---
tags: [stellar, soroban, zk, protocolo, izzy]
aliases: [X-Ray, Yardstick, Protocolo 25, Protocolo 26]
---
## TL;DR
Stellar construyó su capa criptográfica para ZK en dos upgrades seguidos:
- **X-Ray (Protocolo 25)** trajo las *primitivas* ZK al host (BN254 + Poseidon).
- **Yardstick (Protocolo 26)** las hizo *baratas* (mueve la matemática pesada al host layer).

Juntos, más **BLS12-381** (que ya venía del Protocolo 22), le dan a Soroban los bloques para **verificar pruebas zk-SNARK on-chain de forma eficiente**.

> [!important] Son bloques de construcción, no un producto
> Estos upgrades NO te dan pagos privados ni KYC-ZK "de fábrica". Generás la prueba **off-chain** (Noir, Circom, RISC Zero) y deployás un **contrato verificador** en Stellar que la chequea. Ese gap es donde vive el proyecto.

---

## X-Ray — Protocolo 25
**Tema:** "nueva visión" (ver lo que antes no se podía). **Activación:** enero 2026 (testnet 7 ene, mainnet ~22 ene 2026).

Introdujo **host functions nativas para primitivas ZK-friendly**:
- Operaciones sobre la curva elíptica **BN254**.
- Hashing **Poseidon / Poseidon2** (el hash "amigable con circuitos").

Detalle clave: estas funciones **espejan los precompiles EIP-196 y EIP-197 de Ethereum**, lo que hace mucho más fácil **portar proyectos ZK existentes** de EVM a Stellar (misma curva BN254, mismas operaciones).

**Resultado:** se pueden verificar pruebas zk-SNARK (Groth16) **dentro de un contrato Soroban**, probando atributos de un dato sin revelar el dato.

---

## Yardstick — Protocolo 26
**Tema:** precisión / exactitud. **Live en mainnet:** 6 may 2026 (testnet 16 abr, releases estables 8 abr).

La pieza ZK es **CAP-0080: Host functions for efficient ZK BN254 use cases**, que construye sobre el trabajo BN254 de X-Ray y agrega **nueve nuevas host functions**:
- **Multi-scalar multiplication (MSM)** sobre BN254.
- **Aritmética de campo escalar:** add, subtract, multiply, power, inverse.
- **Curve-membership checks** para puntos BLS12-381 **y** BN254.

Mueve la aritmética ZK pesada al **host layer** → la verificación de pruebas (incluidas las de **NoirLang/Ultrahonk**) sale **significativamente más barata** que implementarla del lado Wasm. Si construís ZK en Soroban, es reducción directa de costo.

> [!note] El resto de Yardstick (no-ZK, pero parte del release)
> CAP-0077 Quorum Freeze (congelar ledger keys por consenso) · CAP-0082 aritmética checked de 256 bits · CAP-0078 control fino de TTL · CAP-0079 conversión de strkeys muxed · CAP-0073 SAC para G-accounts.

---

## Comparación rápida

| | **X-Ray (P25)** | **Yardstick (P26)** |
|---|---|---|
| Tema | Nueva visión | Precisión |
| Mainnet | ~ene 2026 | 6 may 2026 |
| Aporte ZK | Primitivas BN254 + Poseidon | +9 host functions (MSM, campo escalar, membership) |
| Efecto | *Se puede* verificar ZK on-chain | Verificar ZK on-chain *barato* |
| CAP ZK | host functions BN254/Poseidon | CAP-0080 |

---

## Por qué importa para [[IZZY]] / personhood
- El **groth16_verifier** en Soroban se apoya en estas host functions (BN254 = curva que sale de Circom/snarkjs).
- El **MSM** de Yardstick es justo lo que arma el `vk_x` del pairing check → verificación más barata.
- Los **curve-membership checks** evitan pruebas malformadas (puntos fuera de curva).
- Poseidon (X-Ray) = el hash para **commitments**, **nullifiers** y el **Merkle tree** de identidades verificadas.

---

## Fuentes
- Yardstick P26 (blog oficial): https://stellar.org/blog/foundation-news/yardstick-stellar-protocol-26
- Guía de upgrade P26: https://stellar.org/blog/foundation-news/stellar-yardstick-protocol-26-upgrade-guide
- Software versions (CAPs): https://developers.stellar.org/docs/networks/software-versions
- Privacidad / ZK en Stellar: https://stellar.org/blog/developers/financial-privacy