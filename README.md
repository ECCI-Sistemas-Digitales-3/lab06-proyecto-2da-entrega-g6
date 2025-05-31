[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=19409290&assignment_repo_type=AssignmentRepo)
# Lab06: Proyecto 2da. entrega

## Integrantes
Laura Hernandez
Sergio Ricardo Leon
Juan Sebastian Alvarez
## Documentación
para la implementacion de la camara se instalo la version de pi-camera 1.3 esto con el objetivo de procesar las inagenes de la camara y reflejarlas en el NODE-red, tambien se utilizo la libreria `UI` cuya funcion es visualizar los bloques en un dashboard y desde la terminal de la raspberry se instalo la libreria `libcamera-still`  a traves del comando
```
sudo apt install-libcamera-still -app
```
y para generar pruebas del funcionamiento de la camara se genero el comando
```
lib-camera -o prueba.jpg
```

Las librerias utilizadas para la elaboracion del dashboard en NODE-red fueron `node-red dashboard` como unica capaz de generar el programa a traves de la IP general que se va a utilizar para el proyecto
 ![imager](
https://github.com/ECCI-Sistemas-Digitales-3/lab06-proyecto-2da-entrega-g6/blob/main/flow.jpg)

## Funcionamiento del codigo de las funciones

En el apartado de node red en la parte de funciones se generaron las instrucciones con el objetivo de crear una variable que permita tomar una foto con  la camara de la raspberry y para ello se implemento un boton que accedera a la libreria lib-camera y que tomara la respectiva imagen y la va a proyectar en el dashboard del proyecto, al accionar este boton se procesa la libreria y aparece un enunciado que indica que esta cargando la foto, despues de reflejarla se implemento una funcion que borra dicho mensaje con el objetivo de visualizar que solamente se ha generado esa funcion.Despues de esto se genera un bloque que permita seleccionar la informacion de la imagen y la procese en base64 y al hacer esto se genera una funcion que permite capturar el color exacto en formato RGB  y CMYK de la imagen tomada para con  ello poder generar el procesamiento de dicho color deseado y enviar esta informacion a traves del protocolo MQTT, esta funcion de procesamiento convierte en un archivo .txt la informacion recibida por parte de la imagen.
## Código del bloque de funcion para mostrar foto
```


<div style="width: 100%; overflow: auto; max-height: 80vh;">
    <canvas id="canvas" style="display: block; width: 100%; height: auto;"></canvas>
</div>
<p id="colorInfo" style="color: white; font-weight: bold;"></p>

<script>
    (function(scope) {
        scope.$watch('msg.payload', function(payload) {
            // Verificar si el payload es una cadena base64 con imagen
            if (typeof payload !== 'string' || !payload.startsWith("data:image")) return;

            const canvas = document.getElementById("canvas");
            const ctx = canvas.getContext("2d");
            const img = new Image();

            img.onload = function () {
                // Calcular la proporción de la imagen
                const aspectRatio = img.width / img.height;

                // Tamaño máximo del canvas (ajustable)
                const maxWidth = 1000;  // Máximo ancho para el canvas
                const maxHeight = 800;  // Máxima altura para el canvas

                // Ajustar tamaño del canvas
                let canvasWidth = img.width;
                let canvasHeight = img.height;

                // Si la imagen es más ancha que el máximo permitido
                if (img.width > maxWidth) {
                    canvasWidth = maxWidth;
                    canvasHeight = maxWidth / aspectRatio;
                }

                // Si la imagen es más alta que el máximo permitido
                if (canvasHeight > maxHeight) {
                    canvasHeight = maxHeight;
                    canvasWidth = maxHeight * aspectRatio;
                }

                // Establecer el tamaño final del canvas
                canvas.width = canvasWidth;
                canvas.height = canvasHeight;

                // Dibujar la imagen en el canvas
                ctx.drawImage(img, 0, 0, canvas.width, canvas.height);

                // Manejar el clic para obtener el color
                canvas.onclick = function(evt) {
                    const rect = canvas.getBoundingClientRect();
                    const scaleX = canvas.width / rect.width;
                    const scaleY = canvas.height / rect.height;

                    const x = Math.floor((evt.clientX - rect.left) * scaleX);
                    const y = Math.floor((evt.clientY - rect.top) * scaleY);

                    const pixel = ctx.getImageData(x, y, 1, 1).data;
                    const rgb = {
                        r: pixel[0],
                        g: pixel[1],
                        b: pixel[2]
                    };

                    document.getElementById("colorInfo").innerText = RGB(${rgb.r}, ${rgb.g}, ${rgb.b});
                    scope.send({ payload: { x, y, color: rgb } });
                };
            };

            img.src = payload; // Establecer la fuente de la imagen (base64)
        });
    })(scope);
</script>
```
## Código del bloque de funcion para capturar el formato RGB

```
 Recibe: msg.payload = { x, y, color: { r, g, b } }
if (msg.payload && msg.payload.color) {
    let r = msg.payload.color.r;
    let g = msg.payload.color.g;
    let b = msg.payload.color.b;

    if (
        Number.isInteger(r) && r >= 0 && r <= 255 &&
        Number.isInteger(g) && g >= 0 && g <= 255 &&
        Number.isInteger(b) && b >= 0 && b <= 255
    ) {
        // Guardar el objeto RGB en el contexto de flujo
        // Puedes usar cualquier nombre para la variable, como 'ultimoColorRGB'
        flow.set('ultimoColorRGB', { r: r, g: g, b: b });

        let hex = #${((1 << 24) | (r << 16) | (g << 8) | b).toString(16).slice(1).toUpperCase()};
        msg.payload = hex; // Asigna el valor hexadecimal directamente a msg.payload
    } else {
        msg.payload = "Valores de color fuera de rango (0-255)";
    }
} else {
    msg.payload = "Mensaje de entrada incorrecto: 'color' no encontrado en el payload.";
}

return msg;
```
## Código del bloque de funcion para transformar las variables RGB a CMYK
```
// Recuperar el último color RGB del contexto de flujo y convertirlo a CMYK,
// luego formatear el payload para el TXT con ambos.

// --- Función de conversión RGB a CMYK ---
function rgbToCmyk(r, g, b) {
    if (r === 0 && g === 0 && b === 0) {
        return { c: 0, m: 0, y: 0, k: 100 }; // Negro puro
    }

    // Normalizar R, G, B a valores de 0 a 1
    r /= 255;
    g /= 255;
    b /= 255;

    // Calcular K (Key/Black)
    let k = 1 - Math.max(r, g, b);

    // Calcular C, M, Y
    let c = (1 - r - k) / (1 - k);
    let m = (1 - g - k) / (1 - k);
    let y = (1 - b - k) / (1 - k);

    // Asegurarse de que los valores no sean NaN o Infinito (pasa si k es 1)
    c = isNaN(c) ? 0 : c;
    m = isNaN(m) ? 0 : m;
    y = isNaN(y) ? 0 : y;

    // Convertir a porcentajes y redondear
    c = Math.round(c * 100);
    m = Math.round(m * 100);
    y = Math.round(y * 100);
    k = Math.round(k * 100);

    return { c: c, m: m, y: y, k: k };
}
// --- Fin Función de conversión RGB a CMYK ---
```
## Código del bloque de funcion para generar el archivo TXT con la informacion completa
```
let ultimoColor = flow.get('ultimoColorRGB');

if (ultimoColor && typeof ultimoColor.r === 'number' && typeof ultimoColor.r === 'number' && typeof ultimoColor.g === 'number' && typeof ultimoColor.b === 'number') {
    let r = ultimoColor.r;
    let g = ultimoColor.g;
    let b = ultimoColor.b;

    if (
        Number.isInteger(r) && r >= 0 && r <= 255 &&
        Number.isInteger(g) && g >= 0 && g <= 255 &&
        Number.isInteger(b) && b >= 0 && b <= 255
    ) {
        // Convertir RGB a CMYK
        let cmyk = rgbToCmyk(r, g, b);

        // Formatear el payload con RGB y CMYK
        msg.payload = RGB: ${r},${g},${b} | CMYK: ${cmyk.c},${cmyk.m},${cmyk.y},${cmyk.k};

        // Si solo quieres CMYK, usa esta línea en su lugar:
        // msg.payload = CMYK: ${cmyk.c},${cmyk.m},${cmyk.y},${cmyk.k};

    } else {
        msg.payload = "Error: El último color RGB guardado está fuera de rango (0-255).";
    }
} else {
    msg.payload = "No se ha seleccionado ningún color RGB aún o el formato es incorrecto.";
}

return msg;
```
## Comandos adicionales
1. en el Exec se genera el comando para que la liberia tome la ejecucion del boton y tome la foto. 
2. En el read que se encuentra posterior al excec se genera la lectura de informacion para ejecutarla en el dashboard
3. El template de mostrar imagen refleja el resultado de la cadena y en el otro template de  cuadro RGB muestra el codigo del color y el color puntual
4. el ultimo bloque corresponde al write que permite escribir toda la informacion a enviar en un .txt

## Visualizacion del dashboard 

![imager](https://github.com/ECCI-Sistemas-Digitales-3/lab06-proyecto-2da-entrega-g6/blob/main/dashboard.jpg)

## Visualizacion de archivo TXT
![imager](https://github.com/ECCI-Sistemas-Digitales-3/lab06-proyecto-2da-entrega-g6/blob/main/imagentxt_obtenido.jpg)