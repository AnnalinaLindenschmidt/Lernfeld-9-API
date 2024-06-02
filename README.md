# Deployment einer Web-Applikation #
Im Folgenden wird erklärt, wie ein Server mit dem Betriebssystem Linux (Raspberry Pi OS) eingerichtet werden muss, damit das Deployment einer Web-Applikation – der Todo-Listen-Verwaltung – möglich ist. Dafür muss der Server einige Anforderungen erfüllen: Zum einen muss eine Netzwerkkonfiguration des Raspberry Pis mit einer statischen IP-Adresse im lokalen Subnetz stattfinden, zum anderen sollen zwei Benutzer angelegt werden. Am Schluss erfolgt dann das Deployment der Web-App „Todo-Listen-Verwaltung“ als Container.
Die Umsetzung der einzelnen Schritte wird in den nachfolgenden Kapiteln genauer erläutert.
# Statische IP-Adresse vergeben #
Als Server werden Raspberry Pis mit der Version Raspberry Pi OS Bookworm als Betriebssystem von der Schule zur Verfügung gestellt, die bereits im Local Area Network (LAN) eine eigene IP-Adresse haben. Um diese bisherige IP-Adresse neu zu konfigurieren, müssen die nachfolgenden Schritte durchgeführt werden.
## Raspberry Pi an Strom und Netzwerk anschließen ##
Im Vorfeld muss eine eigene Speicherkarte (Micro-SD-Karte) vorhanden sein, auf die ein Betriebssystem-Image gespielt wird sowie eine freie Hostadresse (hier: _192.168.24.181_) in dem Subnetz, in dem gearbeitet werden soll.
Danach muss sich der Laptop, der für die Konfiguration des Raspberry Pis verwendet wird, mit dem Raum-WLAN verbinden. Es erfolgt dann das Anschließen des Raspberry Pis an Strom mit Stromkabel und Netzwerk mithilfe eines LAN-Kabels. Auch die eigene Micro-SD-Karte, mit dem zuvor darauf gespielten Betriebssystem-Image, muss in den Raspberry Pi eingelegt werden.
## Einloggen in den Raspberry Pi ##
Wie oben bereits erwähnt, werden Raspberry Pis von der Schule zur Verfügung gestellt. Das bedeutet, dass man sich zunächst mit dem Benutzer _pi_ und der bisherigen IP-Adresse anmelden muss. Dafür öffnet man auf dem Laptop die Command Line (Befehlsleiste) und gibt `ssh pi@192.168.24.xxx` an, wobei _xxx_ für den letzten Block der bisherigen IP-Adresse steht, der bei den verschiedenen Raspberry Pis der Schule entsprechend unterschiedlich ist. Nach der Bestätigung der Eingabe mit Enter wird die Frage gestellt, ob man sich sicher ist, die Verbindung fortsetzen zu wollen (_Are you sure you want to continue connecting?_). Diese Frage muss, um fortfahren zu können, mit `yes` beantwortet werden. Es folgt die Passwortabfrage für den Benutzer. Genauso wie der Benutzer für die schuleigenen Raspberry Pis vorgegeben ist, so ist ebenfalls das Passwort standardmäßig mit _raspberry_ vorbelegt. Es muss also `raspberry` in die Befehlsleiste eingegeben werden. Die Buchstaben des Passwortes werden dabei allerdings nicht in der Befehlsleiste angezeigt.
## Statische IP-Adresse vergeben ##
Im ersten Schritt sollten das Standard-Gateway und die Adresse des DNS-Servers ermittelt werden. Dafür kann der Befehl `ip r` verwendet werden. Als Ergebnis erhält man in diesem Fall die Adresse _192.168.24.0/24_ sowohl für das Standard-Gateway als auch für den DNS-Server. Diese Adressen werden ebenfalls für die Vergabe einer statischen IP-Adresse benötigt.
Durch die Verwendung der Version Raspberry Pi OS Bookworm als Betriebssystem wird der NetworkManager als Standard-Controller für die Netzwerkfunktionalität verwendet. Der NetworkManager enthält ein Kommandozeilentool namens _nmcli_, das hier verwendet wird, um die statische IP-Adresse zu konfigurieren.
Als nächstes muss der Befehl `sudo nmcli -p connection show` angegeben werden. Damit erhält man eine Liste der Verfügbaren Netzwerkschnittstellen und kann damit ermitteln, wie der Name der Netzwerkschnittstelle lautet, die als statisch festgelegt werden soll. Für den richtigen Namen muss in der Zeile der Liste geschaut werden, in der der Type _ethernet_ ist – der Name lautet dann also _Wired connection 1_.
Es folgt nun die Aktualisierung dieser Netzwerkverbindung, indem drei Befehle gesendet werden, in denen die neue IP-Adresse, die Standard-Gateway-Adresse und die DNS-Server-Adresse aktualisiert werden.
```
sudo nmcli c mod “Wired connection 1“ ipv4.addresses 192.168.24.181/24 ipv4.method manual
sudo nmcli con mod “Wired connection 1“ ipv4.gateway 192.168.24.0/24
sudo nmcli con mod “Wired connection 1“ ipv4.dns 192.168.24.0/24
```
Nach der Aktualisierung sollte ein Neustart vorgenommen werden (`sudo reboot`), da ggf. erst dann die Änderungen greifen. Somit ist die statische IP-Adresse vergeben.
# Zwei lokale Benutzer anlegen #
Nachfolgend wird beschrieben, wie die zwei lokalen Benutzer „willi“ und „fernzugriff“ angelegt werden. Der Benutzer „willi“ soll ein normaler Benutzer ohne Administratorrechte sein, der Benutzer „fernzugriff“ soll ein Benutzer für den Zugriff von außen mittels SSH mit sudo-Rechten sein. Dafür muss es für den Benutzer „fernzugriff“ einen SSH-Dienst zur Administration geben.
## „willi“ und „fernzugriff“ anlegen ##
Für das Anlegen der beiden Benutzer empfiehlt es sich, mit den sudo-Rechten zu arbeiten. Sudo steht für „super user do“. Für das Arbeiten mit sudo-Rechten muss der Befehl `sudo -i` in die Befehlsleiste eingegeben werden.
Als nächstes werden die beiden Benutzer mit einem Home-Verzeichnis angelegt. Dafür wird der Befehl `useradd -m {USERNAME}` verwendet. _USERNAME_ muss dann durch „willi“ bzw. „fernzugriff“ ersetzt werden. Im Anschluss daran erfolgt mit `passwd {USERNAME} xxx` eine Passwortvergabe für die beiden Benutzer. Dies ist nötig, da man sich sonst mit den Benutzern nicht anmelden kann. Das _xxx_ steht stellvertretend für ein Passwort, das bei der Eingabe allerdings nicht angezeigt wird. _USERNAME_ muss wieder durch die beiden Benutzernamen ersetzt werden.
Nach dem Anlegen der Benutzer könnte man sich dann mit dem Befehl `cut -d: -f1/etc/passwd` alle vorhandenen Benutzer anzeigen lassen.
## „fernzugriff“ sudo-Rechte geben ##
Für die Vergabe von sudo-Rechten muss die sudoers-Datei bearbeitet werden. Durch den Befehl `sudo visudo` kann man diese Datei editieren. Am Ende der Datei muss die folgende Zeile hinzugefügt werden: `fernzugriff ALL=(ALL) NOPASSWD:ALL`.
Mit „Strg + X“ und dann der Angabe von „Y“ sowie durch das Betätigen der Enter-Taste wird die Änderung gespeichert und die Datei wird wieder geschlossen.
## SSH-Dienst für „fernzugriff“ einrichten ##
Um für den Benutzer „fernzugriff“ einen SSH-Dienst zur Administration einrichten zu können, sollte zunächst geschaut werden, dass der SSH-Dienst bereits aktiviert ist und läuft.
Dafür kann der Befehl `sudo systemctl status ssh` verwendet werden. Sollte der SSH-Dienst nicht installiert sein, so muss dieser noch installiert und aktiviert werden.
Dafür werden die folgenden Befehle verwendet:
```
sudo apt update
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```
Durch die Befehle geschieht Folgendes:
- Raspberry Pi OS wird auf neuesten Stand gebracht
- OpenSSH-Server wird installiert
- SSH wird in Zukunft automatisch gestartet nach Reboot
- SSH-Dienst wird gestartet
## SSH-Zugriff konfigurieren ##
Als nächstes muss der SSH-Zugriff für den Benutzer „fernzugriff“ konfiguriert werden. Dafür muss die SSH-Konfigurationsdatei bearbeitet werden. In den Bearbeitungsmodus gelangt man durch den Befehl `sudo nano /etc/ssh/sshd_config`. Um den SSH-Zugriff einzig auf den Benutzer „fernzugriff“ zu beschränken, muss die Zeile `AllowUsers fernzugriff` entweder der Datei hinzugefügt oder aber, wenn schon vorhanden, für den Benutzer „fernzugriff“ geändert werden. Auch hier erfolgt dann mit „Strg + X“ und dann der Angabe von „Y“ sowie durch das Betätigen der Enter-Taste das Bestätigen der Änderung und die Datei wird geschlossen.
Mit dem Befehl `sudo systemctl restart ssh` sollte anschließend der SSH-Dienst neu gestartet werden, um die Änderungen zu übernehmen.
Damit ist die Einrichtung des SSH-Dienstes für den Benutzer „fernzugriff“ abgeschlossen.
# Deployment der Web-App „Todo-Listen-Verwaltung“ als Container #
Für das Deployment der Web-App mit Docker wurden die dafür notwendigen Dateien als Vorlage zur Verfügung gestellt, die entsprechend der Todo-Listen-Verwaltung hätte angepasst werden sollen. Aber aus zeitlichen Gründen konnte dies nicht mehr erfolgen..
