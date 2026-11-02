Arduino Side Code

void setup() {
    Serial.begin(9600); // HC-05 default baud
}

void loop() {

    // Send message to Raspberry Pi
    Serial.println("Hello Pi");

    // Check incoming Bluetooth data
    if (Serial.available()) {

        String receivedData = Serial.readStringUntil('\n');

        Serial.print("Got from Pi: ");
        Serial.println(receivedData);
    }

    delay(1000);
}

Raspberry Pi Side

import serial
import time

# Connect to HC-05 serial port
bt = serial.Serial("/dev/rfcomm0", baudrate=9600, timeout=1)

while True:
    # Read data from Arduino
    data = bt.readline().decode('utf-8').strip()
    if data:
        print("From Arduino:", data)

    # Send message to Arduino
    bt.write(b'H')
    time.sleep(2)
