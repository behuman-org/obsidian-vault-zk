---
título: Endpoints y pantallas del front definitivo — beHuman
tipo: especificación para el agente de frontend
estado: análisis (no implementa front)
fecha: 2026-06-27
---

# beHuman — Mapa del Front Definitivo (pantallas ↔ endpoints)

> **Para el agente de frontend:** este documento es **autosuficiente**. Construí el front
> definitivo basándote solo en esto. Cada pantalla dice **qué ve el humano**, **qué datos
> mostrar**, **de dónde salen** (REST / contrato Soroban / hook de SDK) y **qué queda oculto**.
> Cada ítem está marcado **[existente]** (ya está en el código) o **[planificado]** (solo diseño).
>
> **Norte:** *lo más humano posible*. Mostrar lo que una persona común necesita; **nunca**
> plomería técnica. Ver §7 (lo que NO se muestra) y §6 (traducción a lenguaje humano).

---

## 1. El producto y sus 3 capas

beHuman es una plataforma donde **personas reales y únicas** participan **sin revelar quiénes
son**. Se apoya en pruebas de conocimiento cero (ZK) sobre Stellar/Soroban. Tiene 3 capas.

- **Capa 1 · Identidad (KYC-ZK).** *Prueba de persona única.* La persona valida su identidad
  una vez (DNI + cara) y obtiene una credencial; un contrato la registra como **"Verificado ✓"**
  sin que su identidad real (PII) quede expuesta nunca. Es el cimiento: todo lo demás exige
  estar verificado. Fuentes: [[IDEA]], [[Vision General]], [[Flujo de KYC]], [[Contrato Verificador (Soroban)]].

- **Capa 2 · Opinión anónima.** Personas verificadas opinan como **humanos únicos** bajo un
  **seudónimo estable** (no atable a su identidad real), con curaduría de IA + moderación. En el
  front definitivo esta capa **no es una pantalla propia**: su modelo de anonimato e identidad
  seudónima alimenta las **opiniones por campaña** (recorrido E). Fuentes:
  [[Plataforma de Opinión Verificada]], [[Identidad Pública vs Anónima]], [[Curaduría y Agentes Validadores]].

- **Capa 3 · Funding ZK.** Crowdfunding **anónimo y condicional** de causas: una persona
  verificada **dona** (su aporte genera rendimiento en un vault Blend vía DeFindex), y los fondos
  solo llegan a la causa si se cumplen las reglas (**meta + hitos + 2-de-3 firmas**); si la
  campaña falla, se **devuelve** el aporte (todo-o-nada). Además se puede **opinar por campaña**,
  con 1 voz por persona. Fuentes: [[01 - Vision y Alcance]], [[04 - Flujo End-to-End (con ZK)]],
  [[10 - Implementación Capa 3 (ground-funding)]].

---

## 2. Principios del front

1. **Cero PII, cero cripto a la vista.** El front nunca muestra DNI, nombre, foto, ni
   addresses, XDR, hashes, commitments, nullifiers, platformId completo, IDs de contrato ni
   proofs. Todo eso se traduce a **etiquetas humanas** (§6) o se esconde (§7).
2. **Anonimato no vinculable.** La identidad del donante y del que opina **nunca** se revela ni
   se insinúa. Un seudónimo corto es lo máximo que se muestra, y solo cuando aporta valor.
3. **Estados, no jerga.** "Fondos entregados a la causa", no "escrow released". "Te devolvimos
   tu aporte", no "refund tx hash…".
4. **Una verdad por pantalla.** Cada vista responde una pregunta humana: *¿soy válido? ¿a qué
   causa ayudo? ¿cuánto junté? ¿ya puedo liberar?*

---

## 3. Mapa de pantallas (front definitivo)

| # | Recorrido | Pantalla | Capa | Estado |
|---|---|---|---|---|
| A1 | Identidad | Acceso ("Entrá con tu wallet") | 1 | [existente] |
| A2 | Identidad | Consentimiento de privacidad | 1 | [existente] |
| A3 | Identidad | Subir foto del DNI | 1 | [existente] |
| A4 | Identidad | Tus datos + cotejo anti-fraude (rebote del DNI) | 1 | [existente] |
| A5 | Identidad | Escaneo de cara (liveness) | 1 | [existente] |
| A6 | Identidad | Resultado "Verificado ✓" | 1 | [existente] |
| A7 | Identidad | Mi estado de verificación | 1 | [existente] |
| B1 | Explorar | Listado de causas / campañas (+ filtros) | 3 | [existente] |
| B2 | Explorar | Detalle de campaña | 3 | [existente] |
| C1 | Donar | Donar (flujo anónimo, XLM) | 3 | [existente] (dev) / [planificado] (on-chain browser) |
| C2 | Donar | Confirmación del aporte | 3 | [existente] |
| C3 | Donar | Mis aportes / mi posición (capital + rendimiento) | 3 | [existente] + hueco (§8) |
| C4 | Donar | Recuperar mi aporte (devolución todo-o-nada) | 3 | [existente] |
| D1 | Validador | Bandeja de campañas a validar | 3 | [existente] |
| D2 | Validador | Hitos (aprobar) | 3 | [existente] |
| D3 | Validador | Liberar fondos a la causa (2-de-3) | 3 | [existente] |
| D4 | Validador | Disputas | 3 | [planificado] |
| E1 | Opiniones | Hilo de opiniones + barómetro de sentimiento | 3/2 | [existente] |
| E2 | Opiniones | Publicar opinión anónima | 3/2 | [existente] |

---

## 4. Fichas de pantalla

> Formato: **Qué ve el humano · Datos · De dónde salen · Oculto · Estado**.
> Convención de tipos: **REST** (matcher :8787 / platform :8788 / funding :8789), **Contrato**
> (Soroban), **SDK** (`@behuman/sdk`, navegador).

---

### Recorrido A — Onboarding / Verificación de identidad (Capa 1)

> Fuentes: [[Flujo de KYC]], [[Matcher de Identidad (Gate de Capa 1)]],
> [[Spec — Matcher DNI + Selfie (Capa 1)]], [[Puente-KYC-a-ZK]], [[Cumplimiento-Argentina]].
> Flujo real en código: `web/src/kyc/KycFlow.tsx`.

#### A1 · Acceso
- **Qué ve:** "Entrá con tu wallet" + una línea: *"Tu identidad queda verificada de forma
  privada; nunca publicamos tus datos."* Botón **Conectar**.
- **Datos:** estado de conexión (conectado / no).
- **De dónde salen:** SDK wallet (Freighter / Stellar Wallets Kit) — `web/src/kyc/wallet.ts` [existente].
- **Oculto:** la address cruda (no se muestra; a lo sumo un avatar/etiqueta "Tu wallet").
- **Estado:** [existente].

#### A2 · Consentimiento
- **Qué ve:** explicación corta de privacidad (la foto del DNI y la cara se usan solo para
  validar y **no se guardan**), botón **Acepto**.
- **Datos:** texto legal/privacidad (estático).
- **De dónde salen:** estático (front). Base: [[Cumplimiento-Argentina]] [existente].
- **Oculto:** nada técnico.
- **Estado:** [existente].

#### A3 · Subir foto del DNI
- **Qué ve:** zona para subir la foto del **frente del DNI** (que se vea la cara). Feedback:
  ✅ "Documento válido" / ❌ "Eso no parece un DNI, probá de nuevo".
- **Datos:** preview de la imagen (local), resultado de validez (ok / motivos en humano).
- **De dónde salen:** **REST** `POST /document` (matcher) — multipart `document` →
  `{ ok, reasons[] }`. Cliente: `web/src/kyc/api.ts → checkDocument` [existente].
- **Oculto:** OCR, detección de cara, conteo de keywords/tokens. Solo se muestra ok/❌.
- **Estado:** [existente].

#### A4 · Tus datos + cotejo anti-fraude
- **Qué ve:** formulario **año de nacimiento · país · nº de documento**. Aviso: *"Estos datos se
  cotejan con tu DNI; si no coinciden, te pediremos un documento válido."* Si **no coinciden**,
  el front **rebota el DNI** (vuelve a A3) con el motivo en humano: *"El DNI no coincide con tus
  datos (el año de nacimiento). Subí un documento válido."*
- **Datos:** los 3 campos declarados; resultado del cotejo (ok / qué campo no coincide).
- **De dónde salen:** **REST** `POST /verify-data` (matcher) — multipart `document, birthYear,
  docId, countryCode` → `{ ok, reasons[], mismatches[] }` (mismatches = nombres de campo:
  `doc_number | birth_year | country`, **nunca** los valores). Cliente: `verifyDocumentData`
  [existente].
- **Oculto:** los valores leídos por OCR del DNI (no se devuelven ni se muestran); el cotejo
  real corre en el backend.
- **Estado:** [existente]. *(El cotejo se vuelve a enforcar autoritariamente en `/enroll`.)*

#### A5 · Escaneo de cara (liveness)
- **Qué ve:** cámara con guía ("parpadeá / girá la cabeza"), barra de progreso de captura.
- **Datos:** frames capturados (en memoria, local), señal de vivacidad.
- **De dónde salen:** captura local (`web/src/kyc/FaceScan.tsx`); el match real ocurre en
  `enroll`. El front **no** llama un endpoint de liveness aparte [existente].
- **Oculto:** descriptores faciales, score de match, distancia, frames (nunca se persisten).
- **Estado:** [existente].

#### A6 · Resultado "Verificado ✓"
- **Qué ve:** **"✅ Sos un humano verificado"**. Si algo falla, mensaje humano (cara no coincide,
  documento ya usado, etc.). Acceso a explorar causas.
- **Datos:** verificado sí/no; motivo si falla.
- **De dónde salen (cadena real):**
  1. **REST** `POST /enroll` (matcher) — multipart `document, selfie[], commitment, docId,
     birthYear, countryCode` → `{ ok, reasons[], issuerRoot?, pathElements?, pathIndices? }`
     (gate cara+doc + cotejo + de-dup anti-Sybil + emisión). Cliente: `enroll` [existente].
  2. **SDK** `generateProof(credencial, address)` → prueba ZK en el dispositivo [existente].
  3. **Contrato** `kyc_verifier.verify_and_register(address, proof, public_inputs)` y luego
     `is_verified(address) → bool` [existente].
- **Oculto:** commitment, secret, issuerRoot, camino Merkle, proof, nullifier, la tx, el address.
  El front solo traduce a "Verificado ✓".
- **Estado:** [existente].
- **Motivos de rechazo → humano:** ver §6 (`face_mismatch`, `already_enrolled`, `data_mismatch`,
  `not_an_id_document`, `document_unreadable`, `no_liveness_motion`…).

#### A7 · Mi estado de verificación
- **Qué ve:** *"Estás verificado ✓"* o *"Todavía no te verificaste"*. (Reentrada con la wallet.)
- **Datos:** verificado sí/no.
- **De dónde salen:** **Contrato** `kyc_verifier.is_verified(address) → bool` (vía SDK
  `isVerified`, por simulación, sin fee) [existente]. UI: `web/src/kyc/Status.tsx`.
- **Oculto:** el address (se usa internamente; se muestra "tu wallet"); cualquier dato del KYC.
- **Estado:** [existente].

---

### Recorrido B — Explorar causas / campañas (Capa 3)

> Fuentes: [[01 - Vision y Alcance]], [[04 - Flujo End-to-End (con ZK)]]. Código: `funding/api`,
> `web/src/funding/`. Tipos: `packages/shared` (`Campaign`, `Milestone`).

#### B1 · Listado de causas / campañas
- **Qué ve:** tarjetas de campaña: **título**, **resumen**, **barra de avance** (juntado / meta),
  **plazo** ("cierra el …"), **estado** (Activa / Meta alcanzada / Entregada / Devolviendo).
  Filtros simples: *Activas · Meta alcanzada · Cerradas*.
- **Datos por tarjeta:** `title`, `summary`, `raisedAmount`, `goalAmount`, `asset` (XLM),
  `deadline`, `state`.
- **De dónde salen:** **REST** `GET /campaigns` (funding) → `Campaign[]` [existente].
- **Oculto:** `causeWallet`, `vaultAddress`, `controllerAddress`, `escrowId`, addresses de
  `signers`, `signerSecretsDev`. **Nada de esto se muestra.**
- **Estado:** [existente].

#### B2 · Detalle de campaña
- **Qué ve:** título, descripción, **avance** (juntado/meta + %), **plazo**, **estado**, lista de
  **hitos** con su estado (Pendiente / Aprobado), **barómetro de sentimiento** (👍/👎) y acceso a
  donar y a opinar.
- **Datos:** `Campaign` completa salvo lo oculto; `milestones[]` (`title`, `status`); resumen de
  sentimiento (de E1).
- **De dónde salen:** **REST** `GET /campaigns/:id` → `Campaign` [existente]; sentimiento de
  `GET /campaigns/:id/opinions` (ver E1) [existente].
- **Oculto:** ídem B1 + el detalle interno de hitos (solo título + estado humano).
- **Estado:** [existente].

---

### Recorrido C — Donar (Capa 3)

> Fuentes: [[02 - DeFindex (Yield en Blend)]], [[03 - Trustless Work (Escrow y Release)]],
> [[06 - ZK, Anonimato y Liberacion de Informacion]]. Código: `funding/api`, `packages/sdk`
> (`createDefindex`), `web/src/funding/Funding.tsx`.

#### C1 · Donar (flujo anónimo)
- **Qué ve:** *"Doná como humano verificado, sin revelar quién sos."* Campo **monto (XLM)**,
  botón **Donar**. Mensajes de progreso en humano: *"Generando tu prueba…"*, *"Tu aporte entra
  a un fondo que genera rendimiento"*.
- **Datos:** monto; confirmación.
- **De dónde salen:**
  - Gating por personhood: **SDK** prueba de pertenencia (membership) en el navegador [existente].
  - **REST** `POST /campaigns/:id/donate` (funding) — body `{ donorWallet, amount,
    membershipProof }` → en dev `{ ok, donation, raisedAmount }`; en real `{ ok, xdr }` para
    firmar [existente (dev)] / [planificado] (firma+envío on-chain en el browser).
  - Detrás: **SDK** DeFindex `buildDeposit → send` (depósito al vault Blend) [existente].
  - Autoridad real: **Contrato** `campaign_controller.donate(donor, amount)` [existente, on-chain].
- **Oculto:** la wallet de donación (efímera/anónima), `membershipProof`, vault, XDR, hash. El
  front nunca asocia la donación con la identidad del KYC.
- **Estado:** [existente] en dev; el envío on-chain desde el navegador es [planificado] (§8).

#### C2 · Confirmación del aporte
- **Qué ve:** **"¡Gracias! Tu aporte de N XLM ya está ayudando a esta causa."** + nota: *"Si la
  campaña no llega a la meta, te lo devolvemos."* (Opcional: enlace "ver comprobante" **solo si
  es una tx real**; en demo no se muestra un hash falso.)
- **Datos:** monto confirmado, estado de la campaña.
- **De dónde salen:** respuesta de `POST /donate` (`raisedAmount` actualizado) [existente].
- **Oculto:** txHash mock/real, wallet, vault.
- **Estado:** [existente].

#### C3 · Mis aportes / mi posición
- **Qué ve:** lista de "tus aportes" por campaña con **capital + rendimiento** estimado
  ("Aportaste 100 XLM · hoy valen ~104 XLM"). 
- **Datos:** por aporte: campaña, monto, valor actual (capital+yield), APY humano (%).
- **De dónde salen:** **REST** `GET /campaigns/:id/position?wallet=&sig=` → `{ shares,
  underlying, apy }` (requiere **prueba de titularidad** de la wallet anónima) [existente];
  detrás **SDK** DeFindex `balance/apy` [existente].
- **Oculto:** `shares`/dfTokens, la wallet, la firma de titularidad; "shares" se muestra como
  capital, `apy` como porcentaje humano.
- **Estado:** [existente] **+ HUECO ABIERTO (§8):** cómo el front recuerda/lista los aportes de un
  donante anónimo sin un "login" que rompa el anonimato.

#### C4 · Recuperar mi aporte (todo-o-nada)
- **Qué ve:** botón **"Recuperar mi aporte"** visible solo si la campaña **falló** (venció el
  plazo sin meta). Resultado: *"Te devolvimos tu aporte de N XLM."*
- **Datos:** monto a devolver, estado.
- **De dónde salen:** **REST** `POST /campaigns/:id/refund` — body `{ donorWallet, signature }`
  (firma de titularidad) → `{ ok, refundedTo, amount }` [existente]; autoridad real **Contrato**
  `campaign_controller.refund(donor)` [existente]; detrás DeFindex `buildWithdraw` [existente].
- **Oculto:** wallet, firma, withdraw del vault, hash.
- **Estado:** [existente].

---

### Recorrido D — Panel de validador (Capa 3)

> Fuentes: [[05 - Roles y Modelo de Confianza]], [[03 - Trustless Work (Escrow y Release)]].
> Código: `funding/api` (firmas verificadas), `campaign_controller` (release/refund),
> `packages/sdk` (`createTrustlessWork`, `signFundingAction`).
> **Nota de estado:** la API verifica **firmas reales** de los validadores [existente]; la
> *orquestación 100% on-chain* y las *disputas* están [planificadas].

#### D1 · Bandeja de campañas a validar
- **Qué ve:** lista de campañas donde la persona es validador (causa / plataforma / neutral),
  con su estado y qué falta ("2 hitos por aprobar", "lista para liberar").
- **Datos:** campañas + hitos + estado.
- **De dónde salen:** **REST** `GET /campaigns` filtrado en el front por rol [existente].
  *(Endpoint "campañas por validador" dedicado: [planificado], hoy se filtra del listado.)*
- **Oculto:** addresses de los firmantes (se muestran roles: "Causa", "Plataforma", "Neutral").
- **Estado:** [existente] (filtrado en front).

#### D2 · Hitos (aprobar)
- **Qué ve:** lista de hitos con botón **Aprobar** (solo para el rol Plataforma). Tras aprobar:
  "Hito aprobado ✓".
- **Datos:** hitos (`title`, `status`).
- **De dónde salen:** **REST** `POST /campaigns/:id/milestones/:mid/approve` — body
  `{ signature: { signer, signature } }` (**firma Stellar** del aprobador sobre un challenge)
  → milestone actualizado [existente]. La firma la produce **SDK** `signFundingAction`
  [existente]. Detrás: TW `approveMilestone` [existente]; orquestación on-chain [planificado].
- **Oculto:** el challenge, la firma, la address; se muestra el rol y el estado del hito.
- **Estado:** [existente].

#### D3 · Liberar fondos a la causa (2-de-3)
- **Qué ve:** acción **"Liberar a la causa"**, habilitada solo si **meta alcanzada + todos los
  hitos aprobados**, y que requiere **2 de 3 firmas** (Causa / Plataforma / Neutral). El front
  muestra cuántas firmas hay ("1 de 2 necesarias"). Resultado: *"Fondos entregados a la causa
  (capital + rendimiento)."*
- **Datos:** condiciones (meta/hitos), firmas reunidas, estado final.
- **De dónde salen:** **REST** `POST /campaigns/:id/release` — body `{ signatures: [{signer,
  signature}] }` → `{ ok, state, txHash, capitalPlusYield, apy }` [existente]. Firmas: **SDK**
  `signFundingAction` / validación `validReleaseSigners` [existente]. Autoridad real:
  **Contrato** `campaign_controller.release(signers)` (2-de-3 + meta + plazo) [existente];
  TW `releaseFunds` + DeFindex `buildWithdraw` [existente]; orquestación on-chain end-to-end
  [planificado].
- **Oculto:** firmas, addresses, escrow, vault, txHash. Se muestra "entregado a la causa".
- **Estado:** [existente] (API con firmas) / [planificado] (on-chain completo).

#### D4 · Disputas
- **Qué ve:** abrir disputa ("Algo no se cumplió") y, para el rol Neutral, resolver
  (entregar a la causa / devolver a donantes). Estado de la campaña: **"En disputa"**.
- **Datos:** estado de disputa, resolución.
- **De dónde salen:** **SDK/TW** `startDispute` / `resolveDispute` [existente en SDK] pero **sin
  endpoint REST ni UI** todavía → [planificado]. El estado `disputed` existe en el modelo.
- **Oculto:** todo lo on-chain.
- **Estado:** [planificado].

---

### Recorrido E — Opiniones por campaña (Capa 3 sobre el modelo de Capa 2)

> Fuentes: [[09 - Opiniones Anonimas sobre la Causa]], [[Identidad Pública vs Anónima]],
> [[Curaduría y Agentes Validadores]], [[Identidad anónima de plataforma (platformId)]].
> Código: `funding/api` (`/opinions`), `packages/sdk` (`generateFundingOpinionProof`).

#### E1 · Hilo de opiniones + barómetro de sentimiento
- **Qué ve:** **barómetro** 👍 a favor / 👎 en contra (conteo) y el **hilo** de opiniones, cada
  una con un **seudónimo corto** y, opcional, un badge **"aportó"**. Las opiniones en revisión
  no se muestran (o se marcan "en revisión").
- **Datos:** `opinions[]` (`handle` corto, `content`, `sentiment`, `ts`, `curation.status`),
  resumen `{ support, oppose }`.
- **De dónde salen:** **REST** `GET /campaigns/:id/opinions` → `{ opinions[], sentiment }`
  [existente].
- **Oculto:** `platformId` completo (solo seudónimo corto / `handle`), `contentHash`, `txHash`,
  nullifier. La identidad real **jamás** se muestra ni se insinúa.
- **Estado:** [existente]. *(Badge "aportó": [planificado] — requiere vincular voz↔aporte sin
  romper anonimato; ver §8.)*

#### E2 · Publicar opinión anónima
- **Qué ve:** caja de texto + selector **A favor / En contra / Neutral**, botón **Opinar**.
  Regla humana: *"1 persona = 1 voz por campaña"*. Si ya opinó, puede comentar pero su voto no
  se vuelve a contar ("ya votaste en esta campaña").
- **Datos:** texto, sentimiento.
- **De dónde salen:** **SDK** `generateFundingOpinionProof(credencial, campaignId, contenido)`
  (prueba en el navegador, anti-Sybil por campaña) [existente]; **REST** `POST
  /campaigns/:id/opinions` — body `{ content, sentiment, opinionProof }` → `CampaignOpinion`
  [existente]. Curaduría IA (Capa 2) corre detrás y decide visibilidad [existente].
- **Oculto:** la prueba, `platformId`, nullifier, `contentHash`. El front nunca liga la opinión a
  la persona ni a su wallet de donación.
- **Estado:** [existente].

---

## 5. Tabla maestra de endpoints / contratos / SDK

> Tipo: **R**=REST, **C**=Contrato Soroban, **S**=SDK. Estado: **E**=existente, **P**=planificado.

| # | Nombre | Tipo | Capa | Alimenta en el front | Estado |
|---|---|---|---|---|---|
| 1 | `POST /document` (matcher) | R | 1 | A3 validar que es un DNI | E |
| 2 | `POST /verify-data` (matcher) | R | 1 | A4 cotejo datos↔DNI / rebote | E |
| 3 | `POST /enroll` (matcher) | R | 1 | A6 gate+cotejo+de-dup+emisión | E |
| 4 | `POST /verify` (matcher) | R | 1 | (gate puro; no en flujo principal) | E |
| 5 | `GET /health` (matcher) | R | 1 | estado del servicio | E |
| 6 | `kyc_verifier.verify_and_register` | C | 1 | A6 registrar identidad | E |
| 7 | `kyc_verifier.is_verified` | C | 1 | A6/A7 "Verificado ✓" | E |
| 8 | SDK `generateProof` / `verifyAndRegister` / `isVerified` | S | 1 | A6/A7 | E |
| 9 | `GET /campaigns` (funding) | R | 3 | B1 listado / D1 bandeja | E |
| 10 | `GET /campaigns/:id` (funding) | R | 3 | B2 detalle | E |
| 11 | `POST /campaigns` (funding) | R | 3 | (alta de campaña — admin) | E |
| 12 | `POST /campaigns/:id/donate` | R | 3 | C1 donar | E (dev) / P (on-chain) |
| 13 | `GET /campaigns/:id/position` | R | 3 | C3 mi posición (capital+yield) | E |
| 14 | `POST /campaigns/:id/refund` | R | 3 | C4 recuperar aporte | E |
| 15 | `POST /campaigns/:id/milestones/:mid/approve` | R | 3 | D2 aprobar hito | E |
| 16 | `POST /campaigns/:id/release` | R | 3 | D3 liberar 2-de-3 | E |
| 17 | `POST /campaigns/:id/opinions` | R | 3 | E2 publicar opinión | E |
| 18 | `GET /campaigns/:id/opinions` | R | 3 | E1 hilo + sentimiento | E |
| 19 | `campaign_controller.donate/release/refund` | C | 3 | C1/D3/C4 (autoridad on-chain) | E |
| 20 | `campaign_controller.state/raised/contribution/goal` | C | 3 | B/C estados y montos | E |
| 21 | SDK `createDefindex` (deposit/withdraw/balance/apy) | S | 3 | C1/C3/C4 yield | E |
| 22 | SDK `createTrustlessWork` (deploy/approve/release/dispute) | S | 3 | D2/D3/D4 workflow | E (deploy/approve/release) / P (dispute UI) |
| 23 | SDK `generateFundingOpinionProof` / `bindFundingOpinion` | S | 3 | E2 prueba de opinión | E |
| 24 | SDK `signFundingAction` / `validReleaseSigners` | S | 3 | D2/D3 firmas de validador | E |
| 25 | `POST /campaigns/:id/donate/confirm` (real) | R | 3 | C1 confirmar tras firmar XDR | P |
| 26 | "campañas por validador" / "mis aportes" | R | 3 | D1 / C3 (hoy se filtra en front) | P |
| 27 | Curaduría IA (`reviewPost`, Claude) | S | 2 | E1/E2 visibilidad de opiniones | E |

> Contexto: el contrato Capa 3 está desplegado en testnet y DeFindex/Trustless Work fueron
> validados on-chain (ver [[10 - Implementación Capa 3 (ground-funding)]]). El front consume la
> **API funding** (que orquesta y refleja las reglas); la API delega la autoridad en el contrato.

---

## 6. Catálogo de estados "humanos" (traducción)

| Señal técnica (oculta) | Lo que muestra el front |
|---|---|
| `is_verified == true` / proof válida | **Verificado ✓** ("Sos un humano verificado") |
| `enroll.reasons: already_enrolled` | "Este documento ya fue validado (no se puede duplicar)" |
| `reasons: face_mismatch` | "La cara no coincide con la del documento" |
| `reasons: no_liveness_motion` | "No detectamos vivacidad — parpadeá o girá la cabeza" |
| `reasons: not_an_id_document` | "Eso no parece un DNI" |
| `reasons: document_unreadable` | "No pudimos leer el documento — subí una foto más nítida" |
| `mismatches: birth_year/doc_number/country` (verify-data) | "El DNI no coincide con tus datos (año/nº/país)" → **rebota el DNI** |
| `campaign.state = fundraising` | **Activa** ("Juntando fondos") |
| `raised >= goal` | **Meta alcanzada** |
| `now > deadline && raised < goal` | **No alcanzó la meta** (habilita devolución) |
| `milestone.status = approved` | **Hito aprobado ✓** |
| release ok (`state = released`) | **Fondos entregados a la causa** (capital + rendimiento) |
| `state = refunding` / refund ok | **Te devolvimos tu aporte** |
| `state = disputed` | **En disputa** |
| posición DeFindex (`underlying`, `apy`) | "Aportaste N · hoy vale ~M (rinde X%/año)" |
| opinión `curation.status = approved` | se muestra normal |
| `curation.status = flagged` | "⚠️ marcada por moderación" (o se oculta, según política) |
| `curation.status = escalated` | "en revisión" (no cuenta para el sentimiento aún) |
| nullifier ya usado en la campaña | "Ya votaste en esta campaña" (puede comentar, no re-votar) |

---

## 7. Lo que el front NO debe mostrar (plomería a esconder)

Nunca, en ninguna pantalla:

- **PII:** nombre, nº de DNI, fecha de nacimiento exacta, foto, cualquier dato del KYC.
- **Addresses crudas** (wallet del KYC, wallet de donación, causa, firmantes, vault). → usar
  etiquetas: "Tu wallet", "la causa", roles "Causa/Plataforma/Neutral".
- **IDs de contrato** (`kyc_verifier`, `opinion_board`, `campaign_controller`, vault DeFindex,
  escrow Trustless Work).
- **Material criptográfico:** `commitment`, `secret`, `nullifier`, `issuerRoot`, camino Merkle,
  `proof`/`publicSignals`, `contentHash`, `platformId` completo (solo seudónimo corto), `XDR`,
  `signature`/challenge, `txHash` (salvo enlace opcional a explorer **solo en tx reales**).
- **Métricas internas del gate:** OCR text, keywords/tokens, score/distancia de cara, frames.
- **Datos internos de campaña:** `vaultAddress`, `controllerAddress`, `escrowId`,
  `signerSecretsDev`, `membershipProof`, `dfTokens`/shares (mostrar como "capital").
- **Cualquier cosa que vincule** una donación o una opinión con la identidad real o entre sí.

---

## 8. Huecos / decisiones abiertas (para resolver antes/junto al front)

1. **[HUECO] "Mis aportes" sin romper el anonimato (C3).** No hay "login" de donante: la wallet
   de donación es efímera y `GET /position` exige *prueba de titularidad* (firma de esa wallet).
   **Falta decidir** cómo el front recuerda y lista los aportes de una persona (¿guardar las
   wallets efímenas en el dispositivo? ¿una "bóveda" local firmable?) sin atarlos a su identidad.
   Afecta C2/C3/C4 (mostrar histórico y habilitar refund).
2. **[planificado] Donar on-chain desde el navegador (C1).** Hoy el modo dev contabiliza directo;
   el modo real devuelve un **XDR** que la wallet efímera debe **firmar** y enviar, y necesita
   estar **fondeada** (friendbot en testnet). Falta: endpoint `POST /donate/confirm` (#25) y el
   flujo de firma en el front.
3. **[planificado] Orquestación de firma por rol en validador (D2/D3).** El release/approve real
   firma con la clave del rol correspondiente (Causa/Plataforma/Neutral). Falta definir cómo el
   validador firma desde su wallet en el front (cada rol es una persona distinta).
4. **[planificado] Disputas (D4).** Existe en SDK/modelo, falta endpoint REST + UI.
5. **[planificado] Badge "aportó" en opiniones (E1).** Mostrar que quien opina también donó, **sin**
   revelar identidad ni monto. Requiere una prueba/relación voz↔aporte anónima — definir si entra.
6. **[planificado] Endpoints de conveniencia (#26):** "campañas por validador" y "mis aportes"
   hoy se resuelven filtrando en el front; evaluar endpoints dedicados.
7. **[decisión] Activo:** por ahora **XLM** en todo el front (testnet). USDC queda para prod.
   Nota: Trustless Work es escrow de **stablecoin** (USDC) — el front no lo expone, pero el
   equipo debe saber que el activo de TW y el del vault pueden diferir (ver
   [[10 - Implementación Capa 3 (ground-funding)]]).
8. **[decisión] Curaduría/moderación:** sin clave de IA, las opiniones quedan "en revisión" y no
   se listan. Definir el texto humano y si hay una vista de moderación (hoy **fuera de alcance**
   del front de usuario).

---

## 9. Anexo — Fuentes del vault ↔ secciones

| Sección | Notas del vault |
|---|---|
| §1 Producto / capas | [[IDEA]], [[Vision General]], [[Casos de Uso]], README |
| A (Identidad) | [[Flujo de KYC]], [[Matcher de Identidad (Gate de Capa 1)]], [[Spec — Matcher DNI + Selfie (Capa 1)]], [[Modelo de Datos]], [[Contrato Verificador (Soroban)]], [[Puente-KYC-a-ZK]], [[Cumplimiento-Argentina]], [[Arquitectura General]] |
| E (Opiniones) | [[Plataforma de Opinión Verificada]], [[Identidad Pública vs Anónima]], [[Curaduría y Agentes Validadores]], [[Identidad anónima de plataforma (platformId)]], [[09 - Opiniones Anonimas sobre la Causa]] |
| B/C/D (Funding) | [[01 - Vision y Alcance]], [[02 - DeFindex (Yield en Blend)]], [[03 - Trustless Work (Escrow y Release)]], [[04 - Flujo End-to-End (con ZK)]], [[05 - Roles y Modelo de Confianza]], [[06 - ZK, Anonimato y Liberacion de Informacion]], [[07 - Arquitectura y que toca en beHuman]], [[08 - Roadmap y Preguntas Abiertas]], [[10 - Implementación Capa 3 (ground-funding)]] |
| Código real | repo `beHuman` (org ACRC-Zk): `identity/`, `platform/`, `funding/`, `packages/sdk/`, `packages/shared/`, `web/` |

---

> **Resumen para el agente de front:** construí los recorridos **A→E**. Consumí los endpoints de
> la **§5** (todos los marcados **E** ya existen). Traducí cada estado con la **§6**, escondé todo
> lo de la **§7**, y dejá resueltos/marcados los **huecos de la §8** (en especial #1 "mis aportes"
> y #2 "donar on-chain"). Norte: **humano, limpio, sin cripto a la vista.**
