---
tags:
  - concepto
---

# Casos de Uso

Escenarios donde un KYC con ZK sobre Stellar aporta valor real. Útil para el pitch y el
[[Plan de Demo]].

## 1. Rampa fiat / on-ramp regulada (el caso estrella)

Una rampa fiat necesita por ley saber que el usuario pasó KYC y no está en una lista de
sanciones. Hoy almacena tu pasaporte. Con ZK: el usuario presenta una prueba *"KYC nivel
2, país permitido"* y la rampa abre el servicio **sin ver ni guardar tu documento**.

Encaja perfecto con el fuerte de Stellar: stablecoins y pagos transfronterizos.

## 2. Acceso a pools / tokens de RWA con compliance

Tokens de activos del mundo real (RWA) suelen estar restringidos a inversores
verificados/acreditados. Un contrato de pool consulta el [[Modelo de Datos|registro KYC]]
para permitir suscripción sólo a addresses con prueba válida — sin saber quién es cada
inversor.

## 3. Pruebas de edad / país sin doxear

Una app prueba que el usuario es **mayor de 18** o **reside en un país permitido** sin
revelar fecha de nacimiento ni dirección. El circuito evalúa el predicado y sólo expone
el booleano. → [[Diseño del Circuito ZK]]

## 4. "Proof of unique person" anti-sybil

Usando un *nullifier* derivado de la identidad, una persona sólo puede registrarse una
vez por contexto (ej. un airdrop, una votación), evitando ataques sybil, sin revelar
identidad. Mismo motor, distinto predicado.

## 5. Credencial reutilizable cross-dApp

El usuario hace KYC una vez y reutiliza la credencial para generar pruebas en cualquier
dApp del ecosistema Stellar que confíe en el mismo issuer. Una verificación, muchos usos.

---

## Matriz de priorización para la demo

| Caso de uso | Impacto en jueces | Esfuerzo | ¿En el MVP? |
|---|---|---|---|
| Rampa fiat regulada | 🔥 Alto (real-world) | Medio | ✅ Historia principal |
| Prueba de edad/país | 🔥 Alto (claro y visual) | Bajo | ✅ Predicado del MVP |
| RWA / pool gated | Alto | Alto | 🔸 Mencionar como extensión |
| Anti-sybil | Medio | Medio | 🔸 Future work |
| Cross-dApp | Medio | Bajo (es consecuencia del diseño) | ✅ Mencionar |

> **Narrativa de la demo:** contar el caso #1 (rampa fiat) usando el predicado del caso
> #3 (edad ≥ 18 + país permitido). Es el más concreto y "real-world".

Relacionado: [[Vision General]] · [[Flujo de KYC]] · [[Plan de Demo]]
