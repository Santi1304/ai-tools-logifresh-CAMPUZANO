# Conclusiones de la actividad: estimación de tokens y costos de IA

## 1. Los tokens de salida explican la mayor parte del costo

**Dato:** en los cuatro escenarios de pregunta y respuesta, los tokens de salida aportaron entre 52.6% y 99.7% del costo de cada llamada.

En GPT-5.6 Sol, partiendo de pregunta corta y respuesta corta:
- Alargar la respuesta (de 41 a 427 tokens) multiplicó el costo por 10.0.
- Alargar la pregunta (de 11 a 185 tokens) solo lo multiplicó por 1.7.

La razón es doble: cada token de salida cuesta de 5 a 8 veces más que uno de entrada, y la respuesta larga agrega más tokens.

## 2. El modelo elegido define el orden de magnitud

**Dato:** con 1,000,000 de llamadas en el escenario A (pregunta larga y respuesta larga):

| Modelo | Costo |
|---|---|
| GPT-5.6 Sol | US$13,735.00 |
| Claude Opus 5 | US$11,600.00 |
| Gemini 2.5 Pro | US$4,501.25 |

Según el escenario, el modelo más caro cuesta entre 3.0 y 3.4 veces lo que Gemini 2.5 Pro, y entre 75 y 84 veces lo que gpt-5-nano.

## 3. Hay conclusiones robustas y otras frágiles

- **Inferencia (robusta):** Gemini 2.5 Pro es más barato que los otros dos modelos en todos los escenarios. Solo dejaría de serlo si contara entre 150% y 204% más tokens.
- **Supuesto (frágil):** que Claude Opus 5 sea más barato que GPT-5.6 Sol depende de un margen de solo 10.5% a 19.9%. Además, sus tokens se midieron con el tokenizador de OpenAI y no con el de Anthropic. Antes de decidir entre ambos hay que medir con la API de cada proveedor.

## 4. Verificar automáticamente evita errores que no se ven a simple vista

- **Precios:** el precio de cada modelo se validó recalculando el costo de referencia que muestra la calculadora de GPT for Work (100 tokens de entrada y 500 de salida). Así se confirmó que cada precio correspondía a su modelo.
- **Colores:** la primera paleta neón del reporte HTML fue **rechazada** por el validador de vizdata, porque el cian y el magenta se confunden con deuteranopía. Se sustituyó por una paleta que sí pasa la prueba.
- **Diseño:** el color nunca es la única señal. Las alertas llevan ícono y texto.

## 5. Las skills convierten el cálculo en un proceso repetible

`tokeniza-ia` calcula y escala los costos por escenario. `calculadora-promaster3000` mide los tokens de un proyecto completo. La calculadora reutiliza a `tokeniza-ia` y a `vizdata` en lugar de duplicar su lógica.

**Dato:** al medir el propio paquete de skills se obtuvieron 43,335 tokens estimados, que caben en una ventana de 128K (ocupan el 33.9%). Dos scripts concentran el 59.5% del total, así que serían los primeros en optimizarse.

## 6. Límites del análisis

- **Conteo:** donde no estaba disponible `tiktoken`, los tokens se estimaron con caracteres ÷ 4. Es una estimación, no un conteo exacto.
- **Precios:** se consultaron el 9 de octubre de 2026 en una fuente de terceros (GPT for Work) y deben reconsultarse antes de usarlos.
- **Tokens de razonamiento:** no se incluyeron. Los modelos de razonamiento pueden generar tokens ocultos que se cobran como salida.
- **Calidad:** el precio no mide la calidad del modelo. La elección final requiere una prueba con casos reales.

## 7. Recomendación de automatización y control de costos

| Hallazgo | Acción | Indicador de seguimiento |
|---|---|---|
| La salida domina el costo | Fijar `max_output_tokens` y pedir respuestas concisas | Tokens de salida promedio por llamada |
| El modelo multiplica el costo | Enrutar: modelo económico por defecto y escalar solo los casos complejos | % de llamadas escaladas y costo promedio por llamada |
| Comparaciones frágiles entre proveedores | Medir tokens con la API de cada proveedor en 50 a 100 casos reales | Diferencia % de tokens frente al tokenizador de OpenAI |
| Riesgo de gasto a escala | Límite mensual, alertas y `--presupuesto` de la calculadora en revisiones automáticas | % del presupuesto consumido |
