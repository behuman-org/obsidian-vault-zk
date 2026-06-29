---
tags:
  - funding
  - capa/3-funding
  - zk
---

# 01 — Visión y Alcance

## El problema

Donar a causas hoy implica **confiar a ciegas**: ¿el dinero llega? ¿se usa para lo que
dijeron? ¿quién controla los fondos en el medio? Y del otro lado, mucha gente **no dona ni
apoya causas sensibles** (políticas, de denuncia, de derechos) por **miedo a quedar
expuesta**. human ya resuelve "humano real + anónimo"; acá lo extendemos a **mover dinero
hacia causas, de forma anónima y con garantías**.

## Qué construimos

Una vertical de **crowdfunding anónimo y condicional** dentro de human:

1. La **plataforma estudia una causa** (due diligence por fuera) y la publica como campaña
   con: **monto meta**, **tareas/hitos**, **wallet de la causa** y **validadores**.
2. Un **donante anónimo pero verificado** (Capa 1) aporta dinero **vía DeFindex** → los
   fondos quedan en un **vault Blend** generando **yield**.
3. Cuando se cumplen **las tareas + el monto total acordado**, **Trustless Work** libera los
   fondos (capital + yield) a la **causa**, con aprobación de **causa + plataforma + neutral**.
4. Si **no** se cumple (no se llega a la meta o no se aprueban las tareas) → **reembolso**
   a los donantes (**todo-o-nada**).

## Para quién (actores)

- **Donante** — humano verificado que quiere apoyar una causa **sin exponer su identidad**.
- **Causa / beneficiario** — quien recibe los fondos al cumplir las condiciones.
- **Plataforma (human)** — estudia causas, orquesta DeFindex + Trustless Work, co-valida.
- **Validador neutral** — tercero que aprueba/disputas para evitar abuso de cualquier parte.

## Principios de diseño

- **ZK primero, identidad nunca revelada.** Ni del donante ni (salvo acuerdo explícito) del
  beneficiario. Ver [[06 - ZK, Anonimato y Liberacion de Informacion]].
- **No-custodial.** Ni human ni nadie puede llevarse los fondos: DeFindex y Trustless Work
  son no-custodiales por diseño; el release exige reglas y multi-firma.
- **Productivo mientras espera.** El dinero recaudado **no queda quieto**: genera yield en
  Blend, y ese yield **va a la causa**.
- **Condicional y verificable.** El dinero solo se mueve si se cumplen tareas + monto, con
  acuerdo de 2+ partes. Todo auditable on-chain (sin PII).
- **Todo-o-nada.** Si la campaña falla, el donante recupera lo suyo. Maximiza confianza.

## Alcance del MVP (testnet)

- 1 causa → 1 campaña → 1 vault DeFindex (Blend) → 1 escrow Trustless Work.
- Activo: **stablecoin** (ej. **USDC** en Stellar testnet) — Trustless Work es escrow de
  stablecoin; el vault de Blend opera el mismo activo.
- Donación **anónima** gateada por personhood (Capa 1). Montos visibles on-chain en el MVP
  (montos confidenciales = mejora futura, ver `06`).
- Release **Single-Release** (todo-o-nada) con tareas como milestones.

## Anti-alcance (por ahora)

- No es un exchange ni on/off-ramp de fiat (eso lo resuelve otra integración).
- No hace scoring crediticio ni AML del donante.
- No define la due diligence legal de la causa (es un proceso humano de la plataforma; acá
  solo registramos su resultado).

## Siguiente
→ [[02 - DeFindex (Yield en Blend)]]
