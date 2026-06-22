# Reglas y Requisitos — Stellar Hacks: Real-World ZK

## Tema

Construir lo que quieras con **Zero-Knowledge sobre Stellar**. El abanico va de lo
*mild* (un verificador de pruebas limpio como PoC) a lo *wild* (una app de pagos
blindados completa). Categorías sugeridas: privacy pools, pagos privados, tokens
confidenciales, **pruebas de identidad y compliance** ← *aquí caemos nosotros*,
computación verificable, datos verificables.

Stellar es conocido por mover **dinero real en el mundo real** (stablecoins, pagos
transfronterizos, RWAs, settlement institucional), así que proyectos que llevan ZK a
esos casos de uso son especialmente bienvenidos. Nuestro proyecto de **KYC / identidad
con ZK** encaja directo en "identity and compliance proofs".

## Por qué ahora

Stellar pasó las últimas releases de protocolo construyendo la base criptográfica que
los sistemas ZK modernos necesitan. Ver detalle en [[Primitivas ZK en Stellar]]:

- **Protocolo 25 ("X-Ray")** — host functions nativas para primitivas ZK-friendly:
  operaciones de curva elíptica **BN254** y hashing **Poseidon / Poseidon2**.
- **Protocolo 26 ("Yardstick")** — nueve host functions BN254 adicionales
  (multi-scalar multiplication, aritmética en el scalar field, curve-membership checks);
  mueve la matemática pesada de ZK a la capa host y abarata la verificación on-chain,
  incluyendo pruebas de NoirLang.
- **BLS12-381** de protocolos anteriores.

> ⚠️ **Importante:** estas primitivas son *building blocks*. No te dan pagos privados
> end-to-end por sí solas. Generas las pruebas **off-chain** con un sistema de alto
> nivel (Noir, Circom, RISC Zero) y despliegas un **contrato verificador en Stellar**
> para chequearlas. Ese hueco entre "primitivas potentes" y "producto terminado" es
> donde viven los proyectos interesantes.

## Requisitos de elegibilidad (deliberadamente ligeros)

1. **Repo open-source** — repositorio público (GitHub/GitLab/Bitbucket) con todo el
   código fuente y un `README.md` claro. Si algo está incompleto o usa datos *mock*,
   decláralo. Prefieren un work-in-progress honesto a un misterio pulido.
2. **Video demo corto** — walkthrough de 2-3 minutos mostrando lo que construiste y
   explicando qué hace el ZK. No tiene que ser técnico ni producido. No hace falta salir
   en el video. → ver [[Plan de Demo]].
3. **ZK + Stellar** — debe usar criptografía Zero-Knowledge de forma significativa y
   tocar Stellar (p.ej. verificar pruebas en un contrato, o integrar testnet/mainnet).
   **El ZK tiene que ser load-bearing**: potencia una parte real del funcionamiento, no
   aparece sólo en una slide.

No hay framework obligatorio, ni contrato boilerplate requerido, ni track específico.

## Opciones probadas de ZK en Stellar

Tres caminos validados (detalle y comparativa en [[Comparativa de Herramientas ZK]]):

- **[[RISC Zero]]** — ejecuta código en una VM remota y prueba que se ejecutó
  correctamente. Bueno para cómputo grande off-chain.
- **[[Noir]]** — lenguaje tipo Rust para circuitos ZK. Fácil de leer y trabajar; las
  pruebas UltraHonk son más grandes y caras de verificar on-chain.
- **[[Circom]]** — circuitos de bajo nivel basados en constraints, más difíciles de
  entender pero **más baratos de verificar** (Groth16).

## Premios — pool de $10,000

| Puesto | Premio |
|---|---|
| 🥇 1º | $5,000 en XLM |
| 🥈 2º | $2,000 en XLM |
| 🥉 3º | $1,250 en XLM |
| 4º | $1,000 en XLM |
| 5º | $750 en XLM |

Track único de innovación abierta.

## Soporte

- **Stellar Dev Discord** — canal `#zk-chat` → https://discord.gg/stellardev
- **Telegram Stellar Hacks** → https://t.me/+e898qibDUVExODkx

> 🚨 Cuidado con scams por DM. El equipo nunca te escribe primero pidiendo keys, seed
> phrases o pagos.

Ver también: [[Fechas Clave]] · [[Recursos Oficiales]]
