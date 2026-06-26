---
tags:
  - implementacion
  - estado/implementado
---

# Estado actual del desarrollo

**Fecha:** 2026-06-26 · **Capas 1 y 2 en `main`** · **Capa 3 en `ground-funding`**

Snapshot del proyecto: qué está hecho, qué está en progreso, qué falta.

---

## ✅ Hecho

### CAPA 1 — Identidad (KYC con ZK para Stellar)

| Componente | Status | Tests | Deploy |
|---|---|---|---|
| **Circuito (Circom/BLS12-381)** | ✅ Completo | ✅ Testdata snapshot | ✅ En testnet |
| **Contrato (kyc_verifier)** | ✅ Completo | ✅ 9 test cases | ✅ En testnet |
| **Matcher (face-api)** | ✅ Funcional | ✅ Fixtures + tests | ✅ Servidor Express 8787 |
| **De-dup anti-Sybil** | ✅ Cableado | ✅ Validación | ✅ En gate |
| **SDK (generateProof, verifyAndRegister)** | ✅ Completo | ✅ Tipos + encoding | ✅ En web + e2e |
| **Frontend (KycFlow)** | ✅ Flujo full | ✅ Estados claros | ✅ En navegador |
| **Registro on-chain desde web** | ✅ Funciona | ✅ Stellar Wallets Kit | ✅ En testnet |
| **E2E demo (Node)** | ✅ Deploy + prueba | ✅ scripts/e2e_demo.sh | ✅ Reproducible |
| **Persistencia (localStorage)** | ✅ En progreso | 🟡 Partial | ✅ Skeleton |

### Documentación

| Nota | Status |
|---|---|
| **Implementación en rama kyc-zk** | ✅ Creada |
| **Registro de cambios y mejoras** | ✅ Creada |
| **Diseño del Circuito ZK** | ✅ Actualizada (BLS12-381) |
| **Flujo de KYC** | ✅ Actualizado (implementación + linkeo) |
| **Matcher de Identidad** | ✅ Actualizado |
| **Modelo de Datos** | ✅ Actualizado (Capa1Credential real) |

---

### CAPA 2 — Plataforma de opinión anónima (primera iteración) ✅

Mergeada a `main` (PR #2). ZK como núcleo. Detalle en [[Implementación Capa 2 (plataforma)]].

| Componente | Status |
|---|---|
| **Circuito de plataforma** (`platform/circuits`, membership + platformId) | ✅ |
| **Contrato `opinion_board`** (register_identity, post, anti-replay) | ✅ con tests |
| **Backend** (`platform/api`: perfil + contenido off-chain) | ✅ en :8788 |
| **Frontend** (`web/src/platform`: identidad anónima + post + feed) | ✅ |
| **Identidad anónima** (`platformId`, cuenta efímera para fee) | ✅ |
| **Primer post** (opinión comida argentina, anclado on-chain) | ✅ en testnet |
| **Curaduría** (agente IA + rúbrica + cola humana) | ✅ off-chain, aditiva |

### Curaduría — detalle (PR #3 y #4)

- **Nivel 1 (IA):** `platform/curation/agent.ts` + `rubric.ts` → `approved`/`flagged`/`escalated`.
- **Nivel 2 (humano):** `queue.ts` + endpoints `/moderation/*` + `web/.../Moderation.tsx`.
- Solo ve contenido + `platformId`; cero address/PII. Fail-safe: ante error → escalar.
- → [[Curaduría y Agentes Validadores]], [[Implementación Capa 2 (plataforma)]].

### CAPA 3 — Funding ZK (rama `ground-funding`, exploratoria) ✅

Crowdfunding anónimo y condicional. Detalle en [[10 - Implementación Capa 3 (ground-funding)]].

| Componente | Status |
|---|---|
| **Circuito `funding_opinion`** (scope/nullifier por campaña, ~3.7k constraints) | ✅ prueba/verifica |
| **Contrato `campaign_controller`** (release 2-de-3 + meta, refund todo-o-nada) | ✅ 14 tests |
| **Backend `funding/api`** (campañas, donar, yield, hitos, opiniones) | ✅ e2e dev |
| **SDK** (defindex, trustlesswork, fundingOpinion) + **Web** (`web/src/funding`) | ✅ dev |
| **Auditoría red/blue team** | ✅ 11 hallazgos remediados |
| Integración real DeFindex/Trustless Work (keys + Manager del vault) | ⏳ pendiente testnet |

> Donación = wallet efímera; opinión = `platformId` por campaña + nullifier (1 voz/humano).
> Cero PII on-chain. Issuer Capa 1 sigue mock (exploratoria).

---

## 🟡 En progreso / pendiente

### Persistencia de credencial (Capa 1)

`web/src/kyc/credentialStore.ts` — permite reanudar registro sin re-enrolar (ya usado por
la Capa 2 para cargar la credencial). Pulir UI de "credencial guardada".

### Mejoras pendientes

- [ ] Username único (hoy libre).
- [ ] Mitigar correlación por timing (KYC ↔ aparición en plataforma).
- [ ] Relayer en producción (en vez de cuenta efímera friendbot).
- [ ] Calibrar la rúbrica del curador con casos reales.
- [ ] Gobernanza de moderadores + apelaciones.

---

## 📊 Métricas

| Métrica | Valor |
|---|---|
| **Tests del contrato** | 9 (todos pasan) |
| **Fixtures del matcher** | 6 imágenes test |
| **Componentes React** | 5 (Consent, DocumentUpload, FaceScan, KycFlow, api) |
| **Funciones en SDK** | 10 (generateProof, verifyAndRegister, encodings, etc.) |
| **Líneas de documentación** | 600+ en vault |
| **Commits en kyc-zk** | 10+ (últimos 7 días) |
| **Decisiones documentadas** | 7 trade-offs |

---

## 🔍 Revisar antes de CAPA 2

**Crítico:**
- [ ] Auditoría informal de la cripto (nullifier, address binding, issuer root).
- [ ] Testnet íntegra: flujo usuario → enroll → on-chain → is_verified.
- [ ] Prueba con múltiples usuarios (anti-replay verificado).

**Nice-to-have:**
- [ ] Optimizar gas del contrato (almacenamiento, operaciones).
- [ ] Documentación de operators (setup RENAPER, mantener issuer root).

---

## 🎯 Próximo sprint

**Prioridad 1:** terminar persistencia (credentialStore integrada en KycFlow).

**Prioridad 2:** auditoría informal de cripto + validación end-to-end (scripts/e2e_demo.sh).

**Prioridad 3:** documentación de operadores (cómo mantener el sistema en producción).

**Prioridad 4:** si todo cierra → comenzar CAPA 2.

---

Relacionado: [[Registro de cambios y mejoras]], [[Implementación en rama kyc-zk]],
[[Flujo de KYC]].
