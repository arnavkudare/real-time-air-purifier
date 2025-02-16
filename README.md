# real-time-air-purifier
Real time Air purifier using Atmega microcontroller and sensors parallel to the esp-02 Wifi Module
ATMega 32 microcontroller is used for processing the sensors values
16x2 I2C LCD used for displaying of the Live Sensor values and Status of the Air quality
Gas Sensors: MQ2, MQ7 and MQ135 gas Sensors with are electronically adaptible to the latest Arduino board configurations are used for air quality sensing.
In addition to this, to wirelessly transmit the values to the user system, ESP-02 wifi module is used that connects to the kernel of the arduino.
To demonstrate this, we have used a Relay which acts as feedback component is the circuit to turn on the exaust fan which is placed along with filters, and the fans switched on upon crossing the threshold values of the Gas Sensors
