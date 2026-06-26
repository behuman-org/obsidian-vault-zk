---
tags:
  - meta
  - capa/1-identidad
  - anonimato
---

# 09 — Cumplimiento y Privacidad (Argentina)

> [!warning] No es asesoría legal
> Esto es una guía técnica de referencia. Antes de producción, **validar con un abogado especializado en protección de datos** y revisar criterios de la **AAIP** (Agencia de Acceso a la Información Pública).

Marco principal: **Ley 25.326 — Protección de los Datos Personales** ("Habeas Data") y normativa/resoluciones de la AAIP.

## Conceptos clave que nos aplican

### Consentimiento
El tratamiento de datos es **ilícito** sin **consentimiento libre, expreso e informado**, que debe **constar por escrito** o medio equiparable. 
→ **Acción**: pantalla de consentimiento explícita **antes** de capturar DNI/biometría, explicando qué se hace, para qué, y por cuánto tiempo se conserva.

### Datos sensibles y biométricos
- Nadie puede ser **obligado** a dar datos sensibles.
- Los **datos biométricos** son **sensibles cuando pueden revelar otros datos** cuyo uso pueda causar discriminación. La biometría facial cae en una categoría a tratar con **máximo cuidado**.
→ **Acción**: tratar selfie/template facial como dato sensible; minimizar, cifrar, y preferir que el **match ocurra en el proveedor/RENAPER** y que nosotros solo recibamos **OK/score**.

### Retención y destrucción
- Cumplida la prestación contractual, los datos **deben destruirse**, salvo autorización expresa; con autorización se pueden conservar con seguridad **hasta 2 años**.
→ **Acción**: política de **borrado automático** de imágenes/PII tras la verificación. Conservar solo lo imprescindible (ej. `hash(id oficial)` para de-dup) y registros de auditoría **sin PII**.

### Derechos del titular (acceso, rectificación, supresión)
Los datos deben almacenarse de modo que el titular pueda ejercer su **derecho de acceso**.
→ **Acción**: proceso para que un usuario consulte/borre sus datos (en la medida en que existan off-chain).

## Cómo nuestro diseño ya ayuda al cumplimiento

| Principio legal | Cómo lo cumplimos por diseño |
|-----------------|------------------------------|
| Minimización | On-chain solo van commitments/proofs/nullifiers, **nunca PII** |
| Destrucción | El issuer **descarta PII** tras emitir la credencial |
| Sensibles con cuidado | Liveness/match en el proveedor; recibimos OK/score, no la imagen |
| No discriminación | El circuito prueba **hechos** (mayor de edad) sin revelar datos |
| Consentimiento | Pantalla explícita previa a la captura |

## El factor "inmutabilidad" de blockchain (riesgo legal)
El derecho de **supresión** choca con la inmutabilidad on-chain. **Por eso es crítico** que **ningún PII** toque la cadena: lo que está on-chain (commitment, nullifier) es **anónimo y no reversible** a la identidad. Si pusiéramos PII on-chain, sería **imposible** cumplir el borrado. → Refuerza la regla de oro de `07`.

## Checklist de cumplimiento (pre-producción)
- [ ] Consentimiento informado por escrito/equiparable, previo a captura.
- [ ] Aviso de privacidad: qué datos, finalidad, plazo, proveedor, derechos.
- [ ] Registro de la base de datos ante la AAIP (si corresponde).
- [ ] Política de retención + borrado automático implementada.
- [ ] Cifrado de datos en tránsito y reposo.
- [ ] Contrato de tratamiento con el proveedor de KYC (encargado de tratamiento).
- [ ] Verificar que el proveedor cumple Ley 25.326 y dónde aloja los datos.
- [ ] Procedimiento de acceso/rectificación/supresión para titulares.
- [ ] DPIA / evaluación de impacto por uso de biometría.
- [ ] **Cero PII on-chain** (verificado técnicamente).

## Fuentes
- [Ley 25.326 — Texto actualizado (Argentina.gob.ar)](https://www.argentina.gob.ar/normativa/nacional/ley-25326-64790/actualizacion)
- [Ley 25.326 — Infoleg (texto)](https://servicios.infoleg.gob.ar/infolegInternet/anexos/60000-64999/64790/texact.htm)
- [Ley simple: Datos personales (Argentina.gob.ar)](https://www.argentina.gob.ar/justicia/derechofacil/leysimple/datos-personales)
