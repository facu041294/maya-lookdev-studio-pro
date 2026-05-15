# 🏷️ Guía de Nomenclatura de Archivos y Carpetas
**Proyecto:** Neo-Noir Interior Pipeline (TAR-021)
**Nivel de Restricción:** Estricto (Obligatorio para Integración)

Esta documentación define la convención estándar (Naming Convention) para garantizar la interoperabilidad, evitar sobreescritura accidental y asegurar una ingesta fluida en el motor de juego.

## 1. Estructura de Directorios (Work vs. Publish)
Separación obligatoria entre archivos en desarrollo y archivos finales listos para integración.

```text
PROYECTO_NeoNoir/
├── 01_Concept/
│   └── Moodboards_Refs/
├── 02_Work/
│   ├── Maya/           # Archivos fuente (.ma) versionados
│   └── Substance/      # Proyectos (.spp) versionados
├── 03_Publish/
│   ├── Meshes/         # Geometría final (.fbx) validada
│   └── Textures/       # Mapas PBR finales a 4K (.png / .exr)
└── 04_Renders/
    └── LookDev_Tests/  # Salidas de Arnold (generadas con LookDev Studio Pro)
```
---

## 2. Nomenclatura de Archivos Fuente (Iterativos)
Regla de oro: Nunca sobreescribir. Utilizar versionado iterativo de tres dígitos.

Estructura Base: [Tipo]_[NombreAsset]_[Etapa]_v[###]

Ejemplos:

MOD_DetectiveDesk_HighPoly_v001.ma

MOD_DetectiveDesk_LowPoly_v004.ma

TEX_DetectiveDesk_PBR_v002.spp

---

## 3. Jerarquía del Outliner (Maya / DCC)
El Outliner es la estructura ósea del proyecto. Nodos genéricos (pCube1, lambert2) causarán el fallo automático del QA Check.

Mallas Estáticas (Static Meshes): Prefijo SM_

Ejemplo: SM_Desk_Drawers, SM_Desk_Leg_L

Grupos Estructurales: Sufijo _GRP

Ejemplo: DetectiveDesk_Main_GRP

Geometría de Soporte (LookDev): Sufijo _GEO

Ejemplo: Cyclorama_GEO (Generado por script)

Cámaras Cinematográficas: Prefijo CAM_

Ejemplo: CAM_IntroShot_Main

Luces: Sufijo del Tipo de Luz (_LGT o descriptivo temporal)

Ejemplo: KeyLight, FillLight, RimLight, Arnold_DomeLight

## 4. Mapas de Textura PBR (Publish)
Los mapas exportados desde Substance Painter deben ser auto-descriptivos para facilitar la reconexión de materiales (o el uso futuro de scripts de batching).

Estructura Base: T_[NombreAsset]_[UDIM_si_aplica]_[TipoDeMapa]_[Resolucion]

Tipos de Mapa Soportados (Engine Ready):

_BaseColor (sRGB)

_Normal (DirectX / Z-Up)

_Roughness (Linear)

_Metalness (Linear)

_Emissive (Linear)

Ejemplo Completo de Exportación (Publish):

T_DetectiveDesk_BaseColor_4K.png

T_DetectiveDesk_Normal_4K.png