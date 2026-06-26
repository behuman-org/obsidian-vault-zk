---
tags:
  - meta
  - capa/1-identidad
---

# KYC-Identidad — Investigación base (beHuman)

Objetivo: construir la capa de **KYC real (sin mocks)** que valida que detrás de una wallet hay un **humano real y único**, para luego alimentar los **circuitos ZK** y marcar `Verified(address)` on-chain en Stellar/Soroban.

Inspiración y modelo a replicar: **[zk.me](https://www.zk.me/)** — KYC con Zero-Knowledge ("One Face, One DID"), pero llevado a Stellar.

## Qué tiene que pasar (resumen de una línea)

Una persona sube su **DNI argentino** + hace un **selfie con liveness por cámara**. **Dos validaciones** deben pasar antes de tocar los circuitos:

1. **Documento** — el DNI es auténtico y sus datos son reales (parseo PDF417 + validación contra RENAPER).
2. **Biometría** — la cara del selfie está viva (anti-spoofing) y **coincide** con la foto del DNI / del registro RENAPER.

Si ambas pasan → se emite una **credencial** → se deriva un **commitment** → recién ahí entra al **circuito ZK** (ver `06` y `07`).

## Mapa de la investigación

| # | Carpeta | Contenido |
|---|---------|-----------|
| 01 | [[01-Vision-y-Alcance/Vision-y-Alcance]] | Qué construimos, alcance, principios (real, no-custodial, PII off-chain) |
| 02 | [[02-Referencia-zkMe/Referencia-zkMe]] | Cómo funciona zkMe y qué emulamos en Stellar |
| 03 | [[03-Verificacion-Documento-DNI/Verificacion-DNI-Argentina]] | DNI argentino, PDF417, RENAPER, autenticidad |
| 04 | [[04-Biometria-Rostro-y-Liveness/Biometria-y-Liveness]] | Face match + liveness/PAD (ISO 30107-3) |
| 05 | [[05-Proveedores-y-Stack/Proveedores-y-Stack]] | RENAPER SID, proveedores, build vs buy, stack |
| 06 | [[06-Puente-KYC-a-ZK/Puente-KYC-a-ZK]] | De credencial verificada → commitment → circuito → Soroban |
| 07 | [[07-Arquitectura-y-Flujo/Arquitectura-y-Flujo]] | Arquitectura end-to-end y flujo de datos |
| 08 | [[08-Roadmap-Implementacion/Roadmap]] | Fases concretas para el equipo |
| 09 | [[09-Cumplimiento-y-Privacidad/Cumplimiento-Argentina]] | Ley 25.326, datos biométricos, retención |

## Decisión central (recomendación)

> Para que sea **lo más real posible en Argentina**, la fuente de verdad es **RENAPER** vía su **Sistema de Identidad Digital (SID)**: compara un selfie con la foto oficial del DNI y devuelve un score en segundos. Se accede de forma directa (siendo organización autorizada) o a través de un **proveedor licenciado** (Didit, Truora, etc.). Esa validación 1:1 contra el Estado es la diferencia entre "KYC de verdad" y un face-match casero.

## MVP testnet: el matcher como gate

Para testnet, el "KYC real" arranca como un **matcher de cara**: el usuario sube la foto del
DNI y escanea su cara con la cámara; solo si **coinciden** se crea la identidad de Capa 1.
Es la versión testnet (swappable por RENAPER después). Spec y arquitectura en las notas del
proyecto: [[Matcher de Identidad (Gate de Capa 1)]] y [[Spec — Matcher DNI + Selfie (Capa 1)]].

> [!warning] La capa ZK NO reemplaza al KYC
> Los circuitos prueban *"pertenezco al set de humanos verificados"* sin revelar quién. Pero **alguien tiene que haber verificado de verdad** primero. Esta carpeta es ese "alguien" (el issuer/oráculo de personhood).
