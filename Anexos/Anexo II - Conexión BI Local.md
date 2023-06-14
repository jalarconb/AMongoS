# SETUP DEL CONECTOR BI EN MONGODB (Replica Set Local - Windows)

Para poder realizar una conexión con Power BI es necesario tener algunas herramientas y realizar algunas configuraciones adicionales.


## MONGODB BI CONNECTOR


### Descargar e Instalar MongoDB BI Connector

[MongoDB BI Connector](https://www.mongodb.com/try/download/bi-connector) es una herramienta necesaria para la visualización de datos en herramientas BI en general. Descargar e instalar.

Por defecto, la ruta de instalación será similar a _C:\Program Files\MongoDB\Connector for BI\2.14_.


### Instalar _mongosqld_ como un servicio



**Abrir una CMD Admin (CMD-1). Ir a la ruta de instalación**


Crear el subdirectorio _log_ y el archivo _mongosqld.log_ dentro de este:
```shell
mkdir log
cd log
notepad mongosqld.log
```


Volver a la ruta de instalación y Crear un nuevo archivo de nombre _mongosqld.conf_:
```shell
cd ..
notepad mongosqld.conf
```


Agregar el siguiente texto, guardar y cerrar:
```txt
systemLog:
  path: 'C:\Program Files\MongoDB\Connector for BI\2.14\log\mongosqld.log'
net:
  bindIp: '127.0.0.1'
  port: 3307
```
En caso de ser necesario, cambiar el path de _systemLog_ en la segunda línea.


Ejecutar los siguientes comandos:
```bat
"C:\Program Files\MongoDB\Connector for BI\2.14\bin\mongosqld.exe" install --config "C:\Program Files\MongoDB\Connector for BI\2.14\mongosqld.conf"

net start mongosql
```
En caso de ser necesario, cambiar los paths del primer comando.


### Agregar a Orígenes de datos ODBC

- Buscar un programa en Windows llamado _Orígenes de Datos OBDC_. Ejecutarlo.
- Ir a la pestaña _DSN del Sistema_.
- Dar click en _Agregar..._
- Escoger el origen de tipo _MongoDB ODBC 1.4.3 Unicode Driver_. Dar en _Finalizar_.
- _Data Source Name_ puede ser cualquier cosa. _TCP/IP Server_ es _localhost_ y el puerto será el _3307_. Escoger en _Database_ la base de datos que se quiere usar.
- (Opcional) Usar _Test_ para ver que la conexión funciona.


### Conexión desde Power BI

Una vez creado el origen de datos, al intentar conectar una nueva fuente de datos de tipo _ODBC_ en power BI, debería aparecer este origen en un menú desplegable.

---

Anexo basado en el tutorial de: https://www.youtube.com/watch?v=QN-qw5Eqiew&list=PLLwY9YsWZ6icHVhs8D4j54A5jeMGcYSsH