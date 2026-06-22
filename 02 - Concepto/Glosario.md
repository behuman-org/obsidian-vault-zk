# Glosario

Términos clave del proyecto. Enlazados desde el resto de notas.

## Zero-Knowledge

- **Zero-Knowledge Proof (ZKP)** — prueba criptográfica de que una afirmación es cierta
  sin revelar nada más que su veracidad.
- **zk-SNARK** — *Succinct Non-interactive ARgument of Knowledge*. Pruebas pequeñas y
  rápidas de verificar. Groth16 y UltraHonk son SNARKs.
- **Circuito** — el programa que define qué se está probando, expresado como un sistema
  de *constraints* (restricciones aritméticas).
- **Witness** — los valores (públicos + privados) que satisfacen el circuito. La parte
  privada es el "secreto" que no se revela.
- **Señales públicas / públicos (public inputs)** — entradas visibles para el verificador
  (ej. la raíz de Merkle del issuer, el predicado, el address).
- **Proving key / Verifying key** — claves generadas en el *setup* del circuito; la VK se
  embebe en el contrato verificador.
- **Trusted setup** — ceremonia para generar parámetros. Groth16 lo requiere (por
  circuito); UltraHonk/PLONK usa un setup universal; RISC Zero lo evita en gran medida.
- **Nullifier** — valor único derivado de un secreto, usado para evitar doble uso / sybil
  sin revelar identidad.
- **Commitment** — compromiso criptográfico a un valor (ej. hash Poseidon de los
  atributos) que puedes revelar/probar después.
- **Merkle tree / Merkle root** — estructura para probar pertenencia a un conjunto (ej.
  "mi credencial está en el set de credenciales válidas del issuer") de forma compacta.

## Curvas y hashes (primitivas de Stellar)

- **BN254 (alt_bn128)** — curva elíptica amigable para pairings; base de muchos SNARKs.
  Stellar la soporta vía host functions (Protocolo 25/26). → [[Primitivas ZK en Stellar]]
- **BLS12-381** — otra curva de pairing, soportada en Stellar desde antes.
- **Poseidon / Poseidon2** — funciones hash diseñadas para ser baratas dentro de
  circuitos ZK ("ZK-friendly"). Nativas en Stellar desde Protocolo 25.
- **MSM (multi-scalar multiplication)** — operación pesada en verificación de SNARKs;
  Stellar la ofrece como host function (Protocolo 26).

## Stellar

- **Stellar** — blockchain enfocada en pagos, stablecoins y RWAs.
- **Soroban** — la plataforma de smart contracts de Stellar (contratos en Rust → Wasm).
- **Host function** — función nativa del runtime de Stellar, mucho más barata que
  implementarla en Wasm. Las primitivas ZK son host functions.
- **XLM (Lumens)** — token nativo de Stellar.
- **Testnet / Mainnet** — redes de prueba y de producción.

## Identidad / KYC

- **KYC (Know Your Customer)** — proceso de verificar la identidad de un cliente.
- **Issuer (emisor)** — entidad de confianza que verifica identidad y emite la credencial
  firmada. → [[Modelo de Datos]]
- **Credencial** — atestación firmada por el issuer sobre los atributos del usuario.
- **PII (Personally Identifiable Information)** — datos personales (nombre, documento…)
  que queremos **no** exponer on-chain.
- **Predicado** — la condición que se prueba (ej. `edad ≥ 18`, `país ∈ permitidos`).
- **Registro KYC on-chain** — contrato que guarda qué addresses están verificadas (sin
  PII).

## Herramientas

- **[[Noir]]** — lenguaje tipo Rust para circuitos (genera pruebas UltraHonk).
- **[[Circom]]** — DSL de bajo nivel para circuitos (genera pruebas Groth16).
- **[[RISC Zero]]** — zkVM para probar la ejecución correcta de programas.
- **UltraHonk** — sistema de pruebas usado por Noir/Barretenberg.
- **Groth16** — sistema de pruebas clásico, pruebas pequeñas, requiere trusted setup.
