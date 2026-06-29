---
tags:
  - meta
  - capa/1-identidad
---

# 05 — Proveedores y Stack (Build vs Buy)

Regla: **no reinventar liveness ni el canal a RENAPER.** Reinventamos solo la **capa ZK** (que es nuestro diferencial). El KYC se apoya en infraestructura real probada.

## Opción A — RENAPER directo (máxima "realness", más burocracia)

- **Qué da**: validación de datos del DNI + **validación biométrica facial** contra la foto oficial (SID).
- **Cómo**: API REST de los **Servicios de Validación de Identidad** de RENAPER.
- **Requisito**: ser **organización autorizada** (convenio/habilitación). Hay fricción legal y de onboarding.
- **Cuándo**: cuando human tenga entidad legal y volumen que justifique el canal directo.

## Opción B — Proveedor licenciado sobre RENAPER (recomendado para arrancar real ya)

Agregadores que **ya tienen el canal a RENAPER** y suman doc + liveness certificado, vía una sola API:

| Proveedor | Fuerte en | Notas |
|-----------|-----------|-------|
| **Didit** | Verificación DNI + RENAPER (doc y non-doc) | Documentan API RENAPER específica para Argentina |
| **Truora** | Identidad digital LatAm / Argentina | Guías 2026 de verificación en AR |
| **Regula** | Autenticidad documental (forense) | Muy fuerte en chequeo de seguridad del documento |
| (otros) | Onfido, Sumsub, Veriff, iProov | Globales; verificar cobertura RENAPER real en AR |

> [!tip] Estrategia recomendada
> **Arrancá con un proveedor licenciado (Opción B)** para tener KYC real desde el MVP, detrás de una **interfaz `IdentityProvider`** propia. Cuando escale, migrás a **RENAPER directo (Opción A)** sin tocar el resto del sistema.

## Opción C — Open source / propio (solo dev y piezas no críticas)

| Pieza | Open source | ¿Sirve para prod? |
|-------|-------------|-------------------|
| Lectura PDF417 | `@zxing/library` | Sí (lectura/UX) |
| OCR documento | `tesseract.js` | Parcial (complementario) |
| Face match 1:1 | `face-api.js`, `InsightFace` | Solo prototipo |
| **Liveness/PAD** | — | **No** (usar certificado iBeta) |
| Validación RENAPER | — | **No existe** open; es canal oficial |

Conclusión: lo open sirve para la **UX de captura** y prototipos; la **validación real** (RENAPER + liveness certificado) va por A/B.

## Stack técnico propuesto

```
Frontend (captura)         → React/TypeScript (encaja con human)
                             - @zxing/library  (PDF417 del dorso)
                             - getUserMedia / cámara (selfie)
                             - SDK del proveedor de liveness (web)

Backend "Issuer service"   → Node/TypeScript (o Rust)
                             - interfaz IdentityProvider (A/B/C intercambiable)
                             - orquesta V1 (doc+RENAPER) y V2 (liveness+match)
                             - firma la credencial, NO guarda PII de más
                             - emite commitment hacia la capa ZK

Capa ZK / on-chain         → Circom (Groth16) + verifier Soroban (ver 06/07)
```

## Interfaz `IdentityProvider` (clave para no atarse a un vendor)

```ts
interface IdentityProvider {
  // V1: documento + validación contra fuente oficial (RENAPER)
  verifyDocument(input: {
    frontImage: Blob; backImage: Blob;
  }): Promise<{ ok: boolean; person?: PersonData; reasons?: string[] }>;

  // V2: liveness + match 1:1 contra foto oficial
  verifyBiometrics(input: {
    selfieSession: LivenessSession; person: PersonData;
  }): Promise<{ ok: boolean; livenessScore: number; matchScore: number }>;
}
```

Implementaciones: `RenaperDirectProvider`, `DiditProvider`, `TruoraProvider`, `DevProvider` (solo local, claramente marcado, **nunca** en prod).

## Criterios para elegir proveedor
1. **Cobertura RENAPER real** en Argentina (no solo "soportamos LatAm").
2. **Liveness certificado iBeta ISO 30107-3** (Level ≥1, ideal 2).
3. **Modelo de datos**: que nos devuelva **OK/score**, idealmente sin que nosotros toquemos la imagen.
4. **Costos por verificación** y SLA/latencia (SID responde <7s).
5. **Cumplimiento** Ley 25.326 / AAIP (ver `09`).

## Siguiente
→ [[06-Puente-KYC-a-ZK/Puente-KYC-a-ZK]]

## Fuentes
- [Verificación de identidad en Argentina — guía 2026 (Truora)](https://blog.truora.com/es/verificacion-de-identidad-digital-en-argentina-guia-completa-2026)
- [Verificación non-doc Argentina RENAPER (Didit)](https://didit.me/es/blog/non-doc-verification-argentina-renaper/)
- [SID — modalidades y productos (Argentina.gob.ar)](https://www.argentina.gob.ar/sid/modalidades-y-productos)
