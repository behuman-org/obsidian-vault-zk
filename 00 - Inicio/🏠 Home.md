# 🏠 Home · KYC-ZK sobre Stellar

Mapa de contenidos (MOC) de la vault. Punto de entrada para todo el proyecto.

> **En una frase:** un sistema donde una persona prueba que pasó un proceso de KYC
> (identidad verificada) **sin revelar sus datos personales**, usando una prueba de
> Zero-Knowledge que se **verifica en un contrato inteligente de Stellar (Soroban)**.

---

## 🎯 Empieza aquí

- [[IDEA]] — **la visión del proyecto** (léela primero)
- [[Prueba de Persona Única]] — el núcleo técnico que nos diferencia
- [[Plataforma de Opinión Verificada]] — la aplicación con impacto social
- [[Arquitectura General]] — cómo encaja todo
- [[Flujo de KYC]] — el recorrido del usuario paso a paso

## 🗂️ Secciones

### 01 · Hackathon
- [[Reglas y Requisitos]]
- [[Fechas Clave]]
- [[Recursos Oficiales]]

### 02 · Concepto
- [[Prueba de Persona Única]]
- [[Plataforma de Opinión Verificada]]
- [[Curaduría y Agentes Validadores]]
- [[Identidad Pública vs Anónima]]
- [[Problema y Solucion]]
- [[Glosario]]
- [[Vision General]] · [[Casos de Uso]] *(genéricas — pendiente realinear a [[IDEA]])*

### 03 · Stellar
- [[Stellar y Soroban]]
- [[Primitivas ZK en Stellar]]
- [[Contrato Verificador (Soroban)]]

### 04 · Zero Knowledge
- [[Fundamentos ZK]]
- [[Comparativa de Herramientas ZK]]
- [[Noir]]
- [[Circom]]
- [[RISC Zero]]

### 05 · Arquitectura
- [[Arquitectura General]]
- [[Flujo de KYC]]
- [[Diseño del Circuito ZK]]
- [[Modelo de Datos]]
- [[Decisiones técnicas y trade-offs]] — por qué cada decisión y sus implicancias
- [[Matcher de Identidad (Gate de Capa 1)]] — validación DNI + cara en vivo

### 06 · Implementación
- [[Setup del Entorno]]
- [[Estructura del Codigo]]
- [[Roadmap]]
- [[Plan de Demo]]
- [[Implementación en rama kyc-zk]] — registro completo del viaje
- [[Registro de cambios y mejoras]] — decisiones y mejoras implementadas
- [[Estado actual del desarrollo]] — qué está hecho, en progreso, y próximos pasos

### 07 · Investigación
- [[Notas y Referencias]]
- [[zkME]]

### 09 · Tools 🧰
- [[🧰 Tools — Índice]] — toolbox para construir el KYC
- [[Recursos ZK & Privacy en Stellar]]
- [[Skills de IA para construir]]
- [[Verificadores ZK de referencia]]
- [[Stack de Privacidad en Stellar]]
- [[Plano del KYC inspirado en zkMe]]
- [[Plan de armado con IA]]

---

## 📌 Estado actual

- [x] Estructura de la vault creada
- [x] **Monorepo `beHuman` scaffoldeado** (por capas: `identity/` + `platform/`) →
  https://github.com/ACRC-Zk/beHuman · ver [[Estructura del Codigo]]
- [x] Skills de IA instaladas en el repo → [[Skills de IA para construir]]
- [x] **Rama `kyc-zk`: CAPA 1 (identidad) IMPLEMENTADA y testeada** ← ¡Acá estamos!
  - [x] Circuito Circom (BLS12-381 + Merkle) → [[Diseño del Circuito ZK]]
  - [x] Contrato `kyc_verifier` (init, verify_and_register, tests con testdata)
  - [x] Matcher (face-api + liveness + de-dup) → [[Matcher de Identidad (Gate de Capa 1)]]
  - [x] SDK (generateProof, verifyAndRegister, addressHash, encodings)
  - [x] E2E demo en testnet (`scripts/e2e_demo.sh`)
  - [ ] Integración en frontend: Stellar Wallets Kit + llamada on-chain desde navegador
- [ ] Implementar capa 2: `opinion_board` + backend + curaduría ([[Plataforma de Opinión Verificada]])
- [ ] Demo end-to-end + video (3 min)

## ❓ Decisiones abiertas

- **Nombre del proyecto:** `beHuman` *(nombre de trabajo, en definición)* → [[Vision General#Nombre del proyecto]]
- **Toolchain ZK:** Noir (favorito) vs Circom vs RISC Zero → [[Comparativa de Herramientas ZK]]
- **Modelo de confianza del issuer:** ¿uno o varios emisores de KYC? → [[Modelo de Datos]]
