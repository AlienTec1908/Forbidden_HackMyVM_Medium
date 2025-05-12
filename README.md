# Forbidden - HackMyVM (Medium) - Writeup

![Forbidden Icon](Forbidden.png)

Dieses Repository enthält einen zusammengefassten Bericht über die Kompromittierung der HackMyVM-Maschine "Forbidden" (Schwierigkeitsgrad: Medium).

## Metadaten

*   **Maschine:** Forbidden (HackMyVM - Medium)
*   **Link zur VM:** [https://hackmyvm.eu/machines/machine.php?vm=Forbidden](https://hackmyvm.eu/machines/machine.php?vm=Forbidden)
*   **Autor des Writeups:** DarkSpirit
*   **Datum:** 8. Oktober 2022
*   **Original Writeup:** [https://alientec1908.github.io/Forbidden_HackMyVM_Medium/](https://alientec1908.github.io/Forbidden_HackMyVM_Medium/)

## Zusammenfassung des Angriffspfads

Die initiale Erkundung mit `nmap` identifizierte einen offenen FTP-Dienst (Port 21, vsftpd 3.0.3) mit erlaubtem anonymen Login und einem beschreibbaren `www`-Verzeichnis sowie einen Nginx-Webserver (Port 80). Eine Analyse des FTP-Servers (`wget -r`) enthüllte eine `note.txt` von Benutzerin `marta`, die behauptete, PHP sei deaktiviert, aber ihr Passwort sei in einer `.jpg`-Datei versteckt.

Trotz der Notiz wurde eine PHP-Reverse-Shell mit der Endung `.php5` erfolgreich über FTP in das `www`-Verzeichnis hochgeladen (`put image.php5`). Der Aufruf dieser Datei via HTTP (`curl http://forbidden.hmv/image.php5`) führte zur Ausführung und gewährte initialen Zugriff als `www-data`.

Die Rechteausweitung von `www-data` zu `markos` erfolgte durch Ausführen eines SUID/SGID-gesetzten Binaries (`.forbidden`) im Home-Verzeichnis von `marta`. Als `markos` wurde die User-Flag gelesen. In Anlehnung an `marta`s Notiz wurde der Dateiname `TOPSECRETIMAGE` (ohne `.jpg`) als Passwort für `marta` erraten und erfolgreich mit `su marta` verwendet (Laterale Bewegung).

Als `marta` zeigte `sudo -l` die Berechtigung, `/usr/bin/join` ohne Passwort auszuführen. Dies wurde genutzt, um den Inhalt von `/etc/shadow` auszulesen (`sudo join ...`). Der Passwort-Hash für den Benutzer `peter` wurde extrahiert und offline mit `john` und `rockyou.txt` geknackt (Passwort: `boomer`). Mittels `su peter` wurde zu diesem Benutzer gewechselt (Laterale Bewegung).

Schließlich zeigte `sudo -l` für `peter` die Berechtigung, `/usr/bin/setarch` ohne Passwort auszuführen. Dies wurde mit `sudo setarch $(arch) /bin/bash` ausgenutzt, um eine Root-Shell zu erhalten und die Root-Flag auszulesen.

## Verwendete Tools (Auswahl)

*   `arp-scan`
*   `vi` (implied)
*   `nmap`
*   `wget`
*   `cat`
*   `mv` (implied for renaming shell?)
*   `ftp` (`ls`, `cd`, `put`)
*   `grep`
*   `nc` (netcat)
*   `curl`
*   `python3` (`pty` module)
*   `export`
*   `find`
*   `su`
*   `sudo`
*   `join`
*   `john`
*   `setarch`
*   `bash`
*   `ls`, `cd`

## Angriffsschritte (Übersicht)

1.  **Reconnaissance:** Ziel-IP (`arp-scan`), Hostname (`forbidden.hmv` in `/etc/hosts`), Portscan (`nmap`) -> Port 21 (FTP, anon, writable `www`), Port 80 (HTTP, Nginx).
2.  **FTP Enumeration & Analyse:** Inhalt via `wget -r` laden. `note.txt` finden (von `marta`, erwähnt deaktiviertes `.php`, Passwort in `.jpg`). `TOPSECRETIMAGE.jpg` im Web-Root identifizieren.
3.  **Upload & Initial Access:** PHP-Reverse-Shell als `image.php5` via FTP in `www`-Verzeichnis hochladen (`put`). `nc`-Listener starten. Shell via `curl http://forbidden.hmv/image.php5` auslösen -> Shell als `www-data`. Stabilisieren (`python pty`).
4.  **Privilege Escalation (`www-data` -> `markos`):** SUID/SGID-Binary `/home/marta/.forbidden` finden und ausführen -> Shell als `markos`.
5.  **Lateral Movement (`markos` -> `marta`):** User-Flag (`/home/markos/user.txt`) lesen. Passwort `TOPSECRETIMAGE` für `marta` raten. Mit `su marta` wechseln.
6.  **Lateral Movement (`marta` -> `peter`):** `sudo -l` -> `NOPASSWD: /usr/bin/join`. `/etc/shadow` mittels `sudo join ...` auslesen. Hash für `peter` extrahieren. Hash mit `john` knacken (`boomer`). Mit `su peter` wechseln.
7.  **Privilege Escalation (`peter` -> `root`):** `sudo -l` -> `NOPASSWD: /usr/bin/setarch`. Mit `sudo setarch $(arch) /bin/bash` Root-Shell erlangen.
8.  **Flags:** Root-Flag (`/root/root.txt`) lesen.

## Flags

*   **User Flag (`/home/markos/user.txt`):** `HMVpussycat`
*   **Root Flag (`/root/root.txt`):** `HMVmymymymymind`

---

## Disclaimer

Die hier dargestellten Informationen und Techniken dienen ausschließlich Bildungs- und Forschungszwecken im Bereich der Cybersicherheit. Die beschriebenen Methoden dürfen nur auf Systemen angewendet werden, für die eine ausdrückliche Genehmigung vorliegt (z.B. in CTF-Umgebungen wie HackMyVM, Penetrationstests mit schriftlicher Erlaubnis). Das unbefugte Eindringen in fremde Computersysteme ist illegal und strafbar. Die Autoren übernehmen keine Haftung für missbräuchliche Verwendung der bereitgestellten Informationen. Handeln Sie stets legal und ethisch verantwortlich.

---
