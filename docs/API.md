---
title: API
---

## Message Types
|Message Type <br> byte 1-2 <br>(uint16_t) | Description|
|-------------------|---------------|
|0                  | Status Code   |
|1                  | Drive Mode    |
|2                  | Sensor Data   |
|3                  | Path Selection|

## Message Type 0: Status Code
|         |  Byte 1-2  | Byte 3 | 
|---------|------------|---------|
|Var Name | msg_type   | status  |
|Var Type | uint16_t   | uint8_t |
|Min Val  | 0          | 0       | 
|Max Val  | 3          | 3       |
|Example  | 0          | 2       |

## Message Type 2: Sensor RGB Data 

|         |  Byte 1-2  |  Byte 3 |
|---------|-----------|----------|
|Var Name | msg_type  | color    |
|Var Type | uint16_t  | uint8_t  | 
|Min Val  | 0         | 0        |
|Max Val  | 3         | 2        |
|Example  | 2         | 1        |

## Message Type 3 : Path Selection  

|         |  Byte 1-2  | Byte 3 |
|---------|------------|--------|
|Var Name | msg_type   | path   |
|Var Type | uint16_t   | uint8_t|
|Min Val  | 0          | 0      |
|Max Val  | 3          | 2      |
|Example  | 2          | 2      |