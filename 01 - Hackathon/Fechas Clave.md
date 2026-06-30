---
tags:
  - hackathon
---

# Fechas Clave

| Evento | Fecha |
|---|---|
| Apertura de submissions | 15 de junio, 12:00 AM PST |
| **Deadline de submission** | **29 de junio, 12:00 PM PST** |

> 🗓️ Hoy es **2026-06-22**. Quedan aproximadamente **7 días** para el deadline.

## Cuenta regresiva mental

```mermaid
gantt
    title Plan de trabajo hasta el deadline (29 jun 12:00 PM PST)
    dateFormat YYYY-MM-DD
    axisFormat %d %b

    section Diseño
    Cerrar arquitectura y circuito      :des1, 2026-06-22, 1d
    Elegir toolchain ZK                 :des2, 2026-06-22, 1d

    section Build
    Circuito ZK (KYC)                   :b1, 2026-06-23, 2d
    Contrato verificador Soroban        :b2, 2026-06-24, 2d
    Integración off-chain / issuer      :b3, 2026-06-25, 2d

    section Cierre
    Demo end-to-end                     :c1, 2026-06-27, 1d
    README + video 3 min                :c2, 2026-06-28, 1d
    Buffer / submission                 :crit, c3, 2026-06-29, 1d
```

## Hitos propios

- [x] **22 jun** — arquitectura y toolchain cerradas
- [x] **25 jun** — circuito + verificador funcionando aislados
- [x] **27 jun** — flujo end-to-end en testnet
- [x] **28 jun** — README pulido + video grabado
- [ ] **29 jun** — submission antes de las 12:00 PM PST (con buffer)

Ver: [[Roadmap]] · [[Plan de Demo]]
