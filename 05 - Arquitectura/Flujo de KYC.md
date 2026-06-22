# Flujo de KYC

El recorrido completo, paso a paso, desde que el usuario verifica su identidad hasta que
una dApp confía en él. Componentes en [[Arquitectura General]].

## Fase 1 — Emisión de la credencial (una sola vez)

```mermaid
sequenceDiagram
    participant U as 👤 Usuario
    participant W as 👛 Wallet
    participant I as 🏢 Issuer KYC

    U->>W: inicia onboarding
    W->>I: envía datos de identidad (documento, etc.)
    I->>I: verifica identidad (KYC real)
    I->>I: commitment = Poseidon(atributos, secret)
    I->>W: credencial firmada (commitment + firma + atributos)
    W->>W: guarda credencial localmente
    Note over U,I: La PII sólo la ve el issuer. Nunca toca la cadena.
```

## Fase 2 — Generación de la prueba (cada vez que hace falta)

```mermaid
sequenceDiagram
    participant W as 👛 Wallet
    participant P as 🔐 Prover (off-chain)

    W->>P: credencial + predicado (ej. edad≥18, país ok) + address
    P->>P: construye witness (privado: PII, secret, firma)
    P->>P: calcula nullifier = Poseidon(secret, addressHash)
    P->>P: genera prueba ZK
    P->>W: prueba + public inputs
    Note over W,P: Sale una prueba, no datos. La PII queda en el witness privado.
```

## Fase 3 — Verificación y registro on-chain

```mermaid
sequenceDiagram
    participant W as 👛 Wallet
    participant V as 📜 KycVerifier (Soroban)
    participant R as ✅ Registro

    W->>V: verify_and_register(address, proof, public_inputs)
    V->>V: issuer_root ∈ confiables?
    V->>V: address == invoker?
    V->>R: nullifier ya usado?
    R-->>V: no
    V->>V: verify_groth16(vk, proof, públicos) ✅
    V->>R: registra address = verificado + guarda nullifier
    V-->>W: ✅ ok
```

## Fase 4 — Consumo por una dApp

```mermaid
sequenceDiagram
    participant U as 👤 Usuario
    participant D as 🪙 dApp (rampa/pool)
    participant R as ✅ Registro

    U->>D: quiere usar servicio regulado
    D->>R: is_verified(address)?
    R-->>D: true
    D-->>U: ✅ acceso concedido (sin saber quién es)
```

## Flujo completo de un vistazo

```mermaid
flowchart LR
    A["1. KYC con issuer<br/>→ credencial"] --> B["2. Genera prueba ZK<br/>off-chain"]
    B --> C["3. Verifica + registra<br/>en Soroban"]
    C --> D["4. dApp consulta<br/>y da acceso"]
    style A fill:#e8f0fe
    style B fill:#fce8e6
    style C fill:#e6f4ea
    style D fill:#fef7e0
```

## Notas de seguridad del flujo

- **Address binding (paso 3):** la prueba está atada al `addressHash`; el contrato exige
  que el invoker sea ese address → nadie puede comprar/reutilizar la prueba de otro.
- **Nullifier (paso 3):** evita doble registro y ataques sybil sin revelar identidad.
- **Issuer root (paso 3):** sólo se aceptan credenciales de issuers de confianza.
- La **PII nunca sale del cliente** salvo hacia el issuer en la Fase 1.

Detalle criptográfico en [[Diseño del Circuito ZK]] y [[Modelo de Datos]].
