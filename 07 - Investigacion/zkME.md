---
tags:
  - investigacion
  - zk
---

# zkMe

> KYC con **Zero-Knowledge** que ya funciona en otras blockchains, pero **no está
> implementado en Stellar**. Es nuestra principal **referencia de producto**: lo estudiamos
> para diseñar el nuestro desde cero.

🌐 https://www.zk.me/

## Por qué nos interesa

zkMe resuelve lo mismo que buscamos: **identidad verificada sin exponer datos personales**,
con un mecanismo anti-sybil (proof of personhood) que garantiza **una persona = una
identidad**. Validas una vez y pruebas muchas veces ante distintas dApps.

Que **no exista en Stellar** es nuestra oportunidad: traer este patrón sería una integración
**construida desde cero**, no una copia ([[IDEA]]).

## Cómo lo adaptamos → ver el blueprint

El análisis detallado de qué tomar de zkMe y cómo bajarlo a Stellar/Soroban está en:

→ **[[Plano del KYC inspirado en zkMe]]** (en `09 - Tools`).

## Relacionado

- [[IDEA]] · [[Prueba de Persona Única]] · [[Plano del KYC inspirado en zkMe]]
- [[Flujo de KYC]] · [[Diseño del Circuito ZK]]
