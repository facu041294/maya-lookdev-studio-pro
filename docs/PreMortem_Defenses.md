# 🚨 Directiva Pre-Mortem: LookDev Studio Pro

**Fecha de Análisis:** Mayo 2026
**Responsable:** Facundo Villarreal (Cinematic Technical Artist)
**Proyecto Asociado:** Neo-Noir Interior Pipeline (Virtual Production / Render)

## Premisa del Ejercicio
El pipeline de integración falló estrepitosamente. Los artistas de Layout no pueden exportar los assets a Unreal Engine de forma consistente, las referencias en Arnold se están rompiendo y el LookDev Studio Pro, en lugar de ahorrar tiempo, está generando retrabajos masivos. 

¿Qué causó el colapso y cómo la arquitectura del script previno estos desastres desde el día 1?

---

### Autopsia de Fallos Potenciales y Defensas Activas

#### Riesgo Crítico 1: El Caos del Outliner (Jerarquía Rota)
* **La Causa del Fallo:** Al ejecutar el script de LookDev, las luces, las cámaras y el ciclorama se crearon sueltos en la raíz (`root`) del Outliner. Un artista intentó agrupar el asset de producción (`SM_DeskChair_01`) junto con las luces temporales por error. Al exportar el `.fbx`, la escena de Unreal Engine importó el estudio completo, rompiendo el *Lightmass* y los polígonos del nivel.
* **La Defensa en Código (Mitigación Activa):** El script impone una estructura de "Jerarquía Aislada e Inmutable". En la **línea 14**, el script crea y fuerza que todos los elementos de validación residan dentro del nodo `PhotoStudio_SETUP_GRP`. El asset de producción jamás se mezcla con la estructura temporal, permitiendo al artista borrar el grupo del estudio con un solo clic o ignorarlo durante el proceso de exportación (Export Selected).

#### Riesgo Crítico 2: Duplicación Cíclica (Name Clashing)
* **La Causa del Fallo:** Un artista generó el estudio, no le gustó el encuadre de la cámara, y volvió a presionar el botón de ejecución 5 veces. Maya generó `KeyLight1`, `KeyLight2`, etc. La escena se sobrecargó de memoria, el render colapsó por exceso de rebotes de luz y el *Naming Convention Strict Check* falló al encontrar nodos con el sufijo `_1`.
* **La Defensa en Código (Mitigación Activa):** Implementación de **Idempotencia** (Safe Mode). En la **línea 13**, el script verifica activamente: `if (\`objExists "PhotoStudio_SETUP_GRP"\`) delete "PhotoStudio_SETUP_GRP";`. Esto garantiza que, sin importar cuántas veces se ejecute la herramienta, el estado de la escena siempre sea limpio y predecible, destruyendo la versión anterior antes de reconstruir.

#### Riesgo Crítico 3: Renders Negros (Arnold Light Linking Failure)
* **La Causa del Fallo:** El script instanció correctamente el entorno físico de luces, pero al lanzar el render de Arnold (TAR-024), la pantalla salió completamente en negro. Maya no vinculó automáticamente las nuevas luces a los objetos existentes en la escena.
* **La Defensa en Código (Mitigación Activa):** Enlace forzado a bajo nivel (Light Linking Force). El script no confía en la automatización de la interfaz de Maya. En las **líneas 46, 52 y 58**, se utiliza el comando `connectAttr -nextAvailable ($[lightName] + ".instObjGroups") "defaultLightSet.dagSetMembers";`. Esto asegura matemáticamente que la luz incidirá sobre todos los objetos preexistentes en la escena, independientemente del historial del archivo.

#### Riesgo Crítico 4: El Error de Escala Silencioso
* **La Causa del Fallo:** El ciclorama se generó demasiado pequeño. Las sombras del asset se cortaban abruptamente en el borde del plano, arruinando la presentación visual.
* **La Defensa en Código (Mitigación Activa):** El script utiliza la evaluación del `exactWorldBoundingBox` (Línea 26) para calcular la dimensión máxima (`$maxDim`) del asset. A partir de esa variable, toda la matemática de la escena (escalado del plano en la línea 36, offset del bisel, posicionamiento y escala de las luces) se vuelve **procedimental y dependiente del volumen del objeto**, garantizando un encuadre perfecto para cualquier geometría válida.

---
*Este documento certifica que el diseño de la herramienta prioriza la estabilidad técnica del pipeline por sobre la simple automatización de tareas de UI.*