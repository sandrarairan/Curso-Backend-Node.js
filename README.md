# Curso-Backend-Node.js
Curso Platzi Escuela JS
## Backend node escuela js
https://github.com/glrodasz/platzi-backend-node/tree/readable-y-writable-streams/learning-node/streams


Diferencias entre Node.js y JavaScript
En JavaScript del lado del cliente tenemos el DOM y el CSSDOM así como el objeto window para manipular los elementos de nuestra página además una serie de APIs, aquí unos ejemplos:
* 		fetch
* 		SessionStorage y LocalStorage
* 		canvas
* 		bluetooth
* 		audio
* 		web authentication
Mientras que en Node.js no tenemos un DOM ni un objeto windows, lo que sí tenemos son una serie de módulos que nos permiten interactuar con los recursos de la máquina como el sistema operativo y el sistema de archivos, por ejemplo:
* 		os
* 		fs
* 		http
* 		util
* 		debugger
* 		stream
* 		events
https://nodejs.org/api/

https://developer.mozilla.org/en-US/docs/Web/API

# Arquitectura orientada a eventos

https://nodejs.org/api/events.html#events_class_eventemitter

```
const promiseFunction = () =>
  new Promise((resolve, reject) => {
    setTimeout(() => {
      if (Math.random() < 0.5) {
        resolve("hello world");
      } else {
        reject(new Error("hello error"));
      }
    }, 500);
  });

async function asyncAwait() {
  try {
    const msg = await promiseFunction();
    console.log("message", msg);
  } catch (err) {
    console.log("error", err);
  }
}

asyncAwait();
``


```
const EventEmmiter = require("events");

class Logger extends EventEmmiter {
  execute(cb) {
    console.log("Before");
    this.emit("start");
    cb();
    this.emit("finish");
    console.log("After");
  }
}

const logger = new Logger();

logger.on("start", () => console.log("Starting"));
logger.on("finish", () => console.log("Finishing"));
logger.on("finish", () => console.log("It's Done"));

// logger.execute(() => console.log("hello world"));
logger.execute(() => setTimeout(() => console.log('hello world'), 500));
```
# Node.js para la web

https://nodejs.org/api/http.html#http_http_createserver_options_requestlistener

https://nodejs.org/es/docs/guides/anatomy-of-an-http-transaction/

Server.js
``` 
const http = require("http");
const server = http.createServer();

server.on("request", (req, res) => {
  res.statusCode = 200;
  res.setHeader("Content-Type", "text/plain");

  res.end("hello world\n");
});

server.listen(8000);
console.log("Servidor en la url http://localhost:8000");
```

Otro servidor:echo-server.js
```
const http = require("http");
const server = http.createServer();

server.on("request", (req, res) => {
  if (req.method === "POST" && req.url == "/echo") {
    let body = [];

    req
      .on("data", chunk => {
        body.push(chunk);
      })
      .on("end", () => {
        res.writeHead(200, { "Content-Type": "text/plain" });
        body = Buffer.concat(body).toString();
        res.end(body);
      });
  } else {
    res.statusCode = 404;
    res.end();
  }
});

server.listen(8001);
console.log("Servidor en la url http://localhost:8001");
```

# Manejo y uso de Streams con Node.js

Introducción a streams
Los Streams son una colección de datos como los arrays o strings sólo que se van procesando pedazo por pedazo, esta capacidad los hace muy poderosos porque podemos manejar una gran cantidad de datos de manera óptima.
https://nodejs.org/api/stream.html#stream_stream

Big-file.js

```
const fs = require("fs");
const file = fs.createWriteStream("./big");

for (let i = 0; i <= 1e6; i++) {
  file.write(
    "Estaba la pájara pinta sentada en un verde limón. Con el pico cortaba la rama, con la rama cortaba la flor. Ay, ay, ay! Cuándo vendrá mi amor... Me arrodillo a los pies de mi amante, me levanto constante, constante. Dame la mano, dame la otra, dame un besito sobre la boca. Daré la media vuelta,  Daré la vuelta entera, Con un pasito atrás,  Haciendo la reverencia. Pero no, pero no, pero no, porque me da vergüenza, pero sí, pero sí, pero sí, porque te quiero a ti."
  );
}

file.end();
```

File-server.js
```
const fs = require("fs");

const server = require("http").createServer();

server.on("request", (req, res) => {
  fs.readFile("./big", (err, data) => {
    if (err) {
      console.log("error", err);
    }

    res.end(data);
  });
});

server.listen(3000);
```
Curl -I localhost:3000 

Stream-server.js
```
const fs = require("fs");

const server = require("http").createServer();

server.on("request", (req, res) => {
  const src = fs.createReadStream('./big');
  src.pipe(res);
});

server.listen(3000);
```

# Readable y Writable streams
Los Readable y Writeable streams tienen los siguientes eventos y funciones respectivamente:
Readable
Eventos
* 		data. Se dispara cuando recibe datos.
* 		end. Se dispara cuando termina de recibir datos.
* 		error. Se dispara cuando hay un error.
Funciones
* 		pipe
* 		unpipe
* 		read
* 		push
Writeable
Eventos
* 		drain. Se dispara cuando emite datos.
* 		finish. Se dispara cuando termina de emitir.
* 		error. Se dispara cuando hay un error.
Funciones
* 		write
* 		end
Recuerda que tienen estos eventos porque los heredan de la clase EventEmitter.

https://nodejs.org/api/stream.html#stream_writable_streams

Writle-strem.js
```
const { Writable } = require("stream");

const writableStream = new Writable({
  write(chunk, encoding, callback) {
    console.log(chunk.toString());
    callback();
  }
});

process.stdin.pipe(writableStream);
```

Readable-stream.js
```
const { Readable } = require('stream');
const readableStream = new Readable();

readableStream.push(`${0/0} `.repeat(10).concat("Batman, Batman!"));
readableStream.push(null);

readableStream.pipe(process.stdout);

```


Readable-steam-on-demand.js
```
const { Readable } = require('stream');
const readableStream = new Readable({
    read(size) {
        setTimeout(() => {
            if (this.currentCharCode > 90) {
                this.push(null);
                return;
            }

            this.push(String.fromCharCode(this.currentCharCode++));
        }, 100);
    }
});

readableStream.currentCharCode = 65;
readableStream.pipe(process.stdout);
``` 

# Duplex y Transforms streams
Ambos sirven para simplificar nuestro código:
* 		Duplex: implementa los métodos write y read a la vez.
* 		Transform: es similar a Duplex pero con una sintaxis más corta.


Duplex.js
```
const { Duplex } = require("stream");

const duplexStream = new Duplex({
  write(chunk, encoding, callback) {
    console.log(chunk.toString());
    callback();
  },

  read(size) {
    if (this.currentCharCode > 90) {
      this.push(null);
      return;
    }

    this.push(String.fromCharCode(this.currentCharCode++));
  }
});

duplexStream.currentCharCode = 65;
process.stdin.pipe(duplexStream).pipe(process.stdout);
```
Trasform.js

```
const { Duplex } = require("stream");
const { Transform } = require("stream");

const transformStream = new Transform({
  transform(chunk, enconding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback();
  }
});

process.stdin.pipe(transformStream).pipe(process.stdout);
```
# Uso de utilidades de Node.js

# Sistema operativo y sistema de archivos
En esta clase vemos dos módulos básicos:
* 		so. Sirve para consultar y manejar los recursos del sistema operativo.
* 		fs. Sirve para administrar (copiar, crear, borrar etc.) archivos y directorios.
Los métodos contenidos en estos módulos (y en todo Node.js) funcionan de forma asíncrona por default, pero también se pueden ejecutar de forma síncrona, por ejemplo el método readFile() tiene su versión síncrona readFileSync().


https://github.com/glrodasz/platzi-backend-node/tree/administrar-directorios-y-archivos/learning-node/so-archivos

Os.js
```
const os = require('os');

// console.log("CPU info", os.cpus());
// console.log("IP address", os.networkInterfaces().en0.map(i => i.address));
// console.log("Free memory", os.freemem());
// console.log("Type", os.type());
// console.log("SO version", os.release());
// console.log("Usr info", os.userInfo());
```

Sistemas de archivos
Sync.file
```

const fs = require("fs");

try {
  const file = process.argv[2];

  const content = fs.readFileSync(file).toString();

  const lines = content.split("\n").length;
  console.log(lines);
} catch (err) {
  console.log(err);
}
```
>node sync-files.js naranja.txt
16

Async-files.js

```
const fs = require("fs");

const file = process.argv[2];

if (!file) {
  throw new Error("Debes indicar el archivo que quieres leer");
}

const content = fs.readFile(file, function(err, content) {
  if (err) {
    return console.log(err);
  }

  const lines = content.toString().split("\n").length;
  console.log(lines);
});
```

# Administrar directorios y archivos

https://github.com/glrodasz/platzi-backend-node/tree/administrar-directorios-y-archivos/learning-node/so-archivos

readdir.js

```
const fs = require("fs");

const files = fs.readdir(__dirname, (err, files) => {
  if (err) {
    return console.log(err);
  }

  console.log(files);
});
```

Mkdir.js
```
const fs = require("fs");

fs.mkdir("platzi/escuela-js/node", { recursive: true }, err => {
  if (err) {
    return console.log(err);
  }
});
```
Copy.js
```
const fs = require("fs");

fs.copyFile("naranja.txt", "limon.txt", err => {
  if (err) {
    console.log(err);
  }

  console.log("naranja.txt fue copiado como limon.txt");
});
```

# Consola, utilidades y debugging

https://github.com/glrodasz/platzi-backend-node/tree/consola-utilidades-y-debugging/learning-node


https://nodejs.org/api/util.html

https://nodejs.org/api/console.html

https://nodejs.org/api/debugger.html

Console-class.js
```
const fs = require("fs");

const out = fs.createWriteStream("./out.log");
const err = fs.createWriteStream("./err.log");

const consoleFile = new console.Console(out, err);

setInterval(() => {
    consoleFile.log(new Date());
    consoleFile.error(new Error("Ooops!"));
}, 2000);
```

Console-utils.js

```
// %s  string
// %d  numero
// %j  son
// console.log("Un %s y un %s", "perrito", "gatito");

// console.info("hello world");
// console.warn("hello error");

// console.assert(42 == "42");
// console.assert(42 === "42");

// console.trace("hello");

const util = require("util");
const debuglog = util.debuglog("foo");

debuglog("hello from foo");
```

Util-deprecate.js
```
const util = require('util');

const helloPluto = util.deprecate(() => {
    console.log('hello pluto')
}, 'pluto is deprecated. It is not a planet anymore');

helloPluto();
```

Debugging

Node —inspect console-class.js

# Clusters y procesos hijos

Una sola instancia de Node.js corre un solo hilo de ejecución. Para tomar ventaja de los sistemas con multiples core, necesitamos lanzar un cluster de procesos de Node.js para manejar la carga.
El módulo cluster nos permite la creación fácil de procesos hijos que comparten el mismo puerto del servidor. Veamos un ejemplo en código:
const cluster = require("cluster");
const http = require("http");


// Requerimos la cantidad de CPUs que tiene la maquina actual
const numCPUs = require("os").cpus().length;


if (cluster.isMaster) {
  console.log(`Master ${process.pid} is running`);


  // Si el cluster es maestro, creamos tantos procesos como numero de CPUS
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }


  // Si por alguna razón el cluster se finaliza hacemos un log
  cluster.on("exit", (worker, code, signal) => {
    console.log(`worker ${worker.process.pid} died`);
  });
} else {
  // Los diferentes workers pueden compartir la conexión TCP
  // En este caso es una servidor HTTP
  http
    .createServer((req, res) => {
      res.writeHead(200);
      res.end("hello world\n");
    })
    .listen(8000);


  console.log(`Worker ${process.pid} started`);
}
Si corremos nuestro archivo de Node.js ahora compartirá el puerto 8000 con los diferentes workers:
$ node server.js
Master 3596 is running
Worker 4324 started
Worker 4520 started
Worker 6056 started
Worker 5644 started
En Windows, todavía no es posible establecer un nombre de proceso server en un worker.


# Crea tu primer proyecto en Express.js
# Request y Response Objects

```
El objeto req (Request) en Express representa el llamado HTTP y tiene diferentes propiedades del llamado, como la cadena de texto query (Query params), los parámetros de la URL (URL params), el cuerpo (Body), los encabezados (HTTP headers), etc.
Para acceder al req basta con acceder al primer parámetro de nuestros router handlers (router middleware) ó middleware.
Como por ejemplo así lo hemos visto siempre:
app.get("/user/:id", function(req, res) {
  res.send("user " + req.params.id);
});
Pero también funcionaria sin problemas:
app.get("/user/:id", function(request, response) {
  response.send("user " + request.params.id);
});
Exploremos las propiedades más importantes
req.body
Contiene los pares de llave-valor de los datos enviados en el cuerpo (body) del llamado (request). Por defecto es undefined pero es establecido cuando se usa algún “body-parser” middleware como body-parser y multer.
En Postman cuando hacemos un request y enviamos datos en la pestaña Body, estos middlewares son los que nos ayudan a entender el tipo de datos que vamos a recibir en el req.body.
Aquí podemos ver como se pueden usar estos middlwares para establecer el valor del req.body:
const app = require("express")();
const bodyParser = require("body-parser");
const multer = require("multer");
const upload = multer(); // Para datos tipo multipart/form-data

app.use(bodyParser.json()); // Para datos tipo application/json
app.use(bodyParser.urlencoded({ extended: true })); // Para datos tipo application/x-www-form-urlencoded

app.post("/profile", upload.array(), function(req, res, next) {
  console.log(req.body);
  res.json(req.body);
});
Más información sobre los diferentes formatos que puede tener el body: https://developer.mozilla.org/es/docs/Web/HTTP/Methods/POST
req.params
Esta propiedad contiene un objeto con las propiedades equivalentes a los parámetros nombrados en la ruta. Por ejemplo, si tenemos una ruta de la forma /user/:name entonces la propiedad name está disponible como req.params.name y allí podremos ver su valor. Supongamos que llamaramos a la ruta con /user/glrodasz, entonces el valor de req.params.name sería glrodasz. Este objeto por defecto tiene el valor de un objeto vacío {}.
// GET /user/glrodasz
req.params.name;
// => "glrodasz"
req.query
Esta propiedad contiene un objeto con las propiedades equivalentes a las cadenas de texto query de la ruta. Si no hay ninguna cadena de texto query tendrá como valor por defecto un objeto vacío {}.
req.query.q;
// => "tobi ferret"

// GET /shoes?order=desc&shoe[color]=blue&shoe[type]=converse
req.query.order;
// => "desc"

req.query.shoe.color;
// => "blue"

req.query.shoe.type;
// => "converse"
Más información sobre los query strings en: https://es.wikipedia.org/wiki/Query_string y https://tools.ietf.org/html/rfc3986#section-3.4
Response Object
El objeto res representa la respuesta HTTP que envía una aplicación en Express.
Para acceder al res basta con acceder al segundo parámetro de nuestros router handlers(router middleware) o middleware.
Como por ejemplo así lo hemos visto siempre:
app.get("/user/:id", function(req, res) {
  res.send("user " + req.params.id);
});
Pero también funcionaría sin problemas:
app.get("/user/:id", function(request, response) {
  response.send("user " + request.params.id);
});
Exploremos los métodos más comunes
res.end()
Finaliza el proceso de respuesta. Este método viene realmente del core de Node.js, específicamente del método response.end() de http.ServerResponse.
Se usa para finalizar el request rápidamente sin ningún dato. Si necesitas enviar datos se debe usar res.send() y res.json().
res.end();
res.status(404).end();
res.json()
Envía una respuesta JSON. Este método envía una respuesta (con el content-type correcto) y convierte el parámetro enviado a una cadena de texto JSON haciendo uso de JSON.stringify().
El parámetro puede ser cualquier tipo de JSON, incluido un objeto, un arreglo, una cadena de texto, un boolean, número, null y también puede ser usado para convertir otros valores a JSON.
res.json(null);
res.json({ user: "tobi" });
res.status(500).json({ error: "message" });
res.send()
Envía una respuesta HTTP. El parámetro body puede ser un objeto tipo Buffer, una cadena de texto, un objeto, o un arreglo. Por ejemplo:
res.send(Buffer.from("whoop"));
res.send({ some: "json" });
res.send("<p>some html</p>");
res.status(404).send("Sorry, we cannot find that!");
res.status(500).send({ error: "something blew up" });

``` 
# ¿Qué es Express.js y para qué sirve?
Express.js es un framework para crear Web Apps, Web APIs o cualquier tipo de Web Services, es libre bajo la licencia MIT.
Express es muy liviano y minimalista además de ser extensible a través de Middlewares.
Los Middlewares interceptan el request y el response para ejecutar una acción en medio.

http://sinatrarb.com/
