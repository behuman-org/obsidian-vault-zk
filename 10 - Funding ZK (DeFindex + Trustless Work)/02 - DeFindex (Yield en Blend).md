---
tags:
  - funding
  - capa/3-funding
  - zk
---

# 02 — DeFindex (Yield en Blend)

Rol en beHuman: **dónde vive el dinero recaudado y cómo genera rendimiento** mientras la
causa se llena. Fuente: https://docs.defindex.io/

## Qué es DeFindex

Protocolo en Stellar/Soroban que hace el yield **plug-and-play**: creás un **vault** sobre un
activo (ej. USDC) y el vault asigna los fondos a una o más **estrategias** (en nuestro caso,
**Blend** — protocolo de lending de Stellar). El usuario deposita y recibe **shares** que
representan su parte; el valor de la share crece con el yield. Auditado, con funciones de
seguridad **rebalance** y **rescue**.

## Conceptos que usamos

- **Vault** — contenedor del dinero de una campaña. 1 campaña = 1 vault (o un vault por
  causa). El activo subyacente es la stablecoin.
- **Strategy (Blend)** — el vault coloca el capital en Blend para generar interés.
- **Shares** — lo que recibe el donante al depositar. **Clave para el reembolso todo-o-nada**:
  si la campaña falla, cada donante retira sus shares y recupera su capital.
- **APY** — rendimiento; en nuestro modelo el yield generado **se destina a la causa**.

## Roles del vault (y cómo los usamos)

DeFindex define roles que son **direcciones** y **pueden ser contratos** (control por
políticas). Esto es central: el "dueño" del vault puede ser un **contrato controlador** que
solo deja mover fondos según las reglas de la campaña.

| Rol DeFindex | Qué hace | En beHuman |
|---|---|---|
| **Manager** | Dueño; configura, asigna roles, puede ser multisig/contrato | **Contrato controlador de campaña** (reglas de release/refund) |
| **Rebalance Manager** | Mueve fondos entre estrategias | Bot/operador de la plataforma (mantiene en Blend) |
| **Emergency Manager** | `rescue`/pausa de una estrategia ante problemas | Multisig de seguridad de la plataforma |
| **Fee Receiver** | Recibe fees del vault | Plataforma (si aplica) |

> [!important] Ningún rol puede retirar los fondos de los usuarios
> La doc lo dice explícito: los roles **no** pueden sacar el dinero de los donantes. El
> capital solo sale por las reglas (release a la causa o refund a donantes). Esto es lo que
> lo hace **no-custodial**.

## Flujo técnico (depósito / retiro)

DeFindex se integra por **API** (`api.defindex.io/vault/{vaultAddress}/{endpoint}`) o **SDK
TypeScript**. Patrón típico:

```
1. Donante pide "depositar X"   → API arma la tx (devuelve XDR)
2. Donante FIRMA el XDR          → con su wallet (anónima)
3. Se envía el XDR firmado       → /send
4. El vault coloca el capital en Blend → empieza a generar yield
```

Endpoints relevantes: `deposit`, `withdraw`, `balance`, `apy`. Ver
[API Integration Guide](https://docs.defindex.io/api-integration-guide/api.md) y el
[SDK TS](https://docs.defindex.io/advanced-documentation/sdks.md).

## Cómo encaja con el resto

- **Mientras se recauda:** todas las donaciones entran al vault → Blend → yield.
- **Al cumplirse las condiciones:** el **controlador** (Manager = contrato) autoriza el
  **withdraw** de Blend para que **Trustless Work** lo libere a la causa (ver `03` y `04`).
- **Si falla:** los donantes **retiran sus shares** (refund todo-o-nada).

> [!note] Punto de integración a definir
> La pieza fina: que el **Manager del vault sea un contrato** cuyo permiso de withdraw esté
> atado al **estado "released" del escrow de Trustless Work**. Decisión de implementación en
> [[07 - Arquitectura y que toca en beHuman]] y [[08 - Roadmap y Preguntas Abiertas]].

## Siguiente
→ [[03 - Trustless Work (Escrow y Release)]]

## Fuentes
- [Welcome / What is DeFindex](https://docs.defindex.io/)
- [Vault Roles](https://docs.defindex.io/getting-started/getting-started/vault-roles.md)
- [API Integration Guide](https://docs.defindex.io/api-integration-guide/api.md)
