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
