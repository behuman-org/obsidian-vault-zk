---
tags:
  - funding
  - capa/3-funding
  - zk
---

# 10 — Funding ZK (DeFindex + Trustless Work)

Nueva integración para human: **crowdfunding anónimo para causas**, construido sobre la
identidad verificada-pero-anónima de [[Arquitectura General|Capa 1]] y dos integraciones de
Stellar:

- **[DeFindex](https://docs.defindex.io/)** → capa de **yield**: el dinero recaudado vive en
  un **vault que asigna a Blend** y genera rendimiento mientras la causa se llena.
- **[Trustless Work](https://docs.trustlesswork.com/trustless-work)** → capa de **escrow /
  liberación**: gobierna **cuándo y si** el dinero sale hacia la causa (tareas + monto
  cumplidos + aprobación de 2+ validadores), con disputas y reembolsos.

> [!info] La idea en una línea
> Una persona **anónima pero verificada** dona vía DeFindex → la plata queda en **Blend**
> generando yield → cuando se cumplen **las tareas y el monto total acordado**, **Trustless
> Work** libera los fondos (capital + yield) a la **wallet de la causa**, con el acuerdo de
> **causa + plataforma + neutral**. Si no se cumple, **se reembolsa** (todo-o-nada).

> [!success] Estado: IMPLEMENTADO (rama `ground-funding`, exploratoria)
> Circuito + contrato `campaign_controller` (14 tests) + API + SDK + Web, e2e en dev verde.
> El detalle de la implementación real (lo construido) está en
> **[[10 - Implementación Capa 3 (ground-funding)]]**. Las notas 01–09 son la visión/diseño.

## Decisiones tomadas (base de todo el diseño)

| Tema | Decisión |
|---|---|
| **Custodia / handoff** | DeFindex envía la plata a **Blend** y queda ahí. Cuando se cumple todo, **Trustless Work** libera esos fondos guardados en Blend hacia la causa que inició la petición. |
| **Destino del yield** | **A la causa** (recauda más de lo donado). |
| **Validadores del release** | **Causa + plataforma + tercero neutral** (multi-firma). |
| **Si no se cumple** | **Todo-o-nada**: se reembolsa a los donantes. |
| **ZK (corazón)** | La **identidad nunca se revela**. Donaciones anónimas; participación gateada por personhood de Capa 1. |

## Mapa de la investigación

| # | Nota | Contenido |
|---|------|-----------|
| 01 | [[01 - Vision y Alcance]] | Qué construimos, para quién, alcance y principios |
| 02 | [[02 - DeFindex (Yield en Blend)]] | Cómo funciona DeFindex, vaults, roles, depósito/retiro |
| 03 | [[03 - Trustless Work (Escrow y Release)]] | Roles, ciclo de vida, single/multi-release, disputas |
| 04 | [[04 - Flujo End-to-End (con ZK)]] | El recorrido completo, paso a paso, con anonimato |
| 05 | [[05 - Roles y Modelo de Confianza]] | Quién hace qué; mapeo a causa/plataforma/neutral/donante |
| 06 | [[06 - ZK, Anonimato y Liberacion de Informacion]] | Cómo se mantiene privado; revelar info con acuerdo de 2+ |
| 07 | [[07 - Arquitectura y que toca en human]] | Componentes y carpetas del monorepo que se tocan |
| 08 | [[08 - Roadmap y Preguntas Abiertas]] | Fases, decisiones pendientes, referencias |
| 09 | [[09 - Opiniones Anonimas sobre la Causa]] | Opinar incógnito (ZK) dentro de cada campaña |
| 10 | [[10 - Implementación Capa 3 (ground-funding)]] | **Lo construido**: circuito, contrato, API, SDK, web, auditoría |

## Cómo encaja con lo que ya existe

- **Reusa Capa 1:** solo humanos verificados (anónimos) pueden donar/validar/opinar. El gate
  es una **prueba ZK de pertenencia** al árbol del issuer (`issuerRoot`), **no**
  `is_verified(address)` — para no deanonimizar. → [[Identidad anónima de plataforma (platformId)]].
- **Es una nueva vertical de aplicación** (junto a la [[Plataforma de Opinión Verificada]]):
  human pasa de "opinar con confianza" a "**financiar causas con confianza**", sin exponer
  identidades.
- **Trae la opinión adentro del funding:** cada campaña tiene su hilo de **opiniones
  anónimas verificadas** (mismo modelo `platformId` de Capa 2, scopeado a la causa) →
  [[09 - Opiniones Anonimas sobre la Causa]].

## Fuentes
- DeFindex: https://docs.defindex.io/
- Trustless Work: https://docs.trustlesswork.com/trustless-work
