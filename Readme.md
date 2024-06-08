# Proyecto de Sistema de Gestión de Inventarios con Python

## Introducción

<div align=center>
    <img src="img/sistema-gestion-inventario.png">
</div>

<div align=justify>
En este tutorial, aprenderás a desarrollar un proyecto de **Sistema de Gestión de Inventarios** utilizando el lenguaje de programación Python.

Un sistema de gestión de inventarios es una herramienta que permite realizar un seguimiento y control de los productos o artículos almacenados en un negocio o empresa.

Algunas de las funcionalidades que implementaremos incluyen:

1. **Registro de productos:** Aprenderás a crear una estructura de datos para almacenar la información de los productos, como su nombre, descripción, precio, cantidad disponible, etc. También aprenderás a agregar nuevos productos al sistema.

2. **Búsqueda y filtrado de productos:** Te enseñaré cómo implementar funciones de búsqueda y filtrado para encontrar productos específicos en base a diferentes criterios, como el nombre, la categoría o el precio.

3. **Actualización de inventario:** Aprenderás a manejar las actualizaciones de inventario, como la compra o venta de productos. Implementaremos funciones que permitan aumentar o disminuir la cantidad disponible de un producto y mantener un registro de estas transacciones.

4. **Generación de informes:** Te mostraré cómo generar informes sobre el estado del inventario, como la lista de productos disponibles, los productos más vendidos, los productos con bajo stock, etc. Utilizaremos técnicas de manipulación de datos y generación de informes para presentar esta información de manera clara y concisa.

A medida que avancemos en el tutorial, también aprenderás buenas prácticas de programación, como la modularidad, la reutilización de código y la implementación de pruebas unitarias para garantizar la calidad y robustez de tu sistema de gestión de inventarios.
</div>


## Paso 1: Diseño del Modelo de Datos en WINDOWS

1. Para instalar sqlite3 en Windows necesitas, en primer lugar, dirigirte al sitio web https://www.sqlite.org/download.html ahí te dirigirás a la sección Precompiled Binaries for Windows, en la que encontrarás los binarios para Windows, de ahí tienes que descargar los archivos:

    sqlite-tools
    sqlite-dll

Descargar sqlite-tools del sitio web https://www.sqlite.org/download.html 
    sqldiff.exe
    sqlite3.exe
    sqite3_analyzer.exe



2. Asegúrate de seleccionar los archivos de descarga adecuados, ya que para ambos existen versiones de 32 bits (x86) y 64 bits (x64).

3. Una vez que descargues los archivos y los descomprimas, ya que originalmente vienen como un .zip, tienes que moverlos a la carpeta (c:\sqlite3) para agregarlos 

4. Una vez realizado creamos una variable de entorno en el sistema que apunte a ese directorio.


5. Abre el terminal o powershell crear base de datos > sqlite3 inventario.db

6. dentro de sqlite digitar. .open inventario.db

2. Define la estructura de la **base de datos SQLite**. Agrega el siguiente código:

 -- Crear la tabla 'productos' para almacenar la información de los productos
    CREATE TABLE productos (
        id INTEGER PRIMARY KEY,
        nombre TEXT NOT NULL,
        precio REAL NOT NULL,
        existencias INTEGER NOT NULL
    );

Este código crea una tabla llamada productos con los campos id, nombre, precio y existencias.



## Paso 2: Desarrollo de la Lógica del Negocio

Crea un archivo llamado logica.py para escribir la lógica del negocio.

Define las funciones necesarias para agregar productos, actualizar existencias, registrar ventas y generar informes.

Agrega el siguiente código:

```python
import sqlite3

def conectar():
    # Función para establecer conexión a la base de datos
    conn = sqlite3.connect('inventario.db')
    return conn

def agregar_producto(nombre, precio, existencias):
    # Función para agregar un nuevo producto a la base de datos
    conn = conectar()
    cursor = conn.cursor()
    cursor.execute("INSERT INTO productos (nombre, precio, existencias) VALUES (?, ?, ?)", (nombre, precio, existencias))
    conn.commit()
    conn.close()

def actualizar_existencias(id_producto, nuevas_existencias):
    # Función para actualizar las existencias de un producto en la base de datos
    conn = conectar()
    cursor = conn.cursor()
    cursor.execute("UPDATE productos SET existencias = ? WHERE id = ?", (nuevas_existencias, id_producto))
    conn.commit()
    conn.close()

def registrar_venta(id_producto, cantidad_vendida):
    # Función para registrar una venta y actualizar las existencias en la base de datos
    conn = conectar()
    cursor = conn.cursor()
    cursor.execute("SELECT existencias FROM productos WHERE id = ?", (id_producto,))
    existencias_actuales = cursor.fetchone()[0]
    nuevas_existencias = existencias_actuales - cantidad_vendida
    actualizar_existencias(id_producto, nuevas_existencias)
    conn.close()

def generar_informe():
    # Función para generar un informe de inventario
    conn = conectar()
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM productos")
    productos = cursor.fetchall()
    conn.close()
    return productos
```
## Paso 3: Interfaz de Línea de Comandos (CLI)

Crea un archivo llamado cli.py para desarrollar la interfaz de línea de comandos.

Utiliza la biblioteca argparse para crear una interfaz fácil de usar.

Agrega el siguiente código:

``` python
import argparse
import logica

def agregar_producto(args):
    logica.agregar_producto(args.nombre, args.precio, args.existencias) # Se llama a la función de logica.py
    print("Producto agregado con éxito.")

def actualizar_existencias(args):
    logica.actualizar_existencias(args.id, args.existencias) # Se llama a la función de logica.py
    print("Existencias actualizadas.")

def registrar_venta(args):
    logica.registrar_venta(args.id, args.cantidad) # Se llama a la función de logica.py
    print("Venta registrada.")

def generar_informe(args):
    productos = logica.generar_informe() # Se llama a la función de logica.py
    print("Informe de inventario:")
    for producto in productos:
        print(producto)

def main():
    parser = argparse.ArgumentParser(description='Sistema de Gestión de Inventarios')
    subparsers = parser.add_subparsers(title='Acciones', dest='accion')

    # Comando para agregar un producto
    parser_agregar = subparsers.add_parser('agregar', help='Agregar un nuevo producto')
    parser_agregar.add_argument('nombre', type=str, help='Nombre del producto')
    parser_agregar.add_argument('precio', type=float, help='Precio del producto')
    parser_agregar.add_argument('existencias', type=int, help='Existencias iniciales del producto')
    parser_agregar.set_defaults(func=agregar_producto)

    # Comando para actualizar existencias
    parser_actualizar = subparsers.add_parser('actualizar', help='Actualizar existencias de un producto')
    parser_actualizar.add_argument('id', type=int, help='ID del producto a actualizar')
    parser_actualizar.add_argument('existencias', type=int, help='Nuevas existencias del producto')
    parser_actualizar.set_defaults(func=actualizar_existencias)

    # Comando para registrar una venta
    parser_venta = subparsers.add_parser('venta', help='Registrar una venta')
    parser_venta.add_argument('id', type=int, help='ID del producto vendido')
    parser_venta.add_argument('cantidad', type=int, help='Cantidad vendida del producto')
    parser_venta.set_defaults(func=registrar_venta)

    # Comando para generar un informe
    parser_informe = subparsers.add_parser('informe', help='Generar un informe de inventario')
    parser_informe.set_defaults(func=generar_informe)

    args = parser.parse_args()
    if 'func' in args:
        args.func(args)

if __name__ == '__main__':
    main()
```

## Paso 4: Prueba del Sistema

Utiliza la interfaz de línea de comandos para probar el sistema. A continuación, algunos ejemplos de comandos:

### Agregar un producto:

```bash
python cli.py agregar "Producto1" 10.99 100
```
El comando anterior agrega un producto llamado "Producto1" con un precio de 10.99 y 100 existencias.

### Actualizar existencias de un producto:

```bash
python cli.py actualizar 1 50
```
El comando anterior actualiza las existencias del producto con ID 1 a 50.

### Registrar una venta:

```bash
python cli.py venta 1 20
```
El comando anterior registra una venta de 20 unidades del producto con ID 1.

### Generar un informe de inventario:

```bash
python cli.py informe
```
El comando anterior genera un informe de inventario con todos los productos almacenados en la base de datos.

<div align=center>
¡Y eso es todo! Has desarrollado con éxito un Sistema de Gestión de Inventarios con Python utilizando SQLite y una interfaz de línea de comandos.
</div>

# Paso Extra: Agregar archivos adicionales

Crea un archivo llamado **requirements.txt** y agrega las dependencias necesarias para tu proyecto. En este caso, solo necesitamos la biblioteca sqlite3.

Crea un archivo llamado **.gitignore** para evitar que ciertos archivos y directorios se incluyan en el repositorio de Git.

``` plaintext
# Archivos y directorios generados por Python
__pycache__/
*.pyc

# Archivos de base de datos
*.db
```

Crea un archivo **Readme.md** que proporcione una descripción del proyecto y una guía para su configuración e instalación.
