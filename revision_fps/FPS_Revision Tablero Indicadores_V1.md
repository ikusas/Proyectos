# FPS — Revisión del Tablero de Indicadores (MVP)

**Repositorio:** `ikusas/Tableros-indicadores-FPS_MVP` (rama `master`, último cambio 27-jun-2026)
**Fecha de revisión:** 13-oct-2026

## Cómo se probó

- Se instaló y se ejecutó la aplicación en un servidor local (`npm run dev`, puerto 3000).
- El inicio de sesión con Google no se puede hacer desde la nube, así que se usó el **simulador oficial de Firebase** con datos de prueba. La base de datos real **no se tocó**.
- Recorrido completo: crear empresa (L6) → elegir 6 indicadores → hojas de cálculo → Tablero General → Medición de Salida → inicio.
- Las capturas están en esta misma carpeta (`FPS_Captura_*.png`).

## Lo que funciona bien

- El flujo es claro y sigue la lógica del programa: catálogo → captura → línea base → meta ±8 % → medición de salida.
- El catálogo tiene 42 indicadores (14 de L1, 12 de L6 y 16 de L7), con su definición, fórmula y naturaleza (aumentar o disminuir).
- Las tablas especializadas de L6 están muy completas: se puede pegar desde Excel, calculan solas y se ve qué columnas se llenan y cuáles son automáticas.
- La narrativa de cálculo del Tablero General se genera sola y se puede editar.
- La aplicación compila sin errores para producción.

## Hallazgos (ordenados por importancia)

| # | Hallazgo | Impacto | Esfuerzo |
|---|----------|---------|----------|
| 1 | **La "Meta" en Medición de Salida aparece en 0.** En la prueba, el Tablero General mostró una meta de 1.150.000, pero Medición de Salida mostró 0,00. Pasa siempre que el consultor no haya escrito la meta a mano. | Alto: el informe de salida muestra un dato falso | Bajo |
| 2 | **"Cumple / no cumple" ignora la meta.** El sistema siempre compara contra ±8 % fijo, aunque el consultor haya ajustado la meta. | Alto: puede marcar mal el cumplimiento | Bajo |
| 3 | **Se guarda en la base de datos con cada tecla.** Escribir "1250000" generó 8 guardados. Esto hace lenta la app (ya se había notado con la red) y aumenta el costo de Firebase. | Medio | Bajo |
| 4 | **La sincronización del catálogo falla en cada inicio de sesión.** Las reglas de seguridad no permiten escribir en `catalogo_indicadores`, así que la app intenta 42 escrituras y todas son rechazadas. No se ve en pantalla, pero es trabajo inútil. El catálogo ya vive dentro del código, así que la sincronización sobra. | Medio | Bajo |
| 5 | **Dos formas distintas de guardar.** Los indicadores se guardan solos, pero los datos de la empresa (periodo, producto, sub-periodos) necesitan el botón "Guardar Datos". Si el consultor lo olvida, pierde esos cambios. | Medio | Bajo |
| 6 | **Si un dato no carga, la pantalla se queda en "Cargando..."** y no muestra ningún error. | Medio | Bajo |
| 7 | **Un resultado final de 0 se trata como vacío** en Medición de Salida. | Bajo | Bajo |
| 8 | **Solo hay tres líneas (L1, L6 y L7).** La documentación del programa también menciona L3 y L9. | Funcionalidad | Alto |
| 9 | **Para revisar con la metodología:** "Porcentaje de Productos sin Renovación Reciente (PPSRR)" está marcado como *Aumentar*, pero por su nombre parece que debería *Disminuir*. | Por validar | Bajo |
| 10 | **Inicio:** las empresas archivadas aparecen mezcladas con las activas, no hay filtro de favoritas y el gráfico muestra solo variaciones positivas. | Experiencia | Medio |

### Hallazgos técnicos (no se ven en pantalla)

- La revisión de tipos (`npm run lint`) falla porque faltan dos paquetes de definiciones (`@types/react` y los tipos de Vite).
- La app pesa 1,17 MB al cargar, principalmente por Firebase y la librería de gráficos. Había además librerías instaladas sin uso (`@base-ui/react`, `react-hook-form`, `yup`, `motion` y otras); ya se quitaron, aunque no afectaban el peso final. Reducir el peso requiere cargar las pantallas por partes; queda como mejora futura.
- Las reglas de seguridad dejan crear indicadores dentro de la empresa de otro consultor si se conoce su identificador. El riesgo es bajo, pero es fácil de cerrar.

## Propuesta de trabajo

**Paquete A, correcciones rápidas (hallazgos 1 a 7 y los técnicos):** son errores de bajo esfuerzo y alto impacto. Se puede hacer en una sola entrega.

**Paquete B, mejoras de experiencia:** guardado automático de los datos de la empresa con indicador de "Guardado", filtros de archivadas y favoritas en el inicio, y un gráfico por empresa.

**Paquete C, ampliación:** tablas especializadas para L1 y L7, siguiendo el patrón de L6 y los formatos base de `docs/metodologia/`, y agregar L3 y L9.

## Estado

- **Paquete A: hecho** (13-oct-2026), en la rama `claude/github-fps-dashboard-search-16rkuk` del repositorio del tablero. Además de los hallazgos 1 a 7, se corrigió que el campo de sub-periodos no dejaba escribir comas. El hallazgo 2 quedó con esta regla: un indicador **cumple** si su variación es de al menos el mínimo del programa (8 %, configurable en `src/lib/reglas.ts`), aunque no alcance una meta más alta puesta por el consultor.
