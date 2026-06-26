---
tags:
  - meta
  - capa/1-identidad
---

# 04 — Biometría: Face Match + Liveness

Objetivo de esta etapa (V2): confirmar que **hay una persona viva** frente a la cámara y que **es la misma** que la del documento/registro.

Son **dos cosas distintas** y ambas hacen falta:

1. **Liveness / PAD** — ¿la cara está viva y presente? (anti-spoofing)
2. **Face match (1:1)** — ¿esta cara = la del DNI/RENAPER? (verificación de identidad)

Un atacante con tu foto de DNI puede pasar el match pero **no** el liveness. Un gemelo/deepfake puede pasar liveness pero **no** el match contra RENAPER. Por eso van juntas.

## V2.a — Liveness / Presentation Attack Detection (PAD)

**Qué ataca:** foto impresa, foto en pantalla, máscara, video replay, deepfake.

**Estándar de la industria:** **ISO/IEC 30107-3**, evaluado por laboratorios acreditados por NIST (el más citado: **iBeta**, vía programa NVLAP).

### Niveles iBeta (ISO 30107-3)
- **Level 1**: ataques de bajo costo (fotos, prints, pantallas).
- **Level 2**: ataques sofisticados (máscaras, maquillaje, modelos 3D).
- **Level 3** (nuevo, más exigente): BPCER/FNMR ≤ **10%** (en L1/L2 el límite es 15%).

> Para producción real, exigir al proveedor **certificación iBeta ISO 30107-3 Level 1 como mínimo, Level 2 deseable**.

### Activo vs Pasivo
- **Activo**: el usuario hace un gesto (parpadear, sonreír, girar). Más fricción, pero "elicita respuesta".
- **Pasivo**: analiza un solo frame/selfie sin pedir gestos. Mejor UX, menos intrusivo.
- **Recomendación**: pasivo + un challenge activo opcional como fallback en casos dudosos.

### Métricas que importan
- **APCER**: % de ataques que pasan como genuinos (cuanto más bajo, mejor).
- **BPCER**: % de genuinos rechazados (fricción para usuarios reales).
- **FNMR/FMR**: tasas de no-match/falso-match del motor de reconocimiento.

## V2.b — Face Match (verificación 1:1)

Comparar el **selfie vivo** contra una **foto de referencia**. Hay dos fuentes de referencia, en orden de "realness":

1. **Foto oficial de RENAPER** (vía SID) — **la mejor**. Comparás contra la foto que tiene el Estado, no contra la que subió el usuario. Devuelve un **score de confianza en <7s**.
2. **Foto del frente del DNI** (extraída por OCR) — sirve, pero la imagen la aporta el usuario → más atacable. Útil como complemento o fallback.

### Umbrales
- No inventar umbrales propios: si usás RENAPER/SID o un proveedor, **adoptá su score y umbral recomendado** (calibrados sobre millones de casos).
- Definir política para el **score intermedio** (zona gris): reintento, challenge activo, o revisión manual.

## El camino "más real" para Argentina: RENAPER SID

El **Sistema de Identidad Digital (SID)** de RENAPER:
- Compara una **selfie** con la **foto registrada en RENAPER** + foto del DNI.
- Devuelve respuesta con **porcentaje de confianza en < 7 segundos**.
- Integración por **API REST**.
- Ya procesa **+700.000 verificaciones/día**; lo usan bancos y fintechs (probado por el BCRA).

> Esto cubre V2.a (liveness en la captura) **y** V2.b (match contra el Estado) en un solo flujo oficial. Es lo que hace que nuestro KYC sea "de verdad" y no un demo.

## Pipeline recomendado (V2)

```
[Cámara] → captura selfie con liveness (PAD)
                 │  PAD falla → RECHAZO (posible reintento/challenge)
                 ▼
   Face match 1:1 contra RENAPER (SID)  ── score < umbral → RECHAZO
                 │ score ≥ umbral
                 ▼
              V2 = OK
```

## Build vs Buy (biometría)
- **Open source** (`face-api.js`, modelos propios): sirve para prototipos y para el *match* básico, **pero el liveness robusto y certificado NO se improvisa**. Los ataques (deepfakes, replay) evolucionan rápido.
- **Real / producción**: proveedor con **iBeta ISO 30107-3** y, para Argentina, **RENAPER SID** (directo o vía Didit/Truora). Ver `05`.

## Privacidad (crítico)
- La **biometría es dato sensible** (ver `09`). 
- El template/selfie **no se guarda on-chain jamás** y se retiene off-chain el mínimo tiempo legal.
- Idealmente el match ocurre en el flujo del proveedor/RENAPER y nosotros solo recibimos **OK/score**, no la imagen.

## Siguiente
→ [[05-Proveedores-y-Stack/Proveedores-y-Stack]]

## Fuentes
- [SID — Sistema de Identidad Digital (Argentina.gob.ar)](https://www.argentina.gob.ar/interior/renaper/sid-sistema-de-identidad-digital)
- [El SID valida tu identidad a distancia (BCRA)](https://www.bcra.gob.ar/noticias/sistema_de_identidad_digital_y_bcra.asp)
- [ISO 30107-3 PAD — iBeta](https://www.ibeta.com/iso-30107-3-presentation-attack-detection-confirmation-letters/)
- [Yoti MyFace passive liveness iBeta L3](https://idtechwire.com/yotis-myface-passive-liveness-meets-ibeta-pad-level-3-under-iso-iec-30107-3/)
- [Qué hay de nuevo en ISO/IEC 30107 (ID R&D)](https://idrnd.medium.com/whats-new-in-the-recent-update-of-iso-iec-30107-for-biometric-a048733c0065)
