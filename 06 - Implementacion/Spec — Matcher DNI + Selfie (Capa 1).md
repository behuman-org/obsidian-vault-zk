---
tags:
  - implementacion
  - capa/1-identidad
  - estado/implementado
---

# Spec — Matcher DNI + Selfie (Capa 1)

> [!abstract] Para el agente
> Este documento es el **brief de implementación**. Describe **qué hay que construir** y
> **qué partes del monorepo `beHuman` se tocan**, sin código todavía. Vista arquitectónica
> en [[Matcher de Identidad (Gate de Capa 1)]]. Investigación de fondo en la carpeta
> `KYC-Identidad/`.

## 1. Objetivo

Construir el **gate de verificación de identidad** que precede a la creación de la
identidad de Capa 1. El usuario **sube la foto de su DNI** y **escanea su cara con la
cámara**; si ambas validaciones pasan (**match 1:1** + **liveness**), recién entonces el
issuer genera la **credencial + commitment** y continúa el [[Flujo de KYC]].

Alcance: **testnet**. No sale a producción. El matcher es **reemplazable** por RENAPER/SID
más adelante sin tocar el resto (misma interfaz `IdentityProvider`).

## 2. Decisiones ya tomadas (contexto del producto)

- El **face match corre en un servidor** (backend) que se encarga completo del matcher
  (decidido con el equipo). El frontend solo captura.
- Entrada = **foto de DNI subida** + **cara escaneada por cámara** (no dos fotos subidas).
- Liveness para testnet = **captura en vivo por cámara** (+ challenge simple opcional). Sin
  PAD certificado por ahora.
- **Ningún PII on-chain.** Lo único que cruza a la cadena son commitment/proof/nullifier
  ([[Modelo de Datos]], [[Cumplimiento-Argentina]]).

## 3. Dónde encaja (no rompe nada existente)

```
[web/] captura  →  [identity/issuer/ + matcher] valida  →  crea identidad Capa 1
                                                          →  [packages/sdk] prover
                                                          →  [identity/contracts/kyc_verifier] on-chain
```

El gate vive **antes** del commitment. La Capa ZK, el contrato y el modelo de datos
on-chain **no cambian**.

## 4. Contrato funcional del gate

**Entrada:**
- `documento`: imagen del DNI (frente, con la foto de la cara).
- `selfie`: frames/imagen capturada **en vivo** por la cámara.

**Proceso:**
1. Detectar y recortar la **cara del documento**. Si no hay cara detectable → error.
2. Detectar la **cara del selfie**. Si no hay cara / no es captura en vivo → error.
3. Calcular **embeddings** de ambas y la **similitud** (coseno).
4. Evaluar **liveness** (captura por cámara, challenge opcional).
5. Decidir: `match = similitud ≥ umbral` ∧ `liveness OK`.

**Salida (lo que consume el issuer):**
```
{
  ok: boolean,            // match ∧ liveness
  matchScore: number,     // similitud 0..1
  livenessOk: boolean,
  reasons: string[]       // motivos de rechazo si ok=false
}
```
> No devolver imágenes ni embeddings hacia afuera. Solo OK/score/razones.

**Criterio de validación (gate):** `ok === true` habilita la creación de la identidad de
Capa 1. Si `ok === false` → rechazo + reintento.

## 5. Qué archivos/carpetas toca (monorepo `beHuman`)

Referencia de estructura: [[Estructura del Codigo]].

| Acción | Ruta en `beHuman` | Qué se hace |
|---|---|---|
| **Nuevo** módulo matcher (servidor) | `identity/issuer/` (submódulo, ej. `identity/issuer/matcher/`) | Servicio backend que recibe documento+selfie, hace detect+embed+compare+liveness, devuelve OK/score |
| **Interfaz** intercambiable | `identity/issuer/` | `IdentityProvider` con impl. `MatcherTestnetProvider` (hoy) y hueco para `RenaperProvider` (futuro) |
| **Modificar** issuer | `identity/issuer/` | Antes de emitir credencial, llamar al gate; solo si `ok` → generar `secret`/`commitment`/firma ([[Modelo de Datos]]) |
| **Nuevo** UI de captura | `web/` | Pantalla: subir foto DNI + componente de cámara (`getUserMedia`) que escanea la cara; postea al backend |
| **Tipos compartidos** | `packages/shared/` | `MatchResult`, `KycVerificationInput`, estados del gate (TS) |
| **Config** | `beHuman/.env.example` | `MATCH_THRESHOLD`, modelo, provider activo (`testnet` / `renaper`) |
| **Sin cambios** | `identity/circuits/`, `identity/contracts/kyc_verifier/`, `platform/*` | El gate es previo al commitment; no se tocan |
| **Docs** | `docs/` → enlaza a esta nota | Declarar que el matcher es **testnet** (lo pide la hackathon: marcar mocks/alcance) |

## 6. Interfaz a respetar (para que sea swappable)

Conceptual (definición fina en [[Puente-KYC-a-ZK]] y [[Proveedores-y-Stack]]):

```
IdentityProvider.verifyBiometrics({ documento, selfie })
   → { ok, matchScore, livenessOk, reasons }
```
- **Hoy:** `MatcherTestnetProvider` (modelo de face match en el servidor).
- **Mañana:** `RenaperProvider` (match contra foto oficial vía SID) — misma firma.

El issuer **no debe saber** cuál provider está atrás. Cambiar de uno a otro = una variable
de entorno.

## 7. Datos y privacidad (obligatorio)

- Documento y selfie son **PII sensible** → manejar efímero, no loguear imágenes, borrar
  tras la verificación. Ver [[Cumplimiento-Argentina]] (Ley 25.326: consentimiento,
  retención, destrucción).
- Pantalla de **consentimiento** antes de capturar cara/documento.
- El **secret** y la identidad de Capa 1: decidir si lo genera el issuer (modelo actual de
  [[Modelo de Datos]]) o el usuario (modelo Semaphore de [[Puente-KYC-a-ZK]]). **Decisión
  abierta** (ver §9).

## 8. Criterios de aceptación (Definition of Done)

1. Un usuario sube foto de DNI + escanea su cara → el backend devuelve `ok=true` y el issuer
   crea la credencial/commitment de Capa 1.
2. Una cara **distinta** a la del DNI → `ok=false`, no se crea identidad.
3. Una **foto estática** frente a la cámara (sin vivacidad) → rechazada (o al menos
   degradada con el challenge).
4. El provider es **intercambiable** por config, sin tocar el issuer ni la Capa ZK.
5. **Cero PII** en logs y on-chain; imágenes borradas tras verificar.

## 9. Decisiones abiertas (resolver antes de codear)

- **Umbral de match**: arrancar conservador y **calibrar** con caras reales (no inventar).
- **Quién genera el secret**: issuer-side ([[Modelo de Datos]]) vs user-side / no-custodial
  ([[Puente-KYC-a-ZK]]). Afecta UX de recuperación.
- **Liveness**: solo captura en vivo vs + challenge (parpadeo/giro) vs + anti-pantalla.
- **Stack del backend**: a definir (Python con modelo de face recognition, o Node). El
  equipo dijo "servidor que se encargue del matcher" → libre, detrás de la interfaz.
- **Modelo/face engine**: elegir motor de embeddings (calidad vs peso/licencia).

## 10. Lo que este gate NO hace (anti-alcance)

- No valida que el DNI sea **auténtico/existente** (eso es RENAPER, producción).
- No hace AML/sanciones (Tier 2 de zkMe, [[Referencia-zkMe]]).
- No toca circuitos ni contrato.
- No guarda biometría on-chain. Nunca.

## Referencias
- Arquitectura del gate: [[Matcher de Identidad (Gate de Capa 1)]]
- Flujo completo: [[Flujo de KYC]] · [[Arquitectura General]]
- Biometría/liveness: [[Biometria-y-Liveness]]
- Puente a ZK: [[Puente-KYC-a-ZK]] · Proveedores: [[Proveedores-y-Stack]]
- Privacidad: [[Cumplimiento-Argentina]]
- Estructura de código: [[Estructura del Codigo]]
