---
tags:
  - implementacion
  - estado/implementado
---

# Plan de Demo

El video de 2-3 min es **requisito** ([[Reglas y Requisitos]]). Tiene que mostrar el
proyecto funcionando y explicar **qué hace el ZK**. No hace falta salir en cámara.

## Guion (≈ 2:30)

| Tiempo | Escena | Qué se ve / dice |
|---|---|---|
| 0:00-0:20 | **Problema** | "El KYC tradicional te obliga a subir tu pasaporte a cada servicio. Cada copia es una fuga." → [[Problema y Solucion]] |
| 0:20-0:40 | **Solución** | "Verificas tu identidad una vez y luego *pruebas* que cumples, sin revelar tus datos, on-chain en Stellar." |
| 0:40-1:00 | **Credencial** | Terminal: issuer mock firma una credencial para el usuario. → [[Modelo de Datos]] |
| 1:00-1:40 | **Prueba ZK** | Terminal: el cliente genera la prueba de `edad ≥ 18`. Mostrar que la entrada privada (fecha de nacimiento) **no** aparece en la prueba ni en los públicos. *Aquí el ZK es load-bearing.* |
| 1:40-2:10 | **Verificación on-chain** | `stellar contract invoke verify_and_register` en testnet → éxito. Mostrar el tx en el explorer. → [[Contrato Verificador (Soroban)]] |
| 2:10-2:30 | **Consumo** | `is_verified(address)` → `true`. dApp da acceso "sin saber quién es el usuario". Cierre. |

## Qué resaltar para los jueces

- 🎯 **ZK load-bearing:** sin la prueba, el contrato no podría confiar en el usuario sin
  recibir su PII. Mostrarlo, no decirlo.
- 🎯 **Toca Stellar de verdad:** verificación **on-chain en testnet**, tx visible.
- 🎯 **Real-world:** encuadrarlo como rampa fiat / compliance (el fuerte de Stellar).
- 🎯 **Honestidad:** decir claramente qué es mock (el issuer, los datos). Lo valoran.

## Checklist de grabación

- [ ] Script `e2e_demo.sh` corre limpio de principio a fin (ensayar 2x)
- [ ] Terminal con fuente grande y legible
- [ ] Tener el explorer de testnet abierto para mostrar el tx
- [ ] Guion locutado o subtítulos
- [ ] Duración final 2-3 min
- [ ] Subir (YouTube/Loom no listado) y enlazar en el README

## Estructura del README del repo de código (también evaluado)

1. Qué es (1 párrafo) + el problema que resuelve.
2. Cómo funciona + **qué hace exactamente el ZK** (diagrama de [[Arquitectura General]]).
3. Cómo correrlo (setup + `e2e_demo.sh`).
4. Qué está terminado y **qué es mock / WIP** (honestidad).
5. Enlace al video y a esta vault de docs.

Relacionado: [[Casos de Uso]] · [[Flujo de KYC]] · [[Roadmap]]
