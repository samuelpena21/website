---
title: Trabajando con WebSockets
prev:
  title: Analizando un JSON en segundo plano
  path: /docs/cookbook/networking/background-parsing
next:
  title: Persistir datos con SQLite
  path: /docs/cookbook/persistence/sqlite
---

Además de las peticiones normales HTTP, puedes conectar a servidores usando 
WebSockets. WebSockets permite una comunicación de dos vías con el servidor sin 
["polling"](https://es.wikipedia.org/wiki/Polling).

En este ejemplo, conectarás a un [servidor de pruebas proporcionado por 
websocket.org](http://www.websocket.org/echo.html). Este servidor simplemente nos devuelve 
el mismo mensaje que le envies.

## Instrucciones

  1. Conecta a un servidor de WebSocket 
  2. Escucha mensajes desde el servidor 
  3. Envía datos al servidor
  4. Cierra la conexión al WebSocket
  
## 1. Conecta a un servidor de WebSocket 

El paquete [web_socket_channel]({{site.pub-pkg}}/web_socket_channel) 
proporciona las herramientas que necesitarás para conectarnos al servidor WebSocket.

El paquete proporciona un `WebSocketChannel` que te permite tanto escuchar mensajes 
desde el servidor como enviar mensajes al servidor. 

En Flutter, crea un `WebSocketChannel` que se conecta al servidor en una 
línea:

<!-- skip -->
```dart
final channel = IOWebSocketChannel.connect('ws://echo.websocket.org');
```

## 2. Escucha mensajes desde el servidor

Ahora que has establecido una conexión, puedes escuchar mensajes desde el 
servidor.

Después que mandes un mensaje al servidor de pruebas, este envia el mensaje de vuelta. 

¿Cómo escuchar por mensajes y mostrarlos? En este ejemplo, usaremos 
un widget [`StreamBuilder`]({{site.api}}/flutter/widgets/StreamBuilder-class.html) 
para escuchar nuevos mensajes y un widget 
[`Text`]({{site.api}}/flutter/widgets/Text-class.html) 
para mostrarlo.

<!-- skip -->
```dart
StreamBuilder(
  stream: widget.channel.stream,
  builder: (context, snapshot) {
    return Text(snapshot.hasData ? '${snapshot.data}' : '');
  },
);
```

### ¿Cómo funciona esto?

El `WebSocketChannel` proporciona un [`Stream`]({{site.api}}/flutter/dart-async/Stream-class.html) 
de mensajes desde el servidor.

La clase `Stream` es una parte fundamental del paquete `dart:async`. Este 
proporciona una manera de escuchar eventos asíncronos desde 
una fuente de datos. Al contrario que `Future`, el cual devuelve una única respuesta 
asíncrona, la clase `Stream` puede entregar muchos eventos a 
lo largo del tiempo. 

El widget [`StreamBuilder`]({{site.api}}/flutter/widgets/StreamBuilder-class.html) 
conecta a un `Stream` y pide a Flutter hacer rebuild cada vez que 
recibe un evento usando la función `builder` proporcionada. 

## 3. Envía datos al servidor

Para enviar datos al servidor, agrega mensajes con el método `add` al receptor `sink` proporcionada 
por el `WebSocketChannel`.

<!-- skip -->
```dart
channel.sink.add('Hello!');
```

### ¿Cómo funciona esto?

El `WebSocketChannel` proporciona una clase 
[`StreamSink`]({{site.api}}/flutter/dart-async/StreamSink-class.html) 
para enviar (_push_) mensajes al servidor. 

La clase `StreamSink` proporciona una manera general para añadir eventos síncronos o asíncronos a una
fuente de datos.

## 4. Cierra la conexión al WebSocket

Después de que hayas usado el WebSocket, cierra la conexión. 
Para hacerlo, cierra el `sink`.

<!-- skip -->
```dart
channel.sink.close();
```

## Ejemplo completo

```dart
import 'package:flutter/foundation.dart';
import 'package:web_socket_channel/io.dart';
import 'package:flutter/material.dart';
import 'package:web_socket_channel/web_socket_channel.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final title = 'WebSocket Demo';
    return MaterialApp(
      title: title,
      home: MyHomePage(
        title: title,
        channel: IOWebSocketChannel.connect('ws://echo.websocket.org'),
      ),
    );
  }
}

class MyHomePage extends StatefulWidget {
  final String title;
  final WebSocketChannel channel;

  MyHomePage({Key key, @required this.title, @required this.channel})
      : super(key: key);

  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  TextEditingController _controller = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Padding(
        padding: const EdgeInsets.all(20.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: <Widget>[
            Form(
              child: TextFormField(
                controller: _controller,
                decoration: InputDecoration(labelText: 'Send a message'),
              ),
            ),
            StreamBuilder(
              stream: widget.channel.stream,
              builder: (context, snapshot) {
                return Padding(
                  padding: const EdgeInsets.symmetric(vertical: 24.0),
                  child: Text(snapshot.hasData ? '${snapshot.data}' : ''),
                );
              },
            )
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _sendMessage,
        tooltip: 'Send message',
        child: Icon(Icons.send),
      ), // Esta coma final hace al auto-formateo más agradable a los métodos de compilación.
    );
  }

  void _sendMessage() {
    if (_controller.text.isNotEmpty) {
      widget.channel.sink.add(_controller.text);
    }
  }

  @override
  void dispose() {
    widget.channel.sink.close();
    super.dispose();
  }
}
```

![Web Sockets Demo](/images/cookbook/web-sockets.gif){:.site-mobile-screenshot}

