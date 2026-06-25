# Diseño del Circuito ZK

El detalle criptográfico de **qué** prueba el circuito. Conceptos base en
[[Fundamentos ZK]]; herramienta en [[Comparativa de Herramientas ZK]].

## Afirmación que prueba el circuito

> *"Conozco una credencial KYC válida emitida por un issuer de confianza
> (`issuer_root`), atada a mi dirección Stellar (`address_hash`), y los atributos de esa
> credencial satisfacen el predicado (`edad ≥ 18` y `país ∈ permitidos`) — sin revelar
> ninguno de esos atributos."*

## Entradas

### Privadas (witness — no se revelan)

| Señal | Descripción |
|---|---|
| `birth_year` | Año de nacimiento |
| `country_code` | Código de país |
| `secret` | Secreto del usuario (semilla del commitment y del nullifier) |
| `issuer_sig` / `merkle_path` | Prueba de que la credencial fue emitida por el issuer |

### Públicas (visibles para el verificador on-chain)

| Señal | Descripción |
|---|---|
| `issuer_root` | Raíz Merkle / clave del issuer de confianza |
| `address_hash` | Hash del address Stellar del usuario (binding) |
| `nullifier` | Identificador anti-replay derivado del secreto |
| `current_year` | Año actual (para calcular la edad) |
| `is_adult`, `country_ok` | Resultado de los predicados (booleanos) |

## Restricciones (constraints) que impone el circuito

```mermaid
flowchart TD
    subgraph Privado
        BY[birth_year]
        CC[country_code]
        SEC[secret]
        SIG[issuer_sig / merkle_path]
    end
    subgraph Público
        ROOT[issuer_root]
        ADDR[address_hash]
        NF[nullifier]
        CY[current_year]
        ADULT[is_adult]
        COK[country_ok]
    end

    BY & CC & SEC --> C1["commitment = Poseidon(birth_year, country_code, secret)"]
    C1 & SIG --> C2["verificar credencial<br/>pertenece a issuer_root"]
    ROOT --> C2
    BY & CY --> C3["age = current_year - birth_year<br/>is_adult ⇔ age ≥ 18"]
    C3 --> ADULT
    CC --> C4["country_ok ⇔ country_code ∈ permitidos"]
    C4 --> COK
    SEC & ADDR --> C5["nullifier = Poseidon(secret, address_hash)"]
    C5 --> NF
```

1. **Commitment correcto** — `commitment = Poseidon(birth_year, country_code, secret)`.
2. **Credencial del issuer** — la credencial (vía firma o Merkle proof) corresponde a
   `issuer_root`. Garantiza que un issuer de confianza atestó esos atributos.
3. **Predicado de edad** — `is_adult == (current_year - birth_year >= 18)`.
4. **Predicado de país** — `country_ok == (country_code ∈ lista_permitida)`.
5. **Nullifier bien formado** — `nullifier == Poseidon(secret, address_hash)`. Liga el
   secreto al address sin revelar el secreto, y permite anti-replay.

## Decisiones de diseño (implementadas en `kyc-zk`)

- **Curva: BLS12-381** (NO BN254 por defecto de Circom).
  - Razón: el verificador on-chain es el `groth16_verifier` oficial de soroban-examples,
    que usa host functions BLS12-381 (CAP-0059, disponible).
  - BN254/Poseidon nativas (CAP-0074/75) siguen siendo propuestas, no disponibles.
  - Implicación: Poseidon con constantes de circomlib (BN254) reusadas sobre el campo
    BLS12-381. Válido para MVP; en producción, parámetros Poseidon específicos del campo.
  - Detalle técnico: ver `identity/circuits/src/kyc.circom`, línea 17-32.

- **Poseidon** para todos los hashes (commitment, nullifier, hash de inclusión Merkle):
  es ZK-friendly. → [[Primitivas ZK en Stellar]]

- **Predicados como salidas públicas booleanas**: el verificador ve sólo `is_adult = 1`,
  nunca la edad real.

- **Address binding** vía `address_hash` en el nullifier y validado por el
  [[Contrato Verificador (Soroban)|contrato]]: evita reventa de pruebas.

- **Issuer: Merkle tree (implementado)**, NO firma EdDSA.
  - El issuer publica un árbol Merkle de credenciales; cada prueba incluye un Merkle path.
  - `issuerRoot` = raíz del árbol.
  - La inclusión Merkle es curva-agnóstica (funciona sobre BLS12-381).
  - EdDSA/BabyJubJub NO funciona: está definido sobre el campo escalar de BN254, inválido
    bajo BLS12-381.
  - Mejor para revocación, anti-sybil, y escala.
  - Implementación: `MerkleInclusion(LEVELS)` template en el circuito.

## Contrato de interfaz con el verificador

El orden y significado de los **public inputs** debe coincidir exactamente con lo que
espera el [[Contrato Verificador (Soroban)]]:

```
public_inputs = [ issuer_root, address_hash, nullifier, current_year, is_adult, country_ok ]
```

## Riesgos / consideraciones

- **Colusión issuer↔verificador:** si comparten un identificador del usuario podrían
  correlacionar. Mitigado por el `secret` privado y nullifiers por contexto.
- **`current_year` como input público:** debe validarse contra el ledger para que el
  usuario no mienta la edad usando un año falso. Documentar en el contrato.
- **Lista de países:** si es grande, considerar Merkle/lookup en vez de comparaciones
  lineales (coste de constraints).

Relacionado: [[Fundamentos ZK]] · [[Contrato Verificador (Soroban)]] · [[Modelo de Datos]] · [[Flujo de KYC]]
