## Documentation

+ General documentation [phidget](https://www.phidgets.com/docs/Phidget_Program_Outline)
+ Load cells documentation [load cell phidget](https://www.phidgets.com/docs/Calibrating_Load_Cells)
+ Load cells [code example](https://www.phidgets.com/?prodid=1270#Tab_Code_Samples)
## Phidget channels

Each channel has a **Channel Class**, corresponding to the type of function the channel performs. For example, a `DCMotor` channel will control a DC motor, a `TemperatureSensor` channel will measure temperature

 Every `VoltageInput` channel has functions to set the data interval and read the voltage on the input.
## General Workflow

![Workflow for programming a phidget ](media/general_phidget_program_flowchart.png)

### 1) Create a channel

### 2) Address Channels (aka **Addressing Phidgets**)

### 3) Open Channels and wait for attachment
### 4) Main loop
### 5) End program ?
### 6) Close channels
### 7) End
