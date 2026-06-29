---
tags:
  - meta
  - capa/1-identidad
  - zk
---

# 06 — El Puente: de KYC verificado → ZK on-chain

Esta es la pieza que **conecta** la investigación de KYC con el diagrama ZK que ya tenemos. Es donde "verifiqué una persona real" se transforma en "puedo probarlo sin revelar quién".

## El problema del puente

El KYC produce un **OK + datos reales de una persona** (PII). Pero on-chain **no podemos** poner nada de eso. Necesitamos transformar ese OK en algo:
- **anónimo** (no revela identidad),
- **único** (1 humano = 1 registro, anti-Sybil),
- **verificable** (un contrato puede confirmarlo).

La respuesta es el patrón **commitment + Merkle tree + nullifier** (estilo **Semaphore**), con un **issuer** que firma.

## Las 3 piezas criptográficas (modelo Semaphore)

Una identidad se compone de:
- **secret / trapdoor** — secreto que controla **el usuario** (no nosotros).
- **nullifier** — derivado del secreto; sirve para marcar "ya me registré" sin revelar quién.
- **commitment** — `Poseidon(secret, nullifier)`; es la "clave pública" de la identidad. Es lo único que se publica.

> El usuario genera estos valores en su dispositivo. Nosotros (el issuer) **nunca vemos el secreto**, solo el commitment.

## Flujo del puente (paso a paso)

```
1. KYC OK (V1 ∧ V2)   →  el issuer sabe "esta persona es real y única"
                          (tiene PII temporalmente, off-chain)

2. El usuario genera en su device:  secret, nullifier
                                    commitment = Poseidon(secret, nullifier)

3. El issuer:
   - verifica que esta persona REAL no tenga ya un commitment
     (de-dup por hash del documento/RENAPER, off-chain, anti-Sybil de origen)
   - agrega el commitment como hoja en el MERKLE TREE de humanos verificados
   - publica/actualiza el issuer_root (raíz del árbol)
   - firma una credencial (opcional) atando commitment ↔ "verificado por human"

4. El issuer DESCARTA el PII (según política de retención, ver 09)

5. Más tarde, para registrarse on-chain, el usuario corre el CIRCUITO (Circom):
   - inputs privados: secret, nullifier, Merkle path
   - inputs públicos: issuer_root, address_hash, nullifier_hash, predicados
   - output: proof Groth16
   prueba: "conozco un secreto cuyo commitment está en el árbol del issuer_root,
            y mi nullifier es este" — SIN revelar cuál commitment.

6. El contrato Soroban (register()) verifica la proof y:
   - chequea issuer_root confiable
   - chequea address_hash == Poseidon(caller)   (binding anti-frontrunning)
   - chequea nullifier NO usado                  (anti-doble-registro)
   - pairing check Groth16                        (caro, al final)
   - marca Verified(caller) + Nullifier(n)=usado
```

(Los pasos 5–6 ya están detallados en el **diagrama del proyecto**, secciones A y C.)

## Dónde está el anti-Sybil (en dos capas)

1. **En origen (off-chain, KYC):** una persona real = **un solo commitment**. Se de-duplica por un hash determinístico de su identidad oficial (ej. `hash(DNI normalizado)` o el identificador que devuelva RENAPER). Esto evita que la misma persona genere 10 commitments distintos.
2. **On-chain (nullifier):** aunque tuviera el secreto, **un nullifier = un registro**. El contrato rechaza nullifiers repetidos.

> La capa 1 es la que liga el **mundo real** (RENAPER) con la **unicidad criptográfica**. Es exactamente el "One Face, One DID" de zkMe (`02`).

## Decisiones de diseño abiertas (para el equipo)

- **De-dup en origen sin guardar PII:** guardar **solo** `hash(identificador_oficial)` (no el DNI en claro) para detectar duplicados. ¿Con sal global? ¿pepper? (definir con `09`).
- **¿Issuer centralizado o federado?** MVP: un issuer (human). Futuro: varios issuers confiables (multi-root), como prevé el contrato (`issuer_root` ∈ set confiable).
- **Custodia del secreto:** ¿lo guarda el wallet del usuario? ¿passkey? Hay que definir UX de recuperación sin romper el no-custodial.
- **Predicados (Tier 2):** `is_adult`, `country_ok` se pueden incluir como señales públicas derivadas del KYC (la fecha de nac. del DNI prueba mayoría de edad **sin revelar** la fecha).

## Interfaz del issuer (borrador)

```ts
// Tras KYC OK, el usuario aporta su commitment
async function enrollVerifiedHuman(input: {
  commitment: bigint;            // generado por el usuario
  officialIdHash: string;        // hash(identificador RENAPER) para de-dup
  predicates: { isAdult: boolean; country: string };
}): Promise<{ added: boolean; issuerRoot: bigint; credential?: SignedCredential }>;
```

## Siguiente
→ [[07-Arquitectura-y-Flujo/Arquitectura-y-Flujo]]

## Fuentes
- [Semaphore v4 — group membership ZK](https://nkapolcs.dev/thoughts/20240728_zero_knowledge_with_semaphore_v4/)
- [Semaphore protocol (ChainScore)](https://chainscorelabs.com/en/glossary/privacy-enhancing-technologies-pets/private-smart-contract-execution/semaphore-protocol)
- [zkMe — How zkKYC works](https://blog.zk.me/how-zkkyc-works-understanding-the-mechanisms-behind-privacy-preserving-verification/)
