---
tags:
  - meta
  - capa/1-identidad
  - zk
---

# 02 — Referencia: cómo funciona zkMe (y qué emulamos)

Fuente del modelo: **[zk.me](https://www.zk.me/)** / [blog técnico](https://blog.zk.me/how-zkkyc-works-understanding-the-mechanisms-behind-privacy-preserving-verification/).

## La idea de zkMe en una frase

Verificar **hechos, no identidades**: el usuario prueba *"soy mayor de 18"*, *"soy humano único"*, *"no estoy en una lista de sanciones"* mediante una **prueba criptográfica**, sin entregar su fecha de nacimiento ni su identidad. La plataforma verifica el hecho, no a la persona.

## Componentes de zkMe que nos importan

### 1. MeID — "One Face, One DID" (anti-Sybil)
El núcleo de proof of personhood: a partir de la **cara** se deriva un identificador descentralizado (DID) **único por persona**. Garantiza "una cara = un humano = un registro", protegiendo la privacidad. Esto es exactamente nuestro **nullifier anti-doble-registro**.

### 2. Verificación off-chain, prueba on-chain
El KYC pesado (documento + biometría) ocurre **off-chain**. El resultado se convierte en **credenciales verificables** que el usuario puede reusar entre servicios sin rehacer KYC. La cadena solo recibe la **prueba ZK**.

### 3. Tiers de verificación
- **Tier 1 — Proof of Personhood**: anti-bot / anti-Sybil. ← **Nuestro MVP.**
- **Tier 2 — Ciudadanía, AML, monitoreo de transacciones (KYT).**
- **Tier 3 — Ubicación, checks de inversor avanzado.**

> Nosotros arrancamos en **Tier 1** con DNI argentino + biometría RENAPER. Tier 2/3 quedan para después.

### 4. Credenciales reutilizables
Una vez verificado, el usuario lleva su credencial zkKYC a múltiples servicios. En nuestro caso: la credencial alimenta el commitment que entra al circuito de human.

## Mapeo zkMe → human (qué construimos nosotros)

| Pieza zkMe | Equivalente human | Dónde |
|------------|---------------------|-------|
| KYC provider (doc + cara) | RENAPER SID + parseo DNI | `03`, `04`, `05` |
| MeID "One Face, One DID" | Secreto de identidad + **nullifier** | `06` |
| Credencial verificable | Credencial firmada por el issuer + **commitment** | `06` |
| Membership en set verificado | **Merkle tree** de commitments (estilo Semaphore) | `06` |
| ZK proof on-chain | Circom (Groth16) → **verifier Soroban** | `07` + diagrama existente |
| Verificación on-chain | Contrato `register()` → `Verified(address)` | diagrama existente |

## Diferencias clave de nuestro contexto

- **Cadena**: zkMe es multi-chain/EVM; nosotros somos **Stellar/Soroban** (BN254 + Poseidon, host functions de pairing). Las primitivas existen pero hay que ensamblarlas (ver diagrama del proyecto).
- **Geografía**: nos enfocamos en **Argentina** primero → la fuente de verdad ideal es **RENAPER**, no un agregador global. Eso nos da un KYC más fuerte y barato localmente.
- **Open / no-custodial**: queremos que el **secreto** que genera el commitment lo controle el usuario (Semaphore-style), para no ser custodios de la identidad.

## Lecciones a copiar

1. **Separar realness (off-chain) de la prueba (on-chain).** No metas biometría en el circuito.
2. **El circuito prueba membership + unicidad, no identidad.**
3. **El nullifier es la pieza anti-Sybil.** Derivado del secreto, único por contexto.
4. **Credencial reutilizable** = mejor UX y menos costo de KYC repetido.

## Siguiente

→ [[03-Verificacion-Documento-DNI/Verificacion-DNI-Argentina]]

## Fuentes
- [zkMe — zk-Identity Layer](https://www.zk.me/did-solution)
- [How zkKYC Works](https://blog.zk.me/how-zkkyc-works-understanding-the-mechanisms-behind-privacy-preserving-verification/)
- [zkMe Unveils zkKYC (GlobeNewswire)](https://www.globenewswire.com/news-release/2025/01/02/3003715/0/en/zkMe-Unveils-zkKYC-A-Fully-Decentralized-and-Privacy-First-KYC-Solution.html)
