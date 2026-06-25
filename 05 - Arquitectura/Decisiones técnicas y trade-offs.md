# Decisiones técnicas y trade-offs

**Rama:** `kyc-zk` · Decisiones que rigen el diseño de beHuman CAPA 1.

Cada decisión tiene una **razón**, un **trade-off**, y un **futuro** (cómo evoluciona).

---

## 1. Curva BLS12-381 (no BN254)

**Decisión:** usar BLS12-381 para el circuito Circom.

**Razón:**
- El `groth16_verifier` oficial de soroban-examples usa **host functions BLS12-381**
  (CAP-0059, disponible en Soroban SDK 25+).
- BN254/Poseidon nativas (CAP-0074/75) siguen siendo **propuestas, no disponibles**.
- Usando BLS12-381 podemos verificar en testnet hoy.

**Trade-off:**
- Poseidon "no estándar": usamos constantes de circomlib (definidas para BN254) reusadas
  en BLS12-381. Son elementos de campo válidos, pero **no es Poseidon específico de BLS**.
- Riesgo: auditoría futura podría exigir parámetros Poseidon propios del campo.

**Futuro:**
- Cuando CAP-0074/75 estén disponibles (Protocol ~27–28), podremos migrar a BN254 +
  Poseidon nativo.
- Esto sería un cambio de circuito + contrato; los usuarios no se verían afectados (el
  `secret` y `address` siguen siendo iguales).

---

## 2. Atestación por Merkle (no firma EdDSA)

**Decisión:** probar que la credencial pertenece al issuer usando Merkle path, no firma
EdDSA.

**Razón:**
- EdDSA con BabyJubJub (en circomlib) está definido sobre el **campo escalar de BN254**.
- BabyJubJub **no es válido** bajo BLS12-381 (curva diferente).
- Merkle tree es **curva-agnóstico**: funciona igual en BN254, BLS12-381, o cualquier curva.

**Trade-off:**
- Merkle proof tiene más datos (pathElements + pathIndices) que una firma simple.
- Merkle path in public expone la **profundidad del árbol** (cuántas credenciales puede tener
  el issuer). Privacidad aceptable.

**Futuro:**
- Si volviéramos a BN254, podríamos usar EdDSA (proof más chico).
- Merkle permite **revocación**: actualizar la raíz invalida credenciales viejas (feature
  para futuro).

---

## 3. `currentYear` como constante (no input público)

**Decisión:** `currentYear` es una constante compilada en el circuito.

**Razón:**
- MVP simple: no requiere oráculos ni inputs del ledger.
- Los tests pueden correr con un año fijo.

**Trade-off:**
- El usuario **podría mentir la edad** si controlara `currentYear`.
- El contrato no valida la edad (solo ve `is_adult` booleano de la prueba).
- Riesgo: si `currentYear` es stale, la edad se calcula mal.

**Futuro:**
- Convertir `currentYear` a **input público** que valide contra el ledger (block timestamp).
- El contrato verificaría: `timestamp_ledger ≤ currentYear_input ≤ timestamp_ledger + 1_year`.
- Así se fuerza que `currentYear` es reciente y real.

---

## 4. Face-api local (no RENAPER en testnet)

**Decisión:** en testnet, usar face-api.js (face matching local) en vez de RENAPER.

**Razón:**
- RENAPER requiere credenciales reales, setup on-site, costo.
- Face-api permite **iterar en testnet sin gobierno argentino**.
- Para testnet es suficiente: demostrar que la cara del DNI y la cara viva coinciden.

**Trade-off:**
- Face-api **no es tan exacto** como RENAPER (que tiene acceso a DNI real).
- Falsos positivos/negativos más altos.
- Threshold `MATCH_THRESHOLD` debe calibrarse con datos reales (default 0.6).

**Futuro:**
- **Production:** reemplazar `testnetProvider.ts` con `renaperProvider.ts`.
- La interfaz `IdentityProvider` permite el swap sin tocar el circuito ni el contrato.

---

## 5. De-dup con `sha256(docId+PEPPER)` (no docId crudo)

**Decisión:** guardar hash criptográfico del docId, no el docId.

**Razón:**
- Si alguien tiene acceso al storage, no puede listar números de documentos reales.
- Previene correlación simple: no se puede hacer `docId_hash → docId` sin conocer PEPPER.
- PEPPER es un secreto compartido (rotable por entorno).

**Trade-off:**
- El hash es **irreversible** — si un usuario dice "mágame sé si ya me enrolé", hay que
  usar el mismo PEPPER (que el sistema debe guardar).
- Revocación es más complicada (no puedes hacer "buscar quién enroló con doc X").

**Futuro:**
- PEPPER debe ser rotado periódicamente (escenario: breach → cambiar PEPPER → viejos hashes
  inútiles).
- Considerar PEPPERING diferenciado por provider (testnet / renaper) para aislamiento.

---

## 6. localStorage para credencial (no backend)

**Decisión:** guardar `Capa1Credential` en localStorage del navegador, no en servidor.

**Razón:**
- **No-custodial:** el usuario tiene su credencial, no el servidor.
- No hay database backend que administrar (data locality).
- El `secret` nunca sale del device.

**Trade-off:**
- Credencial **no sincronizada** entre devices.
- Si borro localStorage, pierdo la credencial (pero el Merkle path sigue en el issuer, así
  que puedo re-enrolar).
- Menos segura si el device está comprometido (XSS podría leer localStorage).

**Futuro:**
- **Option 1:** backend encriptado (cifrar credencial con clave del usuario, guardar server-side).
- **Option 2:** sincronización P2P (IPFS, blockchain).
- **Option 3:** recovery con email/teléfono (rescan cara, re-validar).

---

## 7. Stella r Wallets Kit (no custom signing)

**Decisión:** usar [Stellar Wallets Kit](https://stellarwalletskit.dev/) para signing en el
navegador.

**Razón:**
- Estándar de la comunidad Stellar.
- Soporta todas las wallets (Lobstr, Freighter, Walletless, etc.).
- Handle de sesiones y auth de forma robusta.

**Trade-off:**
- Adiciona dependencia externa (Stellar Wallets Kit).
- Requiere que el usuario tenga una wallet instalada.

**Futuro:**
- Walletless: alternativa para usuarios sin wallet (usar passkeys / email).
- Delegation: permitir que un backend firme en nombre del usuario (para relayers).

---

## 8. Merkle path in public (en proof)

**Decisión:** el Merkle path (pathElements, pathIndices) es público en la prueba.

**Razón:**
- Necesario para verificar que el commitment pertenece al árbol.
- No revela el `secret` (privado).
- No revela atributos (privados).

**Trade-off:**
- Expone la **profundidad del árbol** — si el path tiene 12 elementos, hay ~4000 hojas
  máximo.
- Alguien observando la cadena puede estimar tamaño del árbol de usuarios.
- Privacidad aceptable: no es PII.

**Futuro:**
- Merkle trees "balanceados dinámicos" (agregar noises para ocultar profundidad).
- Ou opción: usar accumulators (criptografía más pesada, pero perfil privacidad mejor).

---

## Matriz de decisiones

| Decisión | MVP | Producción | Crítica | Auditar |
|---|---|---|---|---|
| BLS12-381 | ✅ | ⚠️ (parámetros Poseidon) | 🔴 SÍ | 🔴 SÍ |
| Merkle | ✅ | ✅ | 🟡 | 🟢 NO |
| currentYear const | ✅ | ❌ (validar ledger) | 🔴 SÍ | 🔴 SÍ |
| Face-api | ✅ | ❌ (RENAPER) | 🟡 | 🟢 NO |
| De-dup hash | ✅ | ✅ | 🟡 | 🟢 NO |
| localStorage | ✅ | 🟡 (+encrypted backend opt) | 🟡 | 🟢 NO |
| Wallets Kit | ✅ | ✅ | 🟢 NO | 🟢 NO |
| Merkle path public | ✅ | ✅ | 🟡 | 🟢 NO |

**Crítica:** decisión que afecta seguridad criptográfica.
**Auditar:** decisión que requiere revisión por expertos.

---

Relacionado: [[Diseño del Circuito ZK]], [[Modelo de Datos]], [[Registro de cambios y mejoras]],
[[Implementación en rama kyc-zk]].
