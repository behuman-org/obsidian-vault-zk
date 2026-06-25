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
- [[Identidad anónima de plataforma (platformId)]] — el ZK de la Capa 2

### 06 · Implementación
- [[Setup del Entorno]]
- [[Estructura del Codigo]]
- [[Roadmap]]
- [[Plan de Demo]]
- [[Implementación en rama kyc-zk]] — Capa 1: registro completo del viaje
- [[Implementación Capa 2 (plataforma)]] — Capa 2: plataforma anónima por ZK
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
- [x] **CAPA 1 (identidad) IMPLEMENTADA y testeada** → [[Implementación en rama kyc-zk]]
  - [x] Circuito Circom (BLS12-381 + Merkle) → [[Diseño del Circuito ZK]]
  - [x] Contrato `kyc_verifier` + matcher (face-api + de-dup) + SDK + frontend on-chain
  - [x] E2E demo en testnet
- [x] **CAPA 2 (plataforma anónima) PRIMERA ITERACIÓN** → [[Implementación Capa 2 (plataforma)]]
  - [x] Identidad anónima `platformId` (ZK) → [[Identidad anónima de plataforma (platformId)]]
  - [x] Circuito de plataforma + contrato `opinion_board` (tests)
  - [x] Perfil/username + handle + feed (off-chain)
  - [x] Primer post (opinión comida argentina) anclado on-chain con cuenta efímera
  - [ ] Curaduría (`platform/curation`) → [[Curaduría y Agentes Validadores]]
- [ ] Demo end-to-end + video (3 min)

## ❓ Decisiones abiertas

- **Nombre del proyecto:** `beHuman` *(nombre de trabajo, en definición)* → [[Vision General#Nombre del proyecto]]
- **Toolchain ZK:** Noir (favorito) vs Circom vs RISC Zero → [[Comparativa de Herramientas ZK]]
- **Modelo de confianza del issuer:** ¿uno o varios emisores de KYC? → [[Modelo de Datos]]
