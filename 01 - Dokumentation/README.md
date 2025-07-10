# Projektdokumentation – M158 LB2 <br>WordPress-Migration

## Übersicht
# Projektdokumentation – Webserver-Projekt

Diese Dokumentation ist nach den Aufgaben (1–14) gegliedert. Jede Aufgabe ist in drei Phasen unterteilt. Bitte tragen Sie Ihre Ergebnisse jeweils unter den entsprechenden Abschnitten ein.

---

## Aufgabe 1 – Projektplan erstellen


### Stufe 3

```mermaid
gantt
    title Projektplan Software-Migration
    dateFormat  DD-MM-YYYY
    section Entwicklung & Testing
    Projektplanung : a1, 16-05-2025  , 7d
    Dokumentation / Arbeitsjournal : after a1, 49d
    section Phase 2
    AWS : b2, after a1, 7d
    Snapshot 1  : milestone, sn1
    section Phase 3
    OS-konf : c1, after b2, 7d
    Snapshot 2 : milestone, sn2
    Webserver / DB (PhpMyAdmin) : c2, after c1, 7d
    Snapshot 3 : milestone, sn3
    V-Host : c3, after c2, 7d
    Snapshot 4 : milestone, sn4
    DNS-Server : c4, after c3, 7d
    Snapshot 5 : milestone, sn5
    section Phase 4
    Migr-WP-Files : d1, after c4, 7d
    Migr-WP-DB : d2, after c4, 7d
    WP-Config : d3, after c4, 7d
    Snapshot 6 : milestone, sn6
    section Phase 5
    Docker Compose : e1, after d3, 7d
    Testing : e2, after d3, 7d
```

---

## Aufgabe 2 – Architekturdiagramm erstellen

### Stufe 3

![Architekturdiagram](media/Architekturdiagram.png)

[Hier](<../03 - Diverses/Architekturdiagram.drawio>) ist noch die Datei

---

## Aufgabe 3 – AWS-Umgebung einrichten

### Stufe 1

Die Instanz ist erstellt und man kann per ssh drauf zugreifen.

#### Webserver

![Instanz](media/Instanz.png)

![SSH](media/SSH.png)

#### DNS

![Instanz2](media/Instanz2.png)

![SSH2](media/SSH2.png)

### Stufe 2

Hier sieht man auch die korrekte Konfiguration von den IPs und auch Port Forwarding.

#### Webserver

![IPs](media/IPs.png)

![portforward](media/portforward.png)

#### DNS

![IPs2](media/IPs2.png)

![portforward2](media/portforward2.png)

---

## Aufgabe 4 – DNS-Konfiguration

Ändern Sie die Stufe, für die Sie sich entschieden haben, selbst.

### Stufe 3

Fügen Sie hier Ihre Ergebnisse ein

Ich habe als DNS Server Bind9 verwendet auf einer Ubuntu EC2 Instanz.

Dabei habe ich eine Zone für sybhad.internal erstellt.

die Konfiguration sieht so aus:

```
$TTL 604800
@   IN  SOA dns.sybhad.internal. admin.sybhad.internal. (
            2025061301 ; Serial
            604800     ; Refresh
            86400      ; Retry
            2419200    ; Expire
            604800 )   ; Negative Cache TTL
@   IN  NS  dns.sybhad.internal.
dns IN  A   3.87.81.158
@   IN  A   3.218.233.70
```

Hier sieht man wie es korrekt resolved

![resolve](media/resolve.png)

---

## Aufgabe 5 – Webserver konfigurieren

### Stufe 2

Hier sieht man meine V-Host config. Dort ist ssl aktiviert.

```apache
<VirtualHost *:443>
    ServerName sybhad.internal
    DocumentRoot /var/www/html

    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/fullchain.pem
    SSLCertificateKeyFile /etc/ssl/private/privkey.pem

    <Directory /var/www/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

        # Client-Caching für statische Dateien
    <IfModule mod_expires.c>
        ExpiresActive On
        ExpiresByType image/jpg "access plus 1 month"
        ExpiresByType image/jpeg "access plus 1 month"
        ExpiresByType image/gif "access plus 1 month"
        ExpiresByType image/png "access plus 1 month"
        ExpiresByType text/css "access plus 1 month"
        ExpiresByType application/javascript "access plus 1 month"
        ExpiresByType image/x-icon "access plus 1 year"
        ExpiresDefault "access plus 2 days"
    </IfModule>

        ErrorLog /var/log/apache2/m158_error.log
        CustomLog /var/log/apache2/m158_access.log combined


</VirtualHost>

<VirtualHost *:80>
    ServerName sybhad.internal
    Redirect permanent / https://sybhad.internal/
</VirtualHost>
```

### Stufe 3

Hier ist auch noch die https weiterleitung

```apache
<VirtualHost *:80>
    ServerName sybhad.internal
    Redirect permanent / https://sybhad.internal/
</VirtualHost>
```

Hier sieht man auch wie man per https auf die Webseite zugreifen kann.

Und die standard-Seite ist auch weg.  

![https](media/https.png)

---

## Aufgabe 6 – PHP einrichten

### Stufe 1

Hier sieht man das php auf meinem apache container installiert ist. Abgerufen mit dem Befehl ```php -version```

![phpversion](media/phpversion.png)

Hier auch noch das phpinfo.php file mit meiner Domain.

![phpinfo](media/phpinfo.png)

### Stufe 2

Das php.ini, bei mir custom.ini genannt, ist im apache_php abgelgt.

Diese hat folgende konfiguration gespeichert:

```php
upload_max_filesize = 128M
post_max_size = 128M
memory_limit = 256M
max_execution_time = 300
max_input_time = 300
```

Hier noch die datei: [custom.ini](<../02 - WordPress/php/custom.ini>)

### Stufe 3



---

## Aufgabe 7 – MySQL/MariaDB aufsetzen

### Stufe 2

Hier sieht man alle Nutzer der Datenbank.

Root ist nur von localhost erreichbar.

Für Wordpress hat es den dezidierten Benutzer M158

```mysql
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| M158             | %         |
| root             | %         |
| mysql.infoschema | localhost |
| mysql.session    | localhost |
| mysql.sys        | localhost |
| root             | localhost |
+------------------+-----------+
```

---

## Aufgabe 8 – Web-Datenbanktool (phpMyAdmin/Adminer)

### Stufe 1

Hier sieht man das phpmyadmin läuft mit meiner Domain.

![phpmyadmin](media/phpmyadmin.png)

### Stufe 2

Ich habe SSL aktiviert in dem ich im phpmyadmin container eine eigene V-Host config erstellt habe.

Dabei habe ich das Zertifkat verwendet welches ich bereits bei der Shop-Webseite verwendet habe.

```apache
<VirtualHost *:443>
    ServerName sybhad.internal
 
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/fullchain.pem
    SSLCertificateKeyFile /etc/ssl/private/privkey.pem
 
    DocumentRoot /var/www/html
<Directory /var/www/html>
        AllowOverride All
</Directory>
</VirtualHost>
```

Hier das File: [default-ssl.conf](<../02 - WordPress/phpmyadmin/default-ssl.conf>)

Hier sieht man wie phpmyadmin auf meiner domain mit https läuft.

![sslphpmyadmin](media/sslphpmyadmin.png)

---

## Aufgabe 9 – FTP-Zugang einrichten

### Stufe 1

Fügen Sie hier Ihre Ergebnisse ein

### Stufe 2

Fügen Sie hier Ihre Ergebnisse ein

---

## Aufgabe 10 – WordPress migrieren

### Stufe 1

Hier sieht man in phpmyadmin wie die Datenbank und Tabellen erfolgreich importiert wurden.

![dbimport](media/dbimport.png)

Hier sieht man noch die WP-Files inklusive Berechtigungen im Document Root des apache Containers.

![wpfiles](media/wpfiles.png)

### Stufe 2

Hier sieht man die benötigte DB konfiguration im wp-config

```php
// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress_db' );

/** Database username */
define( 'DB_USER', 'M158' );

/** Database password */
define( 'DB_PASSWORD', 'Nevio1234' );

/** Database hostname */
define( 'DB_HOST', 'mysql_db:3306' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );
```

Das ist noch das ganze File [wp-config](<../02 - WordPress/html/wp-config.php>)

### Stufe 3

Hier sieht man WP_HOME und WP_SITEURL die auf meine Domain angepasst wurden.

![url](media/url.png)

Die Webseite funktioniert und zeigt auch die korrekte Domain an.

![https](media/https.png)

## Aufgabe 11 – Backup-Konzept umsetzen

### Stufe 1

Fügen Sie hier Ihre Ergebnisse ein

### Stufe 2

Fügen Sie hier Ihre Ergebnisse ein

---

## Aufgabe 12 – Testing der Webapplikation

### Stufe 1

Fügen Sie hier Ihre Ergebnisse ein

### Stufe 2

Fügen Sie hier Ihre Ergebnisse ein

### Stufe 3

Fügen Sie hier Ihre Ergebnisse ein

---

## Aufgabe 13 – Deployment automatisieren

### Stufe 1

Fügen Sie hier Ihre Ergebnisse ein

### Stufe 2

Fügen Sie hier Ihre Ergebnisse ein

### Stufe 3

Fügen Sie hier Ihre Ergebnisse ein

---

## Aufgabe 14 – Docker verwenden

### Stufe 2

Ich habe den Webserver, die Datenbank und phpmyadmin in Docker umgesetzt mit dem bereitgestellten docker-compose und Dockerfile.

Hier sieht man die laufenden container:

![dockerps](media/dockerps.png)

Hier sind noch die files:
[docker-compose](<../02 - WordPress/docker-compose.yml>)

[dockerfile](<../02 - WordPress/dockerfile>)

---

