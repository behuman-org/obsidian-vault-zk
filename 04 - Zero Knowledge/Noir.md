# Noir

Lenguaje tipo **Rust** para escribir circuitos ZK, creado por Aztec. Genera pruebas
**UltraHonk** (vía el backend Barretenberg). Es la opción más **legible y rápida de
iterar**.

## Pros / Contras para Stellar

- ✅ **DX excelente** — sintaxis familiar (Rust-like), fácil de leer y razonar.
- ✅ Buen ecosistema de librerías estándar (hashes, Merkle, ECDSA…).
- ✅ La verificación on-chain se abarató con el **Protocolo 26** (host functions BN254).
- ❌ Pruebas **UltraHonk más grandes** → más caras de verificar que Groth16.
- ❌ Verificador en Stellar es **comunitario** (más riesgo en un plazo corto).

## Recursos

- Docs: https://noir-lang.org/docs/
- Verificador UltraHonk para Soroban: https://github.com/yugocabrio/rs-soroban-ultrahonk
- Tutorial E2E: https://jamesbachini.com/noir-on-stellar/

## Cómo se sentiría nuestro circuito KYC en Noir

> Boceto ilustrativo, no código final. Detalle conceptual en [[Diseño del Circuito ZK]].

```rust
// main.nr (pseudocódigo Noir)
fn main(
    // privados (witness)
    birth_year: Field,
    country_code: Field,
    secret: Field,
    issuer_sig: [Field; 2],
    // públicos
    issuer_root: pub Field,
    address_hash: pub Field,
    nullifier: pub Field,
    current_year: pub Field,
    is_adult: pub Field,
) {
    // 1. Recomputar el commitment de la credencial
    let commitment = poseidon::hash([birth_year, country_code, secret]);

    // 2. Verificar que el issuer firmó ese commitment (pertenece al set/root)
    //    verify_merkle_or_signature(commitment, issuer_sig, issuer_root);

    // 3. Predicado: mayor de edad
    let age = current_year - birth_year;
    assert((age as u32 >= 18) == (is_adult == 1));

    // 4. Nullifier liga el secreto a este contexto sin revelarlo
    assert(nullifier == poseidon::hash([secret, address_hash]));
}
```

## Cuándo elegir Noir aquí

Es el **plan B / camino de velocidad**: si iterar en [[Circom]] se vuelve lento, Noir
permite avanzar más rápido a costa de pruebas más caras. Ver decisión en
[[Comparativa de Herramientas ZK]].

Relacionado: [[Circom]] · [[RISC Zero]] · [[Diseño del Circuito ZK]]
