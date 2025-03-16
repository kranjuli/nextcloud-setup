# nextcloud-setup

Guides to install NextCloud server on Raspberry Pi, to upload pictures oder videos from smartphone to NextCloud automatically

## Setup

Install NextCloud auf Raspberry Pi

1. Raspberry Pi vorbereiten

````
sudo apt update
sudo apt upgrade -y
sudo apt dist-upgrade -y
sudo apt autoremove -y
````

2. Abhängigkeiten installieren

````
sudo apt install apache2 php libapache2-mod-php mariadb-server php-mysql php-xml php-zip php-gd php-curl php-mbstring php-bcmath php-json php-ldap wget unzip -y
````

3. MariaDB (MySQL) einrichten

````
sudo msql

CREATE DATABASE nextcloud;
CREATE USER 'nextclouduser'@'localhost' IDENTIFIED BY 'deinPasswort';
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextclouduser'@'localhost';
FLUSH PRIVILEGES;
EXIT;

sudo systemctl start mariadb
sudo systemctl enable mariadb
````

4. Nextcloud herunterladen und installieren

````
cd /var/www/
sudo wget https://download.nextcloud.com/server/releases/nextcloud-24.0.0.tar.bz2  # Ersetze die Version, wenn nötig
sudo tar -xjf nextcloud-*.tar.bz2
sudo chown -R www-data:www-data nextcloud/
sudo chmod -R 755 nextcloud/
`````

5. Apache konfigurieren

Öffne die Apache-Konfigurationsdatei für Nextcloud:

````
sudo nano /etc/apache2/sites-available/nextcloud.conf
````
Füge sicherheitshalber den folgenden Inhalt ein (falls du das noch nicht getan hast):

````
<VirtualHost *:80>
  # ServerAdmin admin@deinedomain.com
  DocumentRoot /var/www/nextcloud
  ServerName deineIP_oder_domai(localhost)

  <Directory /var/www/nextcloud>
    Options +FollowSymLinks
    AllowOverride All
    Require all granted
  </Directory>

</VirtualHost>
````

````
sudo a2ensite nextcloud.conf
sudo systemctl reload apache2

sudo a2dissite 000-default.conf

sudo a2enmod rewrite headers env dir mime
sudo systemctl restart apache2
````

6. Nextcloud im Browser einrichten

Gehe zu deinem Raspberry Pi im Browser (z.B. http://raspberrypi.local oder http://deineIP).

Du solltest nun den Nextcloud-Installationsassistenten sehen. Gib deine Datenbankinformationen ein: Datenbankname nextcloud, Benutzername nextclouduser, Passwort (das du zuvor erstellt hast). Führe die Installation fort und erlaube, dass Nextcloud die Dateien und Datenbankverbindung prüft.

7. Zugriff vom iPhone

7.1. Nextcloud App auf dem iPhone installieren

Falls du dies noch nicht getan hast, lade die Nextcloud-App aus dem App Store herunter und installiere sie.

7.2. Mit dem Nextcloud-Server verbinden

Öffne die Nextcloud-App auf deinem iPhone.
Gib die URL deines Nextcloud-Servers ein (z.B. http://192.168.1.100 oder https://deinedomain.duckdns.org).
Melde dich mit deinem Benutzernamen und Passwort an, um dich mit deinem Nextcloud-Server zu verbinden.

7.3. Automatischen Upload aktivieren

Jetzt, da du mit deinem Nextcloud-Server verbunden bist, kannst du die automatische Foto- und Video-Upload-Funktion aktivieren:

7.3.1 Öffne die Nextcloud App und gehe zu den Einstellungen:

Tippe auf das Menü-Symbol (oben links in der App).
Wähle Einstellungen aus.

7.3.2 Aktivierung des automatischen Foto-Uploads:

Wähle „Automatischer Foto-Upload“ aus den Einstellungen.

7.3.3 Wählen des Ordners für den Upload:

Du kannst auswählen, in welchem Ordner auf deinem Nextcloud-Server die Fotos gespeichert werden sollen. Meistens wird ein Ordner namens „Fotos“ oder „Uploads“ empfohlen.
Ordner auswählen: Wähle den Ordner aus, in dem deine Fotos gespeichert werden sollen. Zum Beispiel: Fotos oder Uploads.

7.4. Fotos und Videos manuell hochladen (optional)

Wenn du den Upload für bestimmte Fotos oder Videos starten möchtest, die du bereits aufgenommen hast, kannst du dies manuell tun:

Öffne die Fotos-App auf deinem iPhone. Wähle die Fotos oder Videos aus, die du hochladen möchtest. Tippe auf das Teilen-Symbol und wähle Nextcloud aus, um die Dateien direkt hochzuladen.
	
8. SSL (Optional, aber empfohlen)

Für eine sichere Verbindung kannst du ein SSL-Zertifikat einrichten (z.B. mit Let's Encrypt). Hierzu kannst du certbot verwenden:

````
sudo apt install certbot python3-certbot-apache -y
sudo certbot --apache
````