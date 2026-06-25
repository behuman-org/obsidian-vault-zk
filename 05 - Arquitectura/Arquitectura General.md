# Arquitectura General

Vista de pájaro de todos los componentes y cómo se conectan. Los flujos detallados están
en [[Flujo de KYC]].

## Componentes

```mermaid
flowchart TB
    subgraph OFF["🌐 Off-chain"]
        ISS["🏢 Issuer KYC<br/>(verifica identidad,<br/>firma credencial)"]
        WALLET["👛 Wallet / Cliente<br/>(usuario)"]
        PROVER["🔐 Prover<br/>(genera la prueba ZK)"]
    end

    subgraph ON["⛓️ On-chain (Stellar / Soroban)"]
        VERIFIER["📜 KycVerifier<br/>(verifica prueba +<br/>registra address)"]
        REGISTRY["✅ Registro KYC<br/>(address → verificado)"]
        DAPP["🪙 dApp consumidora<br/>(rampa, pool, RWA)"]
    end

    USER["👤 Usuario"] --> WALLET
    WALLET -->|datos de identidad| ISS
    ISS -->|credencial firmada| WALLET
    WALLET --> PROVER
    PROVER -->|prueba + públicos| VERIFIER
    VERIFIER -->|usa host functions ZK| VERIFIER
    VERIFIER --> REGISTRY
    DAPP -->|is_verified address?| REGISTRY
```

## Quién hace qué

| Componente | Dónde | Responsabilidad |
|---|---|---|
| **Issuer KYC** | Off-chain | Verifica identidad real (una vez) y emite credencial firmada. En el MVP es un **mock**; para testnet ese paso lo hace el [[Matcher de Identidad (Gate de Capa 1)\|matcher de cara]] (DNI + cámara). → [[Modelo de Datos]] |
| **Wallet / Cliente** | Off-chain | Guarda la credencial del usuario; orquesta el flujo. |
| **Prover** | Off-chain | Genera la prueba ZK a partir de la credencial + predicado. Es el cómputo pesado. → [[Diseño del Circuito ZK]] |
| **KycVerifier** | On-chain (Soroban) | Verifica la prueba con [[Primitivas ZK en Stellar\|host functions]] y registra el address. → [[Contrato Verificador (Soroban)]] |
| **Registro KYC** | On-chain (storage) | Mapa `address → verificado` + nullifiers usados. |
| **dApp consumidora** | On-chain | Consulta `is_verified(address)` para abrir funcionalidad regulada. → [[Casos de Uso]] |

## Principio de diseño clave: dónde corre cada cosa

- **Generar la prueba = off-chain** (caro, privado). El usuario nunca expone su PII.
- **Verificar la prueba = on-chain** (barato gracias a las primitivas de Stellar).

Esta separación es el corazón de por qué el ZK es *load-bearing*: sin él no hay forma de
que el contrato confíe en el cumplimiento del usuario sin recibir sus datos.

## Vista de capas

```mermaid
flowchart LR
    subgraph L1["Capa de identidad"]
        A[Issuer + Credencial]
    end
    subgraph L2["Capa ZK"]
        B[Circuito + Prover + VK]
    end
    subgraph L3["Capa on-chain"]
        C[Verificador + Registro]
    end
    subgraph L4["Capa de aplicación"]
        D[dApps consumidoras]
    end
    L1 --> L2 --> L3 --> L4
```

## beHuman completo: las dos capas juntas

La capa 4 (*dApp consumidora*) **es** la [[Plataforma de Opinión Verificada]].

```mermaid
flowchart TB
    subgraph C1["🔐 CAPA 1 · Identidad (KYC-ZK)"]
        ISS["Issuer + árbol Merkle"] --> PROV["Prover (circuito)"]
        PROV --> VER["kyc_verifier (Soroban)"]
        VER --> REG["✅ Registro: address → verificado + nullifier"]
        ISS --> ROOT["🌳 issuerRoot<br/>(set de humanos verificados)"]
    end
    subgraph C2["🗣️ CAPA 2 · Plataforma de opinión (anónima)"]
        BOARD["opinion_board (ancla: platformId + contentHash)"]
        API["Backend: feed / perfil / contenido off-chain"]
        CUR["🤖 Curaduría (futuro)"]
    end
    ROOT -->|"prueba ZK de pertenencia<br/>(sin revelar quién)"| BOARD
    BOARD --> API
    API --> CUR
    style C1 fill:#e8f0fe
    style C2 fill:#e6f4ea
```

### Los dos puentes entre capas

Hay **dos formas** de consumir la identidad de Capa 1, según se necesite identidad o anonimato:

| Puente | Para qué | Cómo |
|---|---|---|
| **`is_verified(address)`** | dApps genéricas (rampas, pools, RWA): "este address es un humano único". | El consumidor consulta el registro on-chain por address. **Seudónimo (linkeable al address).** |
| **Pertenencia a `issuerRoot`** | La **plataforma anónima**: "soy un humano del set verificado, pero no digo cuál". | Prueba ZK de inclusión Merkle, identidad = `platformId`. **Anónimo (no toca el address).** |

> 🔑 La plataforma de opinión usa el **segundo** puente: nunca el address.
> → [[Identidad anónima de plataforma (platformId)]].

- **Identidad en la plataforma:** `platformId = Poseidon(secret, SCOPE)` → [[Identidad Pública vs Anónima]].
- **Almacenamiento capa 2:** híbrido (ancla on-chain + contenido off-chain).
- **Fee on-chain:** cuenta efímera (no el address del KYC).
- **Curaduría:** off-chain, aún no implementada → [[Curaduría y Agentes Validadores]].

### Mapeo arquitectura → código (monorepo `beHuman`)

| Componente | Capa | Carpeta del repo |
|---|---|---|
| Issuer KYC (mock) | 1 | `identity/issuer/` |
| Circuito + Prover | 1 | `identity/circuits/` + `packages/sdk/` |
| KycVerifier + Registro | 1 | `identity/contracts/kyc_verifier/` |
| Circuito de plataforma (membership + platformId) | 2 | `platform/circuits/` |
| Plataforma (ancla on-chain) | 2 | `platform/contracts/opinion_board/` |
| Backend / feed / perfil / contenido | 2 | `platform/api/` |
| Curaduría (agentes + moderación) | 2 | `platform/curation/` *(futuro)* |
| Frontend (Capa 1 + Capa 2) | — | `web/src/kyc/` + `web/src/platform/` |

→ Estructura completa en [[Estructura del Codigo]].

Relacionado: [[Flujo de KYC]] · [[Diseño del Circuito ZK]] · [[Modelo de Datos]] ·
[[Contrato Verificador (Soroban)]] · [[Plataforma de Opinión Verificada]] ·
[[Identidad anónima de plataforma (platformId)]] · [[Implementación Capa 2 (plataforma)]]
