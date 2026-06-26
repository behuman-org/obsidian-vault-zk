---
tags:
  - implementacion
  - capa/2-plataforma
  - estado/implementado
---

# Implementación Capa 2 (plataforma)

**Estado:** mergeada a `main` (PR #2, rama `capa2-dev`). Primera iteración funcionando en testnet.

Registro de cómo se construyó la **CAPA 2 (plataforma de opinión anónima)** con **ZK como
núcleo**. La identidad del KYC (address de Capa 1) **nunca** se usa: todo gira sobre
`platformId`, un seudónimo derivado del `secret` por ZK.

> 🎯 **Objetivo logrado:** un humano verificado en Capa 1 puede crear una identidad anónima
> de plataforma, ponerse un username, y publicar opiniones (primer test: la comida
> argentina) — y es **criptográficamente imposible** linkear el post a su KYC o a su PII.

---

## El modelo de identidad: `platformId`

La pieza central. Detalle conceptual en [[Identidad anónima de plataforma (platformId)]].

- **`platformId = Poseidon(secret, SCOPE)`** — derivado del `secret` de la
  [[Modelo de Datos|Capa1Credential]]. Poseidon es unidireccional → no se puede volver al
  secret ni al address.
- **Determinístico** (mismo humano → mismo platformId) → perfil persistente + anti-Sybil.
- **Incorrelacionable** con el address del KYC ni con la PII.
- **Handle público** = últimos 5 caracteres del platformId.

---

## 1. Circuito de plataforma (`platform/circuits/src/post.circom`)

Template `PlatformPost(LEVELS=4, SCOPE=700200300400500700800)`.

**Prueba (sin revelar nada):** *"mi commitment pertenece al árbol del issuer (issuerRoot),
mi identidad es platformId = Poseidon(secret, SCOPE), y esta prueba está atada a este
contentHash."*

| Señal | Tipo | Qué es |
|---|---|---|
| `birthYear`, `countryCode`, `secret` | privado | atributos de Capa 1 (witness) |
| `pathElements`, `pathIndices` | privado | Merkle path en el árbol del issuer |
| `contentHash` | **público** | hash del post (atado dentro de la prueba) |
| `issuerRoot` | output público | raíz del árbol (el contrato la exige de confianza) |
| `platformId` | output público | el seudónimo anónimo |

**Reutiliza Capa 1:** mismo `commitment = Poseidon(birthYear, countryCode, secret)`, misma
`MerkleInclusion`, misma curva **BLS12-381**, mismo árbol del issuer (LEVELS=4). Por eso un
humano de Capa 1 ya es miembro válido sin re-enrolar.

**Binding de contentHash:** se fuerza `contentHashSq <== contentHash * contentHash` para que
la prueba quede atada al contenido (cambiar el texto invalida la prueba → anti-replay).
Para "registrar identidad" se usa `contentHash = 0`.

---

## 2. Contrato `opinion_board` (`platform/contracts/opinion_board/src/lib.rs`)

**Sin address, sin is_verified(address).** Gatea con prueba ZK de pertenencia (Groth16
BLS12-381, mismo patrón de pairing que `kyc_verifier`).

| Función | Qué hace |
|---|---|
| `init(admin, trusted_issuer_root, vk)` | Guarda la raíz del issuer (la misma de Capa 1) + la VK del circuito de plataforma. |
| `register_identity(proof, public_inputs)` | Verifica pertenencia → registra el `platformId`. Anti-Sybil: doble registro → `AlreadyRegistered`. |
| `post(proof, public_inputs)` | Exige identidad registrada + verifica prueba + ancla `PostRecord { platformId, contentHash, timestamp }`. |
| `is_registered(platformId)` / `get_post(id)` / `post_count()` | Consultas. |
| `verify_proof(proof, public_inputs)` | Verificación pura del pairing (tests/integración). |

**Public signals (orden):** `[issuerRoot, platformId, contentHash]`.

**Storage (`DataKey`):** `Config` (admin + trusted_issuer_root), `Vk`, `Identity(platformId)`,
`Post(id)`, `PostCount`, `Posted(platformId, contentHash)` (anti-replay).

**Errores:** UntrustedIssuer, InvalidProof, AlreadyRegistered, NotRegistered, AlreadyPosted,
AlreadyInitialized, NotInitialized, MalformedPublicInputs, MalformedVerifyingKey.

**Tests (`test.rs`, con artefactos ZK reales en `testdata.rs`):** verify acepta/rechaza
(tampered), register happy/untrusted issuer/doble, post requiere registro/replay, doble init.
Todos por `platformId`, **cero address**.

---

## 3. Backend off-chain (`platform/api/src/server.ts`)

Express en `:8788`. Guarda **perfil + contenido**, siempre keyed por `platformId`. Cero PII,
cero address.

| Endpoint | Qué hace |
|---|---|
| `POST /profile` | Setea `username` (libre, mutable, max 40 chars). Devuelve `{ username, handle }`. |
| `GET /profile/:platformId` | Lee el perfil (o handle por defecto). |
| `POST /content` | Guarda el texto del post (max 560) + `contentHash` + `txHash` del ancla on-chain. |
| `GET /feed` | Devuelve el feed (posts más recientes primero). |

`handle = platformId.slice(-5)`. Store en archivo JSON (demo, gitignored). El ancla on-chain
la hace el cliente; el backend solo guarda el contenido off-chain (modelo **híbrido**).

---

## 4. Frontend (`web/src/platform/`)

| Archivo | Qué hace |
|---|---|
| `Platform.tsx` | Orquesta: crear identidad anónima → username → publicar tweet → feed. |
| `zk2.ts` | `generatePlatformProof(cred, contentHash)`, `platformIdHex`, `handleOf`, `contentHashField`. |
| `chain2.ts` | `initIfNeeded`, `registerIdentity`, `postTweet`, `ContractError`. |
| `api2.ts` | `getFeed`, `postContent`, `setProfile`. |
| `ephemeral.ts` | `createFundedEphemeral()` — cuenta efímera para pagar fees. |
| `stellar2.ts` | `OPINION_BOARD_CONTRACT_ID`. |

**Flujo:** carga la `Capa1Credential` del device → genera prueba ZK en el navegador → crea
cuenta efímera → init (si falta) → register_identity → post → feed. Cada post muestra el
**link a la tx on-chain** (stellar.expert) para demostrar el anonimato.

---

## 5. La clave del anonimato: cuenta efímera para fees

`ephemeral.ts`: el fee on-chain **NO lo paga el address del KYC** — se genera una cuenta
**efímera** al vuelo, fondeada con friendbot. Sin esto, el fee-payer delataría a la persona.

> Esto resuelve el "¿quién paga el gas sin deanonimizar?" que es la fuga típica de estos
> diseños. → [[Identidad anónima de plataforma (platformId)#Por qué la cuenta efímera]].

---

## 6. Curaduría (`platform/curation`) — moderación off-chain y aditiva

Implementada (PR #3 y #4). **No toca el circuito ni `opinion_board`**: opera solo sobre el
contenido + `platformId`, nunca address ni PII. Dos niveles (como en
[[Curaduría y Agentes Validadores]]).

### Nivel 1 — Agente validador (IA)

`agent.ts`: usa la API de Claude (`@anthropic-ai/sdk`), modelo configurable
(`CURATION_MODEL`, default `claude-opus-4-8`). `reviewPost(input)` → `CurationVerdict`.
Cliente inyectable (`AnthropicLike`) para mockear el LLM en tests.

`rubric.ts` — la rúbrica del sistema (`SYSTEM_RUBRIC`):
- Evalúa **veracidad/fuentes, coherencia, toxicidad, plagio**.
- Decisión: `approved` | `flagged` | `escalated`.
- **Regla de oro:** discrepar con una idea NO es motivo de moderación — solo abuso,
  desinformación o plagio. Las opiniones subjetivas son válidas y no requieren fuentes.
- **Fail-safe:** si el modelo no devuelve un JSON válido → `escalated` (ante la duda, humano).

### Nivel 2 — Cola de moderación humana

`queue.ts`: `escalateToModeration`, `getModerationQueue`, `resolveModeration`. Store JSON
local. ⚠️ Guarda **solo** contenido + `platformId`/handle + motivo. **Nunca address ni PII.**

### Integración en el backend (`platform/api`)

- Al hacer `POST /content`, se llama a `reviewPost` y se guarda el veredicto en el post.
- Si el curador no está disponible → fail-safe `escalated` (revisión humana).
- Los `escalated` **no se publican** en el feed; los `approved` y `flagged` sí.
- Endpoints nuevos: `GET /moderation/queue`, `POST /moderation/resolve`.
- Frontend: `web/src/platform/Moderation.tsx` (vista de la cola).

**Tipos** (`packages/shared`): `CurationStatus`, `CurationVerdict { status, reason }`,
`CurationInput { platformId, handle, content }`.

---

## Invariantes ZK que se respetaron

1. ✅ El address del KYC **nunca** se usa ni se revela en la plataforma.
2. ✅ Identidad = `platformId = Poseidon(secret, SCOPE)`, determinística e incorrelacionable.
3. ✅ Gate por **prueba de pertenencia** al árbol del issuer, NO por `is_verified(address)`.
4. ✅ `contentHash` atado dentro de la prueba (anti-replay).
5. ✅ Fee pagado por cuenta efímera (no el address KYC).
6. ✅ Username/contenido off-chain keyed por platformId; cero PII.

---

## Reproducir (testnet)

```bash
git checkout main
npm install
# circuito de plataforma
cd platform/circuits && npm install && bash scripts/compile.sh && bash scripts/setup.sh
# tests del contrato
cd ../../ && cargo test -p opinion_board
# backend + frontend
npm run serve -w @behuman/api      # :8788
npm run dev -w @behuman/web        # :5173
```

---

## Pendiente / próximos pasos

- **Username único** (hoy es libre, sin unicidad).
- **Mitigar correlación por timing** (KYC y aparición en plataforma).
- **Relayer** en producción (en vez de cuenta efímera friendbot, que es de testnet).
- **Curaduría on-chain (opcional):** hoy los veredictos son off-chain; a futuro se podría
  anclar el hash del veredicto.
- **Calibrar la rúbrica** del curador con casos reales (es la decisión de producto más fina).

---

Relacionado: [[Identidad anónima de plataforma (platformId)]], [[Plataforma de Opinión Verificada]],
[[Decisiones técnicas y trade-offs]], [[Implementación en rama kyc-zk]], [[Modelo de Datos]],
[[Estado actual del desarrollo]].
