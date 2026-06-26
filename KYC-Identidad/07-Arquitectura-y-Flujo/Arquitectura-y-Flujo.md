---
tags:
  - meta
  - capa/1-identidad
---

# 07 — Arquitectura y Flujo End-to-End

Visión completa: del navegador del usuario hasta `Verified(address)` en Soroban. Conecta esta investigación con el **diagrama existente** del proyecto (Secciones A/B/C).

## Diagrama de componentes (texto)

```
┌─────────────────────────── OFF-CHAIN ───────────────────────────┐
│                                                                  │
│  [USER · navegador]                                              │
│   - captura DNI (frente+dorso)                                   │
│   - lee PDF417 (zxing)                                           │
│   - captura selfie + liveness SDK                                │
│   - genera secret, nullifier, commitment   (en su device)       │
│            │ imágenes/sesión                  │ commitment       │
│            ▼                                   │                  │
│  [ISSUER SERVICE · backend]                    │                  │
│   IdentityProvider:                            │                  │
│    V1 doc → RENAPER (existe+coincide)          │                  │
│    V2 bio → liveness + match 1:1 (SID)         │                  │
│   de-dup por hash(id oficial)  (anti-Sybil)    │                  │
│   agrega commitment al MERKLE TREE ────────────┘                  │
│   publica issuer_root · firma credencial                         │
│   DESCARTA PII (retención mínima)                                │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
                         │ issuer_root (público)
                         ▼
┌──────────────────────── ON-CHAIN (Stellar/Soroban) ─────────────┐
│  [CIRCUITO Circom · corre en device del user]                   │
│    proof = Groth16(secret, nullifier, path | root, addr, ...)   │
│            │ proof + public signals                              │
│            ▼                                                     │
│  [register() ] → checks baratos → verify Groth16 → COMMIT       │
│            → Verified(caller), Nullifier(n)=usado               │
└──────────────────────────────────────────────────────────────────┘
```

(El detalle de `register()`, storage y verifier ya está dibujado en las Secciones A y C del diagrama del proyecto.)

## Flujo cronológico (happy path)

1. **Captura** (navegador): DNI frente+dorso, lectura PDF417, selfie con liveness.
2. **V1 — Documento**: issuer valida autenticidad + cruza datos + **RENAPER** confirma existencia/coincidencia.
3. **V2 — Biometría**: liveness (PAD) OK + **match 1:1** contra foto oficial (SID) ≥ umbral.
4. **Generación local**: el device del usuario crea `secret`, `nullifier`, `commitment`.
5. **Enrollment**: issuer de-duplica (anti-Sybil de origen), inserta `commitment` en el Merkle tree, publica `issuer_root`.
6. **Limpieza**: issuer descarta PII (retiene solo `hash(id oficial)` para de-dup, ver `09`).
7. **Registro on-chain** (cuando el usuario quiera): corre el circuito → `proof` → `register()` → `Verified(address)`.

## Dónde vive el PII (mapa de datos sensibles)

| Dato | Dónde puede estar | Dónde NUNCA |
|------|-------------------|-------------|
| Imágenes DNI / selfie | Flujo del proveedor / efímero en issuer | On-chain, logs, S3 sin cifrar |
| Datos del DNI (nombre, etc.) | Memoria del issuer durante verificación | On-chain |
| `hash(id oficial)` (de-dup) | DB del issuer (off-chain) | On-chain |
| `secret`, `nullifier` | Device del usuario | Issuer, on-chain |
| `commitment`, `issuer_root` | Merkle tree / público | (son públicos por diseño) |
| `proof`, `nullifier_hash`, predicados | On-chain | — |

> Regla de oro: **lo único que cruza a on-chain son commitments, roots, proofs, nullifiers y predicados.** Nada de PII.

## Puntos de falla y manejo
- **RENAPER caído / timeout** → encolar reintento; no emitir credencial sin V1+V2 OK.
- **Liveness en zona gris** → challenge activo o revisión manual.
- **Doble registro intentado** → rechazado en de-dup (off-chain) y/o por nullifier (on-chain).
- **Serialización Circom↔Soroban** → fuente #1 de bugs (endianness, orden de coords, length prefixes) — ver callouts del diagrama.

## Qué construir primero (resumen → ver 08)
1. Issuer service con interfaz `IdentityProvider` + un proveedor real (B).
2. Captura web (DNI + selfie).
3. Merkle tree + emisión de `issuer_root`.
4. Integración con el circuito/contrato ya diseñado.

## Siguiente
→ [[08-Roadmap-Implementacion/Roadmap]]
