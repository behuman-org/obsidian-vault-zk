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
| **Issuer KYC** | Off-chain | Verifica identidad real (una vez) y emite credencial firmada. En el MVP es un **mock** (declarado en README). → [[Modelo de Datos]] |
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

Relacionado: [[Flujo de KYC]] · [[Diseño del Circuito ZK]] · [[Modelo de Datos]] · [[Contrato Verificador (Soroban)]]
