# ai-tools-logifresh-CAMPUZANO

Tablero ejecutivo interactivo de desempeño operativo de cadena de frío para **LogiFresh México** (caso sintético), publicado como sitio estático en GitHub Pages.

**URL pública:** https://santi1304.github.io/ai-tools-logifresh-CAMPUZANO/

---

## 1. Propósito y alcance

**Pregunta de decisión que responde:** ¿en qué segmentos debería concentrarse un piloto de mejora operativa de 30 días y qué evidencia adicional hace falta antes de intervenir?

**Qué resuelve**

- Consolida 240 embarques (1 abr – 28 jun 2026) en ocho indicadores ejecutivos con contexto y meta explícita.
- Permite acotar la vista por ocho dimensiones y observar cómo cambian simultáneamente KPIs, gráficas y detalle.
- Separa de forma explícita **hechos**, **hipótesis** y **próximo paso**, y documenta lo que los datos **no** permiten afirmar.

**Qué NO cubre**

- No identifica causas. El archivo no contiene causa raíz validada, tiempos por etapa, condiciones externas ni costo de intervención.
- No sirve para evaluar o sancionar transportistas: el dataset sintético asigna a los cuatro exactamente el mismo cumplimiento (76.7 %).
- No sustituye información operativa real. Los datos son sintéticos y no contienen información personal.

## 2. Datos

| Campo | Valor |
|---|---|
| Fuente | `Datos_sinteticos_LogiFresh_dashboard.xlsx`, hoja `Datos` |
| Registros | 240 embarques, 18 campos, sin nulos, sin `id_embarque` duplicado |
| Periodo | 1 abr 2026 – 28 jun 2026 |
| Naturaleza | Sintética, con fines académicos; sin datos personales ni reales |
| Moneda / unidades | MXN · minutos · °C · escala 1–10 |

El Excel se convirtió una sola vez a `datos.js` (un objeto JSON con `columns` y `rows`). No se recalcula ni se agrega ningún valor en la conversión: es una copia fiel, con la fecha normalizada a `YYYY-MM-DD`. Todos los agregados se calculan en el navegador al vuelo.

## 3. Arquitectura

```
index.html   HTML + CSS + JS en un solo archivo (sin dependencias externas)
datos.js     Dataset embebido (240 filas, ~36 KB)
.nojekyll    Evita el procesamiento Jekyll en GitHub Pages
README.md
REPORTE_VALIDACION.md
```

**Decisiones y por qué**

| Decisión | Razón |
|---|---|
| Sin librerías (sin Chart.js, D3 ni frameworks) | Carga rápida, cero peticiones a CDN, cero riesgo de recurso bloqueado en la URL pública. Total del sitio < 60 KB. |
| Gráficas en SVG generado a mano | Control total de etiquetas, unidades, líneas de referencia y `<title>` accesible; escala sin pérdida en móvil. |
| Datos en `datos.js` y no `fetch()` de un JSON | Evita problemas de CORS y de ruta al abrir el archivo localmente; una sola petición extra en Pages. |
| Rutas relativas en todo el sitio | Requisito para que funcione bajo `usuario.github.io/repositorio/`. |
| Paleta Okabe–Ito | Segura para deficiencias de visión al color (validada con simulación de protanopia, deuteranopia y tritanopia). Naranja = por debajo de meta, azul = en meta; el color nunca es el único canal: siempre hay etiqueta numérica directa. |
| Promedio de retraso sólo sobre tardíos | Incluir los embarques que cumplen (retraso 0) diluiría el indicador y daría 12.1 min en lugar de 51.8 min. |
| Brecha expresada en **pp** | Restar porcentajes produce puntos porcentuales, no por ciento. |

## 4. Contenido del tablero

- **Encabezado:** pregunta de decisión, periodo, meta SLA de 90 % y advertencia de datos sintéticos.
- **8 KPIs con contexto:** embarques, SLA, brecha vs. meta, retraso promedio de tardíos, incidentes, excursiones > 8 °C, reclamaciones y satisfacción. Ninguno es un número suelto: todos llevan denominador, unidad o comparación.
- **8 filtros** (mes, origen, destino, producto, transportista, tipo de ruta, SLA, incidente) con resumen en vivo y botón de restablecer.
- **4 gráficas no redundantes:** SLA por mes con línea de meta; SLA por segmento (dimensión seleccionable); incidentes por categoría; reclamaciones por dimensión seleccionable.
- **Tabla de detalle** paginada, con desplazamiento horizontal propio.
- **Panel Hechos / Hipótesis / Próximo paso**, con piloto de 30 días, basal, criterio de éxito e indicador de seguimiento.
- **Definiciones de cálculo, límites y datos faltantes.**

## 5. Accesibilidad

- Navegación completa por teclado; enlace «Saltar a los indicadores»; foco visible de 3 px.
- Etiquetas `<label>` asociadas a cada control; regiones con `aria-labelledby`; el resumen de filtros y el estado sin datos son `role="status"` con `aria-live`.
- Contraste de texto ≥ 4.5:1; los colores de la paleta se usan sólo como relleno de área, nunca como texto ni línea delgada.
- Cada SVG lleva `role="img"` con `aria-label` descriptivo y `<title>` por barra.

## 6. Privacidad y seguridad

Sin tokens, claves, credenciales ni datos personales en el repositorio. Sin analítica, cookies, almacenamiento local ni peticiones a terceros. El repositorio contiene únicamente los cinco archivos listados arriba.

## 7. Reproducir localmente

```bash
git clone https://github.com/Santi1304/ai-tools-logifresh-CAMPUZANO.git
cd ai-tools-logifresh-CAMPUZANO
python3 -m http.server 8000   # abrir http://localhost:8000
```

## 8. Validación

Ver [`REPORTE_VALIDACION.md`](REPORTE_VALIDACION.md): perfil de calidad, reconciliación de los siete valores de control, matriz de pruebas (esperado / obtenido / estado), fallas detectadas y corregidas, y limitaciones.
