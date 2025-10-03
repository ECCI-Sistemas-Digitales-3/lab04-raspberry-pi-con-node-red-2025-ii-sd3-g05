[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=20887887&assignment_repo_type=AssignmentRepo)
# Lab04: Visualización de Datos en Raspberry Pi con Node-RED 

## Integrantes

Michael Yesid Velasquez V.- Cod: 94882 Yeison Gabriel Niño J. - Cod: 61096 Carlos Eduardo Puentes L. - Cod: 89466


## Documentación

<!-- Incluir diagramas y adjuntar al repositorio, en una carpeta src, el flujo que crearon -->

1) ¿Qué hicimos y por qué?

Montamos un flujo en Node-RED (corriendo en la Raspberry Pi) para:

Elegir un color con un color picker.

Mostrar sus valores RGB en la interfaz.

Guardar esos datos en un archivo de texto para que un script en Python los pueda leer.

¿Qué es Node-RED?
Es una herramienta visual para “programar conectando cajitas” (nodos). Cada nodo hace algo (ej. leer un valor, transformar, guardar, etc.). Los datos viajan como un objeto llamado msg (mensaje) con campos como msg.payload (el contenido principal).

2) Cosas que al principio no entendíamos (y ahora sí)

Nodo “color picker”: envía el color en formato HEX tipo #RRGGBB (ej. #ff3300). Para trabajar con números necesitamos convertirlo a RGB (tres enteros de 0 a 255).

msg y msg.payload: todo lo que circula entre nodos va en msg. Lo más importante suele ir en msg.payload. Si un nodo “no funciona”, casi siempre es porque el msg.payload no tiene lo que esperamos.

Nodo function (JavaScript): es una cajita donde podemos escribir código corto (JS) para transformar datos. En este lab lo usamos para pasar de HEX → RGB y preparar texto para guardar.

Nodo file: guarda el msg.payload en un archivo.

Si pones ruta relativa, escribe respecto al directorio donde corre Node-RED (a veces confunde).

Recomendado: usar ruta absoluta, por ejemplo: /home/pi/labs/rgb.txt.

Dashboard (UI): los nodos del “dashboard” crean una página web simple (con el color picker y texto).

Acceso por navegador: http://<IP>:1880/ui.

Servicio y puerto:

Node-RED sirve su editor en http://<IP>:1880.

Si habilitas el servicio: sudo systemctl enable nodered.service, arranca solo al prender la Pi.

Archivo de flujo: se puede exportar en JSON (es el “código fuente” del diagrama). Abres Node-RED → menú (hamburguesa) → Export → Clipboard.

3) Flujo — Explicación de bloques

ui_color_picker: widget para elegir color (HEX).

function HEX→RGB: convierte #RRGGBB a {r,g,b} y además arma un texto como R, G, B para guardarlo.

ui_text: muestra en la UI los números RGB (ej. 255, 51, 0).

file: guarda en un .txt (una línea por registro, con fecha/hora).

Ojo: en el file node define una ruta absoluta válida en tu Pi, por ejemplo: /home/pi/labs/rgb_log.txt.

## 4. Flujo JSON de Node-RED

> Copia y pega este bloque en *Import > Clipboard* de Node-RED.

```json
[
  {
    "id": "tab-rgb-lab",
    "type": "tab",
    "label": "LAB - RGB",
    "disabled": false,
    "info": ""
  },
  {
    "id": "grp-ui",
    "type": "ui_group",
    "z": "",
    "name": "Controles",
    "tab": "tab-ui",
    "order": 1,
    "disp": true,
    "width": "6",
    "collapse": false
  },
  {
    "id": "tab-ui",
    "type": "ui_tab",
    "z": "",
    "name": "Laboratorio",
    "icon": "dashboard",
    "disabled": false,
    "hidden": false
  },
  {
    "id": "color-picker",
    "type": "ui_colour_picker",
    "z": "tab-rgb-lab",
    "name": "Color",
    "label": "Selecciona color",
    "group": "grp-ui",
    "format": "hex",
    "outformat": "string",
    "showSwatch": true,
    "showPicker": true,
    "showValue": true,
    "showHue": true,
    "showAlpha": false,
    "pickerType": "auto",
    "topic": "ui/color",
    "x": 170,
    "y": 120,
    "wires": [["fn-hex2rgb"]]
  },
  {
    "id": "fn-hex2rgb",
    "type": "function",
    "z": "tab-rgb-lab",
    "name": "HEX → RGB + texto",
    "func": "// Espera msg.payload como string tipo \"#RRGGBB\"\nlet hex = String(msg.payload || \"\").trim();\nif (!/^#?[0-9a-fA-F]{6}$/.test(hex)) {\n  node.warn(\"Formato HEX inválido: \" + hex);\n  return null;\n}\nif (hex[0] === '#') hex = hex.slice(1);\nconst r = parseInt(hex.slice(0,2), 16);\nconst g = parseInt(hex.slice(2,4), 16);\nconst b = parseInt(hex.slice(4,6), 16);\n\n// Mostramos RGB en UI\nmsg.payload = `${r}, ${g}, ${b}`;\nmsg.rgb = { r, g, b };\n\n// Para archivo: agregamos timestamp y CSV simple\nconst ts = new Date().toISOString();\nmsg.filePayload = `${ts};${r};${g};${b}`;\nreturn [msg];",
    "outputs": 1,
    "noerr": 0,
    "initialize": "",
    "finalize": "",
    "libs": [],
    "x": 430,
    "y": 120,
    "wires": [["ui-text","to-file"]]
  },
  {
    "id": "ui-text",
    "type": "ui_text",
    "z": "tab-rgb-lab",
    "group": "grp-ui",
    "order": 2,
    "width": "6",
    "height": "1",
    "name": "Valores RGB",
    "label": "RGB",
    "format": "{{msg.payload}}",
    "layout": "row-spread",
    "x": 690,
    "y": 100,
    "wires": []
  },
  {
    "id": "to-file",
    "type": "file",
    "z": "tab-rgb-lab",
    "name": "Guardar en archivo",
    "filename": "/home/pi/labs/rgb_log.txt",
    "appendNewline": true,
    "createDir": true,
    "overwriteFile": "false",
    "encoding": "none",
    "x": 710,
    "y": 160,
    "wires": []
  }
]


## 6) Problemas típicos que vimos y cómo los resolvimos

“No escribe el archivo”

La ruta no existe → crea la carpeta (mkdir -p /home/pi/labs).

Permisos → prueba en tu home (/home/pi/...) o ejecuta Node-RED con tu usuario.

“El UI no aparece”

Te falta instalar los nodos dashboard.

Revisa http://<IP>:1880/ui.

Haz Deploy después de cambios.

“El texto del file node se ve raro”

Asegúrate que msg.payload sea texto. En el flujo que te paso guardo en msg.filePayload. Si usas ese campo, cambia el file node a “Use msg.filePayload” (propiedad Property). En el JSON que te di ya guardo msg.payload como texto “R, G, B”; si quieres el CSV con timestamp, cambia el nodo file para usar msg.filePayload.

“¿HEX a RGB en Node-RED?”

Lo hacemos en el function con JS (ver bloque en el JSON).
## Conclusiones

Aprendimos a manejar Node-RED con nodos de Dashboard y a transformar datos en el function node.

Entendimos la estructura de msg y la importancia de msg.payload.

Probamos la integración con Python leyendo el archivo generado, que sirve para análisis posterior.

Configurar rutas absolutas y permisos fue importante para que el guardado funcione sin errores.