1. Purpose
> The Data Protocol is used in multimedia sensors network. It provides
datas for abstract sensors. This protocol isn't necessary in a real sensors network.

2. Overview of the Protocol
> To use an abstract sensors network, we need a server simulator. Indeed, sensors
need to transmit data to a supervisor server (see Supervisor Server Protocol).
The server, which implements this protocol, is responsible for dispatching data between
all sensors. So, each sensor has a catching area which represents the part of data
that a sensor can share.
The transaction between server and sensor, is a two steps process. The sensors
initiate the connection to the data server and transmit information such as a catching area.
The server transmits the right data corresponding to the given parameters.

3. Relation to other Protocols
> This protocol works over TCP. Indeed, it needs to use reliable data deliveries. TCP provides
a such feature. This protocol does not need real time features that UDP might offer.

4. Initial
> The connection is initiated by a sensor which wants to retrieve the datas that it has to
transmit. It sends a request to obtain the datas and the server answers to it by giving the datas
the sensor was expecting.
The Data Protocol uses the port number 31002 to communicate with sensors.

5. Data Protocol Packets

> There are Three types of packets in this protocol :

> opcode  operation

  1. request for data (REQDATA)
> 2       reply to request (REPDATA)
> 3       error packet (ACK)

As mentionned above, a request for data is sent by sensors to get information according to
the given parameters :

> 2 bytes     4 bytes         8 bytes       4 bytes
> 
---

> |  Header  |     Id       |  Catch Area   |  Quality  |
> > 
---

> > > Figure 5-1: REQDATA packet

- Id : specifies the sensor's id.
- Catch Area : the catch area of the data
- Quality : the quality of the data catched

Once the REQDATA has been sent, the server can reply either by sending the requested data or either send an error packet. The packet which contains
the request response is described below :


> 2 bytes     4 bytes      4 bytes       4 bytes     n bytes
> 
---

> |  Header  |     Id       |  Mime Type  |  Lenght   |    Data   |
> > 
---

> > > Figure 5-2: REPDATA packet

- Id : specifies the sensor's id.
- Mime Type : the mime type of the current resource
- Lenght : the length of the following data
- Data : the datas corresponding to the request

And the error packet :


> 2 bytes    4 bytes      4 bytes
> > 
---


> |  Header  |     Id       |  Error Code   |
> > 
---


> Figure 5-3: ACK packet

Here are the different error codes :
- 0000.0000 OK - none
- 0000.0001 bad parameters