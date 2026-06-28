---
tags:
  - arquitectura
  - capa/1-identidad
  - anonimato
  - estado/planificado
---

# Onboarding con Pollar (embedded wallet)

**Estado:** EN DISEÑO (no implementado). Rama prevista: `pollar-onboarding`.

Integración de **[Pollar](https://www.pollar.xyz/)** como **onboarding fácil para usuarios
nuevos en web3**: que puedan crear su wallet **con email**, sin perder el anonimato ZK (que es
la prioridad). **Se suma** a Stellar Wallets Kit (Freighter…), no lo reemplaza.

> Decidido: Pollar es la ruta fácil para newbies; los usuarios cripto siguen con Freighter.

## Qué es Pollar

Embedded wallet SDK **nativo de Stellar** (`@pollar/core` + `@pollar/react`, v0.4.3). Crea
cuentas Stellar (G-address) con login por **email/Google/GitHub**, y resuelve la fricción del
newbie: activación, trustlines y **fees patrocinados**. Métodos clave:

```ts
const pollar = new PollarClient({ apiKey, stellarNetwork: "testnet" });
await pollar.login({ provider: "email", email });   // crea wallet
await pollar.buildTx("payment", { ... });
await pollar.signAndSubmitTx(xdr);                    // firma server-side (custodial)
```

## ⚠️ Hallazgo crítico: Pollar (email) es CUSTODIAL

Según sus docs (`docs.pollar.xyz/llms-full.txt`):
- La clave privada **se genera en el servidor de Pollar** (cifrada con AWS KMS).
- **La firma ocurre server-side.**
- **No hay** modo sin persistencia ni self-hosting; el email se usa para OTP.

→ Con email, **un tercero (Pollar) guarda email + clave**. Por eso el anonimato **no puede
depender de la wallet de Pollar**.

## Las dos identidades (la clave de por qué NO se pierde el anonimato)

beHuman separa a propósito dos identidades. El anonimato vive en la **anónima**, no en la wallet.

| | Identidad **pública** (KYC) | Identidad **anónima** (plataforma) |
|---|---|---|
| Qué es | wallet/address verificada | `platformId = Poseidon(secret, scope)` |
| ¿Anónima? | nunca lo fue (`is_verified` on-chain) | **sí, 100%** |
| Qué la protege | — | el `secret` (client-side) + wallets efímeras |

**Pollar solo sabe:** "email → wallet pública". **Pollar nunca sabe:** el `secret`, el
`platformId`, ni una sola opinión. → [[Identidad Pública vs Anónima]],
[[Identidad anónima de plataforma (platformId)]].

## Reglas innegociables (si una se rompe, hay deanonimización)

1. El **`secret`** se genera y queda **solo client-side**. Jamás a Pollar ni a un backend.
2. La wallet de Pollar es **solo la identidad pública** (onboarding/gas/KYC público). Las
   acciones anónimas (posts, opiniones, donaciones) **nunca** la usan → siguen por efímeras.
3. Las efímeras se fondean **independientemente** (friendbot/relayer), **nunca desde la wallet
   de Pollar**.

> 🔎 **Por qué la regla 3:** Pollar firma server-side y puede *registrar* sus tx. Si la wallet
> de Pollar fondeara una efímera que luego postea, quedaría el rastro
> `email → wallet Pollar → efímera → opinión`. Fondear la efímera aparte rompe ese grafo.

## ¿El KYC-ZK sigue funcionando? → Sí, idéntico

La integración es **puramente aditiva al frontend**. Sin cambios en circuito, contrato ni SDK ZK:
- Circuito `kyc.circom` (BLS12-381), contrato `kyc_verifier` (`verify_and_register`,
  `is_verified`): igual.
- La prueba se genera **client-side** desde el `secret`: igual.
- *Address binding* (anti-front-running): se ata a la G-address de Pollar, igual que con Freighter.
- Pollar firma la tx, pero esta solo lleva el **proof público** → el `secret` nunca está ahí.

## Costo honesto

El email "cuesta" que Pollar asocie tu email con tu wallet *pública* (que ya era pública
on-chain). **No te deanonimiza como opinante.** La UX debe ser honesta: *"tu email crea tu
wallet pero nunca se vincula a tu identidad anónima"* — **no** decir "no se guarda nada en
ningún lado" (Pollar sí guarda su parte).

## Futuro: anonimato total sin email

**Pollar Passkeys** (clave en el dispositivo / Secure Enclave, sin email en servidor) = cero
custodia de terceros. Hoy está *coming soon*; cuando salga, migrar el onboarding a esa vía.

---

Relacionado: [[Identidad Pública vs Anónima]], [[Identidad anónima de plataforma (platformId)]],
[[Implementación en rama kyc-zk]], [[Flujo de KYC]], [[Decisiones técnicas y trade-offs]],
[[Estado actual del desarrollo]].
