---
tags:
  - meta
  - capa/1-identidad
---

# 01 — Visión y Alcance

## El problema a resolver

human necesita una **prueba de personhood real**: que detrás de una dirección de Stellar haya un humano **real** (no un bot) y **único** (no la misma persona 1000 veces), **sin exponer su identidad**. La tensión "real + anónimo" se resuelve en dos capas:

- **Capa de realness (KYC, off-chain)** → esta investigación. Verifica al humano de verdad.
- **Capa ZK (on-chain)** → prueba pertenencia al set de verificados + nullifier anti-doble-registro.

Esta carpeta cubre **solo la primera capa**, que es el cimiento. Sin un KYC sólido, todo lo ZK que venga después prueba pertenencia a un set… que podría estar lleno de identidades falsas.

## Qué construimos (alcance)

Un **issuer/oráculo de personhood** que:

1. Captura **DNI argentino** (frente + dorso) desde el navegador.
2. Captura un **selfie con detección de vida (liveness)** usando la cámara.
3. Ejecuta **dos validaciones independientes** que ambas deben pasar:
   - **V1 – Documento**: el DNI es legítimo y sus datos existen/coinciden con el registro oficial.
   - **V2 – Biometría**: la cara está viva y **es la misma persona** del documento/registro.
4. Si V1 ∧ V2 → emite una **credencial verificable** + un **commitment** (`hash(secreto)`) que es lo único que viaja hacia la capa ZK.
5. **Descarta el PII** según política de retención (ver `09`).

## Lo que NO es (anti-alcance, por ahora)

- No es un wallet ni un contrato. Eso es otra capa.
- No guarda biometría on-chain. **Nunca**.
- No hace AML/sanciones todavía (Tier 2 de zkMe). Fase posterior.
- No mockea: aunque para desarrollo local se use un "modo dev", el **diseño objetivo es contra RENAPER real** desde el día 1 (ver `05`).

## Principios de diseño

- **Real primero.** La fuente de verdad para Argentina es **RENAPER** (Estado), no un face-match casero.
- **PII off-chain, siempre.** La blockchain solo ve commitments/nullifiers/proofs. Nunca documento, cara, nombre.
- **Minimización de datos.** Capturar lo mínimo, retener lo mínimo legal (ver Ley 25.326 en `09`).
- **No-custodial del secreto.** El secreto de identidad que genera el commitment lo controla el usuario (estilo Semaphore), no nosotros.
- **Auditable y reproducible.** Cada decisión de "verificado/rechazado" debe loguearse (sin PII) para auditoría.
- **Modular.** El proveedor de KYC (RENAPER directo vs Didit vs Truora) tiene que ser intercambiable detrás de una interfaz.

## Definición de "verificado" (criterio de aceptación)

Una identidad se considera **verificada** sí y solo sí:

```
V1_documento  == OK   (DNI auténtico + datos validados contra RENAPER)
AND
V2_liveness   == OK   (PAD pasa, score de vida ≥ umbral)
AND
V2_match      == OK   (similaridad selfie↔foto oficial ≥ umbral RENAPER/SID)
AND
nullifier     no usado (1 humano = 1 registro; se chequea en la capa ZK)
```

Recién con esto se habilita la emisión de la credencial.

## Siguiente

→ [[02-Referencia-zkMe/Referencia-zkMe]] para entender el modelo que copiamos.
