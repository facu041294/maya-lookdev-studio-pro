# Auditoría Técnica — LookDevStudioPro.mel
**Rol:** Technical Artist Expert | **Fecha:** 2026-06-07  
**Archivo auditado:** `src/LookDevStudioPro.mel` (118 líneas)

---

## Bug #1 — `ls -type "mesh"` ignora la selección del usuario
**Línea 17** | Severidad: **Crítica** | Estado: `[x] Corregido`

```mel
string $allMeshes[] = `ls -type "mesh"`;
```

El README dice "selecciona tu asset". El código **ignora la selección** y toma *todas* las mallas de la escena. Si hay un plano de referencia, un proxy, o cualquier otra geometría en la escena, queda incluida en el bounding box → el studio se genera en el tamaño equivocado.

**Fix aplicado:**
```mel
string $sel[] = `ls -sl`;
if (size($sel) > 0) {
    $sourceMeshes = `ls -sl -dag -type "mesh"`;
} else {
    $sourceMeshes = `ls -type "mesh"`;
}
```

---

## Bug #2 — Las luces son Maya estándar, no Arnold
**Líneas 57, 65, 73** | Severidad: **Alta** | Estado: `[x] Corregido`

```mel
string $keyS = `createNode areaLight ...`;   // Maya nativa
```

`createNode areaLight` crea la luz de área de **Maya nativa**, no de Arnold. Arnold sí las renderiza, pero sin atributos Arnold (spread, normalize, light groups, shadow color, per-light AOVs). Para un LookDev tool de Arnold se requiere `aiAreaLight`.

**Fix aplicado:**
```mel
$keyS = `createNode aiAreaLight -n "Key_LGTShape" -p $keyT`;
```

Irónico que el SkyDome SÍ usa `aiSkyDomeLight` (línea 82) pero las luces principales no.

---

## Bug #3 — `e[3]` reutilizado después de cambio de topología
**Líneas 40 y 44** | Severidad: **Alta** | Estado: `[x] Corregido`

```mel
select -r ($plane[0] + ".e[3]");
polyExtrudeEdge ...;           // cambia la topología
move -r -ws -wd 0 0 ($maxDim * 8);

polyBevel ... ($plane[0] + ".e[3]");  // e[3] ya no es el mismo edge
```

Después del `polyExtrudeEdge`, Maya agrega nuevas aristas y puede reasignar índices. El `polyBevel` sobre `e[3]` en la línea 44 puede estar biselando la arista equivocada — potencialmente el borde inferior del plano en vez de la esquina piso-pared. Requiere capturar el resultado del extrude y operar sobre la arista nueva.

**Fix aplicado:** buscar el edge interior (la arista compartida entre piso y pared) usando `polyListComponentConversion`:
```mel
string $creaseEdges[] = `polyListComponentConversion -toEdge -internal ($plane[0] + ".f[0]") ($plane[0] + ".f[1]")`;
polyBevel -offset ($maxDim * 1.5) -segments 32 -autoFit 1 $creaseEdges;
```

---

## Bug #4 — `viewFit` recibe un shape node, no un panel
**Líneas 112-114** | Severidad: **Media** | Estado: `[x] Corregido`

```mel
viewFit -fitFactor 0.5 $cam1S;  // $cam1S = "Studio_Cam_MainShape"
```

`viewFit` requiere que la cámara esté activa en un panel para tener efecto. Sin especificar el panel activo con `lookThru`, este comando encuadra la cámara activa del viewport activo, **no las cámaras de studio generadas**. El encuadre de las 3 cámaras no tiene efecto real.

**Fix aplicado:** activar cada cámara en el panel, fitear, y restaurar la cámara original:
```mel
string $origCam = `modelPanel -q -camera $panels[0]`;
lookThru $panels[0] $cam1T; viewFit -fitFactor 0.5;
lookThru $panels[0] $cam2T; viewFit -fitFactor 0.5;
lookThru $panels[0] $cam3T; viewFit -fitFactor 0.5;
lookThru $panels[0] $origCam;
```

---

## Bug #5 — Material `lambert` en pipeline Arnold
**Línea 49** | Severidad: **Media** | Estado: `[x] Corregido`

```mel
string $cycMat = `shadingNode -asShader lambert -name "Cyclorama_MAT"`;
```

Lambert no tiene respuesta especular física. En Arnold produce un render visualmente incorrecto para un studio (bordes duros en la transición piso-pared, sin frescura de la luz). El ciclorama debería usar `aiStandardSurface` con roughness alto.

**Fix aplicado:**
```mel
$cycMat = `shadingNode -asShader aiStandardSurface -name "Cyclorama_MAT"`;
setAttr ($cycMat + ".baseColor") -type double3 0.15 0.15 0.15;
setAttr ($cycMat + ".specularRoughness") 0.9;
```

---

## Bug #6 — Sin `undoInfo` chunk — UX rota
**Global** | Severidad: **Media** | Estado: `[x] Corregido`

El script crea ~30+ nodos. Sin el wrapper de undo, el artista necesita presionar `Ctrl+Z` 30+ veces para deshacer. En producción inevitablemente deshacerán trabajo previo.

**Fix aplicado:** wrapper al inicio y fin de `createFullStudio()`:
```mel
undoInfo -openChunk;
// ... todo el código ...
undoInfo -closeChunk;
```

---

## Bug #7 — RimLight más intenso que el KeyLight
**Líneas 60 y 76** | Severidad: **Media** | Estado: `[x] Corregido`

```mel
setAttr ($keyS+".intensity") 240;   // Key
setAttr ($rimS+".intensity") 300;   // Rim → más intenso que el Key
```

En un esquema clásico 3-point: Key > Fill > Rim en términos de intensidad. El Rim a 300 sobreexpone el contorno y produce renders con look incorrecto para LookDev de producción.

**Fix aplicado:** Key 300, Fill 20, Rim 120.

---

## Bug #8 — Light linking con `defaultLightSet` (sistema legado)
**Líneas 59, 67, 75** | Severidad: **Media** | Estado: `[x] Corregido`

```mel
connectAttr -nextAvailable ($keyT + ".instObjGroups") "defaultLightSet.dagSetMembers";
```

Este es el sistema de light linking de mental ray / Maya clásico. Arnold gestiona light linking a través de `lightLinker1` y el comando `lightLink`. Funciona en casos simples pero falla con Light Groups de Arnold o con escenas que tienen configuración de light linking preexistente.

**Fix aplicado:** usar la API de alto nivel `sets -addElement`:
```mel
sets -addElement $keyT  defaultLightSet;
sets -addElement $fillT defaultLightSet;
sets -addElement $rimT  defaultLightSet;
```

---

## Bug #9 — Naming convention inconsistente con la propia documentación
**Líneas 56, 64, 72** | Severidad: **Baja** | Estado: `[x] Corregido`

`NamingConvention_Guide.md` define sufijo `_LGT` para luces. El script genera `KeyLight`, `FillLight`, `RimLight`. Los propios nodos del tool no pasarían el QA check de nomenclatura que la documentación impone.

**Fix aplicado:** renombrados a `Key_LGT`, `Fill_LGT`, `Rim_LGT` (y sus shapes correspondientes).

---

## Bug #10 — Sin validación de plugin `mtoa` para las luces principales
**Líneas 56-78** | Severidad: **Baja** | Estado: `[x] Corregido`

El SkyDome está correctamente guardado con `if pluginInfo -q -loaded "mtoa"` (línea 80). Si en el futuro se migran las área lights a `aiAreaLight` (Bug #2), ese bloque también necesitará el guard para evitar crash en instalaciones sin Arnold.

**Fix aplicado:** las luces principales ahora están dentro de un bloque `if (pluginInfo -q -loaded "mtoa")` con fallback a `areaLight` nativa.

---

## Resumen

| # | Línea | Severidad | Hallazgo | Estado |
|---|-------|-----------|----------|--------|
| 1 | 17 | Crítica | `ls -type mesh` ignora la selección — bounding box incorrecto | `[x]` |
| 2 | 57/65/73 | Alta | `areaLight` nativa en vez de `aiAreaLight` de Arnold | `[x]` |
| 3 | 40/44 | Alta | `e[3]` reutilizado post-extrude — bevel sobre arista equivocada | `[x]` |
| 4 | 112-114 | Media | `viewFit` no apunta las cámaras de studio | `[x]` |
| 5 | 49 | Media | Lambert en vez de `aiStandardSurface` para Arnold | `[x]` |
| 6 | Global | Media | Sin `undoInfo` chunk — undo roto | `[x]` |
| 7 | 60/76 | Media | RimLight (300) más intenso que KeyLight (240) | `[x]` |
| 8 | 59/67/75 | Media | `defaultLightSet` es el sistema de light linking legado | `[x]` |
| 9 | 56/64/72 | Baja | Nombres de luces sin sufijo `_LGT` (violan propia doc) | `[x]` |
| 10 | 56-78 | Baja | Luces principales sin guard de `mtoa` | `[x]` |
