# Paxcounter-SDS011-Lora-Umweltsensor
TTGO Paxcounter mit Feinstaubsensor und BME280

## INHALT:

 1. Hardware 
 2. Zusammenbau / Verdrahtung 
 3. thethingsnetwork 
 4. Software
 5. github-Repository 
 6. Änderungen eintragen 
 7. Decoder & Konverter
 8. Kompilieren und hochladen 
 9. Ausblick 1 node-Red, InfluxDB und Grafana
 10. Ausblick 2 luftdaten & openSenseMap

--------------------

**Hardware:**

 - TTGO ESP32 Paxcounter (für Deutschland EU868 Variante)
 - Feinstaubsensor nova PM sensor Typ SDS011 
 - Umweltsensor BME280 (Temperatur, Luftfeuchtigkeit und Druck)
-----------------------------

**Zusammenbau / Verdrahtung**

Die Stiftleisten werden am Paxcounter-Board und am BME Sensor angelötet, je nach Einbaulage oder Einbauort bietet sich die Stiftleiste auf der Ober- oder Unterseite vom Board an. Die Schraubantenne wird montiert - auch hier kann man je nach Sensoraufbau eventuell ein Verlängerunskabel einbauen. Anschließend werden die beiden Sensoren wie folgt am Paxcounter angeschlossen:

*Der Umweltsensor wird folgendermaßen verdrahtet:*

    VCC (VDC 3,3 Volt) wird mit Pin 3.3V angeschlossen
    GND (Ground) wird mit dem GND am Paxcounter verbunden
    SCL (i2c Bus) --> Pin 22 am Paxcounter (mit SCL oder 22 beschriftet) 
    SDA (i2c Bus) --> Pin 21 am Paxcounter (mit SDA oder 21 beschriftet)
    frei
    frei

*Der Feinstaubsensor wird folgendermaßen verdrahtet:*

    TXD (Daten senden) wird mit Pin 23 am Paxcounter verbunden
    RXD (Daten empfangen) wird mit Pin LoRa2 am Paxcounter verbunden
    GND (Ground) wird mit einem weiteren Pin GND am Paxcounter angeschlossen
    frei
    VCC (VDC 5 Volt) kommt an den 5 Volt Pin (beschriftet mit 5V)
    frei
    frei

> *Schaut Euch bitte die Bilder im Ordner /IMG an, da kann man nochmal
> deutlich die Verdrahtung erkennen ! Für einen stabilen Halt der
> Verbindungskabel auf der Stiftleiste empfehlen wir, die Stecker
> einfach mit etwas Heisskleber zu fixieren. So vermeidet man eine ganze
> Weile das abrutschen der teilwese recht locker sitzenden Kabel.*
-----------------------------
**thethingsnetwork**

Im nächsten Schritt bereiten wir unseren Account bei thethingsnetwork.org vor -- Zuerst wird (sofern nicht schon vorhanden) ein Account angelegt. Anschließend wird eine neue Application erstellt. Neben einem eindeutigen Namen wird hier nur noch der ttn-Handler als "ttn-handler-eu" eingetragen. 

Innerhalb dieser Application legen wir unser Device an - also quasi unseren TTN Feinstaubsensor. Dazu klicken wir auf "register device". Im nächsten Fenster vergeben wir einen kurzen, eindeutigen Namen, beispielsweise "Feinstaubsensor-01" -- die DeviceEUI lassen wir uns automatisch generieren. Dazu klicken wir auf den Doppelpfeil vor dem entsprechenden Feld. Mehr wird an dieser Stelle nicht benötigt, später kann man noch den Standort und ähnliche Daten zum Gerät hinterlegen, wenn man möchte. Das Browser-Fenster mit "Device Overview" lassen wir auf, es wird später benötigt !

-----------------------------
**Software**

Zur Programmierung verwenden wir Microsoft Visual Studio Code mit der Erweiterung PlatformIO IDE. Zuerst wird also Visual Studio Code installiert, anschießend unter "FILE" und "PREFERENCES", "EXTENSIONS" im Suchfeld Platformio ide eingegeben und installiert. Nach einem Neustart der Software ist unsere Programmierumgebung fertig. 

> Hinweis: Zahlreiche umfangreiche Anleitungen im WWW beschreiben die
> Installation und Einrichtung vom VS Code sehr detailliert

-----------------------------
**github-Repository**

Als Basissoftware verwenden wir seit einiger Zeit das Repository von Cybmerman54 aus github. 
https://github.com/cyberman54/ESP32-Paxcounter
Das Repository wird heruntergeladen und entpackt, anschließend im VS Code geöffnet.

-----------------------------
**Änderungen und Anpassungen**

(1) 
Im ersten Schritt wird die Datei platformio_orig.ini aus dem Hauptverzeichnis umbenannt oder kopiert und dann umbenannt zu platformio.ini. 
Danach wird in dieser Datei die Zeile 10 ersetzt zu `;halfile = generic.h`  es wird quasi deaktiviert. Auch in dieser Datei wird Zeile 19 ersetzt zu `halfile = ttgov21new.h` - das Semikolon entfernt und damit aktiviert
(siehe Bild)

(2) 
Im Ordner /src/hal/ wird die Datei "ttgov21new.h" wie folgt angepasst, folgender Code wird in Zeile 21 einfach nur eingefügt:

    // BME280 sensor on I2C bus
    #define HAS_BME 1 // Enable BME sensors in general
    #define HAS_BME280 GPIO_NUM_21, GPIO_NUM_22 // SDA, SCL
    #define BME280_ADDR 0x76 // change to 0x77 depending on your wiring
    
    // SDS011 dust sensor settings
    #define HAS_SDS011 1 // use SDS011
    // used pins on the ESP-side:
    #define SDS_TX 12     // connect to RX on the SDS011
    #define SDS_RX 35     // connect to TX on the SDS011

Kopiert Euch einfach den oben stehenden Code und fügt ihn im VS-Code in der Datei ein. Das Ergebniss kann man im Bild sehen.

(3) /src/ota_sample.conf:
Die Datei wird umbenannt oder kopiert in "ota.conf", in der Datei selber müssen wir keine Änderungen vornehmen

(4) /src/loraconf_sample.h:
Umbenennen oder kopieren in "loraconf.h"
in Zeile 38, 40 und 42 die DEVEUI, APPEUI und APPKEY (alles als MSB) einfügen. Dazu gehen wir zurück in das Browserfenster mit "Device Overview". Um die Schlüssel in das richtige Zahlenformat zu bekommen, wird auf das <> Symbol der jeweiligen Zeile geklickt, der Schlüssel wechselt sein Format. Wir benötigen jeweils das MSB Format 

> EXKURS: Beim Programmieren sind Sie sicherlich schon einmal über die
> Abkürzungen "MSB" und "LSB" gestolpert. Was es mit den Kürzeln auf
> sich hat, erfahren Sie in diesem Praxistipp. Bitwertigkeit: MSB & LSB
> einfach erklärt Die Bitwertigkeit ist dazu da, den Stellenwert jedes Bits 
> festzulegen. Dies ist beispielsweise für serielle Übertragungen
> wichtig.
> 
> - LSB steht für "Least Significant Bit". Wenn eine Bitfolge nach der
> LSB-0-Bitnummerierung nummeriert ist, dann hat das Bit mit dem Index 0
> den geringsten Stellenwert. 
> - MSB steht für "Most Significant Bit". Bei
> der MSB-0-Bitnummerierung hat das Bit mit dem Index 0 den höchsten
> Stellenwert. Wenn bei einer Binärzahl mit den Positionen 0, 1, ...,
> N-1 das Bit mit dem Index 0 den höchsten Stellenwert hat, muss dessen
> Wert mit 2 hoch (N-1) multipliziert werden.

Am Ende der jeweiligen Zeile mit dem Schlüssel kann man sich den gesamten Teil in die Zwischenablage kopieren, zurück im VS-Code werden nacheinander die drei Schlüssel in die entsprechenden Zeilen kopiert - die vorgegebenen Beispielschlüssel werden einfach überschrieben. Auch hier habe ich für Euch ein Bild zur Veranschaulichung erstellt.

(5)
Die Datei /.pio/libdeps\usb/SDS011 sensor library/SDS011.cpp wird nun bearbeitet:

Dieser Code stammt aus https://github.com/cyberman54/ESP32-Paxcounter/issues/597#issuecomment-619936445 und beschreibt das "Arbeitscommando"

Der Text wird in Zeile 35 eingefügt:

    static const byte WORKCMD[19] = {
        0xAA, // head
        0xB4, // command id
        0x06, // data byte 1
        0x01, // data byte 2 (set mode)
        0x01, // data byte 3 (work)
        0x00, // data byte 4
        0x00, // data byte 5
        0x00, // data byte 6
        0x00, // data byte 7
        0x00, // data byte 8
        0x00, // data byte 9
        0x00, // data byte 10
        0x00, // data byte 11
        0x00, // data byte 12
        0x00, // data byte 13
        0xFF, // data byte 14 (device id byte 1)
        0xFF, // data byte 15 (device id byte 2)
        0x06, // checksum
        0xAB  // tail
    };

ausserdem müssen wir Zeile 113 ändern in:

    void SDS011::wakeup() {
        //  sds_data->write(0x01);
        //  sds_data->flush();
        for (uint8_t i = 0; i < 19; i++) {
            sds_data->write(WORKCMD[i]);
        }
        sds_data->flush();
        while (sds_data->available() > 0) {
            sds_data->read();
        }
    }
-----------------------------
**Decoder und Converter**

Zum Abschluß der Software-Änderungen holen wir uns noch aus dem nachfolgenden per Copy & Paste /src/TTN/packed_converter.js -> Payload Formats -> converter den Inhalt dieser Datei und gehen zurück in das Browserfenster. In den Eigenschaften der TTN Application gehen wir auf den Reiter "Payload Formats" und wählen dort zum Einfügen den Converter aus. Sollte im Fenster dort schon Text stehen wird dieser zuerst einfach vollständig gelöscht und dann der Text aus der o.g. Datei dort eingefügt.

Danach holen wir uns im VS Code den Decoder aus der Datei /src/TTN/packed_decoder.js und kopieren diesen genau wie eben aber in das Feld "Payload Formats -> decoder" Hier ist noch eine kleine, manuelle Änderung nötig: 

Füge in Zeile 37 (Payload Formats -> decoder) folgendes hinzu:

        // combined wifi + ble + SDS011
        if (bytes.length === 8) {
            return decode(bytes, [uint16, uint16, uint16, uint16], ['wifi', 'ble', 'PM10', 'PM25']);
        }

Ein kurzer Test in der Konsole bringt Sicherheit, dass der Decoder funktioniert: 16x 0 in das Feld für Payload eingeben, Port auf 1 setzen und "Test" klicken und dann "save payload function" klicken.

-----------------------------
**Kompilieren und hochladen**

als letzter Schritt ist nur noch die Kompilierung des Programms im VS Code nötig, danach wird das gesamte Programm per USB auf den Paxcounter geladen. Der Sensor wird also per Micro-USB Kabel angeschlossen und mit dem Schiebeschalter aktiviert. 

- Unten in der blauen Leiste wird zuerst auf "CLEAN" (Symbol Mülltonne geklickt) Unnötige und Code-Inhalte und Dateien werden automatisch bereinigt. 

- Anschließend klickt man in der blauen Leiste unten auf das Symbol mit dem Haken "Built" - das kann einen Moment dauern, sollte aber nach spätestens ein paar Minuten abgeschlossen sein. 

- Letztendlich wird der gesamte Programmcode an den Sensor gesendet. Dazu einfach auf Rechts-Pfeil "Upload" klicken; das Programm wird übertragen und anschließend der Sensor neu gestartet, wenn die Übertragung erfolgreich beendet wurde.

Klickt man nun noch in der blauen Leiste auf das Stecker-Symbol "Serial Monitor", so kann man in einer Konsole beobachten, was der Sensor aktiv macht und auch Messwerte werden dort angezeigt.

> Für einen direkten Funktionstest in der TTN Konsole ist ein
> TTN-Gateway in Reichweite unabdingbar, da sonst der Sensor seine Daten
> zwar in die Umgebund gesendet, diese aber nirgendwo empfangen werden
> und weiter verarbeitet werden.

Parallel dazu öffnet man also im Browser die TTN-Application, wählt das Device aus und klickt oben rechts auf das "Data" Feld. nach einer kurzen Weile sollten auch hier Daten angezeigt werden, die zu thethingsnetwork per Lora Funktechnik übertragen werden. Hinweis, da die Feinstaubwerte als "Ganzahl" übertragen werden, müssen die in der TTN-Konsole angezeigten Messwerte durch 10 geteilt werden !!! Herzlichen Glückwunsch, Euer Lora-Feinstaubsensor ist fertig !!! In zwei weiteren Kapiteln geben wir einen Ausblick, was mit den gewonnenen Daten gemacht werden kann, und wie man selbige auf Opendata Karten veröffentlichen kann.

-----------------------------
**Ausblick 1 - node-Red, InfluxDB und Grafana**
*hier folgt der Text noch*

-----------------------------
**Ausblick 2 - OpenData Karten, luftdaten und opensensemap**
*hier folgt der Text noch*
