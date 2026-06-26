---
tags:
  - investigacion
---

# Notas y Referencias

Cajón de notas sueltas, dudas y enlaces para investigar. Mover lo que madure a su nota
definitiva.

## Preguntas abiertas

- [ ] ¿El `groth16_verifier` oficial soporta la cantidad de public inputs que necesitamos?
- [ ] ¿Cómo se valida `current_year` contra el ledger para que no se falsee la edad?
      → ver nota en [[Diseño del Circuito ZK#Riesgos / consideraciones]]
- [ ] ¿Qué esquema de firma del issuer es más barato en circuito: EdDSA Poseidon vs Merkle?
- [ ] ¿Coste real (CPU/fees) de verificar Groth16 vs UltraHonk en testnet? Medir.
- [ ] ¿Conviene emitir un *soulbound token* en vez de un flag en storage? (future work)

## Enlaces oficiales (copiados de [[Recursos Oficiales]])

- Circom + Stellar: https://jamesbachini.com/circom-on-stellar/
- Noir + Stellar: https://jamesbachini.com/noir-on-stellar/
- RISC Zero + Stellar: https://jamesbachini.com/stellar-risc-zero-games/
- Groth16 verifier: https://github.com/stellar/soroban-examples/tree/main/groth16_verifier
- Soroban docs: https://developers.stellar.org/docs/build/smart-contracts

## Ideas para explorar más adelante (post-MVP)

- Revocación de credenciales (Merkle root actualizable, o accumulators).
- Federación de issuers + gobernanza on-chain de la lista de confianza.
- Recuperación de credencial perdida (social recovery / re-emisión).
- Selective disclosure de más atributos (renta, acreditación de inversor).
- Integración real con un proveedor de KYC en vez del issuer mock.

## Inbox (volcado rápido)

> Anota aquí cualquier cosa al vuelo y clasifícala después.

-
