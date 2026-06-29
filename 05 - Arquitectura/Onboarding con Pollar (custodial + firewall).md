---
tags:
  - arquitectura
  - capa/1-identidad
  - onboarding
  - estado/implementado
---

# Onboarding con Pollar (custodial + firewall)

> **Mergeado a `main`** (rama `pollar-onboarding`). Pollar (`@pollar/react`, embedded wallet de
> Stellar) se integra **solo** como forma amigable de que un usuario sin wallet cree una cuenta
> con **email/Google**. **No firma nada** de human ni participa del anonimato.
> Docs del repo: `docs/pollar-onboarding.md` + `docs/pollar-audit.md`.

## El problema que resuelve

Un newbie web3 no tiene wallet. [[Identidad Pública vs Anónima|Conectar Freighter]] es fricción.
Pollar permite **crear una wallet con email** en segundos. Pero el login email de Pollar es
**custodial** (genera/guarda la clave en su server, firma server-side y conoce `email → wallet`).

## La regla de oro: el anonimato NO depende de Pollar

```
[Email] --Pollar (custodial)--> [Wallet pública G...]   ← identidad de ENTRADA (opcional)
                                    ⟂  firewall (sin link)
[secret client-side] --Poseidon(secret, scope)--> [platformId]   ← identidad ANÓNIMA
```

- El **`secret`** (raíz del anonimato) se genera y vive **solo en el dispositivo** — igual que
  en [[Implementación en rama kyc-zk]]. Nunca se envía a Pollar ni a human.
- El **`platformId`** se deriva en el navegador (ver [[Identidad anónima de plataforma (platformId)]]).
- Las acciones anónimas (posts, opiniones, donaciones) usan **wallets efímeras** (friendbot),
  **nunca** la wallet de Pollar → sin rastro Pollar → efímera → opinión.
- El **email** vive solo en Pollar; human no lo guarda ni lo mapea a `platformId`.

## Qué firma Pollar en human: **nada**

Decisión de producto: Pollar es **solo creación de wallet**. El registro on-chain de Capa 1
(`verify_and_register`, invoke Soroban que requiere firma del titular) **no** se hace por Pollar;
la participación anónima no lo necesita (se gatea por la credencial ZK + prueba de pertenencia,
no por `is_verified(address)`). Para un usuario Pollar, el KYC corre en modo "credencial":
matcher (DNI + cara) → credencial ZK client-side → listo.

## Invariantes (todos se cumplen)

1. `secret`/`platformId` solo client-side.
2. La wallet de Pollar es entrada pública; las anónimas usan efímeras.
3. Efímeras fondeadas por friendbot, nunca desde Pollar.
4. El email nunca toca human; sin mapeo email↔platformId.
5. Se SUMA a Stellar Wallets Kit (Freighter sigue para usuarios cripto).

## Implementación (web)

- `identity/pollar.tsx` — `PollarRoot` (provider solo si hay `VITE_POLLAR_PUBLISHABLE_KEY`,
  testnet por prefijo) + `PollarEmailLogin` (modal email/OTP de Pollar).
- `AuthPage` — botón "Crear cuenta con email" → `/onboarding?via=email`.
- `KycFlow` — `mode="credential"`: crea la credencial ZK sin wallet ni on-chain.
- `OnboardingPage` — `?via=email` → modo credencial + aviso honesto.

## UX honesta

No se afirma "no se guarda nada en ningún lado" (Pollar guarda su parte). Copy:
*"Tu email crea tu wallet, pero nunca se vincula a tu identidad anónima."*

## Auditoría (`docs/pollar-audit.md`) — ✅ APROBADA

Revisión de funcionamiento + invariantes ZK. Veredicto: **los 5 invariantes se cumplen**.

- **Superficie contenida:** Pollar usa **solo** `openLoginModal` + `isAuthenticated`. **Nunca**
  se usa `walletAddress`, `sendPayment`, `signAndSubmitTx` ni `signTx` → Pollar no firma ni
  fondea nada. El email no se envía a `/content`, `/articles`, `/campaigns` ni `/profile`.
- **Funciona en testnet:** login OK con la `pub_testnet_…`. El 403/CORS previo era por usar una
  key `pat_` en vez de la publishable — resuelto. `typecheck` + `vite build` verdes.
- **2 observaciones, ya corregidas:**
  - **O1 ✅** — navegar al onboarding apenas hay sesión (`isAuthenticated || verified`), sin
    esperar el provisioning de la wallet (no se necesita para el flujo anónimo).
  - **O2 ✅** — quitado el override de `appConfig`: el modal usa la config real del dashboard.

> ⚠️ **N1 — Acción de seguridad pendiente:** durante el setup se compartió una secret key
> `sec_testnet_…` por chat. No está en el repo ni se usa en el cliente, pero **debe rotarse**.
> En el navegador solo va la **publishable** `pub_testnet_…` (segura). N3: queda la correlación
> por timing (Pollar conoce `email→hora`, human `platformId→hora`) — sin id compartido, pero
> es la limitación general ya documentada.

Relacionado: [[Identidad Pública vs Anónima]], [[Identidad anónima de plataforma (platformId)]],
[[Implementación en rama kyc-zk]], [[Flujo de KYC]], [[Estado actual del desarrollo]].
