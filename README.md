# DHP CAN Protocol Description

## This is WIP, not all documentation is ready

---

### What is DHP CAN?
DHP is a home automation project that I am working on. It is inspired by KNX - an industrial grade, open standard system used for building automation. It is robust, stable, future-proof, and therefore it comes with a price.

I designed my system with these goals in mind:
- decentralised system, in case one module dies, other modules remain unaffected
- relatively easy to configure and extend
- consists of modules, some of which some are sensors and actuators
- inexpensive, as module could be implemented on cheap low-end microcontrollers

I chose [CAN bus](https://en.wikipedia.org/wiki/CAN_bus) for communication, because I think TCP/IP is overkill for such a system.

It is a broadcast protocol - messages are sent to all modules on the bus, and each module can either accept it or ignore the message.

---

### Properties
DHP CAN is based on CAN 2.0B where 29-bit identifier is used.

The bus speed is fixed at 125000 bps and at this rate, as stated by CAN specification, bus cable length can be up to 500 meters. Obviously CAN bus repeaters should be used for long distances.

The DHP CAN protocol allows sending messages up to 255 bytes and messages longer than 8 bytes are verified with CRC-16.

CAN bus does not have limitation for connected nodes, it is all within the limits of the cable resistance, but with CAN bus repeaters, you could technically connect an unlimited amount of CAN bus nodes. However, DHP CAN protocol addressing supports 254 nodes per 254 groups, so the theoretical maximum node count on DHP CAN is 64516 nodes.

Each module has its own address (1-254) and group (1-254) number. The values 0 and 255 are both reserved.

---

### DHP CAN Header structure
CAN bus 29-bit identifier is used as a header in DHP CAN protocol, see table below for explanation:

| Name | Length | Description |
| ----------- | ----------- | ----------- |
| Mode | 1 bit | Specify whenever the message format is for bootloader or for regular messages |
| Network Message | 2 bit | This is used to identify multi-frame messages that are sent when the message is longer than 8 bytes |
| Message Type | 2 bit | To identify whenever a message was a notification, request or response |
| Command | 8 bit | Message command type |
| Group | 8 bit | Source group number of the node that sent the message |
| Address | 8 bit | Source address number of the node that sent the message |

See below for possible `Mode` values:

| Value | Description |
| ----------- | ----------- |
| 0 | This message is for the bootloader to flash the node firmware |
| 1 | This message is intended for normal node-to-node communication |

See below for possible `Network Message` values:
| Value | Description |
| ----------- | ----------- |
| 0 | It is a single-frame message |
| 1 | It is a multi-frame message, this is the first frame |
| 2 | It is a multi-frame message, this applies to all frames that come after the first frame |

See below for possible `Message Type` values:
| Value | Description |
| ----------- | ----------- |
| 0 | This message is a request for a node, both actuators and sensors can see and accept it |
| 1 | This is a response to a request that can be sent by both actuators and sensors |
| 2 | This is a notification that is ignored by sensors and accepted by actuators |
