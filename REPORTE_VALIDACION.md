# Reporte de validación — ai-tools-logifresh-CAMPUZANO

**Fecha de validación:** 3 de septiembre de 2026
**Artefacto validado:** tablero estático `index.html` + `datos.js`
**Fuente:** `Datos_sinteticos_LogiFresh_dashboard.xlsx`, hoja `Datos` (240 filas)
**URL pública validada:** https://santi1304.github.io/ai-tools-logifresh-CAMPUZANO/

---

## 1. Perfil de calidad de datos

| Verificación | Resultado |
|---|---|
| Filas / columnas | 240 × 18 |
| Valores nulos | 0 en las 18 columnas |
| `id_embarque` duplicados | 0 |
| Rango de fechas | 2026-04-01 a 2026-06-28, sin huecos de mes |
| Tipos | 1 temporal, 12 categóricas, 5 numéricas; todas consistentes |
| Cardinalidad | origen 6 · destino 8 · producto 5 · transportista 4 · tipo_ruta 3 · unidad 38 · incidente 6 (incluye «Sin incidente») |
| Coherencia interna | `excursion_temp_mayor_8c = "Sí"` coincide exactamente con `temperatura_max_c > 8` (9 de 9 casos) |
| Coherencia interna | `retraso_min > 0` coincide exactamente con `sla_entrega = "No cumple"` (56 de 56 casos) |

## 2. Reconciliación de los valores de control

Calculados de forma independiente con pandas sobre el Excel, y comparados contra el tablero servido desde la URL pública.

| Indicador | Esperado (guía) | Calculado del archivo | Obtenido en la URL pública | Estado |
|---|---|---|---|---|
| Embarques | 240 | 240 | 240 | ✅ |
| SLA | 76.7 % | 184/240 = 76.7 % | 76.7 % | ✅ |
| Retraso promedio de tardíos | 51.8 min | 51.8 min (n = 56) | 51.8 min | ✅ |
| Incidentes | 52 | 52 (14+12+9+9+8) | 52 | ✅ |
| Excursiones > 8 °C | 9 | 9 | 9 | ✅ |
| Reclamaciones | $882,649 MXN | **$882,549 MXN** | $882,549 MXN | ⚠️ **Discrepancia** |
| Satisfacción | 8.5/10 | 8.5/10 | 8.5/10 | ✅ |

**Discrepancia documentada (no corregida a mano).** La suma exacta de `reclamacion_mxn` en las 240 filas es **882,549 MXN**; la guía indica 882,649 MXN. La diferencia es de **$100 (0.011 %)**. Se revisó: no hay nulos, no hay filas excluidas, la columna es entera y sólo 15 registros tienen valor > 0. El tablero muestra el valor **calculado del archivo**, porque alterar el dato para que cuadre con la documentación falsificaría la evidencia. Es una desviación entre la guía y la versión del archivo, no un error de cálculo del tablero.

Frecuencias de incidente reconciliadas con la guía: Falla mecánica 14, Ventana de entrega 12, Temperatura 9, Documentación 9, Tráfico 8. ✅

## 3. Matriz de pruebas

Ejecutadas dos veces: (a) en local con Chromium headless sobre el sitio servido por HTTP, y (b) **sobre la URL pública de GitHub Pages** ya desplegada. Los resultados de ambas ejecuciones coinciden; la columna «obtenido» corresponde a la ejecución pública.

| # | Prueba | Esperado | Obtenido | Estado |
|---|---|---|---|---|
| T1 | Carga sin filtros | 240 · 76.7 % · −13.3 pp · 51.8 min · 52 · 9 · $882,549 · 8.5 | idéntico | ✅ |
| T2 | Filtro individual: Transportista = Norte | 60 · 76.7 % · 50.6 min · 13 inc · 2 exc · $118,700 · 8.0 | idéntico | ✅ |
| T3 | Dos filtros: Norte + Berries (intersección, no suma) | 12 embarques · 75.0 % · 35.0 min · 3 inc · 0 exc · $0 · 8.0 | idéntico | ✅ |
| T4 | Caso sin resultados: Norte + Berries + Abril + «No cumple» | 0 embarques, mensaje claro, sin error | 0 · aviso «Sin resultados» visible · KPIs en «s/d» · tabla con mensaje explícito | ✅ |
| T5 | Restablecer filtros | Regresa a 240 y a todos los valores de control | idéntico a T1 | ✅ |
| T6 | Todos los componentes reaccionan al filtro | KPIs, 4 gráficas, tabla, resumen y panel de hechos cambian | los 4 SVG se redibujan, la tabla repagina (25 filas), el resumen y los hechos se recalculan | ✅ |
| T7 | Cambio de dimensión (transportista → tipo de ruta) | 4 barras → 3 barras | 4 → 3 | ✅ |
| T8 | Vista móvil 375 px sobre la URL pública | Sin desplazamiento horizontal del documento | `scrollWidth` 375 = `clientWidth` 375; 0 elementos desbordados fuera del contenedor de la tabla | ✅ |
| T9 | Consola del navegador | 0 errores | 0 errores | ✅ |
| T10 | Accesibilidad básica | Foco visible, navegación por teclado, `label` por control, `aria-live` en estados | verificado: enlace de salto, foco de 3 px, 10 selects etiquetados, 2 regiones `role="status"` | ✅ |
| T11 | Paleta accesible | Distinguible bajo protanopia, deuteranopia y tritanopia | 10/10 pares aprobados por simulación; el color nunca es único canal (etiqueta numérica directa en cada barra) | ✅ |
| T12 | Carga de recursos desde la URL pública | HTTP 200, rutas relativas, sin 404 | `index.html` y `datos.js` cargan; el tablero renderiza completo | ✅ |
| T13 | Cero dependencias externas | 0 peticiones a terceros | 1 recurso cargado (`datos.js`), 0 peticiones a dominios externos | ✅ |

## 4. Fallas detectadas y corregidas

| # | Falla | Detección | Corrección |
|---|---|---|---|
| F1 | Error de sintaxis JavaScript (paréntesis de cierre sobrante en la construcción del panel de hechos). La página cargaba en blanco: sin KPIs, sin gráficas, sin tabla. | Prueba T1 automatizada en local: `PAGEERROR Unexpected token ')'` y 0 elementos `.kpi` | Se reescribió el bloque con lógica condicional explícita en lugar de ternarios anidados. Re-ejecutada toda la matriz. |
| F2 | Desbordamiento horizontal de 5 px en vista móvil (375 px), causado por el nombre largo del archivo dentro de `<code>` en el encabezado. | Prueba T8: `scrollWidth` 380 > `clientWidth` 375 | Se añadió `overflow-wrap:anywhere` a `code`. Re-medido en local y en la URL pública: 375 = 375. |

Ninguna falla quedó abierta al momento de la publicación.

## 5. Trazabilidad del proceso de publicación

| Momento | Qué ocurrió |
|---|---|
| Inspección | Se leyeron la carpeta de la actividad, el diccionario (`Diccionario_y_control`) y la guía docx. |
| Perfil y reconciliación | pandas sobre el Excel; 6 de 7 valores de control coinciden, 1 discrepancia documentada (§2). |
| Diseño | Se definieron audiencia (dirección), pregunta de decisión, KPIs con contexto, 4 gráficas no redundantes y panel Hechos/Hipótesis/Próximo paso antes de escribir código. |
| Construcción | HTML+CSS+JS en un solo archivo, sin librerías; datos en `datos.js`. |
| Pruebas locales | Chromium headless; 2 fallas detectadas y corregidas (§4). |
| **Bloqueo de permisos** | El entorno de la sesión restringe el acceso a GitHub a repositorios previamente autorizados: no fue posible crear el repositorio ni hacer `git push` mediante el proxy de credenciales (`is not in this session's authorized repository set`), y la extensión de navegador no estaba conectada. |
| **Autorización solicitada** | Se detuvo el proceso, se explicó el bloqueo al responsable y se solicitó autorización explícita antes de publicar. El responsable autorizó la creación de un repositorio público con los cinco archivos listados. |
| Publicación | El agente inició sesión en GitHub a través del panel de navegador del entorno (el responsable capturó sus propias credenciales; el agente nunca las vio ni las escribió) y desde ahí creó el repositorio, subió los cinco archivos y activó GitHub Pages desde la rama `main`, carpeta raíz. |
| Verificación pública | Se abrió `https://santi1304.github.io/ai-tools-logifresh-CAMPUZANO/` y se re-ejecutó la matriz de pruebas completa (§3) contra el sitio desplegado. |

**Intervención humana declarada:** el responsable únicamente (a) autorizó la publicación, (b) capturó sus credenciales de GitHub en la pantalla de inicio de sesión y (c) aprobó los permisos de acceso a sitios del navegador. No escribió código, ni comandos de git, ni configuró GitHub Pages manualmente.

## 6. Supuestos declarados

1. La escala de satisfacción (ordinal 1–10) se promedia como si fuera numérica; es el criterio de la guía, pero un promedio de una escala ordinal es una simplificación.
2. `retraso_min` se interpreta como minutos completos de retraso respecto del compromiso, y su cero significa cumplimiento, no ausencia de dato.
3. El mes se deriva de `fecha_salida` (fecha de salida), no de fecha de entrega, que no existe en el archivo.
4. Las reclamaciones se suman en MXN corrientes, sin deflactar ni ajustar por valor de la carga (no disponible).

## 7. Limitaciones del análisis

- **Asociación ≠ causalidad.** Ningún indicador de este tablero identifica la causa de un retraso, una excursión térmica o una reclamación.
- **La estructura del dataset es sintética y regular.** Los cuatro transportistas registran exactamente 76.7 % de cumplimiento y 60 embarques cada uno; las cinco categorías de incidente se distribuyen de forma periódica. **El corte por transportista no contiene señal utilizable** y no debe usarse para evaluar proveedores.
- **Incidentes e incumplimientos son conjuntos disjuntos.** Los 52 embarques con incidente registrado cumplieron el SLA; los 56 incumplimientos no tienen incidente clasificado. Esto confirma que el registro de incidentes está incompleto respecto de los incumplimientos, y es la razón por la que el piloto propuesto se centra en **completar el registro de causa raíz**, no en corregir una causa aún no identificada.
- **Concentración temporal extrema.** Abril y mayo cumplen al 100 % y junio cae a 30 %. Todo el incumplimiento del trimestre está en un solo mes, lo que limita la validez de cualquier corte que no controle por mes.
- **Concentración económica extrema.** 15 de 240 embarques (6.3 %) concentran el 100 % de las reclamaciones; el máximo individual es de $145,000 MXN. Los promedios por segmento son muy sensibles a estos casos.

## 8. Datos faltantes para decidir

Causa raíz validada por embarque · tiempos por etapa (salida, tránsito, ventana de entrega) · condiciones externas verificadas (clima, tráfico) · valor y sensibilidad térmica de la carga · costo estimado de la intervención · exposición (volumen y valor) por segmento.
