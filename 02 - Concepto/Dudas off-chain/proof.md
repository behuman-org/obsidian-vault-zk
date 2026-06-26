---
tags:
  - concepto
---

La _prueba_ en sí: el resultado que produce el circuito. Es un dato criptográfico que demuestra "yo pertenezco al conjunto de humanos verificados y cumplo las reglas" **sin revelar cuál de ellos sos**. Lo genera el usuario off-chain y lo manda a la cadena junto con los _public inputs_ (señales públicas). El contrato `groth16_verifier` la chequea y responde true/false, sin volver a correr el circuito.