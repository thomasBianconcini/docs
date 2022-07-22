AlphaVersion
## Kafka

Kafka implements asynchronous call in Jolie.

Kafka location is an address exspressed in this URL: `kafka://localhost:9092?topic=Test1&id=id2test&type=byte`.

* `localhost:9092` is the bootstrap Server so the port through the communication with the Broker take places.

* `topic` is the name of the Topic in the Broker Kafka.

* `id` is the groupId name of the consumer group.

* `type` byte is the default way to send message through Topic, for future implementations can be extendended to other types.

For all the info about Kafka: https://kafka.apache.org/

## Test

To run a microservice with Kafka you need two Docker Container running, one for Kafka and one for Zookeeper.
The tests were made using this https://hub.docker.com/r/bitnami/kafka and this https://hub.docker.com/r/bitnami/zookeeper.

After that is possible to start a Server that offer a service as the following :


```jolie
﻿include "console.iol"

type GreetRequest { name:string }
interface GreeterAPI {
    OneWay: greet( GreetRequest )
}

service Greeter {
    execution: concurrent

    inputPort GreeterInput {
        location: "kafka://localhost:9092?topic=Test1&id=id2test&type=byte"
        protocol: sodep
        interfaces: GreeterAPI
    }

    main {
        [greet( request )]{
            println@Console("hello " + request.name)()
        }
    }
}
```

Than is possible to start a Client that send a request to the Server like : 


```jolie
include "console.iol"

type GreetRequest { name:string }
type GreetResponse { greeting:string }

interface GreeterAPI {
    OneWay: greet( GreetRequest )
}

service Greeter {

    outputPort GreeterInput {
        location: "kafka://localhost:9092?topic=Test1&id=id2test&type=byte"
        protocol: sodep
        interfaces: GreeterAPI
    }

    main {
        request.name = "Thomas"
        greet@GreeterInput( request )
    }
}
```
Only the OneWay are available cause RequestResponse call with Kafka would lead to problems.

To communicate with no-Jolie Service is highly recommended to use Sodep Protocol in case of Output port the Service send a message casted in json in this String `{"method":"greet","sodepAsync":"1.0","id":2,"params":{"name":"mick"},"resorucePath":"\/"}`.

If you have to send a request to a Server jolie from outside you need to write in the Topic the same String format.
