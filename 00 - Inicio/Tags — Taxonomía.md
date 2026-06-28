---
tags:
  - moc
---

# Tags — Taxonomía

Esquema de etiquetas de la vault, para navegar por **tema, capa y estado** desde el panel de
tags (`#`) y el grafo, más allá de las carpetas. Cada nota tiene tags en su frontmatter.

> 💡 En Obsidian: abrí el panel **Tags** o buscá `tag:#capa/2-plataforma` para filtrar.

## Por tipo (≈ carpeta)

| Tag | Qué agrupa |
|---|---|
| `#moc` | Índices / mapas de contenido (Home, IDEA, esta nota) |
| `#hackathon` | Reglas, fechas, recursos del hackathon |
| `#concepto` | Visión, problema/solución, glosario |
| `#stellar` | Stellar, Soroban, primitivas |
| `#zk` | Zero-Knowledge (fundamentos, herramientas, circuitos) |
| `#arquitectura` | Diseño del sistema, flujos, modelo de datos, decisiones |
| `#implementacion` | Lo construido, setup, estructura de código, estado |
| `#investigacion` | Notas sueltas y referencias |
| `#tools` | Toolbox: recursos, skills de IA, verificadores |
| `#funding` | Capa 3 — Funding ZK (DeFindex + Trustless Work) |
| `#referencia` · `#meta` | Material externo / archivos de soporte |

## Por capa del producto

| Tag | Capa |
|---|---|
| `#capa/1-identidad` | KYC con ZK (proof of personhood) |
| `#capa/2-plataforma` | Plataforma de opinión anónima |
| `#capa/3-funding` | Crowdfunding anónimo y condicional |

## Por tema transversal

| Tag | Qué marca |
|---|---|
| `#anonimato` | Notas centradas en privacidad / identidad anónima |
| `#zk` | Criptografía ZK (pruebas, circuitos, Groth16) |
| `#soroban` | Contratos Soroban (Rust) |
| `#circom` | Circuitos Circom |
| `#decisiones` | Decisiones técnicas y trade-offs |

## Por estado

| Tag | Qué marca |
|---|---|
| `#estado/implementado` | Notas que documentan algo ya construido |
| `#estado/planificado` | Diseño/plan decidido pero aún no implementado |

## Convención

- Tags en **frontmatter** (`tags:`), no inline, para mantener el cuerpo limpio.
- Tags **anidados** con `/` (`capa/2-plataforma`, `estado/implementado`) → Obsidian los agrupa.
- Para re-aplicar/actualizar el esquema masivamente: script `/tmp/add_tags.mjs` (idempotente).

Relacionado: [[🏠 Home]]
