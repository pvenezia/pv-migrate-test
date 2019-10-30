## AMQP Push Notification Guide 
DTS Connected Radio uses the widely supported, robust, and very popular AMQP protocol for client push notifications.

There are implementations and libraries available for all common languages and frameworks, and we recommend looking through the [RabbitMQ Clients and Developers Reference](https://www.rabbitmq.com/devtools.html), and the [RabbitMQ Client Documentation](https://www.rabbitmq.com/clients.html).

For C++, we recommend the [amqpcpp libraries](https://github.com/akalend/amqpcpp).

### Client Push Notification Subscription Process

All DTS Connected Radio clients should follow this process to properly receive and use push notifications for live data.

  1. Retrieve `/broadcasts` response with arrays of broadcast objects.
  2. Determine which broadcast objects have live data available by inspecting `hasLiveData` values.
  3. Request current live data for the desired or currently playing broadcasts via the `/broadcasts/<broadcastId>/live` API path.
  4. Subscribe to push notifications for those broadcasts using AMQP **without declaring an exchange, and by creating a single server-named exclusive queue, without acknowledgements**.
  5. Update the display as needed when notifications are received.

### Connecting to the AMQP Services

Each client should use the information distributed by the DTS Connected Radio RESTful API to connect to the AMQP service. This eliminates the need to have static hostnames, URIs or authentication information coded into the client.

When a broadcast response is retrieved, it will contain one or more broadcast objects. If the object contains a key named `hasLiveData`, and the value is `true`, then that `broadcastId` has live data available for display and for push notification subscription. 

If the client then wishes to display that live data, or subscribe to the push notifications for that `broadcastId`, it should first request the current live data object by making a live data API call for that `broadcastId`:

```text
  GET /v1/broadcasts/<broadcastId>/live?lat={lat}&lng={lng}&disp={disp}
```

The live data object will be returned, and can be used to display the current live data for that broadcast immediately. 

Following this, the client can subscribe to push notifications using the connection information found in the broadcast record for that `broadcastId`.

The broadcast data object will have several other keys present for use in AMQP service connection, and will look something like this:

```json
{...
  "hasLiveData": true,
  "mqExchange": "live",
  "mqKey": "#.b.84246",
  "mqURI": "amqp://username:password@mq.cnrd.io/",
...}
```

The other three keys and values in that broadcast record will allow the client to build a connection string for any AMQP client library. If a full URI is supported, then it might look like this:

```text
  amqp://username:password@mq.cnrd.io/%2f
``` 

`%2f` denotes the default virthost and may or may not be required by the library. **If the URI received does not have a virthost specifed, and does not have `%2f` specifed, then your client should use it if your library requires it**. If the `mqURI` includes the `%2f` string, then you should use it as it is.  

If the `mqURI` specifies a virthost, it will be specified in the URI after the hostname (e.g. `amqp://username:password@hostname.domain/virthost`), and `%2f` should then not be used. 

AMQP URI construction is pursuant to the [RabbitMQ AMQP URI Guidelines](https://www.rabbitmq.com/uri-spec.html).

Note that the use of SSL and associated `amqps://` scheme may be implemented at any time, so all clients must be able to use SSL AMQP connections.

Once the connection to the server is made, the client should create an **server-named exclusive queue without acknowledgements**, and bind to the exchange and routing keys linked to the desired broadcasts for push notifications. In the example above, that would be 
```json
{...
  "mqKey": "#.b.84246",
  "mqExchange": "live",
...}
```
so the client would bind to the routing key `#.b.84246` on the `live` exchange. Clients can bind to multiple routing keys on the same queue, and should only use a single queue. [Further information on AMQP queues is available here](https://www.rabbitmq.com/queues.html).

Once connected, the client will begin receiving push notifications. In order to conserve client bandwidth, these notifications are very small, and are in the form of `broadcastId,liveId`. For the example above, that might be 
```text
84246,abcde123456
```

When this message is received, the client knows there is new data available for `broadcastId 84246` and can then make an API call to `/v1/broadcasts/84246/live` to retrieve the live data object for `broadcastId 84246`, and use the data in the object to display the live data on the screen.

## Connection Details and Heartbeats

Since the clients will be using cellular data networks, they must be resilient when dealing with poor network connectivity. 

All AMQP clients must be configured to accept and use heartbeats to ensure they can still receive notifications from the server, and must reconnect immediately whenever they detect a dead connection, and retrieve the live data for the currently tuned broadcast in case they missed a notification of a program change.

Further reading on this critical function can be found in the [RabbitMQ Heartbeat documentation](https://www.rabbitmq.com/heartbeats.html). Note that of this writing, the RabbitMQ server version in use is `3.6.15`.

## Code Examples
#### Sample C++ Consumer

```python
#include <iostream>
#include <string>
#include "AMQPcpp.h"

int onCancel(AMQPMessage * message ) {
    std::cout << "cancel tag="<< message->getDeliveryTag() << std::endl;
    return 0;
}

int onMessage( AMQPMessage * message  ) {
    try {
        uint32_t j = 0;
        /* Message format: "broadcastId,liveId". e.g. "33503,108" */
        char* data = message->getMessage(reinterpret_cast<uint32_t*>(&j));
        std::string str_data(data);

        if (str_data.size()) {
            std::cout << "[onmessage] " << str_data << std::endl;

            /* Parse broadcast and live Ids */
            std::string broadcastId, liveId;
            int pos = str_data.find(",");
            if(pos > 0){
                broadcastId = str_data.substr(0,pos);
                liveId = str_data.substr(pos+1, j - broadcastId.length() - 1);
            }

            std::cout << "\t broadcastId = " << broadcastId << "\t";
            std::cout << "liveId = " << liveId << std::endl;
        }

    } catch (AMQPException e) {
        std::cout << "[ERROR] " << e.getMessage();
        return -1;
    }
    return 0;
};

int main () {
    try {
        /* Sample stations from New York market */
        std::string broadcastIds[] = {"#.b.58346","#.b.110194",     \
                                      "#.b.110195","#.b.105344",    \
                                      "#.b.94187", "#.b.97059",     \
                                      "#.b.95462", "#.b.103342",    \
                                      "#.b.103634", "#.b.109914",   \
                                      "#.b.110188", "#.b.102241",   \
                                      "#.b.97109", "#.b.103286",    \
                                      "#.b.57390", "#.b.58703",     \
                                      "#.b.57400", "#.b.59694",     \
                                      "#.b.33503", "#.b.37819",     \
                                      "#.b.57388", "#.b.57394"      \
                                     };

        /* Use the username/password from the mqURI attribute */
        AMQP amqp("<username>:<password>@mq.cnrd.io", 60);   // 60 seconds heartbeat timeout
        AMQPQueue * q = amqp.createQueue();

        /* Declare an exclusive queue */
        q->Declare("", AMQP_EXCLUSIVE);

        /* Bind/Subscribe to the broadcasts to receive AMQP messages for */
        for(int i = 0; i < 22; i++){
            /* Use the mqExchange value here */
            q->Bind("<mqExchange>", broadcastIds[i]);
            q->setConsumerTag(broadcastIds[i]);
            std::cout << "Binding/Subscribing to... " << broadcastIds[i] << std::endl;
        }

        /* Callback function to call when AMQP message is received */
        q->addEvent(AMQP_MESSAGE, onMessage );

        /* Callback function to call if the broadcast is cancelled */
        q->addEvent(AMQP_CANCEL, onCancel );

        std::cout<< "Waiting for AMQP messages..." << std::endl;
        q->Consume(AMQP_NOACK);

        std::cout << "Shouldn't reach here..." << std::endl;
    } catch (AMQPException e) {
        std::cout << e.getMessage() << std::endl;
    }

    return 0;
}
```
#### Sample Python consumer
```python
#!/usr/bin/env python

import pika
import json
import urllib2
import pprint

pp = pprint.PrettyPrinter(indent=4, width=80)

## Connect to AMQP service with the URI
parameters = pika.URLParameters('amqp://username:password@mq.cnrd.io/%2f')
connection = pika.BlockingConnection(parameters)

channel = connection.channel()

## Declare our exclusive queue with server-assigned name
result = channel.queue_declare(exclusive=True)
queue_name = result.method.queue

## Set mock coordinates and `disp` type

query_params = [ 'lat=43.094',
                 'lng=-71.730',
                 'disp=guide'
               ]

## Define the broadcastIds and routing keys we want to bind to for
## push notifications.
## These would be pulled from their respective objects in a
## /broadcasts reponse.

rkeys = { '87452': '#.b.87452',
        '86917': '#.b.86917',
        '87262': '#.b.87262',
        '34656': '#.b.34656',
        '57111': '#.b.57111',
        '32044': '#.b.32044',
        '12050': '#.b.12050',
        '36249': '#.b.36249',
        '36773': '#.b.36773',
        '32008': '#.b.32008',
        '124271': '#.b.124271'
        }
subkeys = []

## Function called from the callback when a notification is received.
## This requests the live data from the CONRAD API for that broadcastId.
## This uses standard CONRAD authorization headers and required params.

def callLivePath(bid):
    url = 'https://api.sandbox.cnrd.io/v1/broadcasts/' + bid + '/live' + '?' + "&".join(query_params)
    headers = { 'mfgid' : 'mymfgId',
                'Authorization' : 'myDeviceId,myDeviceHash'
                 }
    print url
    try:
        req = urllib2.Request(url, headers=headers)
        resp = urllib2.urlopen(req)
        content = resp.read()
        ## Print out the JSON object for reference
        pp.pprint(json.loads(content))

    except Exception as error:
        print ("Could not call live path for " + bid + ": " + repr(error))

## Bind the queue for each routing key we care about. The `live` exchange
## was defined in the broadcast record. Note that we run `callLivePath`
## Here prior to binding to the routing key on the exchange.

for rk in rkeys:
    try:
        callLivePath(rk)
        channel.queue_bind(exchange='live', routing_key=rkeys[rk], queue=queue_name)
        subkeys.append(rk)
    except:
        print('error binding to AMQP')

print ("Subscribed to " + ", ".join(subkeys))

print(' [*] Waiting for notifications. To exit press CTRL+C')

def callback(ch, method, properties, body):
    print("\n [x] %r" % body)
    callLivePath(body.split(',')[0])

## Define necessary parameters, `no_ack=True` is required.
channel.basic_consume(callback,
                      queue=queue_name,
                      no_ack=True)

## Begin consuming notifications.
channel.start_consuming()
```

