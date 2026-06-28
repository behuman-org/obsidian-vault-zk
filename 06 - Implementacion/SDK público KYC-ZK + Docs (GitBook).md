---
tags:
  - implementacion
  - tools
  - capa/1-identidad
  - estado/planificado
---

# SDK público KYC-ZK + Docs (GitBook)

**Estado:** EN DISEÑO / PLAN (no implementado). Rama prevista: `sdk-release`.

Objetivo: empaquetar el **KYC-ZK de beHuman como un SDK público reutilizable** para que
terceros agreguen *proof of personhood anónimo en Stellar* a sus propios proyectos, con
**documentación en GitBook**.

## Punto de partida (`packages/sdk`)

Hoy el `@behuman/sdk` es **interno** y no publicable:
- `private: true`, versión `0.0.0`, `main`/`types` apuntan a `src/` en **TS** (sin build).
- **Mezcla todo:** Capa 1 (KYC), helpers de bajo nivel (`blsEncode`, `poseidonBls`, `merkle`)
  y la Capa 3 de funding (`defindex`, `trustlesswork`, `fundingOpinion`, `fundingAuth`).

→ Hay que **curarlo y empaquetarlo**, no exponerlo tal cual.

## Alcance del SDK público (decisión de diseño)

| Incluir | Dejar fuera |
|---|---|
| **Capa 1 (núcleo):** `generateProof`, `verifyAndRegister`, `isVerified`, config de red/contrato | **Capa 3 (funding):** exploratoria, sigue interna |
| **Capa 2 (modo anónimo):** membership + `platformId` para gatear sin revelar identidad | **Encoders de bajo nivel** (`blsEncode`, `poseidonBls`, `merkle`): marcar internos |

## Packaging publicable

- Sacar `private`, versión `0.1.0`, build a `dist/` (**ESM + `.d.ts`**), `exports`/`main`/
  `module`/`types` → `dist`, `files`.
- `@stellar/stellar-sdk` + `snarkjs` como **peerDependencies**.
- Funciona en **navegador y Node**.
- **Artefactos del circuito** (`kyc.wasm`, `kyc_final.zkey`, `verification_key.json`): definir
  cómo los obtiene el consumidor (incluirlos o exponer loader/URL).
- **Defaults de testnet**: IDs de contrato ya desplegadas (`.deploy/`) como config overrideable.
- **No romper el monorepo:** `web/`, `scripts/`, `funding/` deben seguir compilando.

## Docs en GitBook (`docs/`)

GitBook sincroniza desde Git vía `SUMMARY.md` + markdown. Estructura reescrita para un dev
externo (reusando conceptos de esta vault, no copiando literal):

- Introducción · Conceptos (ZK, anonimato, `platformId`, address binding, nullifier)
- Instalación · Quickstart (gatear una acción por `isVerified` en < 20 líneas)
- Guías: "Verificar identidad", "Gating anónimo (Capa 2)"
- Referencia de API · Contratos desplegados (testnet) · Ejemplos
- **Seguridad y limitaciones** (honestidad): el issuer es un **mock** (no KYC real aún),
  testnet, curva BLS12-381, supuestos. No sobrevender.

## Decisiones abiertas (del usuario)

- **Modo anónimo (Capa 2) en el SDK público:** ¿incluirlo desde v0.1.0 o un primer release
  solo Capa 1? (recomendado: incluirlo, es el diferenciador).
- **Nombre del paquete:** hoy `@behuman/sdk`; para publicar quizá `@behuman/kyc-zk` o
  `@behuman/personhood`.
- **npm publish:** dejarlo listo, pero el `publish` es decisión humana.

---

Relacionado: [[Implementación en rama kyc-zk]], [[Diseño del Circuito ZK]],
[[Estructura del Codigo]], [[Decisiones técnicas y trade-offs]],
[[Onboarding con Pollar (embedded wallet)]], [[Estado actual del desarrollo]].
