---
tags:
  - implementacion
  - estado/implementado
---

# Setup del Entorno

Herramientas necesarias para construir el proyecto. Esta vault es sólo docs; el código
vive en otro repo (ver [[Estructura del Codigo]]).

## Stellar / Soroban

```bash
# Rust (target wasm para Soroban)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup target add wasm32-unknown-unknown

# Stellar CLI (antes soroban-cli)
cargo install --locked stellar-cli
# o vía Homebrew:
brew install stellar-cli

# Verificar
stellar --version

# Crear identidad y fondear en testnet (friendbot)
stellar keys generate --global alice --network testnet
stellar keys fund alice --network testnet
```

## Toolchain ZK — Circom (opción recomendada)

```bash
# circom (requiere Rust)
git clone https://github.com/iden3/circom.git
cd circom && cargo build --release && cargo install --path circom

# snarkjs (pruebas Groth16 + trusted setup)
npm install -g snarkjs

# circomlib (Poseidon, Merkle, comparadores)
npm install circomlib
```

## Toolchain ZK — Noir (plan B)

```bash
# noirup (gestor de versiones de Noir)
curl -L https://raw.githubusercontent.com/noir-lang/noirup/main/install | bash
noirup

# backend de pruebas (Barretenberg)
# ver https://noir-lang.org/docs/ para bbup
nargo --version
```

## Verificadores de referencia (clonar para estudiar)

```bash
# Groth16 verifier oficial (Circom)
git clone https://github.com/stellar/soroban-examples
# carpeta groth16_verifier

# UltraHonk verifier (Noir, comunitario)
git clone https://github.com/yugocabrio/rs-soroban-ultrahonk

# RISC Zero verifier (Nethermind)
git clone https://github.com/NethermindEth/stellar-risc0-verifier
```

## Checklist de entorno listo

- [ ] `rustc` + target `wasm32-unknown-unknown`
- [ ] `stellar --version` funciona
- [ ] Identidad de testnet creada y fondeada
- [ ] Toolchain ZK elegida instalada (`circom` + `snarkjs`)
- [ ] `groth16_verifier` clonado y compilando
- [ ] Tutorial E2E correspondiente seguido una vez ([[Recursos Oficiales]])

Relacionado: [[Estructura del Codigo]] · [[Roadmap]] · [[Comparativa de Herramientas ZK]]
