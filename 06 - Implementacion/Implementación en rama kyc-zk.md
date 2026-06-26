---
tags:
  - implementacion
  - capa/1-identidad
  - zk
  - estado/implementado
---

# Implementación en rama kyc-zk

**Rama:** `kyc-zk` en `/Users/mauricio/PROGRAMACION/beHuman`

Registro de cómo se construyó la **CAPA 1 (identidad)** de beHuman, parte por parte, con
enlaces a las decisiones de diseño y al código real.

> 🎯 **Objetivo:** un usuario valida su identidad (DNI + cara viva) en testnet, recibe una
> identidad criptográfica única que se registra on-chain en Stellar, y esa identidad no
> se puede usar dos veces.

---

## 1. Circuito ZK (`identity/circuits/src/kyc.circom`)

**Decisión crítica: BLS12-381 (no BN254).**

- Raíz: el verificador on-chain que usamos (`groth16_verifier` oficial de soroban-examples)
  usa host functions **BLS12-381** (CAP-0059, disponible en Soroban SDK 25+).
- BN254/Poseidon nativas (CAP-0074/75) siguen siendo propuestas.
- Trade-off: Poseidon con constantes BN254 reusadas en BLS12-381 (válido MVP, no estándar;
  marcar para producción).

**Estructura:** [[Diseño del Circuito ZK#Decisiones de diseño (implementadas en `kyc-zk`)]]

- **Inputs privados:** `birthYear`, `countryCode`, `secret`, `pathElements`, `pathIndices`
  (Merkle path).
- **Públicos:** `commitment`, `nullifier`, `issuerRoot`, `addressHash`.
- **Template `MerkleInclusion`:** verifica que el commitment pertenece al árbol del issuer.
- **Predicados:** `is_adult ⇔ currentYear - birthYear >= 18`, `country_ok`.

**Build:** `identity/circuits/scripts/compile.sh` (circom + snarkjs), `setup.sh` (trusted
setup), `gen_testdata.mjs` (snapshot del testdata para los tests del contrato).

---

## 2. Contrato Verificador (`identity/contracts/kyc_verifier/src/lib.rs`)

**Interfaz:** `init`, `verify_and_register`, `is_verified`.

Mapeo a [[Flujo de KYC|Fases 3–4]]:

| Función | Qué hace |
|---|---|
| `init(deployer, issuer_root, verification_key)` | Guarda el issuer root de confianza + VK. |
| `verify_and_register(address, proof, public_inputs)` | Exige `is_verified` del address (address binding) → valida la prueba Groth16 → registra nullifier + marca el address como verificado. |
| `is_verified(address) -> bool` | Consulta que usan las dApps / el frontend. |

**Seguridad cableada:**

- **Address binding:** `require_auth(address)` + hash del address en `public_inputs`.
- **Anti-sybil:** `nullifier` ya registrado → rechazo.
- **Issuer confiable:** solo acepta pruebas cuyo `issuerRoot` public input coincida con el
  del contrato.

**Tests (`src/test.rs`):** happy path, replay attack (nullifier), address mismatch, issuer
no confiable, address binding.

**Testdata (`src/testdata.rs`):** snapshot de una prueba real generada por el circuito +
los public inputs. Se usa en los tests para validar sin regenerar la prueba cada vez.

Relacionado: [[Contrato Verificador (Soroban)]].

---

## 3. Gate de Identidad (Matcher — `identity/issuer/matcher/`)

**Flujo:** usuario sube DNI → extrae cara → cámara captura cara viva → face match + liveness
→ enroll (generate identity).

Componentes:

| Módulo | Qué hace |
|---|---|
| `documentCheck.ts` | OCR + validación de DNI (extrae cara, verifica que es documento válido). |
| `faceEngine.ts` | Carga modelos de `face-api.js`, calcula embeddings, compara similitud (distancia euclidiana). |
| `liveness.ts` | Desafío en vivo: valida que la cara viene de cámara en tiempo real (opcional: parpadeo/giro). |
| `testnetProvider.ts` | Para testnet: devuelve `MatchResult { ok, matchScore, livenessOk, reasons }`. **Sin imágenes.**|
| `renaperProvider.ts` (stub) | Para producción: stub que podría llamar a RENAPER/SID. |
| `server.ts` | Servidor Express en puerto 8787 que expone `/verify` (POST multipart). |

**De-dup anti-Sybil:** `DEDUP_PEPPER` → guarda `sha256(docId + PEPPER)`, nunca el docId.
En testnet se declara en `.env.example`; en producción es un secreto.

Relacionado: [[Matcher de Identidad (Gate de Capa 1)]].

---

## 4. SDK Cliente (`packages/sdk/src/index.ts`)

**Flujo:** credencial → generateProof → verifyAndRegister (on-chain).

Funciones clave:

| Función | Qué hace |
|---|---|
| `generateProof(credential, address)` | Calcula `addressHash` (sha256 + validación de bits) → corre snarkjs fullProve → devuelve proof + publicSignals. |
| `addressHashField(address)` | Réplica exacta de lo que hace el contrato. Critical: sha256(ScVal(address)) con 2 bits altos en 0. |
| `encodeProof(proof)` | Transforma proof snarkjs a bytes (Groth16 encoding: a ∈ G1, b ∈ G2, c ∈ G1). |
| `encodeVerificationKey(vk)` | VK snarkjs → ScVal para el contrato. |
| `initVerifier(cfg, signerSecret, issuerRoot, vk)` | Invoca `init` en el contrato. |
| `verifyAndRegister(cfg, signerSecret, generatedProof)` | Invoca `verify_and_register`, polling en RPC. |

**Precisión:** los encodings de proof y VK deben coincidir exactamente con lo que espera
Soroban; está documentado en `blsEncode.ts`.

Relacionado: [[Flujo de KYC#Fase 2 — Generación de la prueba (cada vez que hace falta)]],
[[Flujo de KYC#Fase 3 — Verificación y registro on-chain]].

---

## 5. Frontend (`web/src/kyc/`)

**Máquina de estados:** `KycFlow.tsx` orquesta el gate.

| Componente | Qué hace |
|---|---|
| `Consent.tsx` | Acepta la política de privacidad (Ley 25.326). |
| `DocumentUpload.tsx` | Sube foto del DNI. |
| `FaceScan.tsx` | `getUserMedia` + captura en vivo. |
| `KycFlow.tsx` | Estado: consentimiento → documento → cara → resultado. |
| `api.ts` | POST `/verify` al matcher; devuelve `MatchResult`. |

**En testnet:** el matcher es local (face-api en Node). Respuesta: `ok`, `matchScore`,
`livenessOk`, `reasons`. **Sin imágenes ni PII.**

**Plan (next iteration):** conexión de wallet (Stellar Wallets Kit) + llamada a `verify_and_register`
desde el navegador → mostrar `is_verified == true`.

Relacionado: `web/README.md`, [[Flujo de KYC#Fase 1 — Emisión de la credencial (una sola vez)]].

---

## 6. E2E Demo (`scripts/e2e_demo.sh`)

**Prueba completa en testnet (Node, no frontend).**

Pasos:

1. Deploy del contrato → guarda ID en `.deploy/kyc_verifier.id`.
2. Init con issuer root + VK.
3. Generate proof (credencial pre-cocinada).
4. verifyAndRegister (Node SDK).
5. is_verified(address) → true.

**Uso:** `npm install && bash scripts/e2e_demo.sh`.

Esto es lo que el frontend eventualmente replicará en el navegador.

---

## 7. Decisiones pendientes (documentadas, future work)

| Decisión | Notas |
|---|---|
| **Poseidon en BLS12-381** | MVP: constantes BN254. Producción: parámetros específicos del campo. Marcar para auditoría. |
| **currentYear como constante** | MVP: compilada. Producción: input público validado contra ledger (evita mentir edad). Ver [[Diseño del Circuito ZK#Riesgos / consideraciones]]. |
| **Issuer mock → RENAPER** | Testnet: face-api. Producción: stub `renaperProvider.ts` → servidor estatal. Interface ya existe. |
| **De-dup por documento** | Testnet: `sha256(docId+PEPPER)`. Producción: validación de documento real (DNI + RENAPER). |
| **Conexión de wallet** | Frontend: `Stellar Wallets Kit`. Permitir que el usuario firme `verify_and_register` desde el navegador. |
| **Revocación de identidad** | Future: nullifier revocable o issuer root actualizable. |

---

## Vinculos a notas de arquitectura

Todo lo anterior está esquematizado en la vault:

- **Flujo conceptual:** [[Flujo de KYC]] (Fases 1–4, con secuencia Mermaid).
- **Diseño criptográfico:** [[Diseño del Circuito ZK]] (constraints, decisiones de ingeniería).
- **Datos:** [[Modelo de Datos]] (Capa1Credential, estructura de storage).
- **Gate biométrico:** [[Matcher de Identidad (Gate de Capa 1)]] (face match + liveness).
- **Especificación técnica:** [[Puente-KYC-a-ZK]] (si existe; ver `packages/shared/src/index.ts`).

---

## Cómo reproducir (local, MacBook M4)

```bash
# Setup
cd beHuman
git checkout kyc-zk
npm install

# Circuito
cd identity/circuits && npm install && bash scripts/compile.sh && bash scripts/setup.sh

# Tests del contrato
cd ../../ && cargo test

# E2E en testnet
bash scripts/e2e_demo.sh

# Frontend (próximo paso)
npm run dev -w @behuman/web
# (luego: Stellar Wallets Kit + llamada a verify_and_register)
```

---

Relacionado: [[Flujo de KYC]], [[Diseño del Circuito ZK]], [[Contrato Verificador (Soroban)]],
[[Matcher de Identidad (Gate de Capa 1)]], [[Estructura del Codigo]].
