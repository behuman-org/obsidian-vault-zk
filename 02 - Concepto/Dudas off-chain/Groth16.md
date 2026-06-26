---
tags:
  - concepto
  - zk
---

Es un _==esquema de prueba_ Zero-Knowledge== concreto (uno de varios; otro es PLONK). Es la "receta matemática" que permite generar una prueba ZK muy pequeña y muy barata de verificar. Sus dos virtudes: la prueba ocupa poquísimo y verificarla on-chain cuesta poco gas. Su costo: requiere un _trusted setup_ por circuito (la ceremonia de Powers of Tau de la Sección B). En tu producto, el circuito Circom genera la prueba usando Groth16.