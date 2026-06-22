# Vault ZK · KYC sobre Stellar

Esta es una **vault de Obsidian** que funciona como el cerebro del proyecto: aquí vive
todo lo que necesitamos para diseñar, entender y construir un sistema de **KYC con
Zero-Knowledge sobre la red Stellar** para la hackathon *Stellar Hacks: Real-World ZK*.

> ⚠️ Este repositorio es **documentación**, no el código de producción. Aquí guardamos
> diagramas, explicaciones, decisiones de arquitectura e investigación. El código del
> contrato Soroban y los circuitos ZK pueden vivir en otro repo (ver
> [[Estructura del Codigo]]).

## Cómo usar esta vault

1. Instala [Obsidian](https://obsidian.md).
2. *Open folder as vault* → selecciona la carpeta de este repositorio.
3. Abre [[🏠 Home]] (en `00 - Inicio`) como punto de partida.
4. Los diagramas usan **Mermaid**, que Obsidian renderiza de forma nativa.

## Estructura de carpetas

| Carpeta | Contenido |
|---|---|
| `00 - Inicio` | Índice / Mapa de Contenidos (MOC) |
| `01 - Hackathon` | Reglas, fechas, recursos oficiales |
| `02 - Concepto` | Visión, problema/solución, casos de uso, glosario |
| `03 - Stellar` | Stellar, Soroban y las primitivas ZK on-chain |
| `04 - Zero Knowledge` | Fundamentos ZK y comparativa de herramientas (Noir / Circom / RISC Zero) |
| `05 - Arquitectura` | Diseño del sistema, flujos, circuito y modelo de datos |
| `06 - Implementacion` | Setup, estructura de código, roadmap y plan de demo |
| `07 - Investigacion` | Notas sueltas, referencias y enlaces externos |
| `templates` | Plantillas de notas |

## Workflow de git

Esta vault es local y se commitea cuando haga falta. Sugerencia de flujo:

```bash
git add .
git commit -m "docs: <qué cambiaste>"
git push
```

El archivo `.gitignore` evita subir la caché de workspace de Obsidian pero **sí**
conserva la configuración compartible del plugin.
