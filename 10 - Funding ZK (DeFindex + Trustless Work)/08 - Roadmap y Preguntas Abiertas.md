# 08 — Roadmap y Preguntas Abiertas

## Roadmap (fases para implementar después)

### Fase 0 — Fundaciones
- [ ] Conseguir **API keys**: DeFindex (`api.defindex.io`) y Trustless Work (`api.trustlesswork.com`).
- [ ] Elegir **activo**: USDC en Stellar **testnet** (trustlines listas).
- [ ] Definir el **contrato controlador de campaña** (interfaz: `release`, `refund`, estados).

### Fase 1 — DeFindex (recaudación + yield)
- [ ] Deploy de un **vault** con estrategia **Blend** para una campaña de prueba.
- [ ] Flujo de **depósito** anónimo (XDR → firma → send) desde la UI.
- [ ] **Shares** por donante (base del refund); lectura de `balance`/`apy`.

### Fase 2 — Trustless Work (escrow + release)
- [ ] Deploy de **escrow Single-Release** con roles (causa, plataforma, neutral, receiver).
- [ ] **Hitos** = tareas de la campaña; flujo de *update → approve*.
- [ ] **Release multi-firma** (2-de-3) y prueba de **disputa/refund**.

### Fase 3 — Integración (handoff)
- [ ] **Manager del vault = controlador** atado al estado *released* del escrow.
- [ ] Condición compuesta: **meta + tareas + multi-firma** → release a la causa (capital+yield).
- [ ] **Refund todo-o-nada** si no se cumple.

### Fase 4 — ZK / anonimato
- [ ] Gatear donar/validar con **prueba de personhood** de Capa 1 (sin PII).
- [ ] **Disclosure por umbral** (revelar info con 2+ validadores).

### Fase 5 — Hardening
- [ ] Pruebas de no-custodia (nadie puede sacar fondos fuera de las reglas).
- [ ] Auditoría del controlador; manejo de TTL/archival de Soroban.
- [ ] Métricas y logs **sin PII**.

## Criterios de aceptación (MVP testnet)
1. Donantes anónimos aportan a una campaña; el capital genera yield en Blend.
2. Al cumplir **meta + tareas + 2-de-3 firmas**, la causa recibe **capital + yield**.
3. Si la campaña falla, los donantes **recuperan su capital** (todo-o-nada).
4. **Cero PII** on-chain; identidad del donante nunca revelada.
5. Nadie (ni la plataforma) puede liberar/retirar fondos por fuera de las reglas.

## Preguntas abiertas (decidir antes de codear)

- **Handoff exacto**: ¿el escrow de Trustless Work **recibe** físicamente el monto al
  liberar, o el controlador transfiere directo desde Blend **leyendo** el estado del escrow?
  (define el wiring on-chain).
- **Yield en caso de fallo**: si la campaña no se concreta, el **capital** se reembolsa; ¿el
  **yield** acumulado vuelve a los donantes (pro-rata), a la plataforma (cubrir costos), o
  queda para la próxima causa? (En éxito, el yield va a la causa — ya decidido.)
- **Deadline / meta**: ¿hay fecha límite además del monto? ¿meta única o tramos?
- **Disclosure por umbral**: qué info se revela, a quién y bajo qué condición legal; alcance
  mínimo de revelación.
- **Montos confidenciales**: ¿MVP con montos visibles y confidencialidad como mejora futura?
- **Validador neutral**: ¿quién lo provee? ¿un árbitro fijo, un pool, o rotativo por causa?
- **Fees**: ¿la plataforma cobra fee (Platform Address de Trustless Work / Fee Receiver de
  DeFindex)? ¿cuánto?
- **Multi-causa**: ¿un vault por causa o un vault con sub-cuentas? (afecta el modelo de shares).

## Referencias
- DeFindex — [Docs](https://docs.defindex.io/) · [Vault Roles](https://docs.defindex.io/getting-started/getting-started/vault-roles.md) · [API](https://docs.defindex.io/api-integration-guide/api.md)
- Trustless Work — [Docs](https://docs.trustlesswork.com/trustless-work) · [Roles](https://docs.trustlesswork.com/trustless-work/introduction/technology-overview/roles-in-trustless-work.md) · [Lifecycle](https://docs.trustlesswork.com/trustless-work/introduction/technology-overview/escrow-lifecycle.md)
- Blend — protocolo de lending en Stellar (estrategia del vault)
- Contexto interno: [[Arquitectura General]] · [[Estructura del Codigo]] · [[Puente-KYC-a-ZK]]
