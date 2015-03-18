Summary

1. Purpose
> The Sensors Networking Protocol is used in multimedia sensors network.
It enables internal communication between sensors in the network. It is also
responsible for maintaining links : a sensor and its children for example.

2. Overview of the Protocol
> As mentioned above, the protocol is in charge of the communication between
each network sensor.All links are up when each sensor have introduced to its parent through a Hello request.
A sensor should consider that one of its child sensor is down when it cannot communicate with this one anymore.
A request system has been established. Indeed, a sensor can retrieve the data of its children. It sends a query which
specifies the catch area and the quality of the data that each of its children can transmit.
A request implies an answer, also a sensor can answer to a request.

3. Relation to other Protocols
> The data protocol runs over TCP. Indeed, it needs some reliable delivery of datas.
It is not a real time protocol, such like RTP, consequently the data protocol does not need to use UDP as a transport layer.
TCP is a reliable stream delivery service that guarantees to deliver a stream of data sent from one host to another
without duplication or data loss.

4. Initial Networking Sensors
> At first, a sensor tries to communicate with its parent through the port 31001. It sends a HELLO REQUEST to announce
its self. This packet is sent while the parent has not anwsered.

5. Sensors Protocol Packets

The protocol supports 4 types of packets, all of them have been mentioned
above :

1 HELLO REQUEST
2 HELLO REPLY
3 INFOS
4 REQDATA
5 REPDATA
6 ACK

5.1. Set up links
5.1.1. Child discovery
When a sensor starts, it tries to contact its parent sensor with a HELLO REQUEST packet. The parent sensor is set by the initial configuration or by a configuration packet. If the parent sensor of the current one is not null (it is the case just for the root sensor), the current sensor send a HELLO REQUEST packet until the parent sensor replies with an HELLO REPLY packet. By using this algorithm, the child sensor is certified that its parent is up.
If the parent sensor of the sensor changes during it's up, the sensor sends a new HELLO REQUEST packet to its new parent to maintain the network topology.

> 2 bytes  4 bytes   4 bytes
> 
---

> | Opcode |   idD   |  idS  |
> > 
---

> > > Figure 5.1-1 HELLO REPLY packet


idS : id of the current (source) node.
idD : id of the parent (destination) node.

5.1.2. Parent answer
When a sensor node receives a HELLO REQUEST packet on the default port, it adds the source sensor to the list of its children. These children will be used for the data requests later. The sensor has to reply to its new children with a HELLO REPLY packet setting up the error code to null.

2 bytes    4 bytes     4 bytes  4 bytes

> 
---

| Opcode | idD   |  idS   |  error code |					 -----------------------------------------
Figure 5.1.2-1 HELLO REQUEST packet


5.2. Share sensor's parameters

Each sensor has to share its parameters with its parent. These parameters are sent every second in an INFOS packet after an HELLO REQUEST packet. With this, a parent sensor will be able to merge its own information with all information from its children. By recursion, the root sensor (or sink) will be able to retrieve the global information for the complete network. This information is updated each time an INFOS packet is re-sent. This protocol provides a quick-answer process. It avoids asking the information to each sensor if it knows in advance that this sensor doesn't own the information.

If a sensor has no parent (root sensor), this packet is not transmitted.

> 2 bytes   4 bytes	   4 bytes    8 bytes       8 bytes    8 bytes    4 bytes   4 bytes
> 
---

> | Opcode |    IdD    |    IdS   | Catch Area |   Clock   | Autonomy | Quality | Payload  |
> > 
---

> > > Figure 5.2-1 INFOS packet
5.3. Handle data requests

The only sensor which can receive user requests is the root sensor. This sensor is the only one which can give the final answer to this request too. When any sensor receives a data request in a REQDATA packet, it tries to get the maximum datas corresponding to this request. The data request is divided as shown below :


> 4 bytes     8 bytes
> 
---

> | Opcode |  Catch area  |
> > 
---



> Figure 5-2: REQDATA packet

- Catch area : the catch area requested.
- Data : the oldest modification date requested for this area.

The root sensor knows if a request has to be processed or not. Indeed, it stores informations about its children :  catch area, quality etc.
Therefor, it has the possibility to decide which one of its children is able to transmit the data corresponding to the request. In fact, all sensors which own children store
information about them. It avoids sending requests to all of the sensors that could lead to overflow the network.


A request involves an answer. Basically, a parent sensor can receive the answers of its children. If the answers are relevant, it has to merge them. Then, it forwards the merge  to its own parent.
Such a reply packet is described below :

> 4 bytes     8 bytes     4 bytes  n bytes
> 
---

> | Opcode |  catch area  |  quality | data   |
> > 
---



> Figure 5.3-2 REPDATA

Finally, the root sensor can answer to the original request with a positive or negative packet.

> 4 bytes     4 bytes
> > 
---

> > | Opcode |  error code   |
> > > 
---

> > > Figure 5.3-3 Negative ACK



> 4 bytes      4 bytes     n bytes
> 
---

> > | Opcode |  error code  |  data  |

> 
---

> > Figure 5.3-4 Positive ACK

