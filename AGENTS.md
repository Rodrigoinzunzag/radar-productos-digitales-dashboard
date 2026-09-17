# Contexto del proyecto — Radar de Productos Digitales

## Propósito

Este repositorio publica el **Radar vivo de productos digitales**, un dashboard estático que ayuda a Rodrigo Inzunza a:

- monitorear longitudinalmente anunciantes encontrados en Facebook Ads Library;
- comparar el número actual de anuncios con observaciones anteriores;
- detectar crecimiento, caída, estabilidad o reaparición;
- priorizar oportunidades mediante un score calculado;
- filtrar por país, tendencia y score;
- seleccionar productos y copiar un brief para analizar avatar, dolor, mecanismo, oferta, landing y ángulos de venta mejorados.

El alcance comercial del radar son principalmente productos digitales documentales o descargables en español/mercados LATAM: ebooks, PDF, guías, recetarios, plantillas, presentaciones, imprimibles, planos, bibliotecas o archivos digitales. No se deben reincorporar apps, SaaS, software, cursos genéricos, certificaciones, mentorías, eventos, lugares físicos o servicios, salvo que la oferta principal sea claramente un recurso descargable. El generador también excluye filas con señales de WhatsApp como canal principal.

## Regla principal

**No editar `index.html` directamente.**

`index.html` es un artefacto generado y será reemplazado en la siguiente actualización automática. Todo cambio permanente de diseño, lógica, filtros, copy, score o estructura debe hacerse en:

`/opt/data/productos-digitales/radar_vivo_productos_digitales.py`

Después hay que regenerar, publicar y verificar el resultado.

## Arquitectura

El sistema tiene cuatro capas:

1. **Recolección de datos**
   - PDF: cron `6c2b02029514`, `Radar productos digitales Facebook Ads Library`, `0 11 */2 * *` UTC.
   - Mini apps: cron `adc809ca713b`, `Radar mini apps Facebook Ads Library`, `0 13 */2 * *` UTC.
   - Ambos investigan Facebook Ads Library y agregan observaciones a sus hojas respectivas.

2. **Fuente única de datos**
   - Google Sheet: `https://docs.google.com/spreadsheets/d/1qs7JHssbOEZNqxrbC9ux9NN3hIIMS_icsYbJiQ59y2A/edit`
   - Spreadsheet ID: `1qs7JHssbOEZNqxrbC9ux9NN3hIIMS_icsYbJiQ59y2A`
   - Hojas utilizadas: `Seguimiento` (PDF) y `Mini Apps` (herramientas interactivas).
   - El dashboard no consulta Meta directamente: calcula solo sobre las filas existentes en estas hojas.

3. **Generación del dashboard**
   - Generador: `/opt/data/productos-digitales/radar_vivo_productos_digitales.py`
   - Módulo Mini Apps: `/opt/data/productos-digitales/radar_mini_apps.py`
   - Plantilla separada: `/opt/data/productos-digitales/radar_dashboard_template.html`
   - Salida maestra local: `/opt/data/productos-digitales/radar_vivo_productos_digitales.html`
   - Estado de la última generación/Drive: `/opt/data/productos-digitales/last_update.json`
   - El HTML contiene CSS, JavaScript y datos JSON embebidos; no necesita backend en tiempo de navegación.

4. **Publicación**
   - Wrapper: `/opt/data/scripts/update_radar_vivo_productos_digitales.sh`
   - Cron Hermes: `0831e7197d27`
   - Nombre: `Actualizar artefacto vivo - Radar productos digitales`
   - Frecuencia: `40 11 */2 * *` UTC, después del cron recolector.
   - Modo: `no_agent`; no produce mensaje si termina bien y alerta si falla.
   - El wrapper publica la misma versión en Google Drive, GitHub Pages y Apps Script.

## Archivos del repositorio

- `index.html`: dashboard publicado; generado automáticamente.
- `README.md`: resumen corto del repositorio.
- `AGENTS.md`: este contexto operativo para agentes y mantenedores.

Este repositorio no contiene el generador ni la fuente de datos. Su función principal es alojar el HTML estático para GitHub Pages.

## Enlaces estables

- Dashboard principal, GitHub Pages: `https://rodrigoinzunzag.github.io/radar-productos-digitales-dashboard/`
- Repositorio: `https://github.com/Rodrigoinzunzag/radar-productos-digitales-dashboard`
- Respaldo Apps Script: `https://script.google.com/macros/s/AKfycbzZaI_T9Av7lIUdzQw0NvcHL4Hhh04sZMdrlBovGot_tYxm1KUSKChQahMZM2fLFZs/exec`
- Archivo Drive: `https://drive.google.com/file/d/1Chz9HkUsPMtidUZyA4igTiDNSs2Bg5Wi/view?usp=drivesdk`
- Descarga Drive: `https://drive.google.com/uc?id=1Chz9HkUsPMtidUZyA4igTiDNSs2Bg5Wi&export=download`

GitHub Pages es el enlace principal. Drive puede mostrar el código HTML en vez de renderizarlo, y Apps Script puede sufrir redirecciones de múltiples cuentas a rutas `/macros/u/<N>/...`.

## Columnas esperadas en Google Sheets

El generador requiere estas columnas en la primera fila de `Seguimiento`:

- `Nombre del anunciante`
- `Link más información`
- `Producto`
- `Nicho`
- `Fecha de revisión`
- `Número de anuncios`
- `Palabra clave origen`
- `Rango fecha filtrado hasta`
- `Estado anuncios`
- `Idioma`
- `Evaluación`
- `Notas`

Puede usar además `País`, `Pais` o `Country`. Si no existe una columna explícita, el país se infiere desde URL, producto, nicho, palabra clave, idioma, evaluación y notas. Las opciones visibles son Chile, Argentina, Brazil y `Sin país detectado`.

La hoja `Mini Apps` mantiene A:L iguales a `Seguimiento` y agrega, en este orden:

- `País`
- `Tipo de mini app`
- `Precio`
- `Moneda`
- `Modelo de cobro`
- `Plataforma checkout`
- `Tecnología visible`
- `Circulando desde`
- `Días activo máx.`
- `Países vistos`
- `Page ID`
- `Score`
- `Tendencia`
- `Run ID`

No cambiar nombres de columnas sin actualizar y probar el generador. Si falta una columna requerida, la generación falla explícitamente.

## Transformación de datos

El generador:

1. Lee `Seguimiento!A:Z` y `Mini Apps!A:Z` mediante Google Sheets API.
2. Excluye filas con marcadores de WhatsApp/WApp/wa.me como canal principal.
3. Agrupa PDF por `Nombre del anunciante`; Mini Apps usa `Page ID` y, como fallback, el nombre normalizado.
4. Ordena cada grupo por fecha y orden de fila.
5. Usa la observación más reciente como estado actual.
6. Busca hacia atrás la última observación con un número de anuncios interpretable.
7. Calcula valor actual, anterior, delta, cambio porcentual, últimos tres conteos y tendencia.
8. Calcula el score de oportunidad.
9. Embebe ambas pistas dentro de `<script id="radarData" type="application/json">`, preservando intacto el contrato PDF.
10. Inserta el JSON en `radar_dashboard_template.html` con `str.replace`, sin interpretar las llaves ni los escapes del CSS/JavaScript.
11. Renderiza pestañas, vistas, filtros, selección y briefs íntegramente en el navegador.

### Interpretación del conteo

`first_int()` intenta extraer el primer entero pequeño del texto de `Número de anuncios`. Devuelve `null` cuando el valor está vacío o contiene `no visible`. Los textos aproximados o narrativos deben revisarse con cuidado: el número normalizado no reemplaza el valor original, que se conserva como `raw_count`.

### Tendencias

- `Nuevo / sin histórico`: no hay observación anterior numérica.
- `Desaparece`: actual = 0 y anterior > 0.
- `Reaparece`: anterior = 0 y actual > 0.
- `Crecimiento fuerte`: delta >= 3.
- `Crecimiento`: delta > 0.
- `Caída fuerte`: delta <= -3.
- `Decrecimiento`: delta < 0.
- `Estable`: delta = 0.
- `Sin dato`: conteo actual o comparación no disponible.

## Score de oportunidad

El score se limita al rango 0–100 y combina:

- volumen actual: hasta 35 puntos (`current_count * 2`);
- delta: entre -25 y +25 (`delta * 4`, acotado);
- cantidad de observaciones: hasta 14 puntos (`observations * 4`);
- evaluación textual: `alto potencial` +28, `bueno` +18, `dudoso` -8;
- señales de producto digital: hasta 18 puntos;
- enlace HTTP disponible: +5 puntos.

Antes de modificar la fórmula, evaluar casos representativos de volumen alto/estable, crecimiento reciente, caída, nuevo sin histórico y conteos no visibles. Un cambio de score afecta orden, colores, métricas y selección comercial.

### Score de Mini Apps

`radar_mini_apps.py` limita el score a 0–100 y combina volumen hasta 30 puntos, longevidad hasta 20, señal de “varias versiones” +10, presencia observada en dos o más países +8, precio con checkout visible +10, tipo con evidencia +12 y delta entre -10 y +10.

## Funciones de la interfaz

- Pestañas `PDF` y `Mini apps`.
- Vistas `Prioridades` y `Mapa`, con decisión `Replicar ya`, `Investigar`, `Vigilar` o `Descartar` calculada en el navegador.
- Búsqueda, filtro por país y orden por oportunidad, crecimiento, anuncios o fecha; Mini Apps agrega tipo, modelo de cobro y días activo.
- Selección cruzada entre pistas y copia de briefs individuales o combinados.
- Navegación directa mediante `#pdf`, `#pdf/mapa`, `#mini_apps` y `#mini_apps/mapa`.
- Diseño responsive y modo claro/oscuro automático según el sistema.
- `Actualizar ahora` recarga la página; no ejecuta el pipeline del servidor.
- Recarga automática del navegador después de 48 horas si la página permanece abierta.

La fecha se almacena como UTC ISO y se formatea en el navegador con la zona horaria local del usuario.

## Comandos operativos

### Regenerar y publicar todo

```bash
/opt/data/scripts/update_radar_vivo_productos_digitales.sh
```

Este comando:

1. genera el HTML y actualiza Drive;
2. copia el HTML a este repositorio como `index.html`;
3. crea commit y hace push a `main` si hay cambios;
4. actualiza el deployment de Apps Script.

Usa reintentos con backoff para Google/Apps Script. Si una etapa final falla, las anteriores pueden haber quedado publicadas; verificar cada destino antes de reintentar para no diagnosticar incorrectamente.

### Generar sin publicar

```bash
/opt/hermes/.venv/bin/python \
  /opt/data/productos-digitales/radar_vivo_productos_digitales.py \
  --output /tmp/radar-productos-digitales.html \
  --json
```

### Generar y actualizar Drive

```bash
/opt/hermes/.venv/bin/python \
  /opt/data/productos-digitales/radar_vivo_productos_digitales.py \
  --upload-drive \
  --json
```

Usar siempre `/opt/hermes/.venv/bin/python`, porque allí están instaladas las dependencias de Google. No usar el Python del sistema ni intentar instalar paquetes globalmente.

## Flujo correcto para cambios

1. Leer este archivo y el generador completo.
2. Confirmar si el cambio afecta UI, datos, fórmula, fuente, cron o publicación.
3. Modificar `/opt/data/productos-digitales/radar_vivo_productos_digitales.py`, no `index.html`.
4. Hacer una generación de prueba a `/tmp` cuando el cambio sea riesgoso.
5. Revisar sintaxis Python y el HTML/JavaScript resultante.
6. Abrir el HTML en un navegador y probar filtros, orden, selección, copiado y consola.
7. Ejecutar el wrapper completo.
8. Revisar `/opt/data/productos-digitales/last_update.json`.
9. Verificar `git status`, commit y push de este repositorio.
10. Abrir GitHub Pages y comprobar timestamp, métricas y comportamiento.
11. Comprobar Drive y Apps Script por separado.
12. Revisar el estado del cron si el cambio afecta la ejecución programada.

## Verificación mínima

No considerar terminado un cambio solo porque el generador salió con código 0.

- El archivo generado existe y tiene tamaño no trivial.
- El JSON embebido se puede parsear.
- `generated_at` cambió y es UTC ISO válido.
- Las métricas coinciden con los elementos/filas procesados.
- No hay errores JavaScript en consola.
- Búsqueda, filtros y ordenamiento funcionan.
- La cabecera de la tabla permanece visible dentro del contenedor con scroll.
- Seleccionar/quitar actualiza el textarea.
- Copiar brief produce texto completo.
- La vista móvil sigue siendo utilizable.
- `git status --short` queda limpio después de publicar.
- `origin/main` contiene el commit nuevo.
- GitHub Pages entrega la versión nueva, no una copia en caché.
- Drive y Apps Script se verifican como destinos independientes.

## Autenticación y secretos

- Token principal de Google Workspace: `/opt/data/google_token.json`.
- No leer, imprimir, copiar al repositorio ni incluir tokens, secretos OAuth o credenciales en logs/contexto.
- Verificar Google Workspace con el helper `setup.py --check-live` antes de diagnosticar el generador.
- La actualización de Apps Script requiere además scopes `script.projects` y `script.deployments`.
- Si Apps Script responde `403 ACCESS_TOKEN_SCOPE_INSUFFICIENT`, reautorizar esos scopes con `/opt/data/productos-digitales/authorize_apps_script.py`; no modificar la lógica del dashboard para compensar un problema OAuth.
- La autorización requiere consentimiento humano en Google; no prometer automatización total de ese paso.

## Fallas comunes

### `invalid_grant`, token expirado o revocado

Reautorizar Google Workspace y verificar una llamada real antes de volver a ejecutar el wrapper. El watchdog de Google Workspace corre antes de los jobs dependientes, pero los tokens todavía pueden revocarse por cambios de seguridad, consentimiento o cliente OAuth.

### `403 ACCESS_TOKEN_SCOPE_INSUFFICIENT` en Apps Script

La generación, Drive y GitHub Pages pueden haber terminado correctamente aunque el wrapper completo figure como error. Revisar primero:

- `/opt/data/productos-digitales/last_update.json`;
- el commit más reciente del repositorio;
- el timestamp visible en GitHub Pages;
- el artefacto del cron bajo `/opt/data/cron/output/0831e7197d27/`.

Después renovar los scopes de Apps Script con el autorizador específico y volver a ejecutar solo tras confirmar el estado de los destinos ya publicados.

### GitHub Pages no refleja el cambio

Confirmar que hubo diff, commit y push en `main`; luego consultar el HTML publicado y su `generated_at`. Considerar caché de GitHub Pages/navegador antes de generar otra versión.

### Drive muestra código HTML

Es comportamiento normal del visor de Drive. Usar GitHub Pages como dashboard principal o descargar el archivo. No tratar `webViewLink` como hosting web confiable.

### Cambio manual desaparece

Probablemente se editó `index.html`. Reaplicar el cambio al generador y regenerar.

## Git

- Rama de publicación: `main`.
- Remoto: `origin` -> `https://github.com/Rodrigoinzunzag/radar-productos-digitales-dashboard.git`.
- Los commits automáticos usan el formato `Actualizar Radar <timestamp UTC>`.
- El wrapper solo hace commit cuando `index.html` cambió.
- No reescribir historia, hacer force push ni borrar commits para una actualización normal.
- No incluir archivos de credenciales, tokens, respaldos del Sheet ni datos temporales.

## Principios de mantenimiento

- Google Sheets es la fuente única; no inventar ni completar datos ausentes en el dashboard.
- Distinguir observaciones del Sheet de anunciantes únicos.
- Mantener trazabilidad entre observación, fecha, conteo original y cálculo normalizado.
- Preferir fuentes y enlaces directos verificables en la recolección upstream.
- Cambiar el generador, no el artefacto.
- Mantener sincronizados copy visible, lógica, frecuencia del cron y recarga del navegador.
- Probar con datos reales, pero evitar cambios destructivos en el Sheet durante QA.
- Si se cambia el alcance de países o tipos de productos, actualizar conjuntamente el cron recolector, el generador, los filtros y este contexto.
