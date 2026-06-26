---
tags:
  - funding
  - capa/3-funding
  - zk
  - estado/implementado
---

# 10 — Implementación Capa 3 (ground-funding)

**Rama:** `ground-funding` (exploratoria, aún no mergeada a `main`).

Registro de cómo se construyó la **CAPA 3 — Funding ZK**: crowdfunding **anónimo y
condicional** para causas, sobre la identidad verificada-pero-anónima de
[[01 - Vision y Alcance|Capa 1]]. Las notas 01–09 son la visión; **esta es la realidad
implementada**.

> 🎯 Un humano verificado **dona y opina** sobre causas sin revelar quién es, con reglas que
> **nadie puede saltear**: release 2-de-3 + meta, o refund todo-o-nada. Cero PII on-chain.
> Doc del repo: `docs/funding.md`.

---

## Qué hace (resumen)

1. **Donación anónima:** aporta XLM (testnet) desde una **wallet efímera** (no el address del
   KYC), gateado por una prueba de *personhood*. El capital entra a un **vault Blend
   (DeFindex)** y genera **yield**.
2. **Condicional (todo-o-nada):** el contrato `campaign_controller` custodia el capital.
   - **Release** solo con **2-de-3 firmas** (causa + plataforma + neutral) **y meta alcanzada**.
   - **Refund** si vence el deadline sin meta → cada donante recupera su aporte.
3. **Opinión por campaña:** opina de incógnito; un **nullifier scopeado a la campaña** asegura
   **1 humano = 1 voz** (no se puede inflar el sentimiento).

---

## 1. Circuito de opinión por campaña (`funding/circuits/src/funding_opinion.circom`)

Template `FundingOpinion(LEVELS=4)`. ~3.7k constraints, BLS12-381, snarkjs.

**Diferencia clave con el circuito de Capa 2:** `scope` y `nullScope` son **inputs públicos
(runtime)**, derivados de la campaña:

- `scope = "funding:" + campaignId` → `platformId` distinto e **incorrelacionable entre
  campañas**.
- `nullScope = "funding-opinion:" + campaignId` → `nullifier` de campaña (anti-Sybil del
  sentimiento, 1 voz por humano por campaña).

**Public signals:** `[issuerRoot, platformId, nullifier, scope, nullScope, contentHash]`.

Reutiliza Merkle/Poseidon de Capa 1/2 (mismo árbol del issuer, mismo `commitment`). **No
modifica** `kyc.circom` ni `post.circom`.

> 🔎 Detalle de auditoría (RT-10): el binding de `contentHash` NO depende de una restricción
> interna — `contentHash` es input **público**, así que Groth16 lo liga por el vector IC
> (alterarlo → `verify=false`). Se eliminó un `contentHashSq <== contentHash*contentHash`
> que era una **restricción muerta** (daba falsa sensación de seguridad).

---

## 2. Contrato `campaign_controller` (`funding/contracts/`)

Custodia **no-custodial** (Soroban + Rust). Nadie —ni la plataforma— puede mover fondos fuera
de las reglas: **no hay retiro discrecional**.

| Función | Qué hace |
|---|---|
| `init(admin, asset, cause, goal, deadline, signers[3])` | Configura. Exige `admin.require_auth()` (RT-03: evita front-running del init) y que `admin` sea uno de los signers. |
| `donate(donor, amount)` | Acumula aporte de una **wallet efímera**; rechaza tras deadline o si ya hubo release. |
| `release()` | Solo **2-de-3 firmas** (causa/plataforma/neutral) **y meta alcanzada** → transfiere a la causa. |
| `refund(donor)` | Si venció el deadline **sin** meta → cada donante recupera (todo-o-nada). |
| views | `Raised`, `State` (Fundraising/Released/Refunding), `Contribution(addr)`. |

**Activo configurable** por su SAC (`asset`): XLM en testnet, USDC en prod.

**Tests:** 14 snapshots verdes — release 2-de-3, rechazo de no-firmante/duplicado/bajo-meta/
tras-deadline, refund todo-o-nada, donate tras deadline/release, init auth, doble init.

> 🔌 Integración futura: que el **Manager** de un vault DeFindex(Blend) sea este contrato
> (las donaciones generan yield en Blend). Hoy el controller custodia el capital y enforcea
> las reglas; el yield se modela en `funding/api`.

---

## 3. Backend (`funding/api/`)

Orquesta campañas + DeFindex (yield) + Trustless Work (workflow/disputa).

- **`gating.ts`:** gating por **personhood** (membership, NO `is_verified(address)`). La
  donación presenta prueba de pertenencia (VK de plataforma); las opiniones, prueba del
  circuito de funding. Verifica con snarkjs + re-deriva y compara `contentHash` (binding).
- **`server.ts`:** endpoints de campañas, `/donate`, posición/yield, hitos, `/release`,
  `/refund`, `/campaigns/:id/opinions`. El `platformId`/`nullifier` salen **de la prueba**.
- **Abstracción de proveedores** (`FUNDING_PROVIDER`): `dev` (mocks deterministas, sin red) o
  `real` (DeFindex + Trustless Work vía `@behuman/sdk` con keys).

---

## 4. SDK y tipos

- **`packages/sdk/`:** `defindex.ts`, `trustlesswork.ts` (real/dev), `fundingOpinion.ts`
  (prover browser/Node + verificación + binding).
- **`packages/shared/`:** `Campaign`, `Donation`, `Milestone`, `VaultPosition`,
  `CampaignOpinion`, `Sentiment`.

## 5. Frontend (`web/src/funding/`)

UI: **donar anónimo**, **panel validador** (aprobar hitos + release 2-de-3) y **hilo de
opiniones** con sentimiento por campaña.

---

## Invariantes ZK (cómo se cumplen)

| Invariante | Implementación |
|---|---|
| Identidad nunca revelada | Donante = wallet efímera; opinante = `platformId = Poseidon(secret, scope_campaña)`. El address del KYC no aparece. |
| Donación no atable al KYC | Wallet de donación es seudónimo efímero, sin relación con la credencial. |
| Gate por personhood | Prueba de pertenencia Merkle bajo `issuerRoot`, no `is_verified(address)`. |
| 1 humano = 1 voz por campaña | `nullifier = Poseidon(secret, "funding-opinion:"+campaignId)`. |
| Incorrelacionable entre campañas | `scope` por campaña → `platformId` distinto. |
| Nadie mueve fondos fuera de las reglas | `campaign_controller` sin retiro discrecional. |
| Opinión atada a campaña + contenido | `scope`, `nullScope`, `contentHash` públicos + binding verificado en la API. |

---

## Auditoría (red/blue team)

Se corrió una auditoría con **11 hallazgos remediados** (último commit de la rama). Ejemplos:

- **RT-03:** `init` exige `admin.require_auth()` (evita front-running de la config).
- **RT-10:** binding de `contentHash` por IC público; se eliminó una restricción muerta.
- No enlazar a stellar.expert con hashes mock en modo dev.

---

## Estado y pendientes

- ✅ Contrato (14 tests), circuito (~3.7k constraints, prueba/verifica), API+SDK+Web (e2e dev verde).
- ⏳ **Para testnet real:** keys de DeFindex/Trustless Work + que el Manager del vault Blend
  sea el `campaign_controller` (cross-contract).
- ⏳ Curaduría de opiniones requiere `ANTHROPIC_API_KEY` (sin ella → se escalan).
- ⚠️ Issuer de Capa 1 sigue siendo **mock**; esta capa es **exploratoria** (rama `ground-funding`).

---

## Reproducir (dev)

```bash
(cd funding/circuits && npm i && bash scripts/compile.sh && POWER=14 bash scripts/setup.sh)
npm i
FUNDING_PROVIDER=dev npm run -w @behuman/funding-api serve   # :8789
npm run -w @behuman/web dev
```

---

Relacionado: [[04 - Flujo End-to-End (con ZK)]], [[06 - ZK, Anonimato y Liberacion de Informacion]],
[[09 - Opiniones Anonimas sobre la Causa]], [[08 - Roadmap y Preguntas Abiertas]],
[[Identidad anónima de plataforma (platformId)]], [[Implementación Capa 2 (plataforma)]],
[[Estructura del Codigo]], [[Estado actual del desarrollo]].
