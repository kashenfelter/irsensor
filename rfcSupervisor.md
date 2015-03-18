Ingenieurs2000											MIETTE Tom
> MOREAU Alan
> MOURET Sebastien
> PONS Julien
RFC												janvier 28, 2008


> SUPERVISOR SERVER PROTOCOL (SSP)

1. Purpose

The attempt in this specification is to specify the diverse needs to permit communication and configuration
between sensors and their server. The objectives of SSP are described below:

> - to register sensors
> - to provide configuration information to sensors by supervisor server (passive and active sensor configuration),
> - to provide sensor states,
> - to retrieve information from sensors by server,
> - to start a script which aims at configuring network sensors.



2. Overview of the protocol

This protocol provides communication rules between the supervisor server and its sensors.
Sensors, which do not belong to a specific network, try to establish a communication with the supervisor server. This implementation implies that sensors know the supervisor server IP.

Once communication has been established, the server tries to retrieve the sensors state. A node can be set to different state among (stopped, standby or ready). Depending on the nodes state, the server might choose between sending the nodes configuration parameters, or alter their state.



3. Relation to other protocols
This protocol runs over TCP. The use of this protocol involves the knowledge of TCP and Sensor Data Protocol.



4. Initial Networking Sensors

The use of Supervisor Server Protocol involves that a supervisor server waits for sensor connection
s on port 31000. When the server starts, it listens to this port. The server has to set up the networking configuration. It expects a predefined number of connection requests.

After having received a connection request from a network sensor, the SS starts to configure it.
When all sensors are configured, it can switch them on. The supervisor server will automatically
discard a useless sensor contained in the network. That is to say, that the SS will accept the
sensor request for connection, but it won't send it its configuration parameters.


5. Terminology

Sensor or node: a sensor is a networking unit, which has to be configured by a supervisor server in order to live.

Supervisor server or SS: a supervisor server represents a networking unit with a configuration script, which aims at setting up the network.

6. Supervisor Server protocol packets

6.1. Server side

6.1.1. Starting SS

When the SS starts, it listens to port 31000 for REQCON packets. Whenever, the SS receives this type
> of packet, it will try to register the sensor from which the packet came from.

6.1.2.  Register node

When SS receives a REQCON packet from a node (see sensor side 6.2.1.), the server answers to it with
a REPCON packet to attribute it a unique identifier. This identifier enables to differentiate each sensor. Therefor, several sensors can run over a single computer.

> The REPCON packet, allows the SS to retrieve the sender's IP address, consequently, the supervisor server
> will be able to send a configuration packet. If a problem occurs during this step, the sensor sends an ACK packet with a CONNECT-ERR code (see error codes 6.4.1.).

The REPCON packet is sent to the same port that the REQCON packet was sent to. It's the only packet, which is sent this way. Other packets will use the specific supervisor server protocol port 31000.

2 bytes     4 bytes

---

|  Header  |     Id     |

---

Figure 6-1-2-1: REPCON packet


6.1.3. Configure node

After registering a new node, SS has to configure it before it can start it. A SETCONF packet in
> which each field specifies a value sets the configuration. The SETCONF packet is sent to each sensor
> to the port 31000.

2 bytes 4 bytes 8 bytes    4 bytes  4 bytes    4 bytes   1 byte

---

| Header | Id | Catch Area | Clock | Autonomy | Quality | Payload |

---


6 bytes n         4 bytes      n bytes

---

| Parent address | ID parent | Optional |

---

Figure 6-1-3-1: SETCONF packet


Id: id of the sensor configured.

Catch area: two integers, which define a rectangle. This rectangle corresponds to the area that the node is able to "see".

Clock: interval between two data captures

Autonomy: time in seconds the sensor can be up (states UP and PAUSE).

Quality: quality of the data catches.

Payload: maximal number of requests a sensor can handle.

Root: ip address of the parent sensor of this one (null if the current sensor is the network root).

To modify the configuration of a sensor already configured, the SS has to send another SETCONF packet
> with the new values. Even if only one parameter has to be modified, all values have to be rewritten as
> the sensor reloads its configuration from scratch.

The sensor answers to the SETCONF packet with an ACK packet containing an error code (see sensor side 6.2.2.).

6.1.4 Retrieving configuration

The SS can retrieve the configuration of a sensor by sending a GETCONF packet to this sensor.
The sensor will send its configuration with a REPCONF packet (see sensor side 6.2.3.).

2 bytes 4 bytes

---

| Header | id |

---

Figure 6-1-4-1: GETCONF packet


6.1.5 Setting sensor state

The SS can modify the state of a sensor. The different states are UP, DOWN and PAUSE. Default state of
> a sensor is set to DOWN.

A sensor which state is different from UP will be unable to handle requests. When a sensor is down,
it loses its configuration whereas a paused sensor keeps it.

To alter the sensors' state, the SS sends a SETSTA packet with the corresponding state code.

2 bytes 4 bytes 2 bytes

---

| Header | id | state |

---

Figure 6-1-5-1: SETSTA packet

If the sensor isn't already configured, the sensor's state cannot be set to UP or PAUSE
> (if it happens, a CONFIGURATION-ERR is raised). The sensor answers to the SETSTA packet with
> an ACK packet containing an error code (see sensor side 6.2.4.).


6.1.6. Retrieving sensor state

The SS can retrieve the sensor state by sending a GETSTA packet. The sensor will give its state
> information thanks to a REPSTA packet. (see sensor side 6.2.5.).

2 bytes 4 bytes

---

| Header | id |

---

Figure 6-1-6-1: GETSTA packet


6.2. Sensor side

6.2.1. Start sensor

When a sensor wants to join to the network, it sends a REQCON packet on port 31000 to the
> supervisor ip address. Note that every network sensor knows the SS IP address.

2 bytes

---

| Header |

---

Figure 6-2-1-1: REQCON packet

The SS replies with a REPCON packet (see server side 6.1.2.).

6.2.2 Set configuration

When a sensor receives a SETCONF packet from the SS (see server side 6.1.3.), it modifies its
configuration with the specified parameters. If the configuration process succeeds the sensor
sends an ACK packet with a null error code. If one or more parameters cannot be read correctly,
> the sensor sets up the error code of the ACK before sending the packet. (See error codes 6.4.2.).

2 bytes 4 bytes    1 bytes

---

| Header | id | error code |

---

Figure 6-2-2-1: ACK packet

6.2.3. Get configuration

When a sensor receives a GETCONF packet from the SS (see server side 6.1.4.), it sends its current
> configuration to the server in a REPCONF packet.

2 bytes 4 bytes 8 bytes     4 bytes 4 bytes       4 bytes 1 byte

---

| Header | Id | Catch Area | Clock | Autonomy | Quality | Payload |

---

Figure 6-2-3-1: GETCONF packet





6.2.4. Set state

When a sensor receives a SETSTA packet from the SS (see server side 6.1.5.), it modifies its
> current state and replace it with the new one. If the state can be read correctly, the sensor replies
> with an ACK packet with a null error code. Otherwise the sensor sets up the error code and send
> the ACK packet (see error codes 6.3.2.).

2 bytes 4 bytes    1 bytes

---

| Header | id | error code |

---

Figure 6-2-4-1: ACK packet

6.2.5 Get state

When a sensor receives a GETSTA packet from the SS (see server side 6.1.6.), it sends its
current state to the server in a REPSTA packet.

2 bytes 4 bytes 2 bytes

---

| Header | id | state |

---

Figure 6-2-5-1: REPSTA packet



6.3. Data requests

6.3.1. Server request

When all sensors are correctly configured and set to the UP state, the SS can submit requests
> to the root sensor. These requests are contained in REQDATA packets. These requests are a
simulation of user's purpose.

1 byte 2 bytes       4 bytes                4 bytes

---

| Header | id | catch area | min quality requested |

---

4 bytes

---

| min date requested |

---

Figure 6-3-1-1: REQDATA packet

Catch area: the catch area requested.

Quality: the minimum quality requested for this area.

Date: the oldest modification date requested for this area.

The root sensor replies with a REPDATA packet.

6.3.2. Root reply

When the root sensor receives a REQDATA from the SS, it tries to reply to this request
(the request is handled by a sensor internal protocol). If the root sensor failed to reply,
it sends an ACK packet with the correct error code. Otherwise, the root sensor sends a REPDATA
> packet with the requested datas.
2 bytes 4 bytes

---

| header | id | datas |

---

Figure 6-3-2-1: REPDATA packet

6.4. Error codes
6.4.1. Server error codes
0000.0000 - OK - none
0001.0000 - CONNECT-ERR - if a problem occurs during the connection between a sensor and the SS.

6.4.2. Sensor error codes
0000.0000 - OK - none
0000.0001 - CONFIGURATION-ERR - if a problem occurs during the configuration of the sensor
> (syntax or semantic errors).
0000.0010 - STATE-ERR - if a problem occurs during the change of state of the sensor
> (syntax or semantic errors).
0000.0011 - IS-DOWN-ERR - if the sensor is down (it cannot handle a configuration packet).
0000.0100 - CANNOT-REPLY - if the root sensor cannot reply to a specific request.



6.5. Header specification

1 byte 1 byte

---

| Opcode | Optional |

---

Figure 6-5-1: Header

The header field in each packet specifies the unique code of the packet to identify it.
0000.0000 - REQCON
0000.0001 - REPCON
0000.0010 - SETCONF
0000.0011 - GETCONF
0000.0100 - REPCONF
0000.0101 - SETSTA
0000.0110 - GETSTA
0000.0111 - REPSTA
0000.1000 - ACK
0000.1001 - REQDATA
0000.1010 – REPDATA



6.6. State specification

1 byte

---

| State |

---

Figure 6-6-1: State

A state is represented by a code:
00000000 – UP
00000001 – DOWN
00000010 – PAUSE or STANDBY