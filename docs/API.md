---
title: API
---

## Team ID
| Member Name | Subsystem       | ID (char) |
|-------------|--------------------|--------|
| JC R.       | Human Machine Interface | H |
| Eric M.     | RGB Sensor              | S |
| Marcus P.   | MQTT Server             | M |
| Bradley P.  | Actuator                | A |
| Broadcast   | ALL                     | X |

## Message Types
|Message Type <br> byte 1-2 <br>(uint8_t) | Description|
|----------------------|---------------|
|0x00                  | Status Code   |
|0x01                  | Drive Mode    |
|0x02                  | Sensor Data   |
|0x03                  | Path Selection|

## Message Type 0: Status Code
|         |  Byte 1  | Byte 2 | 
|---------|------------|---------|
|Var Name | msg_type      | initialize |
|Var Type | uint8_t       | uint8_t    |
|Min Val  | 0x00          | 0x00       | 
|Max Val  | 0x00          | 0x00       |
|Example  | 0x00          | 0x00       |

## Message Type 2: Sensor RGB Data 

|         |  Byte 1  |  Byte 2 |
|---------|-----------|----------|
|Var Name | msg_type  | color    |
|Var Type | uint8_t   | uint8_t  | 
|Min Val  | 0x00         | 0x00        |
|Max Val  | 0x03         | 0x02        |
|Example  | 0x02         | 0x01        |

## Message Type 3 : Path Selection  

|         |  Byte 1  | Byte 2 |
|---------|------------|--------|
|Var Name | msg_type   | path   |
|Var Type | uint8_t    | uint8_t|
|Min Val  | 0x00          | 0x00      |
|Max Val  | 0x03          | 0x02      |
|Example  | 0x02          | 0x02      |