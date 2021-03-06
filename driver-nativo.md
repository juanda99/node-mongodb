# Acceso a MongoDB mediante driver nativo



## Objetivos
- Saber realizar operaciones básicas
  - CRUD
- Familiarizarnos con la documentación y saber usar la API del driver
- Trabajar de forma correcta con el modelo asíncrono de node.js



## Referencias
- [Web MongoDB Node.JS driver](http://mongodb.github.io/node-mongodb-native/)
- [Manual referencia versión 2.2](http://mongodb.github.io/node-mongodb-native/2.2/)  




## Crear el proyecto
- Creamos la carpeta para nuestro proyecto llamada mongodbDriver
- Inicializamos nuestro proyecto mediante el comando **npm init**
- Instalamos nuestras dependencias (mongodb y eslint)

``` 
mkdir mongodbDriver
cd mongodbDriver
npm init
npm install mongodb --save
npm install eslint --save-dev
```



## Configurar eslint

```
./node_modules/.bin/eslint --init
? How would you like to configure ESLint? Answer questions about your style
? Are you using ECMAScript 6 features? Yes
? Are you using ES6 modules? Yes
? Where will your code run? Node
? Do you use JSX? No
? What style of indentation do you use? Spaces
? What quotes do you use for strings? Single
? What line endings do you use? Windows
? Do you require semicolons? No
? What format do you want your config file to be in? JSON
Successfully created .eslintrc.json file in ....
```



## Crear conexión
- Creamos un fichero de conexión (app.js)
- [Modificamos la url si fuera necesario](http://mongodb.github.io/node-mongodb-native/2.2/tutorials/connect/)

```
var MongoClient = require('mongodb').MongoClient

// Connection URL
var url = 'mongodb://localhost:27017/test'

// Use connect method to connect to the server
MongoClient.connect(url, function(err, db) {
  if (err) console.log(err.message)
  else {
    console.log('Connected successfully to server')
    db.close()
  }
})
```



## Probar conexión

- Ejecutamos el fichero anterior mediante node:

  ```
  node app.js
  ```

- ¿Qué mensaje nos muestra la consola?




## Creamos una colección con validación:
- [Siguiendo la documentación oficial](http://mongodb.github.io/node-mongodb-native/2.2/api/Db.html#createCollection), crearemos una colección con validación:

```
var createValidated = function (db, callback) {
  db.createCollection("contactos",
    {
      'validator': {
        '$or':
        [
          { 'telefono': { '$type': "string" } },
          { 'email': { '$regex': /@mongodb\.com$/ } },
          { 'estadoCivil': { '$in': ["Soltero", "Casado"] } }
        ],
        nombre: { $type: "string" },
        edad: { $type: "int", $gte: 18 }
      }
    },
    function (err, results) {
      if (err) console.log(err)
      else console.log("Colección creada")
      callback(err, results)
    }
  )
}
```



- ¿Sabrías integrar esto en el código del fichero app.js?
- Recuerda que node.js 
    - funciona de modo asíncrono
    - hay que utilizar las funciones de callback
    - ¡ojo donde cierras la base de datos, se puede quedar la colección sin hacer!




```
var MongoClient = require('mongodb').MongoClient
// Connection URL
var url = 'mongodb://localhost:27017/test'
// Use connect method to connect to the server
MongoClient.connect(url, function (err, db) {
  if (err) console.log(err.message)
  else {
    console.log('Connected successfully to server')
    
    createValidated(db, function(){
      db.close()
    })
  }
})

var createValidated = function (db, callback) {
  db.createCollection("contactos",
    {
      'validator': {
        '$or':
        [
          { 'telefono': { '$type': "string" } },
          { 'email': { '$regex': /@mongodb\.com$/ } },
          { 'estadoCivil': { '$in': ["Soltero", "Casado"] } }
        ],
        nombre: { $type: "string" },
        edad: { $type: "int", $gte: 18 }
      }
    },
    function (err, results) {
      if (err) console.log(err)
      else console.log("Colección creada")
      callback(err, results)
    }
  )
}

```



## Comprobamos que la colección se crea

- Accediendo a Robo3T y navegando hasta la colección
- Vemos sus condiciones de validación de registros:
  ```
  db.getCollectionInfos()
  ```



## Datos para la colección

- Queremos que a la vez que se crea la colección, se inserten unos registros de ejemplo.

- Datos de ejemplo:
```
const contactos = [
  {
    nombre: 'pepe',
    edad: 20,
    estadoCivil: 'Soltero',
    telefono: '444444444'
  },
  {
    nombre: 'juan',
    edad: 40,
    estadoCivil: 'Casado',
    email: 'juan@midominio.com',
  },
  {
    nombre: 'marta',
    edad: 30,
    estadoCivil: 'Soltero',
    telefono: '999999999'
  }
]
```


## Función para insertar contactos

```
var insertarContactos = function (db, contactos, callback) {
  // Obtenemos la colección
  var collection = db.collection('contactos')
  // Insertamos los documentos
  collection.insertMany(contactos, function (err, result) {
    console.log('Insertados los contactos')
    callback(result)
  })
}
```

- Intenta integrarlo y prueba que funcione. 



## Código con la solución
  
  ```
  var MongoClient = require('mongodb').MongoClient
  // Connection URL
  var url = 'mongodb://localhost:27017/test'
  // Use connect method to connect to the server
  MongoClient.connect(url, function (err, db) {
    if (err) console.log(err.message)
    else {
      console.log('Connected successfully to server')
      createValidated(db, function () {
        insertarContactos(db, contactos, function () {
          db.close()
        })
      })
    }
  })
  
  var createValidated = function (db, callback) {
    db.createCollection("contactos",
      {
        'validator': {
          '$or':
          [
            { 'telefono': { '$type': "string" } },
            { 'email': { '$regex': /@midominio\.com$/ } }
          ],
          nombre: { $type: "string" },
          edad: { $type: "int", $gte: 18 },
          estadoCivil: { '$in': ["Soltero", "Casado"] }
        }
      },
      function (err, results) {
        if (err) console.log(err)
        else console.log("Colección creada")
        callback(err, results)
      }
    )
  }
  
  const contactos = [
    {
      nombre: 'pepe',
      edad: 20,
      estadoCivil: 'Soltero',
      telefono: '444444444'
    },
    {
      nombre: 'juan',
      edad: 40,
      estadoCivil: 'Casado',
      email: 'juan@midominio.com',
    },
    {
      nombre: 'marta',
      edad: 30,
      estadoCivil: 'Soltero',
      telefono: '999999999'
    }
  ]
  
  var insertarContactos = function (db, contactos, callback) {
    // Obtenemos la colección
    var collection = db.collection('contactos')
    // Insertamos los documentos
    collection.insertMany(contactos, function (err, result) {
      console.log('Insertados los contactos')
      callback(result)
    })
  }
  ```



## Búsqueda de registros

- Se utiliza el método find del objeto collection:

```
var encontrarContactosPorNombre = function (db, nombre, callback) {
  var collection = db.collection('contactos')
  collection.find({ 'nombre': nombre }, { 'nombre': 1, 'edad': 1 }).toArray(function (err, docs) {
    if (err) console.log(err)
    else {
      console.log("Encontrados los siguientes contactos")
      console.log(docs)
      callback(null, docs)
    }
  })
}
```



## Borrar registros
- Se deja como ejercicio, utilizando el [Manual referencia](http://mongodb.github.io/node-mongodb-native/2.2/)  




## Estructura acceso a base de datos en una aplicación
- Para las operaciones en la base de datos necesitamos el objeto db.
- El método [connect](http://mongodb.github.io/node-mongodb-native/2.2/api/MongoClient.html#connect) no es [singleton](https://es.wikipedia.org/wiki/Singleton)
- La conexión a base de datos es un proceso costoso (asíncrono) y lo suyo sería hacerlo una sola vez al arrancar el sistema, no por cada operación sobre la base de datos. 
- Si no utilizamos el parámetro de callback, el método connect devuelve una promesa.




## Módulo de conexión a base de datos

```
// bbdd.js
var MongoClient = require('mongodb').MongoClient;
let connection = null;

module.exports.connect = () => new Promise((resolve, reject) => {
    MongoClient.connect(url, option, function(err, db) {
        if (err) { reject(err); return; };
        resolve(db);
        connection = db;
    });
});

module.exports.get = () => {
    if(!connection) {
        throw new Error('Call connect first!');
    }
    return connection;
}
```



- La promesa la construimos "a mano"
- Si el método connect no tiene callback devuelve directamente una promesa, así queda más corto:
```
module.exports.connect = () => MongoClient.connect(url, option)
```



## Estructura aplicación
- Cargamos el módulo de base de datos y si todo va bien, arrancamos el resto de la aplicación

```
  
// sería nuestro fichero index.js o app.js
  const db = require('./bbdd');
  db.connect()
      .then(() => console.log('database connected'))
      .then(() => bootMyApplication())
      .catch((e) => {
          console.error(e);
          // Always hard exit on a database connection error
          process.exit(1);
      });
 ```



- Cualquier módulo que acceda a la base de datos:

  ```
  const db = require('./bbdd');
  db.get().find(...)...
  ```



## Problemática

- Nuestros módulos que acceden a base de datos, tienen la dependencia del módulo bbdd.
  - Esto hace más complejos los tests (hacer fakes)
  - El código está más enmarañado y se hace más dificil de mantener.




## Patrón de dependency injection
- *En vez de crear la dependencia en mi módulo o llamar a algún objeto para obtener mi dependencia, las ponemos como algo externo (parámetro) y el problema se va del módulo.*

- También podemos guardar toda la configuración de nuestra app (de momento solo la bbdd en un fichero específico.



## Estructura final
- Fichero de configuración *config.js*:

```
const app = {
  url: 'mongodb://localhost:27017/test',
  options: {}
}
module.exports = app
```



- Módulo de conexión a la base de datos:

```
var MongoClient = require('mongodb').MongoClient
const bbdd = require('./config')
module.exports.connect = () =>  MongoClient.connect(bbdd.url, bbdd.options) 
```



- Módulo principal (main):

```
const db = require('./bbdd')
db.connect()
  .then(() => console.log('Conectado a base de datos'))
  // paramos la dependencia conmo parámetro 
  // .then(() => appCode(db)) 
  .catch((e) => {
    console.log('Error al conectar con la base de datos')
    //console.error(e)
    // Always hard exit on a database connection error
    process.exit(1)
  })
```



## ¿Usamos async/await?
- En la documentación hay ejemplos con:
  - Promesas
  - Generadores y el módulo co
- No hay ejemplos con async/await (ES8)



## Problemática
- [Falta compatibilidad con node](http://node.green/)
- Para que nos funcione deberemos utilizar babel-node, o en producción compilarlo.
  ```
  npm i -D babel-cli
  ```
- Además necesitamos instalar los plugins que hacen las traducciones de código "nuevo" a código "viejo". A veces es conveniente instalar un preset (colección de plugins)

```
npm install --save-dev babel-plugin-transform-async-to-generator
npm i -D babel-preset-es2015
npm i -D babel-polyfill
```



- Una vez instalados los módulos configuramos babel para que los use, creando un fichero .babelrc con el siguiente código:

```
{
  "presets": ["es2015"],
  "plugins": ["transform-async-to-generator"]
}
```



## Resultado final con JavaScript 2017

- Fichero app.js

```

import db from './bbdd'
import 'babel-polyfill'

(async () => {
  try {
    await db.connect
    console.log('Conectado a base de datos')
  } catch (error) {
    console.log('Error al conectar con la base de datos')
    console.error(error)
    // salida forzada con código de error
    process.exit(1)
  }
})()
```



- Fichero bbdd.js

```
import {MongoClient} from 'mongodb'
import bbdd from './config.js'
const connect = () =>  MongoClient.connect(bbdd.url, bbdd.options) 
export default connect
```



## Ejecución

- Si queremos ejecutarlo habrá que utilizar babel-node:

```
node_modules/.bin/babel-node app.js
```



## Compilación

- Otra opción sería compilarlo. ç
- Introducimos los scripts en el fichero package.json (así nos ahorramos el PATH):

```
  "scripts": {
    "start": "babel-node app.js",
    "build": "babel *.js -d dist"
  },
```

- Ejecución mediante *npm run start* o *npm run build*.
- Otra opción utilizar una versión de node.js más actual (no LTS)

