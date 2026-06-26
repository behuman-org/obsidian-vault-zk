---
tags:
  - meta
  - capa/1-identidad
  - estado/implementado
---

# 08 — Roadmap de Implementación

Objetivo: KYC **real** (no mock) lo antes posible, con piezas intercambiables. Cada fase deja algo funcionando de punta a punta.

## Fase 0 — Fundaciones (1ª semana)
- [ ] Definir entidad legal / vía de acceso a RENAPER (directo vs proveedor licenciado). **Bloqueante para "real".**
- [ ] Elegir proveedor (Didit / Truora / otro) con **cobertura RENAPER** + **liveness iBeta**. Ver `05`.
- [ ] Definir política de datos con base en Ley 25.326 (consentimiento, retención). Ver `09`.
- [ ] Diseñar interfaz `IdentityProvider` (contrato de tipos).

## Fase 1 — Captura web (UX)
- [ ] Pantalla de captura DNI (frente/dorso) con guías de encuadre.
- [ ] Lectura **PDF417** del dorso con `@zxing/library` (manejar encoding "ñ").
- [ ] Captura de **selfie** + integración del **SDK de liveness** del proveedor.
- [ ] Generación local de `secret`/`nullifier`/`commitment` (lib Poseidon/Semaphore en el cliente).
- [ ] Consentimiento informado explícito antes de capturar biometría.

## Fase 2 — Issuer service (backend) con proveedor REAL
- [ ] Implementar `IdentityProvider` con el proveedor elegido (Opción B).
- [ ] **V1**: validar documento + **RENAPER** (existe + coincide).
- [ ] **V2**: liveness (PAD) + **match 1:1** contra foto oficial (SID).
- [ ] Regla de aceptación `V1 ∧ V2` (criterio en `01`).
- [ ] `DevProvider` aparte para tests locales (claramente marcado, jamás en prod).

## Fase 3 — Anti-Sybil + Merkle tree
- [ ] De-dup por `hash(identificador oficial)` (sin guardar DNI en claro).
- [ ] Construir/actualizar el **Merkle tree** de commitments.
- [ ] Publicar `issuer_root`; firmar credencial (opcional).
- [ ] Job de **retención/borrado de PII** post-verificación.

## Fase 4 — Integración con la capa ZK (ya diseñada)
- [ ] Conectar el `commitment` con el **circuito Circom** (inputs privados/públicos).
- [ ] Pruebas de **serialización Circom ↔ Soroban** (endianness, coords, length prefixes).
- [ ] `register()` end-to-end en testnet → `Verified(address)`.
- [ ] Caso de **doble registro** rechazado (de-dup + nullifier).

## Fase 5 — Hardening y auditoría
- [ ] Métricas: APCER/BPCER, tasa de aprobación, latencia (SID <7s).
- [ ] Logs **sin PII** para auditoría.
- [ ] Revisión de seguridad del verifier (el de soroban-examples es **PoC no auditado**).
- [ ] Pentest del flujo de captura (replay, deepfake, document spoofing).
- [ ] Revisión legal AAIP / Ley 25.326.

## Hitos / "Definition of Done" del MVP
1. Un usuario real con DNI argentino completa DNI+selfie y queda `Verified(address)` en **testnet**, con validación **RENAPER real**.
2. Un segundo intento de la **misma persona** es **rechazado** (anti-Sybil).
3. **Ningún PII** aparece on-chain ni en logs.
4. El proveedor de KYC es **reemplazable** sin tocar la capa ZK.

## Riesgos principales
- **Acceso RENAPER** (legal/tiempos) → mitigar con proveedor licenciado en paralelo.
- **Costo por verificación** → modelar con volumen esperado.
- **Serialización ZK** → presupuestar tiempo, es la fuente #1 de bugs.
- **Cumplimiento biométrico** → involucrar legal temprano (`09`).

## Siguiente
→ [[09-Cumplimiento-y-Privacidad/Cumplimiento-Argentina]]
