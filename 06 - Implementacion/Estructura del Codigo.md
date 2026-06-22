# Estructura del Código

Esta vault es **documentación**. El código de producción (circuitos, contrato, cliente)
vive en un **repositorio separado** que será el entregable open-source del hackathon
(requisito de [[Reglas y Requisitos]]).

> 📌 Decisión: mantener docs (esta vault) y código en repos distintos para que la vault
> de Obsidian quede limpia. Enlazar el repo de código aquí cuando exista:
> **Repo de código:** _(pendiente de crear)_

## Estructura propuesta del repo de código

```text
stellar-zk-kyc/
├── README.md                  # Qué es, cómo correrlo, qué hace el ZK (requisito)
├── circuits/                  # Circuito ZK (Circom)
│   ├── kyc.circom
│   ├── package.json
│   └── scripts/
│       ├── compile.sh         # circom → r1cs/wasm
│       ├── setup.sh           # powers of tau + zkey
│       └── prove.sh           # genera prueba de ejemplo
├── contracts/                 # Contrato Soroban (Rust)
│   └── kyc_verifier/
│       ├── src/lib.rs         # verify_and_register + is_verified
│       ├── Cargo.toml
│       └── tests/
├── issuer/                    # Issuer KYC mock (off-chain)
│   └── src/                   # firma credenciales de prueba
├── client/                    # Cliente / demo (CLI o web)
│   └── src/                   # orquesta: credencial → prueba → tx Stellar
├── scripts/
│   ├── deploy_testnet.sh
│   └── e2e_demo.sh            # flujo completo para el video
└── docs/
    └── README → enlaza a esta vault de Obsidian
```

## Mapeo docs → código

| Nota de la vault | Componente de código |
|---|---|
| [[Diseño del Circuito ZK]] | `circuits/kyc.circom` |
| [[Contrato Verificador (Soroban)]] | `contracts/kyc_verifier/src/lib.rs` |
| [[Modelo de Datos]] | credencial en `issuer/`, storage en `contracts/` |
| [[Flujo de KYC]] | `client/` + `scripts/e2e_demo.sh` |
| [[Plan de Demo]] | `scripts/e2e_demo.sh` (lo que graba el video) |

## Convenciones

- **Commits** en esta vault: `docs: <qué cambió>`.
- **Commits** en el repo de código: convencional (`feat:`, `fix:`, `chore:`…).
- Declarar explícitamente en los README qué partes son **mock** (issuer, datos de
  ejemplo) — lo pide la hackathon.

Relacionado: [[Setup del Entorno]] · [[Roadmap]] · [[Arquitectura General]]
