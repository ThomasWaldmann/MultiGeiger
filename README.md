# MultiGeiger
Das Projekt "Multigeiger" wurde von Jürgen Böhringer entwickelt und ist auf seiner Webseite http://www.boehri.de beschrieben.

Die Dokumentation dazu ist im Verzeichnis 'doc' zu finden (incl. Bauanleitung) und der Sourcecode liegt im Verzeichnis 'multigeiger' als .ino-Datei und kann mit der *Arduino-IDE* oder mit *Platformio  und Visual Code* übersetzt werden.

## Installation
Das Verzeichnis clonen oder als .zip runterladen und entpacken. Mit der Arduino-IDE in dem neuen Verzeichnis die Datei *multigeiger.ino* im Verzeichnis *multigeiger* öffnen. Als Board in der IDE das **Heltec Wifi 32 Kit** einstellen (für die Version **ohne** LoRa).  
Für Platformio muss in **platformio.ini** in dem Abschnitt **[platformio]** der Eintrag für *default_envs* so aussehen: **default_envs = wifi**. Dann wird für das *Heltec Wifo Kit 32* übersetzt.

#### ACHTUNG
Für die Platine Version V1.3 **unbedingt** den Branch V1.61 verwenden, **nicht** den Master-Branch. Es gibt für die neueren Platinen Änderungen am Pinning !

Als externe Libraries werden benötigt:
 * U8g2 von Oliver, aktuelle Version 2.16.4  
Falls der Compiler andere Libraries anmahnt, diese bitte in der Arduino IDE per *Sketch -> Include Library -> Manage Libraries ..* installieren. 

 
In dem Sourcecode ( in der Datei **userdefines.h**) können vor dem Übersetzen noch folgende Grundeinstellungen gemacht werden:
 * **#define CPU**  
Eine der drei mögliche CPUs durch auskommentieren auswählen.
 * **#define ROHRNAHME**  
Einen der möglichen Zählrohr-Namen durch auskommentieren auswählen.
 * **#define SERIAL_DEBUG**  
Einen der möglichen Debug Levels zur Ausgabe auf der seriellen Schnittstelle (USB) durch auskommentieren auswählen
 * **#define CONNECT_TIMEOUT**  
 Zeit in Sekunden, die das Programm versucht, sich mit den eingestellten WLAN zu verbinden (Standardwert ist 30 sec)
  * **#define WAIT_4_CONFIG**  
 Zeit in Skeunden, die das Programm wartet, bis sich ein Client auf den internen Access-Point verbindet (Standard 180sec) 
 * **#define SPEAKER_TICKS 0/1**  
Einschalten (1) oder Aussschalten (0) der Knackgeräusche.
 * **#define LED_TICK 0/1**  
Ein- oder Ausschalten des Blitzens der LED bei einem Zählpuls.
 * **#define PLAY_SOUND 0/1**  
 Wenn eingeschaltet, wird beim Restart ein Sound abgespielt 
 * **#define SEND2MADAVI 0/1**   
 Wenn der Wert auf 1 steht, werden die Daten zum Madavi-Server gesendet
 * **#define SEBD2LUFTDATEN 0/1**  
 Auch hier, wenn der Wert auf 1 steht, werden die Daten zum Luftdaten-Server gesendet. Dieser sollte immer auf 1 stehen, damit die Daten dort immer gespeichert werden.
 * **#define SEND2LORA 0/1**  
Steht der Wert auf 1, so wird via LoRa-WAN an TTN gesendet (die TTN-Konfiguartion muss in der Date **lorawan.cpp** erfolgen). Wird oben bei der CPU-Auswahl **WIFI** gewählt, so wird das Senden zu TTN automatisch abgeschaltet, da die Hardware ja dann kein LoRa an Board hat.
 * **#define DEBUG_SERVER_SEND 0/1**  
 Wenn auf 1, dann wird jedesmal beim Senden der Daten zum Server (madavi oder luftdaten) auf der seriellen Schnittstelle (USB) Debug-Info mit ausgegeben.

## Ablauf nach dem Start
Das Gerät baut einen eigene WLAN-Accesspoint (AP) auf. Die SSID des AP lautet **ESP32-xxxxxxxx**, wobei die xxx 
die Chip-ID (bzw. die MAC-Adresse) des WLAN-Chips sind (Beispiel: **ESP32-51564452**).  
**Bitte diese Nummer notieren, sie wird später noch gebraucht.**  
Dieser Access-Point bleibt für 30sec aktiv. Danach versucht das Gerät, sich mit dem (früher) eingestellten WLAN
zu verbinden. Dieser Verbindungsversuch dauert ebenfalls 30sec. Kommt keine Verbindung zu Stande, wird wieder der
eigene AP für 30sec erzeugt. Wenn das WLAN nicht erreicht werden kann, läuft dieses Spiel endlos.  
Solange keine Verbindung zum WLAN besteht, wird auf dem Display in der untersten Zeile ganz klein *connecting ...*
angezeigt. Diese Anzeige verschwindet, sobald eine WLAN-Verbindung hergestellt ist.

## Einstellung des WLAN
Wenn das Gerät den eigenen AP aufgebaut hat, verbindet man sich mit diesem. Entweder mit einem Handy oder einem PC o.ä.. 
Die Verbindung fragt nach einem Passwort, es lautet **ESP32Geiger**.  
Ist die Verbindung mit dem Accesspoint hergestellt, so bleibt das Timeout von 30sec stehen, d.h. man hat beliebig Zeit, die Daten einzugeben. Es öffnet sich **automatisch** die Startseite des Gerätes. Es braucht also - in der Regel - nicht extra der Browser aufgerufen werden. Falls die Startseite ausnahmsweise doch nicht erscheint, 
so muss mit dem Browser die Adresse **192.168.4.1** aufgerufen werden und nun erscheint die Startseite. Diese besteht nur aus einer Zeile *Go to __configure page__ to change settings*. Hier auf den blauen Teil klicken und man kommt zur Einstellungsseite:  

![config](/images/config1.png)  
Diese hat die folgenden 4 Zeilen:  
 * Thing Name  
 Dies ist die SSID des Gerätes und kann zwar geändert werden, sollte aber nicht!
 * AP password  
 Dies ist das Passwort für den AP. Dieses **MUSS** beim ersten mal geändert werden. Es kann natürlich auch das gleiche Passwort wieder verwendet werden - wichtig ist nur, dass da was reingeschrieben wird und dass man das **nicht vergessen** darf.
 * WiFi SSID  
 Hier muss nun die SSID des eigenen WLANs eingegeben werden.
 * WiFi password
 Und hier das zugehörige Passwort.
 
Ist alles eingegeben, kann man auf **Apply** drücken. Nun werden die eingestellten Daten übernommen und in das interne EEPROM gespeichert. Nun bitte **unbedingt** über **Abbrechen** diese Seite verlassen! Nur dann verlässt das Programm den Config-Mode und verbindet sich mit dem heimischen WLAN. Wenn es kein **Abbrechen** gibt, dann wieder zurück in die WLAN-Einstellungen des Gerätes gehen und da dann das normale Heim-Netzwerk wieder einstellen.

## Server
Der Messzyklus beträgt 10min, d.h. es werden 10min lang die Impulse gezählt und dann die "Counts per Minute" (cpm) berechnet. 
Jeweils nach diesen 10min werden die Daten zu den Servern bei *luftdaten.info* und bei *madavi.de* gesendet.  
Bei *luftdaten* werden die Daten gespeichert und stehen am nächsten Tag zum Abruf als CSV-Datei bereit:  
http://archive.luftdaten.info/date/date_radiation_sbm-20_sensor_SID.csv  
wobei date = Datum im Format YYYY-MM-DD ist (beides mal gleich) und SID die Sensornummer des Sensors (**nicht** die ChipID).   
Bei *madavi* werden die Daten in einer RRD-Datenbank abgelegt und können direkt aktuell als Grafik über diesen Link betrachtet werden:  
https://www.madavi.de/sensor/graph.php?sensor=esp32-CHIPID-sbm20  
Hier ist dann CHIPID die ChipId (also die Ziffern der SSID des internen Accesspoints).  
Während der Übertragung der Daten zu den Servern wird in der Statuszeile (unterste Zeile) des Displays kurz der Name des Servers eingeblendet.

## Anmeldung bei luftdaten.info
Damit die Daten, die der Sensor nach luftdaten.info schickt, von dem Server auch angenommen werden, muss man sich dort anmelden. Das geschieht über die Seite https://meine.luftdaten.info.  
Zuerst über den *Registrieren*-Knopf einen Account anlegen. Dann damit über *Login* einloggen und *Neuen Sensor registrieren* anklicken.  
Dann das Formular ausfüllen:
 * Erste Zeile, Sensor ID:  
 Hier die Nummer (nur die Zahlen) der SSID des Sensors eingeben (z.B. bei dem Sensor ESP-51564452 also dann nur 51564452 eingeben) 
 * Zweite Zeile, Sensor Board:  
 Hier *esp32* auswählen (über die kleinen Pfeile rechts)
 * Basisinformation:  
 Hier die Adresse eingeben (mit dem Land!). Der interne Name des Sensors kann beliebig vergeben werden, muss aber eingegeben werden. Bitte den Haken bei **Indoor-Sensor** setzen, so lange der Sensor wirklich innen ist.
 * Zusätzliche Informationen:  
 Kann freigelassen werden, darf aber auch ausgefüllt werden.
 * Hardware-Konfiguration:  
 Hier als Sensor-Typ den Eintrag **Radiation SBM-20** (oder ggf. entsprechend) auswählen. Für den zweiten Sensor kann DHT22 stehen bleiben, das ist für uns irrelevant.
 * Position  
 Hier bitte die Koordinaten eingeben, so genau wie möglich (oder über den rechten Knopf die Koordinaten rechnen lassen). Dies wird benötigt, um den Sensor später auf der Karte anzeigen zu können.

 Nun mit *Einstellungen speichern* das Ganze beenden. Dann auf der Übersichts-Seite  bei diesem Sensor auf *Daten* klicken. Nun steht hinter *Sensor ID* die ID des Sensors. Diese bitte merken: sie wird für die Abfrage bei luftdaten.info bzw. bei der Anzeige auf https://geiger.rexfue.de benötigt (zur Zeit heißt das noch https://test1.rexfue.de und ist laufend in Arbeit - kann also ab und zu ausfallen :wink: )





 
