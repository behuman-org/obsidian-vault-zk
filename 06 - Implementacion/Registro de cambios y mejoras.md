---
tags:
  - implementacion
  - estado/implementado
---

# Registro de cambios y mejoras

**Rama:** `kyc-zk` en `/Users/mauricio/PROGRAMACION/beHuman`

Historial de cómo se construyó beHuman CAPA 1, con decisiones, cambios y mejoras. Esto es
la **documentación viva del viaje de desarrollo**.

---

## Fase 1: Circuito + Contrato (MVP básico)

### ✅ Circuito Circom (BLS12-381)

**Decisión:** usar BLS12-381 (no BN254) porque el `groth16_verifier` oficial de
soroban-examples usa CAP-0059 (disponible).

- Merkle de inclusión (no firma EdDSA): curva-agnóstica.
- Poseidon con constantes BN254 en BLS12-381 (MVP; marcado para auditoría en producción).
- `currentYear` como constante de compilación (MVP; futuro: input público validado).

**Archivos:** `identity/circuits/src/kyc.circom`, `poseidon2.circom`, `poseidon3.circom`.

### ✅ Contrato `kyc_verifier` (Soroban + Rust)

**Interfaz:**
- `init(trusted_issuer_root, verifying_key)` — guarda config.
- `verify_and_register(address, proof, public_inputs)` — valida Groth16 + registra.
- `is_verified(address)` — consulta de dApps.

**Seguridad:**
- Address binding: `require_auth(address)` + hash del address en público.
- Anti-sybil: nullifier en storage, rechazo de replays.
- Issuer confiable: validar `issuerRoot` en público.

**Testdata snapshot** en `src/testdata.rs` — prueba real generada por el circuito,
reutilizada en tests para evitar regenerar cada vez.

**Tests:** happy path, replay, address mismatch, issuer untrusted, address binding.

---

## Fase 2: Gate de identidad (Matcher) + Enroll

### ✅ Matcher — Face-api + OCR + Liveness

**Arquitectura:** `identity/issuer/matcher/`

- **`documentCheck.ts`** — OCR (tesseract.js) + validación de DNI (España/Argentina).
  - Extrae cara del documento.
  - Valida keywords y tokens (DOC_MIN_KEYWORDS, DOC_MIN_TOKENS en `.env`).
- **`faceEngine.ts`** — face-api.js (carga modelos pesados).
  - Embeddings de cara en DNI vs cara en vivo.
  - Distancia euclidiana (no coseno: face-api descriptores).
  - Umbral `MATCH_THRESHOLD` configurable (default 0.6).
- **`liveness.ts`** — valida que la cara viene de cámara en vivo (optional: challenge).
- **`testnetProvider.ts`** — para testnet: devuelve `MatchResult { ok, matchScore, livenessOk, reasons }`.
  - **Sin imágenes**, solo validación.
- **`renaperProvider.ts`** (stub) — para producción: RENAPER/SID.
- **`server.ts`** — Express en puerto 8787, POST `/verify` multipart.

**De-dup anti-Sybil:**
- `DEDUP_PEPPER` — configurado en `.env`.
- Guarda `sha256(docId + PEPPER)`, nunca el docId.
- Rechaza doble enroll: "este documento ya fue validado".

**Cambios clave:**
1. ✅ CORS habilitado para el frontend.
2. ✅ OCR obligatorio antes del escaneo de cara.
3. ✅ De-dup en el gate (anti-Sybil off-chain).

---

## Fase 3: Identidad de Capa 1 (On-chain)

### ✅ Enroll y creación de `Capa1Credential`

**Flujo:**
1. Usuario pasa gate (DNI + cara viva + match + liveness).
2. Issuer genera `Capa1Credential` con:
   - `attributes` (birthYear, countryCode).
   - `secret` (elemento de campo, generado en device).
   - `commitment = Poseidon(birthYear, countryCode, secret)`.
   - `issuerRoot + pathElements + pathIndices` (Merkle proof).
3. Credencial **guardada en device** (localStorage, no se persiste en servidor).

**Tipos:** `packages/shared/src/index.ts` — `Capa1Credential`, `IdentityAttributes`,
`MatchResult`, `EnrollmentResult`.

### ✅ SDK — `packages/sdk/src/index.ts`

**Funciones:**
- `addressHashField(address)` — sha256(ScVal(address)), validación de bits. **Crítico:**
  debe coincidir exactamente con el contrato.
- `generateProof(credential, address)` — snarkjs fullProve: witness → proof + publicSignals.
- `verifyProofLocally(proof, vk)` — sanity check antes de on-chain.
- `encodeProof(proof)` — Groth16 a bytes (a ∈ G1, b ∈ G2, c ∈ G1).
- `encodeVerificationKey(vk)` — VK snarkjs → ScVal.
- `initVerifier(cfg, signerSecret, issuerRoot, vk)` — invoca `init` en el contrato.
- `verifyAndRegister(cfg, signerSecret, generatedProof)` — invoca `verify_and_register`,
  polling en RPC.

**Decisiones clave:**
- `addressHash` se calcula en el SDK y se envía al circuito. El contrato lo valida.
- Public signals order: `[commitment, nullifier, issuerRoot, addressHash]` (acuerdo
  circuito ↔ contrato).

---

## Fase 4: Frontend — Integración completa

### ✅ KycFlow — Máquina de estados

**Componentes en `web/src/kyc/`:**

1. **`Consent.tsx`** — consentimiento (Ley 25.326, privacidad).
2. **`DocumentUpload.tsx`** — sube foto DNI (validación previa opcional).
3. **`FaceScan.tsx`** — `getUserMedia` + captura en vivo.
4. **`KycFlow.tsx`** — orquesta el flujo:
   - Estado: idle → consentimiento → documento → cara → enviando → resultado.
   - Si falla en cualquier punto: mostrar error y permitir reintentar.
5. **`api.ts`** — POST `/verify` al matcher backend.
   - Input: FormData (DNI + foto cara).
   - Output: `MatchResult { ok, matchScore, livenessOk, reasons }`.

### ✅ Credencial en el navegador — `credentialStore.ts`

**Nueva:** persistencia de `Capa1Credential` en localStorage por `docId`.

- `saveCredential(docId, cred)` — guarda la credencial en device.
- `loadCredential(docId)` — la recupera (permite reanudar sin re-enrolar).
- Manejo de errores: si localStorage no disponible, sigue sin persistir.

**Ventaja:** si el usuario cierra el navegador tras enrolar, puede continuar el registro
on-chain sin repetir la validación biométrica.

### ✅ Registro on-chain desde el navegador

**Nuevo:** integración con `packages/sdk/` en el frontend.

1. Usuario conecta wallet (Stellar Wallets Kit).
2. Frontend genera la prueba Groth16 (generateProof).
3. Frontend invoca `verify_and_register` (verifyAndRegister).
4. Stellar Wallets Kit firma la transacción.
5. Frontend espera confirmación y muestra `is_verified == true`.

**Cambios:**
- ✅ Stellar Wallets Kit integrado.
- ✅ Soporte de snarkjs/ffjavascript en el navegador (nodePolyfills en vite.config.ts).
- ✅ Se ve el estado: `IdentityStatus` mostrado con wallet + `is_verified` del contrato.

---

## Fase 5: E2E y Reproducibilidad

### ✅ `scripts/e2e_demo.sh` — Flujo completo en testnet (Node)

**Pasos:**
1. Deploy del contrato (crea cuenta con Friendbot si falta).
2. Init con issuer root + VK.
3. Generate proof (credencial de prueba pre-cocinada).
4. `verify_and_register` on-chain.
5. `is_verified(address)` → true.

**Archivos generados:**
- `.deploy/kyc_verifier.id` — ID del contrato.
- `.deploy/deployer.secret` — keypair del deployer.

**Permite:** reproducir en cualquier testnet sin dependencias de UI.

---

## Mejoras implementadas (por tema)

### Privacidad
- ✅ No se guardan imágenes on-chain.
- ✅ No se guardan números de documento on-chain.
- ✅ `secret` nunca sale del device.
- ✅ De-dup anti-Sybil usa `sha256(docId+PEPPER)`, no el docId.
- ✅ localStorage con try-catch (no falla si unavailable).

### Usabilidad
- ✅ Máquina de estados clara en el KycFlow.
- ✅ Mensajes de error legibles (MatchResult.reasons).
- ✅ Reintentos simples tras fallos.
- ✅ Persistencia de credencial permite reanudar.
- ✅ Indicador de `is_verified` en vivo.

### Seguridad
- ✅ Address binding (address ∈ proof ∧ invoker == address).
- ✅ Nullifier en storage (anti-replay).
- ✅ Issuer root validado.
- ✅ Tests con testdata snapshot (validación off-chain antes de on-chain).
- ✅ CORS habilitado con control granular.

### Arquitectura
- ✅ SDK compartido (`packages/sdk/` reutilizable en web + scripts).
- ✅ Tipos comunes (`packages/shared/` TypeScript en web + backend).
- ✅ Providers intercambiables (testnet / renaper / dev).
- ✅ Soroban-sdk v25 (CAP-0059 para BLS12-381).

---

## Decisiones y trade-offs documentados

| Decisión | Razón | Trade-off | Futuro |
|---|---|---|---|
| **BLS12-381** | CAP-0059 disponible (host functions) | Poseidon BN254 en BLS12-381 (no estándar) | Parámetros Poseidon específicos en prod |
| **Merkle** (no firma) | Curva-agnóstica | Más proof size que EdDSA | Revocación y escala mejoran |
| **currentYear const** | MVP simple | Usuario puede mentir edad | Validar contra ledger (future input) |
| **Face-api local** | Rápido, testnet | No es RENAPER | Renaper provider stub pronto |
| **localStorage** | Persistencia local | No sincronizado entre devices | Opción: guardar en backend (encrypted) |
| **Merkle path en público** | Prueba de inclusion | Expone profundidad del árbol | Privacidad aceptable (no indexada) |

---

## Próximos pasos (candidatos)

1. **Integración de RENAPER** — reemplazar testnetProvider en producción.
2. **Revocación de identidad** — nullifier revocable o issuer root actualizable.
3. **Recovery** — recuperación de identidad perdida (social recovery).
4. **Documentación OCR regional** — más países y documentos.
5. **Sincronización entre devices** — guardar credencial cifrada en backend (con consentimiento).
6. **CAPA 2** — plataforma de opinión (`platform/contracts/opinion_board`, backend, curaduría).

---

Relacionado: [[Implementación en rama kyc-zk]], [[Flujo de KYC]], [[Diseño del Circuito ZK]],
[[Contrato Verificador (Soroban)]], [[Matcher de Identidad (Gate de Capa 1)]].
