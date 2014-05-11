# Remote services

[![Build Status](https://drone.io/github.com/enyo/remote-services/status.png)](https://drone.io/github.com/enyo/remote-services/latest)

A complete abstraction of client <-> server communication.

It is basically a [remote procedure call](http://en.wikipedia.org/wiki/Remote_procedure_call)
implementation in dart. You can call the methods on your services and get the
result back in futures without having to think about the communication.

## Usage


You can look at the [example repository](https://github.com/enyo/remote-services-example)
for an implementation.

### On the server

Define your services:

```dart
class UserService extends Service {
  
  @Procedure()
  Future<CreateUserResponse> create(Context context, CreateUserRequest request) {
  
  }

}

ServiceDefinitions getServiceDefinitions() {
  return new ServiceDefinitions()
      ..addService(UserService)
      ..addServer(new HttpServiceServer());
}
```

Then in your `/bin/` folder you have a script that calls:

```dart
import "my_services.dart";

getServiceDefinitions().startServers();
```

And in your `build.dart` you set the script to generate the client classes:


```dart
library build;

import 'package:remote_services/builder/builder.dart' as remote_services;

import "lib/service_definitions.dart";


const RS_TARGET = "lib/client_services";

const RS_PROTO_BUFFER_MESSAGES = "lib/proto/messages.dart";

const RS_SERVICES_DIR = "lib/services/";


void main(List<String> args) {

  remote_services.build(getServices(), RS_TARGET, RS_PROTO_BUFFER_MESSAGES, args: args, includePbMessages: true, servicesDirectory: RS_SERVICES_DIR);

}


```

### On the client


You import the generated classes from the server, and use the services like this:


```dart
import "package:remote_services/client/browser_http_client.dart";

main() {
  var client = new HttpServiceClient(Uri.parse("http://localhost:8088"));

  var services = new Services(client);

  AuthenticationRequest req = new AuthenticationRequest()
      ..email = "e@mail.com"
      ..password = "password";

  services.userService.auth(req).then((User user) => doSomething(user));

}
```



## Advanced configuration

### Context initializers

Every *procedure* and *procedure filter* receives a `Context` object that gets
instantiated for every request. If you don't define a `ContextInitializer`
yourself, you will always receive the default `Context` implementation, which
only holds the `ServiceRequest` object.

If you want to have additional information in you context (like session data),
you can define your own context class and provide a `ContextInitializer` to
create that object for you.

> `ContextInitializers` are the first thing called when a request comes in.
> After that all filters are called sequentially, and then your procedure with
> the initialized `Context`.

This is the `typedef` for `ContextInitializer`s:

```dart
typedef Future<Context> ContextInitializer(ServiceRequest req);
```

and here an example implementation:

```dart
/**
 * Your own `Context` class
 */
class MyContext extends Context {

  /// An additional field in your context to hold the session information.
  final Session session;

  MyContext(ServiceRequest req, this.session) : super(req);
}

/**
 * Now define your context initializer
 */
Future<MyContext> myContextInitializer(ServiceRequest req) {
  // This can do anything needed for context initialization. Example:

  // Load session info from the memory cache
  myMemoryCache.loadSession(req.cookies["sessionId"])
      .then((Session session) {
        // And return your context, *with* a session
        return new MyContext(req, session);
      })

}


ServiceDefinitions getServiceDefinitions() {
  // And where you create you service definitions, you now pass the context
  // initializer
  return new ServiceDefinitions(myContextInitializer)
      ..addService(UserService)
      ..etc...
}
```

So, every time you receive a `Context` object, it is now a `MyContext` instance.



### Filters

Often you need your procedures to be filtered, for example if you need authentication.

Filters are defined with the `Procedure` annotation and this is their `typedef`:

```dart
typedef Future<bool> FilterFunction(Context context);
```

You can define filters in your service like this:


```dart
Future<bool> authenticationFilter(Context context) {
  // Make sure the user is authenticated.
  return new Future.value(true);
}

class UserService extends Service {
  
  @Procedure(filters: const[authenticationFilter])
  Future<CreateUserResponse> create(Context context, CreateUserRequest request) {
  
  }

}
```

If a filter returns `false`, the procedure will *not* be called, and the an error
will be sent to the client.

> After the `ContextInitializer` function, all defined filters will be called
> **sequentially and in the defined order** and processing the request is
> immediately stopped when one filter returns `false`.

If you have set a `ContextInitializer` all filter functions will receive the
context returned by this function.

