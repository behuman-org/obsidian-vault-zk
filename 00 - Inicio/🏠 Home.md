# 🏠 Home · KYC-ZK sobre Stellar

Mapa de contenidos (MOC) de la vault. Punto de entrada para todo el proyecto.

> **En una frase:** un sistema donde una persona prueba que pasó un proceso de KYC
> (identidad verificada) **sin revelar sus datos personales**, usando una prueba de
> Zero-Knowledge que se **verifica en un contrato inteligente de Stellar (Soroban)**.

---

## 🎯 Empieza aquí

- [[Vision General]] — qué construimos y por qué
- [[Problema y Solucion]] — el dolor que resolvemos
- [[Arquitectura General]] — cómo encaja todo
- [[Flujo de KYC]] — el recorrido del usuario paso a paso

## 🗂️ Secciones

### 01 · Hackathon
- [[Reglas y Requisitos]]
- [[Fechas Clave]]
- [[Recursos Oficiales]]

### 02 · Concepto
- [[Vision General]]
- [[Problema y Solucion]]
- [[Casos de Uso]]
- [[Glosario]]

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

### 06 · Implementación
- [[Setup del Entorno]]
- [[Estructura del Codigo]]
- [[Roadmap]]
- [[Plan de Demo]]

### 07 · Investigación
- [[Notas y Referencias]]

---

## 📌 Estado actual

- [x] Estructura de la vault creada
- [ ] Decidir herramienta ZK definitiva (ver [[Comparativa de Herramientas ZK]])
- [ ] Diseñar el circuito de credencial KYC ([[Diseño del Circuito ZK]])
- [ ] Implementar contrato verificador en Soroban
- [ ] Demo end-to-end + video (3 min)

## ❓ Decisiones abiertas

- **Nombre del proyecto:** aún sin definir → ideas en [[Vision General#Ideas de nombre]]
- **Toolchain ZK:** Noir (favorito) vs Circom vs RISC Zero → [[Comparativa de Herramientas ZK]]
- **Modelo de confianza del issuer:** ¿uno o varios emisores de KYC? → [[Modelo de Datos]]
