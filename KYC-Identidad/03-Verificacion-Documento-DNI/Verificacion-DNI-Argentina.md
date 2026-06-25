# 03 — Verificación de Documento: DNI Argentino

Objetivo de esta etapa (V1): confirmar que el documento es un **DNI argentino legítimo** y que sus datos **existen y coinciden** con el registro oficial.

## Anatomía del DNI argentino (tarjeta)

El DNI tarjeta tiene un **código de barras 2D PDF417** que contiene los datos identificatorios de la persona en texto plano, con campos separados por el carácter `@`.

### Contenido típico del PDF417
Campos separados por `@` (el orden puede variar según versión de credencial):

```
<nro tramite>@<apellido>@<nombre>@<sexo>@<nro DNI>@<ejemplar>@<fecha nac>@<fecha emision>...
```

> [!note] Detalle técnico real
> El PDF417 en **modo texto** solo soporta ASCII imprimible hasta 127. La **"ñ"** es ASCII extendido → puede leerse mal. Hay que manejar este caso (normalización/encoding) o vas a tener bugs con apellidos como "Peña". Documentado por la comunidad (ver fuentes).

Cobertura real reportada: ~**99,975%** de los DNI se leen correctamente con un buen escáner PDF417.

## Las 3 sub-verificaciones del documento

### V1.a — Lectura / parseo (client-side, UX)
Escanear el **PDF417** desde la webcam o desde la foto del dorso y parsear los campos. Esto da los datos estructurados (nombre, DNI, sexo, fecha nac.).
- Librerías: `zxing` / `@zxing/library` (JS, lee PDF417 en el navegador), o `Dynamsoft` (comercial, más robusto).
- PoCs argentinos de referencia (open source) en las fuentes.
- **Esto solo lee, no valida.** Un PDF417 se puede falsificar/recrear. No alcanza por sí solo.

### V1.b — Autenticidad del documento (anti-falsificación)
Verificar que la **imagen** del DNI es un documento real y no una edición/print/pantalla:
- Chequeos de **medidas de seguridad** del DNI (microtextos, roseta, tintas, MRZ si aplica) — ver [características de seguridad oficiales](https://www.argentina.gob.ar/interior/dni/caracteristicas-y-medidas-de-seguridad-de-tu-dni).
- Detección de **document presentation attacks**: foto de una foto, screenshot, documento recortado/editado.
- Coherencia: que los datos del **PDF417** coincidan con los datos **OCR del frente** (mismo nombre/DNI). Una inconsistencia = fraude probable.
- En la práctica esto lo hacen bien los proveedores (Didit/Truora/Regula) o se aproxima con modelos propios. Hacerlo 100% propio y robusto es difícil.

### V1.c — Validación contra la fuente de verdad (RENAPER) ← lo que lo hace REAL
Confirmar que el DNI **existe** y los datos **coinciden** con el registro nacional:
- **RENAPER** expone servicios para, dado **nro de documento + sexo**, obtener los datos identificatorios oficiales e incluso correr **validación biométrica** (rostro/huella).
- Esto convierte "leí un código de barras" en "el Estado confirma que esta persona existe y estos son sus datos".
- Acceso: como organización autorizada o vía proveedor licenciado (ver `05`).

> [!important] Sin RENAPER, el documento NO está realmente validado
> Parsear el PDF417 + OCR + anti-spoof sube la barra, pero la **prueba de existencia real** la da RENAPER. Es el corazón del "sin mockear".

## Pipeline recomendado (V1)

```
[Frente DNI]  → OCR + anti-spoof + extracción de foto facial
[Dorso DNI]   → lectura PDF417 → parseo de campos
                      │
                      ▼
        Cross-check (PDF417 vs OCR frente)  ── inconsistencia → RECHAZO
                      │ OK
                      ▼
        RENAPER: datos(DNI+sexo) existen y coinciden  ── no → RECHAZO
                      │ OK
                      ▼
                  V1 = OK  → pasa a V2 (biometría)
```

## Riesgos / gotchas reales
- **Encoding "ñ"** en PDF417 (arriba).
- DNIs **viejos vs nuevos** (distintos layouts y contenido del PDF417).
- **Calidad de cámara**: el escaneo PDF417 falla con baja resolución/desenfoque → guiar UX (recuadro, foco).
- **Falsos PDF417**: por eso V1.b + V1.c son obligatorios, no opcionales.
- **Acceso RENAPER**: requiere habilitación legal/contrato. Plan B: proveedor licenciado que ya tiene el canal.

## Siguiente
→ [[04-Biometria-Rostro-y-Liveness/Biometria-y-Liveness]]

## Fuentes
- [Parseo del código PDF417 del DNI argentino](https://estandaresparadummies.blogspot.com/2020/12/parseo-del-codigo-pdf417-del-dni.html)
- [PoC lectura PDF417 DNI (GitHub, Naahuel)](https://github.com/Naahuel/poc-dni-pdf417)
- [Problemas de escaneo PDF417 y cómo resolverlos](https://estandaresparadummies.blogspot.com/2020/12/problemas-para-el-escaneo-del-codigo.html)
- [Características y medidas de seguridad del DNI (Argentina.gob.ar)](https://www.argentina.gob.ar/interior/dni/caracteristicas-y-medidas-de-seguridad-de-tu-dni)
- [API de Verificación de DNI RENAPER (Didit)](https://didit.me/es/blog/argentina-renaper-dni-verification-api/)
