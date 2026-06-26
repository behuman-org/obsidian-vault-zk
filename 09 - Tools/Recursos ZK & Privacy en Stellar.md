---
tags:
  - tools
  - anonimato
  - zk
---

# Recursos ZK & Privacy en Stellar

Catálogo **maestro** de los recursos oficiales para construir ZK y privacidad en Stellar
(fuente: *Stellar Hacks: Real-World ZK* / DoraHacks + docs oficiales). Es el índice de
enlaces; el *cómo usarlos* está en las otras notas de [[🧰 Tools — Índice]].

> 💡 Esta nota **amplía** [[Recursos Oficiales]] (que es la versión corta). Aquí está
> todo, clasificado y anotado para nuestro caso de uso ([[IDEA|KYC con ZK]]).

---

## 1. Empieza aquí — ZK & Privacy en Stellar

| Recurso | Enlace | Por qué importa |
|---|---|---|
| **ZK Proofs on Stellar** (docs) | https://developers.stellar.org/docs/build/apps/zk | La referencia central. Explica las host functions **BN254** y **Poseidon/Poseidon2**, qué hacen y cómo se verifica una prueba on-chain. Incluye ejemplos. |
| **Privacy on Stellar** (docs) | https://developers.stellar.org/docs/build/apps/privacy | El mejor mapa del stack de privacidad: Privacy Pools, Confidential Tokens, verificadores ZK on-chain y primitivas cripto. |
| **Stellar X-Ray (Protocol 25)** | https://stellar.org/blog/developers/announcing-stellar-x-ray-protocol-25 | Por qué se añadieron estas primitivas y la estrategia de privacidad de largo plazo. |
| **Yardstick (Protocol 26)** — upgrade guide | https://stellar.org/blog/foundation-news/stellar-yardstick-protocol-26-upgrade-guide | Qué añadió P26 para builders ZK y por qué la verificación de pruebas se abarató. |

→ Contexto conceptual en [[Stack de Privacidad en Stellar]] y
[[Stellar X-Ray (P25) y Yardstick (P26) — la base ZK]].

---

## 2. Tooling de circuitos ZK

| Herramienta | Doc | Modelo | Nota de la vault |
|---|---|---|---|
| **Noir** (Aztec) | https://noir-lang.org/docs/ | DSL tipo Rust, DX amable. P26 abarató su verificación. Pareja con UltraHonk. | [[Noir]] |
| **RISC Zero** (zkVM) | https://dev.risczero.com/ | Escribes un programa Rust normal y pruebas su ejecución. | [[RISC Zero]] |
| **Circom** | https://docs.circom.io/ | Constraints de bajo nivel + Groth16. Lo usa el PoC de Privacy Pools. | [[Circom]] |

**Primitivas en el Soroban SDK (claves para verificar on-chain):**

- BN254: https://docs.rs/soroban-sdk/latest/soroban_sdk/_migrating/v25_bn254/index.html
- Poseidon: https://docs.rs/soroban-sdk/latest/soroban_sdk/_migrating/v25_poseidon/index.html
- Ejemplos preview P25 (Soroban): https://github.com/jayz22/soroban-examples/tree/p25-preview/p25-preview

**CAPs del protocolo (deep cuts):**

- BN254 → **CAP-0074**
- Poseidon/Poseidon2 → **CAP-0075**
- BLS12-381 → **CAP-0059**

> 🎯 Para nuestro KYC la decisión es **Circom + Groth16** (verificación más barata,
> verificador oficial), con **Noir** como plan B. Argumentado en
> [[Comparativa de Herramientas ZK]].

---

## 3. Verificadores ZK on-chain (starter code)

Resumen aquí; detalle y criterio de elección en [[Verificadores ZK de referencia]].

| Verificador | Repo | Sistema de prueba |
|---|---|---|
| RISC Zero (Groth16) | https://github.com/NethermindEth/stellar-risc0-verifier | Groth16 desde zkVM RISC Zero |
| UltraHonk (Noir) | https://github.com/yugocabrio/rs-soroban-ultrahonk · https://github.com/indextree/ultrahonk_soroban_contract | UltraHonk / Barretenberg |
| Privacy Pools PoC | https://github.com/NethermindEth/stellar-private-payments | Circom + Groth16 |
| Groth16 oficial | https://github.com/stellar/soroban-examples/tree/main/groth16_verifier | Groth16 (Circom) |

Artículo RISC Zero: https://stellar.org/blog/developers/risc-zero-verifier
Docs Privacy Pools PoC: https://nethermindeth.github.io/stellar-private-payments/

> ⚠️ El PoC de Privacy Pools es **prototipo de investigación, no auditado**. Para estudiar,
> no para activos reales.

---

## 4. Core Stellar Dev Tools

| Herramienta | Enlace | Para qué |
|---|---|---|
| Stellar Docs | https://developers.stellar.org/ | Documentación raíz. |
| SDKs | https://developers.stellar.org/docs/tools/sdks | Librerías por lenguaje (usar la última versión para soporte P26). |
| Stellar CLI | https://developers.stellar.org/docs/tools/cli | Build/deploy/interactuar con contratos Soroban. |
| Lab | https://developers.stellar.org/docs/tools/lab | Explorar/testear en el navegador; generar + fondear cuentas de testnet. |
| Quickstart | https://developers.stellar.org/docs/tools/quickstart | Red local Stellar vía Docker. |
| Scaffold Stellar | https://scaffoldstellar.org | CLI para el ciclo completo de la app (contratos, tests, deploy). |
| Stellar Wallets Kit | https://stellarwalletskit.dev/ | Conexión de wallets con API unificada. |
| OpenZeppelin on Stellar | https://www.openzeppelin.com/networks/stellar | Librería auditada, Contracts Wizard, MCP server, detectores de seguridad. |

**Building blocks de smart contracts (docs):**

- Getting Started: https://developers.stellar.org/docs/build/smart-contracts/getting-started
- Authorization: https://developers.stellar.org/docs/build/guides/auth
- Storage: https://developers.stellar.org/docs/build/guides/storage
- Testing: https://developers.stellar.org/docs/build/guides/testing

→ Cómo instalar el entorno en [[Setup del Entorno]].

---

## 5. Desarrollo asistido por IA

Catálogo completo en [[Skills de IA para construir]]. Enlaces base:

- Building with AI (docs): https://developers.stellar.org/docs/build/building-with-ai
- Stellar Skills: https://skills.stellar.org/
- ZK Proofs skill: https://skills.stellar.org/skills/zk-proofs/SKILL.md
- stellar-dev-skill (repo): https://github.com/stellar/stellar-dev-skill
- stellar-build (42 skills): https://github.com/kaankacar/stellar-build
- OpenZeppelin Skills: https://github.com/OpenZeppelin/openzeppelin-skills
- llms.txt (digest para LLMs): https://developers.stellar.org/llms.txt

---

## 6. Contexto extra de privacidad

| Recurso | Enlace |
|---|---|
| Confidential Token Association | https://www.confidentialtoken.org/ |
| Demo Confidential Tokens (video) | https://www.youtube.com/watch?v=6NnDqVQYOHM |
| Privacy Pools whitepaper (Buterin et al.) | https://privacypools.com/whitepaper.pdf |

→ Desarrollado en [[Stack de Privacidad en Stellar]].

---

## 7. Comunidad y descubrimiento

| Recurso | Enlace |
|---|---|
| Stellar Ecosystem Resources | https://github.com/stellar/ecosystem-resources/ |
| Stellar Hackathon FAQ | https://github.com/briwylde08/stellar-hackathon-faq |
| Stellar Ecosystem DB (buscar proyectos existentes) | https://github.com/lumenloop/stellar-ecosystem-db |
| Discord `#zk-chat` | https://discord.gg/stellardev |

---

## Relacionado

- [[🧰 Tools — Índice]] · [[Skills de IA para construir]] · [[Verificadores ZK de referencia]]
- [[Stack de Privacidad en Stellar]] · [[Plano del KYC inspirado en zkMe]]
- [[Comparativa de Herramientas ZK]] · [[Primitivas ZK en Stellar]]
