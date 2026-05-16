# 🎬 LookDev Studio Pro

![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)
![Target](https://img.shields.io/badge/target-Maya_2020%2B_%7C_Arnold-orange.svg)
![License](https://img.shields.io/badge/license-MIT-lightgrey.svg)
![Status](https://img.shields.io/badge/status-Active_Development-success.svg)

**LookDev Studio Pro** es una herramienta de Technical Art procedimental construida en **MEL (Maya Embedded Language)**. Diseñada como el núcleo de estandarización visual y control de calidad (QA) para pipelines de producción 3D y Virtual Production.

Reduce un proceso de setup manual de 15-20 minutos a **un solo clic (< 5 segundos)**, permitiendo a los artistas centrarse en la topología y el texturizado, mientras el script garantiza un entorno de revisión físicamente correcto y matemáticamente seguro para su exportación a motores de juego (UE5 / Unity).

---

## ✨ Características Principales (Core Features)

* **Escalamiento Procedimental Inteligente:** Calcula el Bounding Box exacto del mesh y genera un ciclorama (Infinity Background) proporcional a la escala del asset.
* **Iluminación Física (Arnold):** Instancia y conecta al *Node Graph* un esquema de luces cinemáticas (Key, Fill, Rim) mediante *Area Lights* escaladas dinámicamente, respaldadas por un SkyDome.
* **Tri-Cam Setup Automático:** Genera y encuadra tres cámaras de producción (Main, Side, High) apuntadas al centro de masa del objeto, listas para *Contact Sheets*.
* **Z-Up Pipeline Ready:** Construcción nativa en coordenadas Z-Up para mantener la consistencia milimétrica con exportaciones a Unreal Engine.

---

## 🛡️ Pipeline Integration & Pre-Mortem Defenses
Esta herramienta opera como un **Gatekeeper (Guardián)** pre-integración, diseñada con un enfoque defensivo contra los fallos más comunes de un pipeline:

* **Jerarquía Inmutable:** Previene el desorden del Outliner creando el grupo central `PhotoStudio_SETUP_GRP`. Aísla el entorno del asset de producción, permitiendo exportaciones limpias sin "basura" residual.
* **Idempotencia (Safe Mode):** Limpia automáticamente ejecuciones anteriores y nodos huérfanos. Garantiza un entorno de trabajo predecible sin choques de nombres (*Name Clashing*).
* **Arnold Light Linking Force:** Evita los "Renders Negros" forzando la conexión de las luces a bajo nivel directamente al `defaultLightSet.dagSetMembers`.

---

## 🛠️ Bitácora de Desarrollo (Tech Notes)
El desarrollo de este script implicó resolver inconsistencias históricas en la arquitectura interna de Maya:

1. **Topología en Z-Up y Extrusiones (Prevención del "Techo Negro"):**
   Al intentar levantar la pared del ciclorama en Z-Up, `polyExtrudeEdge` en espacio local generaba geometría horizontal. Se forzó la transformación en espacio global (`-ws -wd`) y se identificó algorítmicamente la arista instanciada correcta (`e[3]`).
2. **Tipos de Retorno Inconsistentes en MEL:**
   Se reemplazó el uso obsoleto de `shadingNode` por la creación explícita vía `createNode transform` y `createNode areaLight`, asegurando el control absoluto de las variables de tipo *Shape* y sus jerarquías.
3. **El Comportamiento de `viewFit`:**
   Se aisló la selección en un *array* temporal de mallas válidas (`$validMeshes`), excluyendo expresamente al ciclorama generado para evitar que Maya invirtiera la cámara al intentar encuadrar un fondo infinito.
4. **Optimización de AimConstraints:**
   Se unificaron todas las entidades de render en un *Array* procesado mediante un bucle `for`, apuntando hacia un único *locator* temporal que luego es purgado de la memoria.

---

## 🚀 Instalación y Uso

**Sin dependencias.** MEL puro. No requiere Python ni plugins externos (solo el plugin nativo `mtoa` cargado).

1. Abre el **Script Editor** en Autodesk Maya.
2. Pega el código de `/src/LookDevStudioPro.mel`.
3. Selecciona todo el texto y arrástralo a tu *Shelf* para crear un botón.
4. **Ejecución:** Selecciona tu asset (agrupado con `Ctrl+G`), presiona el botón y el entorno se generará al instante.

<video width="100%" controls>
  <source src="./examples/video/Previewlookdevstudioprov1.0.mp4" type="video/mp4">
  Tu navegador no soporta la etiqueta de video.
</video>

https://github.com/user-attachments/assets/0ce183ec-5260-474c-bce9-671c765813e2

---

## 📁 Documentación Anexa
Para entender el flujo completo de validación de este script dentro de la producción, consulta nuestra documentación de pipeline:
* [Guía de Nomenclatura y Estructura Work/Publish (TAR-021)](./docs/NamingConvention_Guide.md)

---

## 🗺️ Roadmap y Evolución de la Herramienta (Releases)

Este repositorio se encuentra en desarrollo activo. La arquitectura modular actual en MEL sienta las bases para escalar la herramienta hacia una suite de pipeline completa.

### 📍 Fase 1: Expansión de UI y Render (Ciclo v1.x.x)
El objetivo a corto plazo es mejorar la flexibilidad del *setup* y la experiencia de usuario (UX) para el artista, inspirándonos en entornos de LookDev de nueva generación.

- [ ] **v1.1.0 (Camera & Render Update):** - Generación de un segundo grupo de cámaras para **vistas ortográficas** (Top, Front, Side) aisladas del grupo de perspectiva.
  - Implementación de inyección de código MEL en los *Render Settings* de Arnold para ofrecer **3 presets de renderizado de 1-clic** (Draft 540p, Review 720p, Final 1080p sin ruido).
- [ ] **v1.2.0 (The MetaHuman UI Update):** - Rediseño de la interfaz para incluir un panel visual de presets de estudio.
  - Selectores de temperatura de color (Kelvin) para el *Light Rig*.
  - Selector interactivo de color difuso para el ciclorama y menús desplegables para HDRIs personalizados integrados al SkyDome.

### 📍 Fase 2: Refactorización y Automatización (Ciclo v2.x.x)
Evolución del núcleo para operaciones masivas y sin interfaz gráfica (*headless*). Aumento de versión MAYOR debido a la ruptura de compatibilidad con el código MEL heredado.

- [ ] **v2.0.0 (Python Core Migration):** Traducción del motor lógico de MEL a **Python (PyMEL / maya.cmds)**. Esto permitirá ejecutar el script en modo *batch* mediante consola, generando y renderizando estudios de LookDev para cientos de assets en segundo plano.

### 📍 Fase 3: Ecosistema de Pipeline End-to-End (Ciclo v3.x.x)
LookDev Studio Pro absorberá módulos satélite para convertirse en un validador completo desde el modelado hasta el motor. Nueva arquitectura orientada a la ingesta externa.

- [ ] **v3.0.0 (Smart UV & PBR Ingest):** - Integración de algoritmos de corrección de UVs (orientación vertical forzada en eje Z y detección de cortes lógicos).
  - Módulo de Exportación Automática (USD/FBX) sanitizada.
  - **Material Auto-Linker:** Script de escucha que recibe mapas PBR (Albedo, Normal, ORM) exportados desde Substance Painter y los conecta automáticamente al *aiStandardSurface* dentro del entorno de LookDev generado.

---
*Desarrollado por Facundo Villarreal — Lead Cinematic Technical Artist.*