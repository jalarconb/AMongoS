# SETUP INICIAL DE INSTANCIA EN AWS


## Creación de la instancia en AWS Lightsail


Ya estando registrado y logueado en una cuenta de AWS, es necesario dirigirse a [AWS Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home). Allí crearemos una instancia con las siguientes características:


![AWSInstance](../Screenshots/AWSInstance.png)



## Asignación de IP pública estática


Es necesario que la instancia tenga una _IP pública estática_ para faciltiar los procesos de conexión entre los diferentes elementos del sistema. Al entrar en la instancia, seguir este orden de menús y activar la IP estática:


![AWSStaticIP](../Screenshots/AWSStaticIP.png)



## Apertura de puerto


El puerto por defecto de mongo es el _27017_. En el mismo menú en el que asignamos la _IP pública estática_ debemos indicar este puerto explícitamente para que quede abierto:


![AWSOpenPort](../Screenshots/AWSOpenPort.png)



## Instalación de MongoDB


Siguiendo la [documentación oficial](https://www.mongodb.com/docs/v4.4/tutorial/install-mongodb-on-ubuntu/#install-mongodb-community-edition), procedemos a instalar MongoDB:
```sh
sudo apt-get install gnupg

curl -fsSL https://pgp.mongodb.com/server-4.4.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-4.4.gpg --dearmor

echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-4.4.gpg ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list

sudo apt-get update

sudo apt-get install -y mongodb-org
```


## Ejecución de MongoDB

Nuevamente, seguimos la [documentación oficial](https://www.mongodb.com/docs/v4.4/tutorial/install-mongodb-on-ubuntu/#start-mongodb). Inicialmente, es posible que aparezca un error al ejecutar el comando _1_; únicamente en ese caso, ejecutar el comando _2_:
```sh
sudo systemctl start mongod

sudo systemctl daemon-reload
```


Verificamos que el daemon de Mongo esté corriendo:
```sh
sudo systemctl status mongod
```

Si todo está bien, configuramos el daemon para que inicia siempre que el sistema haga reboot:
```sh
sudo systemctl enable mongod
```


## Configuración de credenciales de acceso

### Creación de usuario administrador

Ingresamos al shell de Mongo:
```sh
mongo
```

Seleccionamos la base de datos _admin_
```sh
use admin
```

Creamos un usuario con sus respectivas credenciales (reemplazar _username_ y _password_ por los valores deseados) y le asignamos permisos de adminsitración en todas las bases de datos:
```sh
db.createUser(
  {
    user: "username",
    pwd: "password",
    roles: [{role: "userAdminAnyDatabase", db: "admin"}, "readWriteAnyDatabase"]
  }
)
```

Finalizamos la sesión en el shell
```sh
exit
```


### Habilitación de autenticación en mongo


Ingresamos al archivo _mongod.conf_
```sh
sudo nano /etc/mongod.conf
```


Navegando por el archivo hay que encontrar la etiqueta _"#security:"_ y quitar  el _"#"_. Insertar un salto de línea al final de esta y en la línea siguiente escribir: _"  authorization: "enabled" "_. Nótese que hay dos espacios antes de la palabra authorization; estos son importantes. Debería quedar algo similar a esto:
```conf
security:
  authorization: "enabled"
```

También, buscar la etiqueta _#replication:_ y hacer el mismo proceso. En este caso, _"ReplicaName"_ es el nombre del Replica Set que deseemos. Debe quedar así:
```conf
replication:
  replSetName: "ReplicaName"
```


Luego hay que buscar la etiqueta _"#network interfaces"_. En el atributo _bindIp_ se debe agregar la _IP privada_ de la instancia. Suponiendo que la IP es _"XXX.XXX.XXX.XXX"_, la línea debería quedar así:
```conf
  bindIp: 127.0.0.1, XXX.XXX.XXX.XXX
```
Guardamos y cerramos el archivo.


Reiniciar Mongo
```sh
sudo systemctl restart mongod
```

Instalamos Mongo Shell. Será usado en pasos posteriores
```sh
sudo apt install mongosh
```

**Este proceso de creación de instancia debe realizarse 3 veces para así crear 3 instancias que harán las veces de nodo en nuestro cluster**

---

# SETUP DE REPLICA SET EN AWS


## Revisar conectividad entre instancias

Hay que comprobar que las tres instancias que hemos creado anteriormente puedan comunicarse entre ellas. Para esto, es necesario ejecutar el siguiente comando desde todas las terminales:
```sh
mongosh --host IP --port 27017 -u username -p password
```
_username_ y _password_ se deben reemplazar por los valores asignados al crear el usuario administrador en cada instancia. _IP_ corresponde a la _IP pública estática_ de la instancia objetivo. La idea es que desde la instancia **1** se ejecute el comando con las direcciones IP de las instancias **2** y **3**, y así sucesivamente. En todos los casos debería ser posible acceder al Mongo Shell.



## Configuración del Replica Set

### Agregar el nodo primario

Ingresar a la consola de Mongo en la instancia que queramos que quede de _PRIMARY_ y ejecutamos los comandos _2_ y _3_ en donde _XXX.XXX.XXX.XXX_ es la _IP pública estática_ de la instancia en la que estamos actualmente y _PPPP_ es el puerto que dejamos asignado (_27017_, por defecto):
```sh
mongo

rsconf={_id:"ReplicaName",members:[{_id:0,host:"XXX.XXX.XXX.XXX:PPPP"}]}

rs.initiate(rsconf)
```

Ahora, usamos este comando para ver que el Replica Set ya tiene un miembro:
```sh
rs.status()
```

### Agregar los nodos secundarios

En el mismo nodo que dejamos de primario, ejecutamos esto:
```
rs.add("XXX.XXX.XXX.XXX:PPPP")
```
Aquí, la _IP_ es la _IP privada_ de cada nodo y su respectivo _puerto_.



# Conexión desde MongoDBCompass

Es posible también conectarse a los nodos del cluster a través de MongoDBCompass. Para ello, basta con escribir un string de conexión como el siguiente:
```
mongodb://user:password@XXX.XXX.XXX.XXX:PPPP/
```
Reemplazar así:
- _user_: Usuario creado en el proceso de autenticación.
- _password_: Contraseña creada en el proceso de autenticación.
- _XXX.XXX.XXX.XXX_: IP pública estática del nodo deseado.
- _PPPP_: Puerto abierto del nodo deseado.

---

Anexo basado en tutorial de: https://youtu.be/2GN2w9X7FBQ