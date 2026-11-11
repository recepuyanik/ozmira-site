# OZMIRA

[Türkçe](README.md) · [English](README.en.md) · [Deutsch](README.de.md) · Español

Una aplicación local que muestra tus propios sistemas de IA en una única escena 3D. Cada sistema aparece con su propio color y forma (galaxia, fibra holográfica, doble hélice, agujero negro, esfera). Cuando un agente empieza a trabajar, una señal viaja desde el centro hasta él, regresa cuando termina el trabajo y los tokens gastados aparecen en la escena.

Sin cuenta y sin nube. Las definiciones de tus sistemas, los eventos y tu clave de acceso se quedan solo en tu equipo. Existe un uso compartido anónimo opcional, desactivado por defecto (detalles abajo).

## Iniciar

Necesitas: Python 3.8 o superior (probado con 3.13) y un navegador actualizado. No hay nada más que instalar.

| Sistema | Ejecutar |
| --- | --- |
| Windows | doble clic en `ozmira.bat` |
| macOS | doble clic en `ozmira.command` (la primera vez: clic derecho, Abrir) |
| Linux | `./ozmira.sh` |
| Todos | `python3 ozmira.py` (en Windows: `py ozmira.py`) |

El navegador se abre solo. Si no lo hace, ve a la dirección `http://localhost:8787/` que aparece en el terminal. Para detener, pulsa Ctrl+C en el terminal.

En el primer inicio aparecen una escena de ejemplo y un asistente de configuración. En el asistente le pones nombre al centro y añades tus sistemas. Hay plantillas listas (Claude, ChatGPT, Gemini, Grok, Otro); puedes cambiar nombre, estilo y color. Después puedes cambiarlo todo con el botón Editar de abajo.

## Abrir como ventana de aplicación

Puedes abrir OZMIRA en su propia ventana, sin barra de direcciones, en lugar de una pestaña del navegador. Para ello necesitas Chrome, Edge o Brave en tu equipo (Edge ya viene con Windows 10 y 11).

| Sistema | Ejecutar |
| --- | --- |
| Windows | doble clic en `ozmira-uygulama.bat` (no abre ventana de consola) |
| macOS | doble clic en `ozmira-uygulama.command` |
| Linux | `./ozmira-uygulama.sh` |
| Todos | `python3 ozmira.py --uygulama` |

Al cerrar la ventana, OZMIRA también se detiene. La ventana usa una carpeta de perfil aparte (`.ozmira/pencere`), así que tus sesiones personales del navegador y tus inicios de sesión quedan fuera.

Para poner un acceso directo en el escritorio (y, en Linux, en el menú de aplicaciones), ejecuta esto una vez:

```
python3 ozmira.py kisayol
```

El acceso directo abre la ventana de aplicación y usa el icono de OZMIRA (`ozmira.ico`, `web/icon.png`). Es un acceso directo, no un `.exe` de un solo archivo ni un instalador, y necesita que Python esté instalado.

## Idioma

La interfaz está disponible en turco, inglés, alemán y español. En el primer inicio se usa el idioma del navegador (inglés si el idioma no es compatible). Puedes cambiarlo en cualquier momento con el botón de idioma al final del dock (TR, EN, DE, ES); tu elección se recuerda en este navegador. La salida de la línea de comandos sigue el idioma del sistema; puedes forzarlo con `OZMIRA_DIL=tr|en|de|es`. Este README también existe como `README.md` (turco), `README.en.md` y `README.de.md`.

Nota: las palabras de los comandos están en turco y son iguales en todos los idiomas: `olay` = evento, `basla` = inicio, `bitti` = terminado, `kisayol` = acceso directo, `--uygulama` = ventana de aplicación.

## Gratis y Pro

| | Gratis | Pro |
| --- | --- | --- |
| Escena 3D, aviso de eventos, contador de tokens, funcionamiento local | Sí | Sí |
| Sistemas de IA (Claude, ChatGPT, Gemini, etc.) | Hasta 2 | Sin límite* |
| Agentes | Hasta 7 en total | Sin límite* |
| Estilos visuales | Galaxia, Esfera | Los cinco (incluidos Fibra, Doble hélice, Agujero negro) |

\* En Pro no hay un límite de pago. Solo existe un tope técnico para que la escena siga fluida: 24 sistemas y 200 agentes por sistema.

En la versión gratuita, si tu perfil contiene más de 7 agentes (incluidos los leídos de una carpeta) recibes un aviso al guardar. Si tu carpeta de agentes crece después, en la escena aparecen los primeros 7 agentes y el resto se indica en pantalla con la nota «+N agentes fuera del límite gratuito»; no se borra nada.

La clave de licencia Pro se pega desde el botón Cuenta del dock (en la versión gratuita dice «Gratis»). La clave se verifica sin conexión en este equipo; no se conecta a ningún servidor. Una licencia puede tener caducidad; al caducar, la aplicación vuelve a los límites gratuitos, tu perfil guardado no se borra pero al guardar de nuevo se aplican los límites.

## Añadir tus agentes

Para cada sistema hay dos vías, y puedes usar las dos a la vez:

- **Carpeta:** señala una carpeta de agentes formada por archivos `.md`. El nombre de cada archivo pasa a ser el nombre del agente; si el archivo empieza con líneas `name:` y `description:`, se usan. Solo se leen los primeros 4 KB de cada archivo y solo se toman esas dos líneas; el contenido no se guarda.
- **A mano:** escribe los nombres de los agentes uno por línea o separados por comas.

## Mostrar el trabajo en vivo (Conectar)

OZMIRA no puede saber por sí sola cuándo trabaja un agente; hay que avisarle. El botón Conectar del dock te permite copiar comandos ya preparados para tus sistemas.

```
python3 ozmira.py olay <sistema> <agente> basla
python3 ozmira.py olay <sistema> <agente> bitti 1200 "etiqueta de tarea"
```

`<sistema>` es el ID corto del sistema (derivado de su nombre; la ventana Conectar lista los ID). `1200` es el número de tokens gastados y es opcional. Lo mismo se puede hacer por HTTP:

```
curl -X POST http://localhost:8787/api/olay \
  -H "X-Token: <tu clave>" -H "Content-Type: application/json" \
  -d '{"sistem":"<sistema>","ajan":"<agente>","olay":"bitti","token":1200}'
```

Puedes llamarlo desde tus propios scripts, automatizaciones o los pasos previos y posteriores de tus herramientas. Lo único que se envía es el sistema, el nombre del agente, el tipo de evento, el número de tokens y, si quieres, una etiqueta de tarea de hasta 120 caracteres. No se envían prompts, respuestas ni código.

Si un agente avisa `basla` y no avisa `bitti` en 30 minutos, deja de aparecer como en ejecución en la escena.

## Privacidad y seguridad

- **Dónde están los datos:** en la carpeta `.ozmira` de tu directorio personal (`profil.json`, `olaylar.jsonl`, `ayar.json`, `kullanim.json`, `.token`). Para otro lugar, define la variable de entorno `OZMIRA_HOME`. Mientras no compartas esa carpeta, nadie ve tu sistema. La carpeta de la aplicación en sí no contiene datos personales y se puede compartir.
- **Cerrada a la red:** el servidor solo escucha en `localhost`. Otros equipos no pueden conectarse.
- **Clave:** en la primera ejecución se genera una clave aleatoria (`.ozmira/.token`; en macOS y Linux solo tu usuario puede leerla; en Windows los permisos de archivo funcionan de otra forma, así que si compartes el equipo con otras personas, coloca la carpeta en un lugar protegido). Todas las peticiones a la API exigen esta clave; se comprueban las cabeceras Host y Origin.
- **Compartir está desactivado por defecto.** Si lo permites en la ventana Cuenta y con esta copia llegó una dirección de recolección (`geri_bildirim_adresi.txt`), se envía de forma anónima como máximo una vez al día: versión, plan, un ID de instalación aleatorio y contadores (eventos recibidos, veces que se guardó el perfil). No se envían los nombres de tus sistemas ni de tus agentes, las etiquetas de tarea, las rutas de carpetas, las cantidades de tokens ni tu dirección IP. El texto que escribes en la ventana de comentarios también se envía solo con este permiso; lo que escribes se guarda en todos los casos además en la carpeta de datos de este equipo.
- **Otra conexión externa:** la carga de la biblioteca three.js desde jsdelivr. Para desactivarla, mira «Uso sin conexión» más abajo.
- **Protección del navegador:** se usa una Content-Security-Policy estricta; los textos que vienen de archivos, como nombres o descripciones de agentes, se escriben en la página como texto plano, no como HTML.
- **Para restablecer:** detén la aplicación y borra la carpeta `.ozmira`.

## Uso sin conexión

```
python3 ozmira.py vendor
```

Este comando descarga three.js una vez y lo coloca en `web/vendor/three` (con su licencia MIT). Después OZMIRA funciona sin conectarse a internet en absoluto. Si no hay internet, el comando se detiene con un mensaje de error claro y no deja archivos a medias.

## Verlo desde el teléfono (opcional, con cuidado)

```
python3 ozmira.py --lan
```

El servidor pasa a ser accesible desde la misma red y el terminal muestra una dirección para el teléfono. La página no se abre sin la clave `?k=` al final de la dirección. Aun así, cualquiera en la misma red puede alcanzar el servidor; no lo uses en redes Wi-Fi compartidas (cafetería, hotel, escuela). Cuando termines, reinicia de la forma normal.

## Archivos

| Archivo | Función |
| --- | --- |
| `ozmira.py` | Servidor y línea de comandos (inicio, `--uygulama`, olay, vendor, bilgi, kisayol) |
| `ozmira.bat`, `ozmira.command`, `ozmira.sh` | Lanzadores para el navegador |
| `ozmira-uygulama.bat`, `.command`, `.sh` | Lanzadores para la ventana de aplicación |
| `ozmira.ico`, `web/icon.png`, `web/icon.svg` | Iconos |
| `web/index.html`, `web/stil.css` | Página y aspecto |
| `web/app.js` | Panel, asistente de configuración, ventanas Conectar, Cuenta, Comentarios e Idioma |
| `web/i18n.js` | Textos de los cuatro idiomas y traducción de los mensajes de error del servidor |
| `lisans_acik.json` | Clave de verificación de licencias (puede ser pública; la añade el vendedor) |
| `web/sahne.js` | La escena 3D (cinco estilos, de 1 a 24 sistemas) |

`python3 ozmira.py bilgi` muestra la carpeta de datos y el número de eventos.

## Límites

- La escena es una imagen 3D pesada. En tarjetas gráficas antiguas o con el ahorro de batería activado, la calidad baja sola; el brillo puede desactivarse.
- Tope técnico (también en Pro): 24 sistemas, 200 agentes por sistema. Con más de 8 sistemas la escena crece y la cámara se aleja; esta disposición aún no se ha probado visualmente en una tarjeta gráfica real.
- Versión 0.3.0. Diseñada solo para uso local.
- La ventana de aplicación y el acceso directo del escritorio se probaron con pruebas automáticas en Linux; el comando del acceso directo aún no se ha probado en un equipo real con Windows o macOS. Si no funciona, usa `ozmira-uygulama.bat` / `.command`.
- Las traducciones de los cuatro idiomas se prepararon con IA; un hablante nativo de inglés, alemán o español debería revisarlas.
- La escena se probó en un único equipo Windows con tarjeta gráfica real. Si en otras tarjetas la escena no aparece en el primer inicio, comprueba que WebGL esté activado en el navegador.

## Novedades de 0.3.0

- **Panel:** el botón Panel abre el panel de control: tarjetas de resumen, barras de los últimos 7 días, tabla de agentes con búsqueda y orden, lista de eventos y exportación CSV. Iniciar y Finalizar registran un evento a mano; no ejecutan el agente.
- **Guía:** el botón Guía explica la configuración y temas avanzados en diez lecciones cortas.
- **Conexión con Claude Code:** `ozmira_hook.py` envía un evento a OZMIRA cuando Claude Code inicia y termina un subagente. En la sección `hooks` de los ajustes de Claude Code, añade este comando para `PreToolUse` y `PostToolUse` (matcher: `Task|Agent`): `python ozmira_hook.py` (con la ruta completa del archivo). Si el id de tu sistema no es `claude`, define la variable de entorno `OZMIRA_SISTEM`. Probado con entradas simuladas; aún no con un Claude Code real.
- **Flujo:** la pestaña Flujo del panel muestra los tokens de entrada y salida por sistema y agente. Para registrar el reparto: `ozmira.py olay <sistema> <agente> bitti girdi=800 cikti=400` (girdi = entrada, cikti = salida). Si introduces precios, estima el coste; los precios solo se guardan en el navegador.
- **Derechos de autor:** © 2026 OZMIRA, todos los derechos reservados. Consulta `LISANS.txt`.
