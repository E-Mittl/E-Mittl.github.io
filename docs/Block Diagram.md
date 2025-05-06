---
title: Block Diagram
---

![Block Diagram](Images/diagrams/EGR314%20-%20Individual%20Block%20Diagram.drawio.png)

## Design Choices

The sensor subsystem is designed to be as simple as possible for easy, accurate, and efficient operation. The block diagram above helped to ensure the dsign meets these requirements, as well as project requirements. It shows how the PIC communicates with other subsystems over UART, as well as how it uses I2C to communicate with the Microchip SNAP programmer and OPT4048 RGB sensor. Pin allocation ensures there is enough I/O for the red and green debugging LEDs, white illumination LED, and debugging pushbutton.