
# WebSocket Actor

Authors: Andrea Vaccaro - David Grollier

# Description

A library implementing a high-level actor-based WebSocket communication. An actor can subscribe to a WebSocket Actor (either server or client) to facilitate WebSocket-based communication.

Please note that within this library **resource names** as defined in
the WebSocket protocol specification are referred to as **services.**

# Library

<img src="./media/image1.png" style="width:3.55864in;height:3.54197in"
alt="A screenshot of a computer program Description automatically generated" />

# Actors

## WebSocket Client.lvclass

It forwards packets received from the connected WebSocket Service to its
subscriber as "WebSocket Connection Handler Msg" messages. It allows its
subscriber to send data to a specified service to which the WebSocket
client is connected ("Send to Peer Msg" message" message of the parent
"WebSocket Connection" class). Before starting the actor, the subscriber
must provide, by means of suitable properties, a data handler message in
form of a concrete implementation of the "WebSocket Connection Handler
Msg" abstract message used to handle the connection events corresponding
to the WebSocket communication, a concrete implementation of the
"WebSocket Server-Service Enqueuer Msg" abstract message used to receive
the WebSocket service actor enqueuer, and the WebSocket service URI to
which the client should connect.

## WebSocket Server Manager.lvclass

This actor's role is solely that of launching and managing the stop of
"WebSocket Server" actors listening on at a specific port and interface
address ("Launch Server Msg" and "Stop Server Msg" messages). The actor
will ensure that the launched "WebSocket Server" doesn’t conflict on
interface and port resources. If the server is already running at that
port or on the specified interface the method sends just the server
enqueuer to the client. If the server is not running it will be
launched.

## WebSocket Server.lvclass

Defines a WebSocket Server characterized by an interface and a port on
which it listens. Allows for the definition of WebSocket Services.
Services can be defined by the server subscriber by sending the "Add
Service" message to the actor. The subscriber must provide a concrete
implementation of the "WebSocket Server-Service Enqueuer Msg" abstract
message used to let its subscriber receive the WebSocket service actor
enqueuer and a data handler message in form of a concrete implementation
of the "WebSocket Connection Handler Msg" abstract message used to
handle the connection events corresponding to the WebSocket
communication. For each service defined a "WebSocket Service" actor is
started that will send data to the subscriber that has defined the
service. The actor listens according to the server specifications on a
port and on an interface if not all, when a connection is established,
the actor performs the WebSocket handshake. If the service requested is
defined in this server, it informs the WebSocket service Actor who will
start a corresponding WebSocket connection actor. If either the service
is not defined, or the handshake fails, we do nothing, the WebSocket
protocol will take care of communicating the failure to the client.

## WebSocket Service.lvclass

It forwards packets received from the connected WebSocket Clients to its
subscriber actor as "WebSocket Connection Handler Msg" messages. It
allows the subscriber to send data to a specified client ("Send to Peer
Msg" message of the parent "WebSocket Connection" class) or to all
connected clients (“Broadcast Msg” message). When launching the actor by
means of the “WebSocket Server.lvclass” actor, the subscriber must
provide a data handler message in form of a concrete implementation of
the "WebSocket Connection Handler Msg" abstract message used to handle
the connection events corresponding to the WebSocket communication and a
concrete implementation of the "WebSocket Server-Service Enqueuer Msg"
abstract message used to let the server subscriber actor receive the
WebSocket service actor enqueuer.

## Abstract messages

Two abstract messages must be subclassed to allow the interaction of either a WebSocket Server, a WebSocket Service or a WebSocket Client with the corresponding subscriber actor.

### WebSocket Connection Handler Msg.lvclass

This abstract message is sent by either the WebSocket Client or by the
WebSocket Service to the subscriber Actor on four events:

-   Connect

-   Disconnect

-   Message

-   Drop

For each subscriber Actor a concrete subclass of the message should be created by overriding the methods:

-   "On Connect.vi"

-   "On Disconnect.vi"

-   "On Message.vi"

-   "On Drop.vi"

The subscriber actor can program the required action for the
corresponding event.

### WebSocket Server-Service Enqueuer Msg.lvclass

This abstract message is used either by the WebSocket Server Manager or
by the WebSocket Server to send the just launched/added Server/Service
Enqueuer to the subscriber actor requesting the operation. Each
subscriber actor requesting the operation should subclass this message
and then override the Do.vi in which the Server/Service Enqueuer is
stored in the requesting actor.

# API

## WebSocket Server Manager

### WebSocket Server Listener Settings

<img src="./media/image2.png" style="width:1.77108in;height:1.25017in"
alt="A screenshot of a computer Description automatically generated" />

**timeout ms** specifies the time, in milliseconds, that the function
waits for a connection. If a connection is not established in the
specified time, the function returns an error. The default value is –1,
which indicates to wait indefinitely.

**resolve remote address** indicates whether to call the IP To String
function on the remote address.

**net address** specifies on which network address to listen. Specifying
an address is useful if you have more than one network card, such as two
Ethernet cards, and want to listen only on the card with the specified
address. If you do not specify a network address, LabVIEW listens on all
network addresses.

**port** is the port number on which you want to listen for a
connection.

### Launch Server

<img src="./media/image3.png" style="width:4.77541in;height:3.16694in"
alt="A screenshot of a computer Description automatically generated" />

**Message Enqueuer in** specifies the enqueuer of the WebSocket Server
Manager actor.

**Listener Settings** specifies the listener settings for the Server.

**Handshake timeout** specifies the timeout of the handshake when the client establishes a WebSocket connection to the server.

**Receive Server Enqueuer Handler** specifies the concrete message (to be created) child of the abstract message “*WebSocket Server-Service
Enqueuer Msg.lvclass*”.

### Stop a Server

<img src="./media/image4.png" style="width:4.5754in;height:2.87525in"
alt="A computer screen shot of a message Description automatically generated" />

**Subscriber Enqueuer** specifies the enqueuer of the actor that has
subscribed to the previously launched WebSocket Server actor.

**Message Enqueuer in** specifies the enqueuer of the WebSocket Server
Manager actor to be stopped.

**Listener Settings** specifies the listener settings for the previously
launched WebSocket Server actor.

**Handshake timeout** specifies the timeout of the handshake when the
client establishes a WebSocket connection to the server.

**Receive Server Enqueuer Handler** specifies the concrete message (to
be created) child of the abstract message “*WebSocket Server-Service
Enqueuer Msg.lvclass*”.

## WebSocket Server

In case the WebSocket Server Actor is manually launched the following
properties are available to configure it (the port must be entered)
before its launch

<img src="./media/image5.png" style="width:1.81275in;height:0.55216in"
alt="A close-up of a sign Description automatically generated" />

**Listener Settings** specifies the settings for the Server (see
definition above).

**Handshake timeout** specifies the timeout of the handshake when the
client establishes a WebSocket connection to the server.

### Add Service

<img src="./media/image6.png" style="width:4.77541in;height:3.01693in"
alt="A computer screen shot of a message Description automatically generated" />

**Receive Connection Enqueuer** specifies the concrete message (to be
created) child of the abstract message “*WebSocket Server-Service
Enqueuer Msg.lvclass”.*

**Message Enqueuer in** specifies the enqueuer of the WebSocket Server
actor.

**Service Name** specifies the name of the service.

**Subscriber Enqueuer** specifies the enqueuer of the actor that subscribes to the WebSocket Server Actor.

**WebSocket Data Handler** specifies the concrete message (to be
created) child of the abstract message “*WebSocket Connection Handler
Msg.lvclass”.*

## WebSocket Service

### Broadcast

<img src="./media/image7.png" style="width:4.17536in;height:2.16685in"
alt="A screen shot of a computer Description automatically generated" />  
**Message Enqueuer in** specifies the enqueuer of the WebSocket Service
actor.

**Data** specifies the data to be sent.

### Send

<img src="./media/image8.png" style="width:4.14203in;height:2.34187in"
alt="A computer screen shot of a message Description automatically generated" />

**Message Enqueuer in** specifies the enqueuer of the WebSocket Service
actor.

**Data** specifies the data to be sent.

**Client ID** ID that identifies the specific WebSocket client to which
we want to send the data

## WebSocket Client 

The following properties of the WebSocket Client actor must be set
before launching it:

<img src="./media/image9.png" style="width:1.79167in;height:0.91667in"
alt="A screenshot of a computer Description automatically generated" />

**WS URI** specifies the URI of the client with the following format:

*ws://hostname:port/service*

**Timeout (ms)** is the timeout to establish the connection to the
server. A value of –1 indicates to wait indefinitely.

**WebSocket Data Handler** specifies the concrete message (to be
created) child of the abstract message “*WebSocket Connection Handler
Msg.lvclass”.*

**Subscriber Enqueuer** specifies the enqueuer of the actor (must be
created) that subscribes to the WebSocket Client Actor.

## Send to Peers

<img src="./media/image10.png" style="width:4.20036in;height:2.6419in"
alt="A computer screen shot of a message Description automatically generated" />

**Message Enqueuer in** specifies the enqueuer of the WebSocket Client
actor.

**Data Format (Text)** Specifies an Enum allowing to specify if the data format is either “Text” or “Binary”.

**Timeout ms (5000)** is the timeout to establish the connection to the server. A value of –1 indicates to wait indefinitely.

## Abstract Messages
Abstract messages have private properties that carry useful information to be used when the subscriber actor implementing the Do.vi method ("WebSocket Server-Service Enqueuer Msg" message) or the "On Close.vi", "On Connect.vi", "On Message.vi", "On Drop.vi" ("WebSocket Connection Handler Msg" message) to act on the subscriber actor

### WebSocket Server-Service Enqueuer Msg

![WebSocket Server-Service Enqueuer Msg](media/WebSocket%20Server-Service%20Enqueuer%20Msg.png)

**Error** Carries the error resulting from either launching a server, or adding a service to a server or connecting to a client

**Service Name** String detailing the name of the service corresponding to the service that is being started and to which we are subscribing

**WebSocket Server-Service Enqueuer** the enqueuer of the server/service that was just started and to which we are subscribing

### WebSocket Connection Handler Msg
![WebSocket Connection Handler Msg](media/WebSocket%20Connection%20Handler%20Msg.png)

**Client/Service Enqueuer** The enqueuer of the Client/Service to which we subscribed that triggered the event

**Connection Event** Enum specifying the type of event being triggered. Can take values "On Connect", "On Message", "On Close", and "On Drop"

**Data** String containing the data being received if the event triggered is of type "On Message"

**Data Format** Enum specifying the type of data being received in case of a "On Message" event. It can take values "Binary" and "Text"

**Peer ID** String specifying the Peer ID from which the event is being triggered if the event is originating from a client

**Service Name** String specifying the service name from which the event is being triggered