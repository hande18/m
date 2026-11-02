#include <SoftwareSerial.h>

SoftwareSerial BT(10, 11);
// D10 = RX
// D11 = TX

void setup() {
    Serial.begin(9600);
    BT.begin(9600);
    Serial.println("HC-05 Ready");
}

void loop() {
    // HC-05 → Arduino

    if (BT.available()) {
        char data = BT.read();
        Serial.print("Received: ");
        Serial.println(data);
    }

    // Arduino Serial Monitor → HC-05
    
    if (Serial.available()) {
        char data = Serial.read();
        BT.write(data);
    }
}
