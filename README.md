# 🎬 LookDev Studio Pro (Pipeline Edition)

![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)
![Target](https://img.shields.io/badge/target-Maya_2024%2B_%7C_Arnold-orange.svg)
![License](https://img.shields.io/badge/license-MIT-lightgrey.svg)

**LookDev Studio Pro** es una herramienta de Technical Art construida en **MEL (Maya Embedded Language)**. Actúa como el núcleo de estandarización visual y control de calidad (QA) para pipelines de producción 3D y Virtual Production.

Desarrollada bajo la directiva del proyecto *Neo-Noir Interior*, esta herramienta resuelve dos problemas críticos en la producción: la **pérdida de tiempo en setups manuales** y la **integración de assets corruptos** al motor de juego (Unreal Engine / Unity).

## 🎯 El Problema vs. La Solución

* **El Problema:** Preparar una escena fotorealista para revisión manual consume entre 15 y 20 minutos por asset. Además, el error humano al nombrar archivos o agrupar jerarquías en el *Outliner* rompe frecuentemente las exportaciones *batch* y las referencias en el motor.
* **La Solución:** Con **1 solo clic (menos de 5 segundos)**, LookDev Studio Pro ejecuta un análisis de la geometría (Bounding Box), construye un estudio Arnold físicamente correcto y prepara el asset para la validación de nomenclatura y topología antes de su publicación (`Publish`).

## ✨ Características Principales (Pipeline Integration)

### 1. 1-Click LookDev (Automatización Procedimental)
* **Escalamiento Inteligente:** Calcula el Bounding Box exacto del mesh y genera un ciclorama (Infinity Background) proporcional.
* **Iluminación Física:** Instancia y conecta al *Node Graph* un esquema de luces cinemáticas (Key, Fill, Rim) mediante *Area Lights* de Arnold y un SkyDome de respaldo.
* **Tri-Cam Setup:** Genera y encuadra automáticamente tres cámaras de producción (Main, Side, High) listas para renderizar *Contact Sheets*.
* **Z-Up Pipeline Ready:** Toda la geometría se construye en el eje Z-Up, garantizando la consistencia matemática al exportar a Unreal Engine.

### 2. Pre-Integration Gatekeeper (QA & Naming)
Basado en los estándares de nomenclatura estricta del estudio, la herramienta está diseñada para operar junto al pipeline de validación:
* **Jerarquía Inmutable:** Previene el desorden del Outliner creando el grupo central `PhotoStudio_SETUP_GRP`. Aísla el entorno del asset de producción (ej. `SM_DeskChair_01`), permitiendo exportaciones limpias.
* **Idempotencia / Safe Mode:** Limpia ejecuciones anteriores, borra *locators* de apuntado temporal y previene el *Name Clashing* en escenas complejas.

## 🛡️ Defensas Pre-Mortem (Gestión de Riesgos)
Esta herramienta fue diseñada con un enfoque defensivo contra los fallos más comunes del pipeline:
* **Prevención de "Techo Negro" (Extrusión Z-Up):** Usa transformación en espacio global (`-ws`) durante la creación del ciclorama para evitar colapsos de normales al rotar ejes entre Maya y UE5.
* **Arnold Light Linking Force:** Evita los "Renders Negros" conectando las luces a bajo nivel directamente al `defaultLightSet.dagSetMembers`.

## 🚀 Instalación y Uso

**Sin dependencias.** MEL puro. No requiere Python ni plugins externos (solo `MtoA` cargado).

1. Abre Autodesk Maya.
2. Descarga el archivo `/src/LookDevStudioPro.mel`.
3. Arrastra el archivo directamente al *Viewport* de Maya, o copia su contenido en el **Script Editor** y guárdalo en tu *Shelf*.
4. **Ejecución:** Selecciona tu asset (agrupado con `Ctrl+G` para evitar cálculos erróneos de offset) y presiona el botón **Generar Estudio**.

## 📁 Documentación Anexa
Para entender el flujo completo de validación de este script dentro de la producción, consulta nuestra documentación de pipeline:
* [Guía de Nomenclatura y Estructura Work/Publish (TAR-021)](./docs/NamingConvention_Guide.md)

---
*Desarrollado por Facundo Villarreal — Lead Cinematic Technical Artist.*