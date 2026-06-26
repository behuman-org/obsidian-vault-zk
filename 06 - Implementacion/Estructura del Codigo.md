---
tags:
  - implementacion
  - estado/implementado
---

# Estructura del Código

Esta vault es **documentación**. El código de producción (circuitos, contrato, cliente)
vive en un **repositorio separado** que será el entregable open-source del hackathon
(requisito de [[Reglas y Requisitos]]).

> 📌 **Decisiones:**
> - **Docs ↔ código en repos distintos** (esta vault queda limpia).
> - **Un solo monorepo, organizado por capas** (`identity/` + `platform/`): refleja las dos
>   capas de [[IDEA]] y deja explícito que **todo funciona junto** pero las capas son claras.
> - Organización GitHub: **`ACRC-Zk`** · **Repo de código:** `beHuman`
>   (https://github.com/ACRC-Zk/beHuman) — **✅ scaffolding por capas creado**.
> - **Frontend único en React + Vite + TypeScript** (`web/`): te verificás (capa 1) y eso
>   desbloquea opinar/publicar (capa 2).
> - **El puente entre capas es uno solo:** `is_verified(address)` del contrato `kyc_verifier`.
> - **Capa 2 híbrida:** ancla on-chain (`opinion_board`) + contenido off-chain (`platform/api`).
> - **Identidad en la plataforma: seudónimo estable** (posts linkeables, PII oculta) →
>   [[Identidad Pública vs Anónima]].

## Estructura del monorepo (creada, por capas)

```text
beHuman/
├── README.md · CLAUDE.md · Cargo.toml · package.json · Makefile · .env.example
│
├── identity/                 # ── CAPA 1 · KYC con ZK ──
│   ├── circuits/             #   Circom — src/kyc.circom + scripts (compile/setup/prove)
│   ├── contracts/            #   Soroban — kyc_verifier (verify_and_register + is_verified)
│   └── issuer/               #   Issuer KYC (mock) + matcher/ (gate DNI+selfie, testnet)
│
├── platform/                 # ── CAPA 2 · Plataforma de opinión (anónima por ZK) ──
│   ├── circuits/             #   Circom — post.circom (membership + platformId + contentHash)
│   ├── contracts/            #   Soroban — opinion_board (ancla: platformId + contentHash)
│   ├── api/                  #   Backend: feed, perfil/username, contenido off-chain (TS)
│   └── curation/             #   Agentes validadores (IA, Claude) + cola de moderación (TS)
│
├── funding/                  # ── CAPA 3 · Funding ZK (rama ground-funding) ──
│   ├── circuits/             #   Circom — funding_opinion (scope/nullifier por campaña)
│   ├── contracts/            #   Soroban — campaign_controller (no-custodial: release 2-de-3 / refund)
│   └── api/                  #   Backend: campañas, donar, yield, hitos, opiniones (TS)
│
├── packages/
│   ├── sdk/                  #   Prover + tx Stellar + defindex/trustlesswork + fundingOpinion
│   └── shared/               #   Tipos TS (VerifiedAddress · Post · CurationVerdict · Campaign · Donation…)
│
├── web/                      # Frontend React + Vite + TS (único)
│   └── src/                  #   kyc/ (Capa 1) + platform/ (Capa 2)
├── scripts/                  # deploy_testnet.sh · e2e_demo.sh
└── docs/                     # enlaza a esta vault
```

> Los contratos Rust (`kyc_verifier`, `opinion_board`, `campaign_controller`) son miembros
> del **workspace Cargo raíz** (`/Cargo.toml`): `stellar contract build` compila las tres
> capas. Las **skills de IA** (`stellar-dev`,
> `openzeppelin-skills`) están en `.claude/settings.json` → [[Skills de IA para construir]].

## Mapeo docs → código

| Nota de la vault | Componente de código |
|---|---|
| [[Diseño del Circuito ZK]] | `identity/circuits/src/kyc.circom` |
| [[Contrato Verificador (Soroban)]] | `identity/contracts/kyc_verifier/src/lib.rs` |
| [[Modelo de Datos]] | credencial en `identity/issuer/`, storage en `identity/contracts/` + `packages/shared` |
| [[Flujo de KYC]] | `packages/sdk/` + `web/` + `scripts/e2e_demo.sh` |
| [[Spec — Matcher DNI + Selfie (Capa 1)]] | `identity/issuer/matcher/` + `web/` (captura) + `packages/shared` |
| [[Plataforma de Opinión Verificada]] | `platform/contracts/opinion_board` + `platform/api` |
| [[Curaduría y Agentes Validadores]] | `platform/curation` |
| [[Plan de Demo]] | `scripts/e2e_demo.sh` (lo que graba el video) |

## Rama `kyc-zk` — CAPA 1 IMPLEMENTADA

**Status:** circuito compilando, contrato con tests, matcher validando DNI+cara, SDK con
generateProof/verifyAndRegister, e2e_demo en testnet ✅

Detalles en [[Implementación en rama kyc-zk]].

Próximo paso: integrar SDK en el frontend (`web/`), conectar Stellar Wallets Kit, permitir
que `verify_and_register` se llame desde el navegador.

---

## Convenciones

- **Commits** en esta vault: `docs: <qué cambió>`.
- **Commits** en el repo de código: convencional (`feat:`, `fix:`, `chore:`…).
- Declarar explícitamente en los README qué partes son **mock** (issuer, datos de
  ejemplo) — lo pide la hackathon.

Relacionado: [[Setup del Entorno]] · [[Roadmap]] · [[Arquitectura General]] · [[Implementación en rama kyc-zk]]
