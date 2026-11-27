# Architektur
## FHS
### `/bin` / `/sbin`
> in modernen Systemen nur noch Verweise (Symlinks) auf /usr/bin und /usr/sbin 
> (systemweite Befehle und Admin-Befehle, vom Paketmanager verwaltet)
### `/boot` 
> `/boot` beinhaltet Bootloader-Dateien, Linux-Kernel und Initramfs 
> (Start-Tresor des Rechners) 

> Dateien:
> 
> `vmlinuz` komprimierter Linux-Kernel 
> 
> `initrd.img`(Initial RAM Disk) temporäres "Mini-Linux" im Arbeitsspeicher, wird gebraucht, um Treiber für die echte Festplatte zu laden
> 
> `config` Bauanleitung des Kernels 
> (welche Funktionen und Treiber beim Erstellen ein- oder ausgeschaltet wurden)
> 
> `System.map` verknüpft unverständliche Speicheradressen wie 0xffffffff mit echten Funktionsnamen für das Debugging
>  (Telefonbuch des Kernels)
> 
> `grub/` Menü- und Steuerungszentrum des Bootloaders (bestimmt, welches Betriebssystem beim Anschalten geladen wird)
### `/dev` 
> `/dev` steht für Gerätedateien (Device Files)
> -> Schnittstelle zwischen User-Space-Prozessen und dem Kernel/Hardware. 
> -> Unter Linux gilt "alles ist eine Datei". Hardware wird im FHS nicht als physisches Objekt, sondern als virtuelle Gerätedatei (Device Node) repräsentiert.

> 1) Block Devices (Kennzeichnung: b)
> 
> -> liest und schreibt Daten in festen Blöcken (individuell adressierbar).
> -> Nutzung: Speichermedien wie NVMe, SSDs, HDDs, Partitionen, USB-Sticks, CD-ROMs. 
> 
> -> Beispiele: 
> `/dev/sda` erste SATA/SAS/USB-Festplatte 
> `/dev/sda1` erste Partition der ersten Festplatte
> `/dev/nvme0n1p1` erste Partition auf dem ersten NVMe-Laufwerk

> 2) Character Devices (Kennzeichnung: c)
> 
> -> liest und schreibt Daten fortlaufend, Zeichen für Zeichen (gepuffert oder ungepuffert, nicht adressierbar).
> -> Nutzung: Eingabegeräte, serielle Schnittstellen, virtuelle Terminals. 
> 
> -> Beispiele: 
> `/dev/console` Haupt-Systemkonsole 
> `/dev/tty1` erstes virtuelles Terminal /
> `/dev/ttyS0` erste serielle Schnittstelle

> 3) Pseudo-Devices (Spezialdateien) 
> 
> -> virtuelle Geräte ohne physische Hardware-Entsprechung für logische Kernel-Aktionen. 
> `/dev/null` verwirft alle hineingeschriebenen Daten spurlos
> `/dev/zero` liefert bei Lesezugriff unendlich viele Null-Bytes (z. B. zum Nullen von Festplatten oder Erstellen leerer Dateien mit dd). 
> `/dev/urandom` nicht-blockierender Kernel-Zufallszahlengenerator (Krypto, Schlüssel-Generierung)

> **Praxis-Identifikation**
> Unterscheidungsmerkmal: erstes Zeichen der Dateirechte-Spalte (b = Block, c = Character)
```bash
ls -l /dev/sd* /dev/tty*
```
### `/etc` 
#### Allgemein
> `/etc` (Editable Text Configuration / früher "et cetera")
> Zentrales Verzeichnis für systemweite Konfigurationsdateien (reine Textdateien). Steuert Verhalten des Gesamtsystems und aller Programme

> Benennungsmuster:
>
> ohne Endung / ad hoc: Zentrale Systemdateien (z. B. passwd, shadow, hosts, hostname, fstab)
> 
> `.conf` klassische Konfigurationsdateien (z. B. resolv.conf, sysctl.conf)
> 
> `.rc` / `rc-Dateien` Run-Control / Initialisierungsskripte (z. B. bash.bashrc)
> 
> `.d/` modulare Konfigurationsverzeichnisse (z. B. cron.d/, sources.list.d/, logrotate.d/)
> erlauben Paketen eigene Konfigurationen hinzuzufügen, ohne Hauptdateien zu überschreiben

> Abgrenzung:
> `/etc/` systemweit
> 
> `~/.dotfiles` benutzerspezifisch 
> (überschreiben systemweite Einstellungen aus `/etc` für den jeweiligen User)
#### Pfade
### `/home` 
#### Verzeichnisse
> Home-Verzeichnisse der normalen Benutzer
> `~/.config/` Konfigurationsdateien nur für den User
> (Einstellungen, wie ein Programm arbeiten soll)

> `~/.local/` ist das private Gegenstück zu `/usr/local/`und Dachverzeichnis für benutzerdefinierte Software und Skripte ohne Root-Rechte
> (`pip install --user`)

> `~/.local/share/` Nutzdaten & Zustand
> (Womit arbeitet das Programm, Suchverlauf, Cursor-Position)

> `~/.local/bin/` eigene Skripte und Programme

> `~/.local/state/` Zustand von Programmen 
> (Logfiles oder temporäre Session-Daten)

> `~/.ssh/` speichert Benutzerspezifische SSH-Zugänge 
> -> authorized_keys: öffentlichen Schlüssel der PCs, die sich als Benutzer auf dem Server einloggen dürfen
> -> id_ed25519 / id_rsa: privater Schlüssel, um sich vom Server aus auf andere Systeme einzuloggen
> -> known_hosts: gespeicherte Fingerabdrücke von fremden Servern, mit denen man sich verbunden hat

> `~/.cache/` speichert temporäre Pufferdaten, die Programme beschleunigen oder Zwischenzustände sichern
> (es ist möglich den Inhalt dieses Ordners gefahrlos zu löschen, weil Programme die benötigten Dateien bei Bedarf neu erstellen)
#### Dateien
> `.bashrc` / `.zshrc` Einstellungen für interaktive Shells (z. B. Aliase, Prompt-Look, eigene PATH-Anpassungen)

> `.profile` wird beim ersten Einloggen (Login-Shell) einmalig ausgeführt und setzt umgebungsweite Variablen

> `.bash_logout` Befehle, die automatisch beim Abmelden ausgeführt werden
> (automatische Löschen des Bildschirms, Backups, Bereinigungen von temporären Dateien, Logfile-Eintrag)

> `.bash_history` speichert Befehlsverlauf
### `/lib` 
> essenzielle System-Bibliotheken für Programme in /bin und /sbin

> `/lib/systemd/systemd` Hauptprogramm von PID 1
> (früher /sbin/init ist heute nur noch Symlink)
### `/mnt` 
> Einhängepunkte für temporäre/wechselbare Datenträger
### `/proc` 
> virtuelles Dateisystem mit Prozess- und Kernel-Informationen
>
> -> liegt nicht auf Festplatte, sondern existiert nur im RAM
> -> Der Kernel erzeugt hier virtuelle Dateien (Pseudo-Dateien), damit man den Zustand des Systems einfach mit Textbefehlen (wie cat) ablesen und verändern kann
> -> zeigt alle PIDs

> `cpuinfo`  Hardware-Details des Prozessors
> 
> `cmdline` Start-Befehle, die der Bootloader beim Einschalten an den Kernel mitgegeben hat
> 
> `modules` Liste aller geladenen Kernel-Treiber
> 
> `sys/` Steuerzentrale für Live-Einstellungen des Kernels 
> 
> `net/ipv4/ip_forward` steuert das IP-Routing
> -> `0` Paketverarbeitung nur für den eigenen Rechner, empfangene Pakete an fremde IP-Adressen werden verworfen
> -> `1` Kernel leitet Pakete, die nicht für ihn selbst bestimmt sind, an andere Netzwerk-Schnittstellen weiter
> 
> `kernel/pid_max` maximale Anzahl an Prozessen, die gleichzeitig laufen dürfen

> **/proc Funktionsweise**
> 
> Das Grundproblem:
> Der Linux-Kernel arbeitet tief im Arbeitsspeicher mit komplexen Bit-Mustern. Ein Administrator kann diese im RAM nicht direkt lesen oder verstehen.
> 
> Die Lösung: 
> virtuelle Dateien in /proc. 
> Der Kernel schaltet sich als Übersetzer dazwischen und tut so, als wäre /proc ein Ordner voller Textdateien.
> 
> Ablesen (Hardware & Status):
> -> Wenn man cat /proc/cpuinfo eingibt, liest man keine Datei von der Festplatte. 
> -> Der Kernel bemerkt die Anfrage, fragt live die CPU ab und gibt das Ergebnis als Text auf den Bildschirm.
> 
> Steuern & Schalten (/proc/sys/):
> -> Das Unterverzeichnis /proc/sys/ bietet direkte Schreib-Schnittstellen zur Laufzeit-Konfiguration des Kernels.
> -> Änderungen werden sofort ohne Neustart wirksam.
> -> Beispiel IP-Forwarding: ip_forward steuert das Routing von IPv4-Paketen zwischen verschiedenen Netzwerkschnittstellen:
> `echo 1 > /proc/sys/net/ipv4/ip_forward` setzt den Wert auf 1, womit der Kernel die Paketweiterleitung für Fremdadressen aktiviert. 
### `/sys` 
> `/sys` (sysfs) ist ein virtuelles Pseudo-Dateisystem im Arbeitsspeicher. Es exportiert die interne Objektstruktur des Kernels (Geräte, Treiber, Busse, Kernel-Module) in den User-Space.

> **Hauptaufgabe & Abgrenzung**
> 
> -> Schnittstelle für Hardware-Struktur & Status: zeigt die physische und logische Hierarchie aller vom Kernel erkannten Komponenten.
> 
> -> Abgrenzung zu `/dev`: 
>  /dev liefert die Gerätedateien (Nodes), über die Prozesse Daten an Hardware senden oder lesen. 
> /sys liefert die Meta-Informationen und Zustände dieser Hardware (z. B. MAC-Adresse, Temperatur, Lüfterdrehzahl, Power-Management).
> 
> -> Abgrenzung zu `/proc`:
> /sys ist seit Kernel 2.6 die strukturierte Ergänzung zu /proc. 
> Während /proc primär prozessbezogene Daten und Laufzeit-Parameter enthält, ist /sys strikt nach Hardware, Bussen und Treibern kategorisiert.

> **Unterverzeichnisse**
> 
> `/sys/class/` logische Gruppierung von Geräten nach Funktion (z. B. net für Netzwerkkarten, block für Speichermedien, power_supply für Akkus).
> `/sys/bus/` Sortierung nach physischen Bussen (z. B. pci, usb, i2c).
> `/sys/devices/` die tatsächliche globale Gerätehierarchie des Systems (alle anderen Verzeichnisse wie /sys/class verlinken oft hierher).
> `/sys/module/` geladene Kernel-Module und deren Parameter.

> **Praxis-Beispiele (Auslesen & Steuern)**
```bash
cat /sys/class/net/eth0/address
```
> liest die MAC-Adresse der Netzwerkkarte aus
```bash
cat /sys/class/net/eth0/operstate
```
> zeigt den Verbindungsstatus (up/down)
```bash
echo 1 > /sys/class/backlight/intel_backlight/brightness
```
> Laufzeit-Konfiguration (Hardware-Steuerung ohne Reboot):
> -> setzt Bildschirmhelligkeit
### `/tmp` 
> temporäre Dateien (wird bei Neustarts oder automatisch bereinigt)
### `/usr`
> `usr` bedeutet heute "Unix System Resources", früher "User", jedoch findet sich dieser historische Bereich heute in "/home"
> Programme sind typischerweise Binärdateien, ihre Konfigurationsdateien hingegen Textdateien

> `/usr/bin` / `/usr/sbin` systemweite ausführbare Befehle und Admin Befehle (System-Binaries), die vom Paketmanager (apt/dnf) verwaltet werden 

> `/usr/local/bin/` / `/usr/local/sbin/` systemweite manuell installierte normale- und Admin-Software (am Paketmanager vorbei)

> `/usr/share/` plattformunabhängige Daten, Dokumentationen und Hilfeseiten (man pages)

> `/usr/lib/` / `/usr/lib64/` Bibliotheken und Hilfsdateien, die Programme aus `/usr/bin` zum Laufen brauchen

> `/usr/include/` Header-Dateien für die Programmiersprache C/C++ (für Software selbst kompilieren)

> `/usr/src/` Quellcode des Linux-Kernels oder von Treibern
### `/var` 
> `/var` steht für variable Daten, also Daten, die sich während des laufenden Systembetriebs verändern.
> 
> Typische Inhalte sind:  
> -> System- und Anwendungs-Logs  
> -> Zustandsdaten von Programmen  
> -> Cache-Daten  
> -> Spool-Daten (z. B. Druckaufträge)  
> -> Mail-Daten  
> -> dynamische Anwendungsdaten

> **Unterverzeichnisse**
>
> `/var/log/` System- und Anwendungsprotokolle
> 
> `/var/lib/` dauerhafte Zustandsdaten von Programmen und Diensten
> 
> `/var/cache/` zwischengespeicherte Daten, die bei Bedarf neu erzeugt werden können
> 
> `/var/spool/` Daten, die auf eine spätere Verarbeitung warten (z. B. Druckaufträge)
> 
> `/var/tmp/` temporäre Dateien, die Neustarts überleben sollen. Im Gegensatz zu `/tmp` sind sie nicht grundsätzlich für die Löschung beim Boot vorgesehen.
> 
> `/var/run/` historischer Pfad für Laufzeitdaten; moderne Systeme verwenden dafür `/run`. `/var/run` ist häufig ein Symlink auf `/run`.

> **Abgrenzung: `/var` vs. `/run`**

> `/var` enthält grundsätzlich variable Daten, die teilweise über einen Neustart hinaus erhalten bleiben.
> 
> `/run` enthält dagegen flüchtige Laufzeitdaten des aktuell laufenden Systems und wird beim Boot neu aufgebaut.
### Andere
#### `/`  
> Ausgangspunkt des gesamten Dateisystems  (Wurzelverzeichnis)
#### `/cdrom` 
> historischer Einhängepunkt für optische Laufwerke (heute unter /media)
#### `/opt` 
> zusätzliche, proprietäre Softwarepakete werden manchmal in /opt abgelegt

> Abgrenzung: 
> -> `usr/local` Software verstreut sich auf bin, lib, share
> -> `/opt` komplettes Programm liegt isoliert in einem einzigen Ordner
#### `/root` 
> Home-Verzeichnis des Root-Benutzers
#### `/run`
> flüchtige Laufzeitdaten seit dem letzten Systemstart
#### `/srv` 
> vom System bereitgestellte Daten für Server-Dienste
## Dateisystem
### Dateiformate
> -> in Linux werden Datei-Endungen vom Kernel nicht berücksichtigt (Ausnahme: Programme zur Zuordnung / Benutzerkomfort) 
> -> Dateitypen werden primär über File Header (Magic Bytes) bestimmt

> Standard-Formate:
> `.txt` Textdatei (unformatierter Text)
> `.log` System- und Anwendungslog
> `.conf/.cfg` Konfigurationsdatei 
> `.sh` Executable Shell-Skript
> `.odp` Präsentation (OpenDocument)
> `.png` Rastergrafik (verlustfrei)
> `.jpg/.jpeg` Rastergrafik (komprimiert)

>  System & Konfiguration:
> `yaml/.yml` strukturiertes Konfigurationsformat (z. B. Netplan, Ansible) 
> `.json` Datenaustauschformat 
> `.desktop` Anwendungsverknüpfung für GUI-Umgebungen 
> `.service` Systemd-Dienst-Konfiguration 
> `.bak` Backup-Kopie einer Datei

> versteckte Dateien:
> `ls -a` zum anziegen
>  -> beginnen immer mit einem Punkt
>  -> im Home-Verzeichnis oft benutzerspezifische Konfigurationsdateien
## Hintergrundwissen
### Bootprozess
> **Startprozess in 4 Phasen**

> Phase 1: Hardware und GRUB
> 
> Beim Anschalten sucht die Hardware den Bootloader (GRUB). 
> GRUB schaut in den Ordner /boot/grub/ und zeigt das schwarze Auswahlmenü an ("Ubuntu starten").

> Phase 2: Kernel und Notfall-Kernel
> 
> Der Kernel (vmlinuz) ist noch schlafend/komprimiert und kennt zu Beginn noch keine Festplatte.
> Deshalb lädt GRUB zusätzlich die initrd.img (Initial RAM Disk) in den Arbeitsspeicher. 
> Analogie: Die initrd ist wie ein Werkzeugkoffer mit Universal-Schlüsseln (Basistreibern). Erst mit diesen Treibern kann der Kernel die echte Festplatte (/) aufschließen und lesen.

> Phase 3: Der erste Prozess (PID 1)
> 
> Sobald die Festplatte erreichbar ist, verwirft der Kernel den Notfall-Kernel und startet genau einen Hauptprozess: die Prozess-ID 1.
> Dieser Prozess ist heute systemd (physikalisch gespeichert unter /lib/systemd/systemd)
> PID 1 bringt das System in den Betriebszustand. Es startet und überwacht alle Hintergrunddienste in der richtigen Reihenfolge, startet abgestürzte Dienste neu und adoptiert verwaiste Prozesse, um deren Speicher aufzuräumen. PID 1 läuft ununterbrochen, da sein Beenden das gesamte System stoppt.

> Phase 4: Hintergrunddienste & Live-Status
> 
> Der Bauleiter (systemd) startet nun nacheinander alle anderen Prozesse (Netzwerk, Benutzeroberfläche, SSH-Server, /proc).
### Kernel
> **Die 3 Formen des Kernels**

> 1) `/boot/vmlinuz` (Der Betriebskernel) 
> 
> -> Status: fertig kompilierte, binaere und mit zstd/gzip komprimierte Ausführungsdatei. 
> -> Funktion: wird beim Systemstart direkt vom Bootloader (GRUB) in den RAM geladen, entpackt sich selbst und übernimmt die volle Kontrolle über Hardware, Prozessverwaltung und Speicher. 
> -> Besonderheit: Das "vm" steht für Virtual Memory (virtueller Speicher), das "z" für die Komprimierung; enthält nur die essenziellsten Basisfunktionen für maximale Effizienz.

> 2) `/boot/initrd.img`(Das Initial RAM Disk Image)
> 
> -> Status: ein temporäres, komprimiertes Dateisystem (CPIO-Archiv), das komplett im Arbeitsspeicher liegt. 
> -> Funktion: liefert dem Kernel während der ersten Sekunden des Bootvorgangs die nötigen Treiber (z. B. für NVMe, RAID, LVM oder verschlüsselte LUKS-Dateisysteme), die nicht fest in `vmlinuz` eingebaut sind. 
> -> Ablauf: Sobald die echte Root-Partition (/) gemountet ist, übergibt initrd die Kontrolle an den echten Init-Prozess (systemd, PID 1) und wird aus dem RAM gelöscht.

> 3) `/usr/src/` (Der Quellcode)
> 
> -> Status: unkompilierter C- und Assembler-Textcode (mehrere Gigabyte groß). 
> -> Funktion: dient als Bauplan; wird im laufenden Betrieb nicht benötigt, sondern nur für das manuelle Erstellen eines angepassten Kernels oder das Kompilieren externer Treibermodule (z. B. DKMS für proprietäre Nvidia-Treiber). 
> -> Struktur: enthält die Architekturbeschreibungen (x86_64, ARM) sowie den vollständigen Katalog aller verfügbaren Treiber und Dateisystem-Logiken.

> **Begriff kompilieren:**
> 
> Übersetzungsprozess, bei dem ein spezielles Programm (Compiler) den von Menschen geschriebenen Quellcode (z. B. C-Code) analysiert, prüft und in eine maschinenlesbare Sprache (Maschinencode) umwandelt.
> Ein kompiliertes Programm (Binary) liegt als fertige Binärdatei vor und kann direkt vom Betriebssystem bzw. der CPU ausgeführt werden, ohne dass der ursprüngliche Quellcode oder zusätzliche Interpreter benötigt werden.
### Code
> **Der Weg vom Quellcode zur Hardware**

> 1) Quellcode in C (Höhere Programmiersprache)
> 
> -> Charakter: menschenlesbarer Text in `/usr/src/`, der Abstraktionen wie Variablen, Schleifen, Funktionen und Zeiger (Pointer) nutzt. 
> -> Plattformunabhängigkeit: der C-Code beschreibt Logik und Algorithmen generell; er weiß zu diesem Zeitpunkt noch nicht, auf welcher spezifischen CPU-Architektur er später laufen wird. 
> -> Vorverarbeitung: der Präprozessor (cpp) löst vorab Makros und `#include-Befehle` auf, um eine einzelne, zusammenhängende C-Datei für den nächsten Schritt zu erzeugen.

> 2) Der Compiler-Prozess (gcc / Assembler)
>
> -> Syntaktische Übersetzung: der Compiler (z. B. gcc) liest den C-Code, optimiert ihn und übersetzt ihn zuerst in Assembler-Code – die architekturspezifische Textform der Ziel-CPU (z. B. x86_64). 
> -> Mnemonic-Befehle: Assembler nutzt kurze, hardwarenahe Textkürzel wie mov (Daten verschieben), add (addieren) oder jmp (Springen), um direkt Prozessorregister anzusprechen. 
> -> Assemblierung & Linker: der Assembler (as) wandelt diese Textbefehle 1:1 in Objektcode um; der Linker (ld) fügt alle Teile schließlich zu einer einzigen ausführbaren Binärdatei zusammen.

> 3) Maschinencode (Ausführung auf Transistorebene)
> 
> -> Binärstruktur: das fertige Binary (z.B. `vmlinuz`) besteht aus reinem Maschinencode – Folgen von Bytes (0 und 1), unterteilt in Opcode (Befehl) und Operanden (Speicheradresse/Register). 
> -> Fetch-Decode-Execute-Zyklus: die CPU liest die Bitmuster aus dem RAM (Fetch), decodiert sie über interne Logikschaltungen (Decode) und führt die Rechenoperation aus (Execute). 
> -> Elektrische Signale: auf physikalischer Ebene schalten die Nullen (0V) und Einsen (~1-3V) Milliarden von Transistoren auf dem Prozessor, legen Spannungen an Mainboard-Leitungen an und steuern so Peripherie und Speicher.
### Größeneinheiten
> **Basisbausteine**
> 
> -> Bit (b): kleinste digitale Informationseinheit. Kann genau zwei Zustände annehmen: 0 oder 1
> -> Byte (B): Ein Verbund aus genau 8 Bits. Repräsentiert z. B. ein einzelnes Textzeichen (ASCII).

> **Speichergrößen vs. Netzwerkgeschwindigkeiten**
> 
> -> Dateigrößen & RAM werden in Bytes angegeben 
> -> Netzwerk-Bandbreiten werden in Bits pro Sekunde angegeben (z.B. Mbit/s, Gbit/s).

> **Die zwei Standards: SI (10er-Basis) vs. IEC (2er-Basis)**
>
> 1) SI-Standard (Dezimal / 10er-Potenzen)
> 
> -> Basis: $10^3 = 1.000$
> ->Verwendungszweck: Festplattenhersteller (Marketing), Netzwerkbandbreiten.
> -> Einheiten:
> Kilobyte (KB) = 1.000 Bytes ($10^3$)
> Megabyte (MB) = 1.000.000 Bytes ($10^6$)
> Gigabyte (GB) = 1.000.000.000 Bytes ($10^9$)
> Terabyte (TB) = 1.000.000.000.000 Bytes ($10^{12}$)
> 
> 1) IEC-Standard (Binär / 2er-Potenzen)
> 
> -> Basis: $2^{10} = 1.024$
> -> Verwendungszweck: Betriebssysteme (Linux, RAM-Adressierung), Befehle (free, ls, df).
> -> Einheiten: 
> Kibibyte (KiB) = 1.024 Bytes ($2^{10}$) 
> Mebibyte (MiB) = 1.048.576 Bytes ($2^{20}$) 
> Gibibyte (GiB) = 1.073.741.824 Bytes ($2^{30}$) 
> Tebibyte (TiB) = 1.099.511.627.776 Bytes ($2^{40}$)

> Differenz:
> Die Festplatte wird vom Hersteller in GB/TB (1000er-Basis) verkauft. 
> Das Betriebssystem rechnet im Hintergrund jedoch mit GiB/TiB (1024er-Basis). 
> Dadurch entsteht eine scheinbare Differenz von ca. 7 %.
# Grammatik
## Befehls-Verknüpfung
### Aufbau
```bash
Befehl <Option(en)> <Argument(e)>
```

> `;` oder `Zeilenumbruch` sequentielle Ausführung
 -> führt Befehle nacheinander aus, egal ob der vorherige erfolgreich war

> `&&` Success-Chain
> -> führt rechten Befehl nur aus, wenn linker Befehl erfolgreich war

> `||` Fallback
> -> führt rechten Befehl nur aus, wenn linker Befehl fehlgeschlagen ist

> `#` Kommentar
> -> alles danach wird von der Shell ignoriert

> `!` Invertierung (Verneinung)
> -> kehrt den Exit-Code um
### übergreifende Optionen
> `-v` ausführliche Ausgabe(cp, mv, rm, tar, mkdir) 
> `-r -R` Rekursiv (cp, rm, ls, chown, chmod) 
> `-i` fragt nach Bestätigung (cp, mv, rm) 
> `-f` Erzwingen (rm, cp) 
> `-h` lesbare Größenangaben (ls, df, du)
## I/O Redirection
> Befehlszeile leitet Informationen über Standardkanäle um:
> -> Standardeingabe (Standard Input / stdin / Kanal 0) = Tastatur
> -> Standardausgabe (Standard Output / stdout / Kanal 1) = Bildschirm
> -> Fehlerausgabe (Standard Error / stderr / Kanal 2)
> 
> `|` (Pipe) übergibt die Ausgabe des linken Befehls als Eingabe an den rechten Befehl
> `| xargs` übergibt die Ausgabe des linken Befehls als Kommandozeilen-Argumente an den rechten Befehl

> `>` überschreibt Datei (stdout) oder erstellt neue Datei
> 
> `>>` fügt Informationen zur Datei hinzu Standardausgabe (stdout) oder erstellt neue Datei

> `<` liest Eingabe aus einer Datei statt von der Tastatur (stdin)
> `<<` Text- oder Code, der an den Befehl übergeben wird

> `1>`/`>` Standardausgabe (stdout, Kanal 1)
> `2>` Fehlerausgabe (stderr, Kanal 2)
> `2>&1` leitet Fehlerausgabe dorthin, wo auch stdout hingeht (muss nach der regulären Umleitung stehen, z. B. `Befehl > Ziel.txt 2>&1`).
> 
> `2>/dev/null` verwirft alle Fehlermeldungen
> `/dev/null` = Daten-Mülleimer (akzeptiert Eingaben, aber macht nichts damit)
>
> `&>` / `&>>` leitet normale Ausgabe und Fehler zusammen um
## Quoting
> 1. Ohne Quoting
> `<Befehl> $VAR`
> -> trennt Argumente an Leerzeichen
> -> Variablen ($VAR) werden aufgelöst
> 
> 2. Soft Quoting `""`
> `<Befehl> "$VAR"`
> ->  hält Text mit Leerzeichen zusammen
> -> Variablen ($VAR) werden trotzdem aufgelöst
>
> 3. Hard Quoting `''`
> `<Befehl> '$VAR'`
> ->  hält Text zusammen und schaltet alle Sonderzeichen ab
> -> Variablen werden nicht aufgelöst
> 
> 4. Escape-Zeichen `\`
> `<Befehl> \$VAR`
> ->  neutralisiert genau ein nachfolgendes Sonderzeichen oder Leerzeichen
> (sucht in diesem Fall nach dem echten Dollarzeichen im Namen)

>  Sonderzeichen-Liste:
`~, !, @, #, $, %, ^, &, *, (, ), -, _, +, =, {, }, [, ], |, , :, ;, ", ', <, >, ,, ., ?, /, sowie Leerzeichen, Tabulator und Zeilenumbruch`
## Globbing (Shell Expansion)
> `*` entspricht beliebiger Anzahl von Zeichen
> (Beispiel: `ls <Datei>*` Ausgabe: Datei123, DateiXYZ)

> `?` entspricht genau einem Zeichen
> (Beispiel: `ls <Datei>?.txt` Ausgabe: Datei1.txt, DateiX.txt)

> `[-]` entspricht einzelner/mehrerer oder einer Klasse von Zeichen
> (Beispiel: `ls <Datei>[0-5]` Ausgabe: Dateien, die auf Zahlen 0-5 enden)
>  (Beispiel: `ls <Datei>[1X]` Ausgabe: Dateien mit 1 oder X)
> -> mehrere Klassen durch mehrere Bindestriche seperieren
> -> `[^]` alles außer den nachfolgenden Zeichen

> `[[:Zeichenklasse:]]` entspricht einer Zeichenklasse
> (Beispiel: `ls <Datei>[[:digit:]]` Ausgabe: Dateien, die auf Zahlen enden)
> 
> Optionen:
> `[:alnum:]` Buchstaben und Zahlen
> `[:alpha:]` Groß- oder Kleinbuchstaben
> `[:blank:]` Leerzeichen und Tabs.
> `[:cntrl:]` Steuerzeichen
> `[:digit:]` Zahlen 
> `[:graph:]` graphische Zeichen (alle Zeichen außer ctrl/Leer)
> `[:lower:]` Kleinbuchstaben 
> `[:print:]` druckbare Zeichen 
> `[:punct:]` Interpunktionszeichen
> `[:space:]` Whitespace-Zeichen, (Tabs, Leerzeichen, Zeilenumbrüche
> `[:upper:]` Großbuchstaben 
> `[:xdigit:]` hexadezimale Zahlen (0123456789abcdefABCDEF)
## Regex (Regular Expressions)
### Anker
> `^<Suchbegriff>` findet nur Zeilen, die mit "Suchbegriff" beginnen

> `<Suchbegriff>$` findet nur Zeilen, die auf "Suchbegriff" enden.
### Zeichen
> `.` beliebiges Zeichen
> (Beispiel: `grep "b.t" datei.txt` findet "bat", "bit", "b-t", etc.)

> `[...]` Auswahl
>  (Beispiel: `grep "b[ai]t" datei.txt` findet nur "bat" oder "bit")

> `[<von>-<bis>]` Bereiche
>(Beispiel: `grep "[0-9]" datei.txt` findet alle Zeilen mit mindestens einer Ziffer)

>`[^<von>-<bis>]` Verneinung
> (Beispiel: `grep "[^0-9]" datei.txt` findet Zeichen, die KEINE Ziffer sind)
### Quantifizierer
>`*` 0-mal oder öfter 
> -> Das Zeichen davor darf beliebig oft vorkommen oder fehlen
> (Beispiel: `egrep "go*l" datei.txt` findet "gl", "gol", "gool", "gooool")

> `+` 1-mal oder öfter
> -> Das Zeichen davor muss mindestens 1-mal vorkommen
> (Beispiel: `egrep "go+l" datei.txt` findet "gol", "gool", aber NICHT "gl")
   
> `?` 0-mal oder 1-mal
> -> Das Zeichen davor ist optional
> (Beispiel: `egrep "https?" datei.txt` findet "http" und "https")
   
> `{n}` / `{n,m}` exakte Anzahl
> (Beispiel: `egrep "[0-9]{3}" datei.txt` findet genau 3 Ziffern in Folge)
> (Beispiel: `egrep "a{2,4}" datei.txt` findet "aa", "aaa" oder "aaaa")

> `<Zeichenkette>|<Zeichenkette>` Übereinstimmung mit einer der angegebenen Zeichenketten
> (Beispiel: `egrep "http|https" datei.txt` sucht nach "http" oder "https")

> `( ) |` Gruppierung & ODER-Verknüpfung
> (Beispiel: egrep `"^(#|$)"` datei.txt -> findet Zeilen, die mit # beginnen ODER komplett leer sind ($))
### Anwendungsbereiche
> `grep -E` Durchsuchen von Logdateien und Befehlsausgaben

> `apt` präzise Suche nach Paketnamen

> `sed` & `awk` automatisiertes Ersetzen und Bearbeiten von Texten/Dateien

> `find` erweiterte Dateisuche mit `regex` (Spezialfälle)
## Variablen
### lokale Variablen
```bash
<VAR>=<Wert>
```
> erstellt lokale Variable
```bash
echo $<VAR>
```
> gibt Wert aus
```bash
unset <VAR>
```
> löscht Variable
```bash
set
```
> zeigt alle Shell- und Umweltvariablen sowie Funktionen
### Umgebungs-Variablen
```bash
export <VAR>=<Wert>
```
 > erstellt Umgebungsvariable
```bash
export <VAR>
```
> macht bestehende Variable zur Umgebungsvariable
```bash
env
```
> zeigt Umgebungsvariablen
### System-Variablen
##### `PATH`
 ```bash
 PATH
 ```
> Umgebungsvariable mit allen Verzeichnissen, in denen die Shell nach Befehlen sucht
```bash
PATH=$PATH:<neues-Verzeichnis>
```
> Verzeichnis hinzufügen
```bash
PATH=<Verzeichnisse>
```
> Verzeichnisse exakt setzen
##### `USER`
```bash
$USER
```
> aktueller Benutzer
##### `HOME`
```bash
$HOME
```
> Home-Verzeichnis
##### `SHELL`
```bash
$SHELL
```
> aktive Shell
##### `TZ`
```bash
TZ=<Zeitzone> date
```
> temporäre Zeitzone setzen
# Standard-Befehle
## Systemnavigation
### Orientierung
#### Benutzer
```bash
whoami
``` 
> zeigt, als welcher Benutzer man eingeloggt ist
```bash
who
``` 
> zeigt alle aktuell angemeldeten Benutzer mit Terminal und Login-Zeit
```bash
w
``` 
> zeigt angemeldete Benutzer inklusive ihrer aktuellen Aktivität und Systemauslastung
```bash
$ 
``` 
> Prompt für Standard-Benutzer (eingeschränkte Rechte)
```bash
#
``` 
> Prompt für Root (Vollrechte)
#### Pfade
```bash
pwd
``` 
> zeigt den aktuellen Pfad an
```bash
$PWD
``` 
> Pfad bis zum aktuellen Verzeichnis (Systemvariable!)
```bash
.
```
> im aktuellen Verzeichnis
```bash
~
```
> persönliches Home-Verzeichnis (/home/$USER)
```bash
<Befehl> ~<Benutzer>/<Pfad>
```
> auf Pfad/Datei eines anderen Benutzers zugreifen (Relative-to-Home-Pfad)
```bash
/
```
> Root-Verzeichnis
#### `ls`
 ```bash
 ls
 ```
> zeigt Inhalt des aktuellen Verzeichnisses an
 ```bash
 ls <Pfad>
 ```
> zeigt Inhalt eines bestimmten Ziels an
> 
> Optionen:
> `-l` Langformat mit Berechtigungen, Besitzer und Größe (Alias: `ll`)
> (Aufbau: Dateityp, Rechte, Links zur Datei, Eigentümer, Gruppe, Bytes, letzte Änderung, Dateiname )
> `-lh` Langformat mit lesbarer Dateigröße (MB/GB)
> `-t` sortiert nach Zeitpunkt letzter Änderung
> `-lrt` sortiert umgekehrt nach Zeitpunkt letzter Änderung
> `-lX` sortiert nach Dateiendungen
> `-S` sortiert nach Dateigröße
> `-a` alle Dateien inklusive versteckter
> `-F` klassifiziert Dateien und Verzeichnisse durch Symbole
> `-d` nur Verzeichnisse
> `-d */` zeigt nur direkte Unterverzeichnisse
> `-R` zeigt alle Unterverzeichnisse (Rekursion/unendliche Tiefe)
> `-i` zeigt die jeweilige Inode-Nummer (eindeutige Identifikationsnummer im Dateisystem)
### Navigation
#### Sitzungen
```bash
su 
``` 
> wechselt zum Root-Benutzer unter Beibehalten der User-Pfade und Umgebungsvariablen
```bash
su -
``` 
> wechselt zum Root-Benutzer und lädt dessen Umgebungsprofil
```bash
su <Benutzername>
``` 
> wechselt in Shell-Sitzung eines anderen Benutzers
```bash
su - <Benutzername>
``` 
> wechselt Benutzer und lädt dessen Umgebungsprofil
```bash
sudo -i
``` 
> Anmeldung als Root-Benutzer mit User Passwort (nur, wenn User in Sudo-Gruppe ist)
```bash
sudo -u <Benutzer> <Befehl>
```
> Befehl mit anderem Konto ausführen
```bash
logout
```
> ordnungsgemäßes Abmelden
```bash
exit
``` 
> verlässt bestimmte Prozesse / Root-sitzung / Shell-Sitzung
```bash
q
```
> verlässt bestimmte Prozesse
```bash
<Strg + c>
```
> beendet aktuellen Prozess / blockierte Eingabe
#### `cd`
##### relative Pfade
```bash
cd <Verzeichnis>
```
> wechselt in das angegebene Unter-Verzeichnis
```bash
cd
```
> springt sofort ins Home-Verzeichnis
```bash
cd ..
```
> springt eine Ebene nach oben
```bash
cd ../..
```
> springt 2 Ebenen nach oben
##### absolute  Pfade
```bash
cd <Pfad>
```
> springt in den angegebenen Pfad
## Dateimanagement
### Verwaltung
#### `cp`
```bash
cp <Datei> <Ziel>
``` 
> kopiert Datei
> 
> -> `-p` erhält Berechtigungen und Änderungszeiten des Original
> -> `-u` kopiert Datei nur, wenn das Ziel noch nicht existiert oder älter als die Quelldatei ist
```bash
cp <Datei1> <Datei2>
``` 
> überschreibt Datei2 mit Datei1
```bash
cp <Quelle> <Quelle> <Verzeichnis>
```
> kopiert mehrere Dateien in ein Verzeichnis hinein
```bash
cp -r <Verzeichnis> <Ziel>
``` 
> kopiert Verzeichnis rekursiv (inklusive aller Unterordner und Dateien)
```bash
cp -a <Verzeichnis> <Ziel>
``` 
> Archiv-Kopie: kopiert rekursiv und erhält Rechte, Besitzer und Zeitstempel
#### `mv`
```bash
mv <Quelle> <Ziel>
``` 
> verschiebt Datei/Verzeichnis oder benennt sie um
> 
> -> `-i` Sicherheitsfrage anfordern vor Namensänderung
```bash
mv <Quelle> <Quelle> <Ziel>
``` 
> verschiebt mehrere Dateien/Verzeichnisse
#### `mk`
```bash
mkdir <Verzeichnis>
``` 
> erstellt Verzeichnis
```bash
mkdir <Name> <Name> <Name>
``` 
> erstellt mehrere Verzeichnisse
```bash
mkdir -p <Name>/<Name>
``` 
> erstellt Eltern-Verzeichnis mit Unterverzeichnis
#### `touch`
```bash
touch <Datei>
``` 
> erstellt Datei
```bash
touch <Datei>{1..5}
``` 
> erstellt mehrere Dateien mit nummerierten Endungen
```bash
touch <Datei> <Datei> <Datei> 
``` 
> erstellt mehrere Dateien
#### `rm`
```bash
rm <Datei/Verzeichnis>
``` 
> löscht Datei/Verzeichnis
>
> -> `-r`  löscht Verzeichnis samt Inhalte mit Rückfrage bei schreibgeschützten Dateien 
> -> `-i` stellt für jede Datei/Unterverzeichnis Rückfrage
> -> `-f` löscht ohne Rückfrage (force)
```bash
rmdir <Verzeichnis>
```
>  löscht leeres Verzeichnis
#### `ln`
 ```bash
 ln <Pfad> <Link>
 ```
> erstellt Hardlink (fester Verweis auf denselben Inode auf der Festplatte, nur für Dateien möglich)
```bash
ln -s <Pfad> <Link>
```
> erstellt symbolischen Link (Softlink) auf eine Datei oder ein Verzeichnis
> (symbolische Links beziehen sich auf den Standort des Links selbst; werden Link oder Zieldatei verschoben, bricht die Verknüpfung, aber Verwendung absoluter Pfade verhindert dies)
### Dateien bearbeiten
#### `nano`
```bash
nano <Datei>
```
> öffnet Texteditor im Terminal
> 
> -> `+<Zahl>` bei einer bestimmten Zeilennummer öffnen 

> Steuerung:
> -> `Strg k` Zeile löschen
> ->  `Alt a`  Text markieren + anschließend Strg k = Text löschen
> ->  `Strg w` Suche nach Wort/Zeichen
#### `vi/vim`
```bash
vim <Datei>
``` 
> öffnet Texteditor im Terminal und startet im Befehlsmodus
> 
> Steuerung:
> Befehlsmodus:
> -> `i` Insert (Einfügemodus)
> -> `:w` Save
> -> `:q` Quit
> -> `:wq` = Save & Quit
> -> `:q!` Zwangs-Beenden ohne Speichern
> 
> Einfügemodus:
> -> `Esc` verlässt Einfügemodus
> -> `d` Markierungen löschen
#### `echo`
```bash
echo "<Text>" > <Datei>
```
> überschreibt Benutzer-Datei mit Text
```bash
echo "<Text>" >> <Datei>
```
> fügt Text der Datei hinzu
#### `tee`
```bash
<Befehl> | tee <Datei>
```
> schreibt den Output gleichzeitig auf den Bildschirm (stdout) und in die angegebene Datei
> `-a` anhängen statt überschreiben (append)
#### `Here-Doc'`
```bash
tee <Datei> << 'EOF' 
<Text>
EOF
```
> leert Datei und überschreibt sie mit Text (bis zum Wort EOF)
>
> `-a` hängt Text der Datei an
```bash
cat << 'EOF' > <Datei>
<Text>
EOF
```
> leert Datei und überschreibt sie mit Text (bis zum Wort EOF)
>
> `>>` hängt Text der Datei an
### Dateien auslesen
```bash
cat <Datei>
``` 
> gibt Inhalt aus
> 
> -> `-n` zeigt auch Zeilennummern
```bash
less <Datei>
``` 
> öffnet Datei zum Blättern
>
> Steuerung:
> -> `Leertaste` eine Seite vor
> -> `b` eine Seite zurück
> -> `Pfeiltasten` Zeilen entlang fahren
> -> `q` beenden
```bash
more <Datei>
``` 
> zeigt Textdateien Bildschirm für Bildschirm
```bash
head -n <Zahl> <Datei>
``` 
> zeigt erste Zeilen der Datei (Standard 10)
```bash
tail -n <Zahl> <Datei>
``` 
> zeigt letzte Zeilen der Datei (Standard 10)
```bash
tail -f <Datei>
``` 
> Datei Live-Verfolgung
### Suchen
#### `find`
```bash
find
```
> zeigt Liste aller Dateien, Verzeichnisse und Unterverzeichnisse des aktuellen Verzeichnisses (ohne Option eingegeben) 
```bash
find <Pfad> -<Option> <Suchmuster>
``` 
> sucht im Pfad nach Suchmuster
> 
> -> `-name "<Datei/Verzeichnis>"` findet Datei/Verzeichnis 
> -> `-type f` Dateien, `-type d` Verzeichnisse, `-type l` Symlinks
> 
> -> `-maxdepth <N>` geht maximal N Ebenen tief
> -> `-mindepth <N>` Suche erst ab Ebene N
> -> `-executable` ausführbare Dateien
> -> `-not -path "<Suchmuster>"` Ausschluss von Suchmustern
> 
> -> `-perm <Oktalwert>` nur Dateien mit spezifischer Berechtigung
> -> `-perm -<Oktalwert>` Dateien, bei denen mindestens diese Bits gesetzt sind
> -> `-perm /<Oktalwert>` Dateien, bei denen irgendeines dieser Bits gesetzt ist 
> -> `-user <Benutzer>` Dateien, die dem User gehören
> -> `-group <Gruppe>` Dateien, die der Gruppe gehören
> -> `-nouser`/`-nogroup` Dateien, deren Besitzer/Gruppe nicht mehr existieren
> 
> -> `-mtime <Zahl>` Dateien, die in den letzten x Tagen geändert wurden
> -> `-mmin <Zahl>` Dateien, die in den letzten x Minuten geändert wurden
> (`-<Zahl>`= jünger als, `<Zahl>`= exakter vor, `+<Zahl>`= älter als)
> 
> -> `!`/`-not` (vor der flag -name): Negation
> -> `<Pfad> \( -name "<Suchbegriff>" -o -name "<Suchbegriff>" \)` für "Oder"
>
> -> `-exec <Befehl> {} +` führt Befehl für jede gefundene Datei aus (z.B. `-exec rm -f {} +` oder `-exec chmod u+x {} +`)
> 
> Option: `-size <Zahl>c` Dateien mit exakter Größe
> -> `+` vor Zahl für größer 
> -> `-` vor Zahl für kleiner
> -> `c` Bytes
> -> `k` KB
> -> `M` MB
> -> `G` GB
#### `plocate`
```bash
sudo updatedb
```
> aktualisiert die Index-Datenbank (Inhaltsverzeichnis)
```bash
plocate "<Datei/Verzeichnis>"
```
> durchsucht den Index statt die Festplatte (schneller als find, aber nur Pfade)
### Textverarbeitung
#### `echo`
```bash
echo <Text/Variable>
```
> gibt Text oder Variableninhalte aus

> Optionen mit `-e`:
> -> `-e` aktiviert Verarbeitung von Sonderzeichen
> -> `\n` (Newline): erzeugt einen Zeilenumbruch
> -> `\t` (Tab): fügt einen Tabulator ein
> -> `\\` (Backslash): gibt einen echten Backslash aus
#### `tr`
```bash
tr 'a-z' 'A-Z' < <Datei>
```
> Klein- in Großbuchstaben umwandeln
```bash
tr -d '<Zeichen>' < <Datei>
```
> bestimmte Zeichen löschen
```bash
tr -s ' ' < <Datei>
```
> doppelte Zeichen komprimieren
```bash
#1. bearbeiten und in eine temporäre Datei schreiben
tr -s ' ' < datei.txt > datei.tmp

#2. ursprüngliche Datei mit der temporären überschreiben
mv datei.tmp datei.txt
```
> Datei überschreiben
#### `grep`
```bash
grep "<Suchbegriff>" <Datei>
```
> durchsucht Dateien nach Textmuster
>
> -> `-A <Zahl> ` gibt die nächsten Zeilen mit aus
> -> `-B <Zahl> ` gibt Zeilen davor mit aus
> -> `-C <Zahl> ` gibt Zeilen davor und danach mit aus (Context)
> -> `-c` zählt Anzahl der Treffer
> -> `-l` gibt nur Dateinamen aus
> -> `-w` sucht nach ganzen Wörtern
> -> `-n` gibt Zeilennummer aus
> -> `-i` ignoriert Groß-/Kleinschreibung
> -> `-r` rekursiv (Suche auch in Unterverzeichnissen)
> -> `-v` gibt Zeilen aus, die nicht dem Suchbegriff entsprechen
> -> `-E` nutzt Regex-Funktion für Quantifizierer (alias: `egrep`)
```bash
grep -f "<Suchliste.txt>" <Ziel>
```
> durchsucht Dateien nach allen Wörtern aus der Suchliste
#### `strings`
```bash
strings <Datei>
```
> filtert alle lesbaren Textzeichen aus der Datei
```bash
strings <Datei> | grep "<Suchbegriff>"
```
> filtert alle Zeilen heraus, die den “Suchbegriff“ enthalten.
#### `wc`
```bash
wc <Datei>
```
> Ausgabe: Zeilen, Wörter, Bytes
>
> -> `-l` Zählt Zeilen
> -> `-w` Zählt Wörter
> -> `-m` Zählt die Zeichen
> -> `-c` Zählt Bytes
#### `sort & uniq` 
```bash
sort <Datei>
```
> listet den Inhalt der Datei alphabetisch/numerisch sortiert auf
> 
> -> `-r` für umgekehrte Reihenfolge
> -> `-n` für numerische Sortierung
> -> `-k <Zahl>` sortiert nach einer bestimmten Spalte
> -> `-f` ignoriert Groß-/Kleinschreibung beim Sortieren
> -> `-u` entfernt doppelte Zeilen und gibt jede Zeile nur einmal aus
```bash
uniq <Datei>
```
> Ausgabe mit Entfernung doppelter, direkt aufeinanderfolgenden Zeilen
> 
> -> `-u` zeigt Zeile die nur 1x existiert
> -> `-d` zeigt Zeilen die mehrmals existieren
> -> `-c` zählt Vorkommen jeder Zeile
```bash
sort <Datei> | uniq -dc
```
> Verknüpfung: alle Zeilen einer Datei, die mehrfach existieren und wie oft
#### `cut`
> -> cut trennt Textzeilen an einem gewählten Zeichen (alles davor ist ein Feld)
> -> typische Anwendung: Spalten aus Tabellen, CSV-Dateien oder Logfiles isolieren
```bash
cut -f <Feld> -d '<Trennzeichen>' <Datei>
```
> Standard-Syntax
```bash 
echo "max:mustermann" | cut -f 1 -d ':'
```
> Beispiel (Ausgabe: max)
> 
> -> `-c <Zeichen>` schneidet feste Zeichenpositionen aus 
> (z. B. `-c 1-5` für die ersten 5 Zeichen jeder Zeile)
> -> `-f <Feld> -d '<Trennzeichen>' --complement` lässt Spalte aus
#### `awk`
> **Allgemein:**
> 
> `awk` kann Textzeilen in Spalten (Felder) zerlegen. Während `cut` starr ist, kann `awk` mit Spalten rechnen, Bedingungen prüfen und Text formatiert ausgeben.

> **Funktionsweise:**
>
> Wenn `awk` eine Zeile einliest, schneidet es sie automatisch in Felder (Standardtrenner ist ein Leerzeichen oder Tabulator).
> 
> Felder:
> -> `$0` gesamte Zeile
> -> `$1` erste Spalte
> -> `$2` zweite Spalte (usw.)
> -> `$NF` letztes Feld

> **Allgemeine Syntax:**
> 
> ```
> awk 'Muster { Aktion }' <Datei>
> ```
>
> -> `Muster` ist optional. Eine Bedingung (z. B. "Mach das nur, wenn Spalte 3 größer als 100 ist"). Ohne Muster gilt die Aktion für jede Zeile
> -> `{Aktion}` was genau passieren soll (z. B. `{print $1}`, also "Gib Spalte 1 aus")

> **Optionen:**
```bash
awk '{print $1}' <Datei>
```
> gibt von jeder Zeile nur die erste Spalte aus
```bash
awk '{print $1, $3}' <Datei>
```
> gibt Spalte 1 und Spalte 3 aus (standardmäßig durch ein Leerzeichen getrennt
```bash
awk -F 'Trennzeichen' '{print $1}' <Datei>
```
> Field Separator (falls nicht durch Leerzeichen getrennt z.B. durch **`:`**)
```bash
awk '{t=$1; $1=$2=$3=""; print t, $0}' <Pfad>
```
> speichert Spalte 1 in der Variablen t, leert Spalten 1,2,3 und gibt Spalte 1 sowie den Rest nach Spalte 3 aus
```bash
awk -F ':' '$3 > 100 {print $1}' /etc/passwd
```
> Beispiel Zeilen filtern: wenn 3. Spalte (UID) größer ist als 100, gib $1 (Usernamen) aus
#### `diff`
```bash
diff <Datei1> <Datei2>
```
> zeigt alle Zeilen, die sich unterscheiden
> -> Zeilen mit `<` stehen nur in Datei 1
> -> Zeilen mit `>` stehen nur in Datei 2
#### `source`
```bash
source <Datei>
```
> führt ein Skript in aktueller Shell-Sitzung aus
#### `sed`
**Ersetzen und Löschen:**

```bash
sed '<Befehl>' <Datei>
```
> Stream Editor ist ein nicht-interaktiver Texteditor. Er liest Text zeilenweise ein, wendet Befehle darauf an und gibt das Ergebnis im Terminal aus (standardmäßig bleibt die Originaldatei unberührt)
```bash
sed 's/<Suchmuster>/<Ersatztext>/g' <Datei>
```
> "s" ersetzt ein Suchmuster mit einer Eingabe in einer Datei und "/g" (global) ersetzt alle Vorkommen in einer Zeile, ansonsten wird nur das erste Vorkommen je Zeile ersetzt
```bash
sed '<Zahl>d' <Datei>
```
> löscht nur die entsprechende Zeile
```bash
sed '/<Suchmuster>/d' <Datei>
```
> löscht alle Zeilen, die das Suchmuster enthalten

> Allgemeine Optionen:
> -> `-i` schreibt Änderungen in Originaldatei
> -> `-E` nutzt Regex
> -> `|`/`#` andere Sonderzeichen zur Abgrenzung wählen
> (z.B. `sed 's#/bin/bash#/bin/sh#g' /etc/passwd`)
> -> `-e` mehrere sed-Befehle nacheinander setzen
> (z.B. `sed -e 's/Apfel/Kirsche/g' -e 's/Banane/Mango/g' datei.txt`)

**Suchen und Filtern:**

```bash
sed -n '<Zahl>p' <Datei>
```
> gibt nur die angegebene Zeile aus
```bash
sed -n '<Start>,<Ende>p' <Datei>
```
> gibt nur einen bestimmten Zeilenbereich aus (z.B. Zeilen 1 bis 5)
### Archive
#### tar
> (bündelt viele Dateien und Verzeichnisse zu einer einzigen Datei. Archive erhalten standardmäßig die Endung `.tar`, komprimieren aber nicht selbst)
```bash
tar -cf <neues-Archiv>.tar <Pfad>
``` 
> erstellt ein unkomprimiertes tar-Archiv
```bash
tar -xf <Archiv>.tar
``` 
> entpackt ein unkomprimiertes tar-Archiv

> -> `-c` Archiv erstellen
> -> `-f`Name erstellen
> -> `-x` Archiv entpacken
> -> `-u` fügt einem unkomprimierten Archiv Dateien hinzu
> -> `-t`zeigt Inhalt an, ohne zu entpacken
> -> `-v`Namen der Dateien ausgeben lassen, mit denen tar beim Erstellen oder Extrahieren eines Archivs arbeitet

> -> `z` mit gzip komprimieren
> -> `j` mit bzip2 komprimieren
> -> `J` mit xz komprimieren

> -> `-xf <Archiv>.tar <Archiv-Pfad>` entpackt nur einen Teil aus dem tar-Archiv
> -> `-xf <Archiv.tar> -C <Zielverzeichnis>` extrahiert Archiv in ein bestimmtes Zielverzeichnis
> -> `-xf <Archiv.tar> -C <Zielverzeichnis> <Archiv-Pfad>` extrahiert einen Teil des Archives in ein Zielverzeichnis

> ```bash
> file <Dateiname>
> ```
> -> zeigt Dateityp
#### Synchronisation
> **Backups-Standard**
```bash
rsync -avh --delete <Quelle> <Ziel>
```

> **Allgemein**
> ```bash
> rsync <Optionen> <Quelle> <Ziel>
> ```
> synchronisiert Verzeichnisse lokal oder über ein Netzwerk hinweg und überträgt dabei standardmäßig nur Änderungen (Delta-Transfer)
> 
> -> `-a` (Archive-Modus) entspricht `-rlptgoD` (rekursiv, erhält Symlinks, Berechtigungen, Eigentümer, Gruppen und Zeitstempel)
> -> `-v` (Verbose) Ausführliche Ausgabe
> -> `-h` (Human-readable) lesbare Größenangaben
> -> `--delete` löscht Dateien im Zielverzeichnis, die in der Quelle inzwischen ebenfalls gelöscht wurden (unverzichtbar für exakte Spiegelungen/Backups)
#### gzip/bzip2/xz
```bash
<Befehl> <Datei>
```
> komprimieren
```bash
<Befehl> -<Zahl> <Datei>
```
> Komprimierungsstufen
```bash
<Befehl> -d <Datei><Endung>
```
> entpacken

> Befehle:
> `gzip` Standard für Einzeldateien; Kompromiss aus Tempo und Größe
> -> Datei-Endung: `.gz` 
> `bzip2` stärkere Komprimierung als gzip, braucht mehr Rechenleistung
>  -> Datei-Endung: `.bz2` 
> `xz` starke Komprimierung
> -> Datei-Endung: `.xz`
> 
> -> entpacken klassisch ohne "-d" `gunzip`/`bunzip2`/`unxz`

> Interaktion mit Archiven:
> `z<Befehl> <Archiv>` für gzip
> `bz<Befehl> <Archiv>` für bzip2
>`xz<Befehl> <Archiv>` für xz
#### zip
> `zip` / `unzip` ist der weltweite Standard auf Windows und macOS
> 
> -> `zip` ist Bündeln und Komprimieren in einem einzigen Schritt
> -> Zweck: Linux nutzt nativ meist `.tar.gz`, Windows-Nutzer können damit ohne Zusatzprogramme (wie 7-Zip) nichts anfangen
```bash
zip -r <neues-Archiv>.zip <Pfad> 
```
> zip erstellen
```bash
unzip <Archiv>.zip 
```
> zip entpacken
### Datenkonvertierung
#### `file`
```bash 
file <Datei>
```
> ermittelt tatsächlichen Dateityp anhand des Inhalts, unabhängig von Dateiendung
> 
> Optionen:
> -> `-b` kurze Ausgabe (versteckt Dateinamen)
> -> `-i` gibt MIME-Type und Zeichenkodierung aus
> 
> Dateitypen:
> -> ASCII text / UTF-8 Unicode text = Textdateien, Konfigurationsdateien
> -> ELF 64-bit LSB executable = ausführbare Linux-Programme (Binärdateien)
> -> POSIX shell script = Shell-Skripte
> -> gzip compressed data / Zip archive data = komprimierte Archive
> -> symbolic link to "Ziel" =  Verknüpfungen (Symlinks)
> -> data = unbekannte Binärdaten (oder verschlüsselte/beschädigte Dateien)
> -> directory = Verzeichnis
#### `base64`
```bash
base64 <Datei>
```
> kodiert Daten in Base64
```bash 
base64 -d <Datei>
```
> dekodiert Daten in Base64
> 
> -> `-i` ignore-garbage
#### `md5sum`
```bash
md5sum <Datei>
```
> erstellt einen eindeutigen Hashwert von Texten oder Dateien
> -> Nutzen: Prüfung, ob zwei Dateien exakt identisch sind oder beim Download beschädigt wurden
## Systeminfo
### Software
#### Befehls-Analyse
##### Befehls-Status
```bash
which <Befehl>
```
> zeigt exakten Pfad zur ausführbaren Datei eines Befehls an
```bash
whereis <Befehl>
```
> zeigt Pfad zur Programm-Datei und seine Dokumentationsdateien
```bash
type <Befehl>
```
> beschreibt, wie ein Befehl interpretiert wird (Alias, Shell-Builtin, Pfad)
```bash
echo $?
```
> gibt Exit-Code des zuletzt ausgeführten Befehls zurück
> -> `0` erfolgreich
> -> `1-255` Fehler
##### `info`
```bash
info <Befehl>
```
> Info-Seiten für GNU-Werkzeuge
##### `help`
```bash
help <Befehl>
```
> Informationen für Shell-Builtins
##### `man`
```bash
man <Befehl>
```
> Informationen für externe Programme und System-Konfigurationen

> -> `man` verwenden `less` zum Blättern
> -> Wortsuche vorwärts: `/<Wort>`
> -> Wortsuche rückwärts: `?<Wort>`

> Kategorien:
> 1 = Benutzerbefehle
> 2 = Systemaufrufe
> 3 = Funktionen der C-Bibliothek
> 4 = Treiber und Gerätedateien
> 5 = Konfigurationsdateien und Dateiformate
> 6 = Spiele
> 7 = Verschiedenes 
> 8 = Systemadministrator-Befehle
> 9 = Kernel-Funktionen (nicht Standard)
> 
> -> mehrere Kategorien können Manpages mit gleichen Namen enthalten (z.B. Befehl `passwd` oder Passwortdatenbankdatei `/etc/passwd`)
> -> gewünschte Manpage aufrufen mit: 
```bash
man <Zahl> <Wort>
```
#### Andere
##### Historie
```bash
history
```
> zeigt Liste bisheriger Befehle an
```bash
clear
```
> leert Anzeige des Terminalfensters
```bash
!!
```
> führt letzten Befehl aus
```bash
!string
```
> führt letzten Befehl aus, der mit "string" begann
##### `hostname`
```bash
hostname
```
> zeigt Hostnamen
```bash
sudo hostnamectl set-hostname <neuer-Hostname>
```
> ändert Hostnamen
##### `alias`
```bash
alias 
```
> Auflistung aktiver Abkürzungen
```bash
alias <Name>='<Befehl>'
```
> erstellt eine Abkürzung für einen (langen) Befehl
```bash
unalias <Name>
```
> löscht Alias
### Hardware
```bash
uname -a
```
> zeigt Kernel-Version, System-Architektur und Hostnamen
> -> `-r` zeigt nur Kernel-Version
> -> `-s` zeigt Betriebssystem-Name
```bash
lscpu
```
> listet detaillierte Informationen zur Prozessor-Hardware auf
```bash
lsblk
```
> listet alle erkannten Block-Geräte (Festplatten, Partitionen, USB-Sticks) auf
```bash
lspci
```
> listet alle am PCI-Bus angeschlossenen Hardware-Komponenten auf
```bash
free
```
> zeigt Speicherwerte im Standard-Format (Kibibyte)
> 
> `-h` zeigt Arbeitsspeicher-Belegung in lesbaren Einheiten
> `-m` zeigt Arbeitsspeicher-Belegung in Mebibytes
> `-g` zeigt Arbeitsspeicher-Belegung in Gibibytes
```bash
df -h
```
> zeigt Speicherplatzbelegung aller eingehängten Dateisysteme
```bash
du -h <Pfad/Datei>
```
> zeigt Speicherverbrauch eines Ordners/Datei
>
> -> `-a` alle Unterordner und Dateien
> -> `-d<Zahl>` ist Shortcut für `--max-depth=<Zahl>`
```bash
uptime
```
> die letzten 3 Ziffern zeigen die durchschnittliche Load des Systems der letzten 5/10/15 Minuten
# Systemsteuerung
## Shell
```bash
Strg a
```
> zum Beginn der Eingabezeile springen (Shortcut)
```bash
chsh -s /bin/<Shell>
```
> change shell (z.B. bash, zsh)
```bash
exec <Shell> -l
```
> Shell neustarten
```bash
echo $0
```
> aktive Shell anzeigen
```bash
echo $$
```
> gibt die PID der aktuellen Shell-Sitzung au
```bash
bash -c <Befehl>
```
> führt einen Befehls-String in einer neuen Sub-Shell aus 
## Paketverwaltung
### Debian-basiert
#### Allgemeines
> **Repository-Zeilen-Format**
>  
> -> `deb http://archive.ubuntu.com/ubuntu/ noble main restricted universe multiverse`
> -> Type: `deb` (Binärpakete) oder `deb-src` (Quellcode)
> -> URI: Server-Adresse des Spiegelservers (Mirror)
> -> Distribution: Release-Codename (z. B. `noble`, `jammy`) oder Suite (`stable`)
> -> Komponenten: `main` (offiziell unterstützt, Open Source), `restricted` (proprietäre Treiber), `universe` (Community-gepflegt), `multiverse` (urheberrechtlich eingeschränkt)

> **FHS**
> 
> `/etc/apt/sources.list` Hauptdatei für die Paketquellen (Repositories)
> `/etc/apt/sources.list.d/` Verzeichnis für zusätzliche Repositories (z. B. Fremdquellen, PPAs als .list-Dateien)
> `/etc/apt/apt.conf.d/` Verzeichnis für globale apt-Konfigurationen (z. B. Timeouts, Proxies, automatische Updates)
> `/var/lib/apt/lists/` Zwischenspeicher für heruntergeladene Paketlisten (Metadaten der Repositories von apt update)
> `/var/cache/apt/archives/` lokaler Cache für heruntergeladene .deb-Pakete (hier landen die Installationsdateien, bevor sie installiert werden – wird mit apt clean geleert)

> **Befehle**
```bash
sudo nano /etc/apt/sources.list.d/custom-repo.list
````
> manuelles Hinzufügen einer Paketquelle als `.list`-Datei
```bash
sudo add-apt-repository ppa:<user>/<ppa-name>
```
> fügt ein Personal Package Archive (PPA) hinzu (unter Ubuntu)
```bash
sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/example.gpg example.gpg.key
```
> moderner Standard: Importiert den GPG-Schlüssel des Repositories direkt nach "/etc/apt/trusted.gpg.d/"
#### High-Level
```bash
sudo apt <Befehl> 
```
> -> `update` Paketquellen (Repositories) synchronisieren (in Skripten: `apt-get`)
> -> `upgrade` installierte Pakete aktualisieren
> -> `full-upgrade` Systemupgrade inklusive Kernel- und Abhängigkeitswechsel
> 
> -> `install <Paket>` Paket installieren
> -> `remove <Paket>` Paket deinstallieren (Konfigurationsdateien bleiben erhalten)
> -> `purge <Paket>` Paket vollständig löschen (inklusive Konfiguration)
> -> `autoremove && sudo apt clean` nicht mehr benötigte Abhängigkeiten und Cache aufräumen
> 
> -> `search <Suchbegriff>` nach Paketen suchen (besser mit Regex: `'^...$'`)
> -> `show <Paket>` Details zu einem Paket anzeigen

> -> `apt-key list` listet vertrauenswürdige Signatur-Schlüssel auf
#### Low-Level
```bash
dpkg -i <Paket.deb>
```
> installiert lokale .deb-Datei (löst keine Abhängigkeiten auf)
```bash
dpkg -l
```
> listet alle installierten Pakete auf
```bash
dpkg -L <Paket>
```
>zeigt alle Dateien, die durch ein Paket installiert wurden
```bash
dpkg -S <Pfad-zur-Datei>
```
> zeigt, welches Paket eine bestimmte Datei installiert hat
### Red-Hat-basiert
#### Allgemeines
> -> Red-Hat-basiert (RHEL, Fedora, Rocky Linux, AlmaLinux)
> -> Paketformat: .rpm
#### High-Level
```bash
dnf search <Suchbegriff>
```
> sucht nach Paketen
```bash
dnf info <Paket>
```
> zeigt Paketinformationen
```bash
dnf install <Paket>
```
> installiert Paket inkl. Abhängigkeiten
```bash
dnf remove <Paket>
```
> entfernt Paket
```bash
dnf check-update
```
> prüft auf verfügbare Paket-Updates
```bash
dnf upgrade
```
> aktualisiert alle installierten Pakete
```bash
dnf autoremove
```
> entfernt ungenutzte Abhängigkeiten
#### Low-Level
-> lokale .rpm-Dateien verwalten
```bash
rpm -i <paket.rpm>
```
> installiert lokale .rpm-Datei (löst keine Abhängigkeiten auf)
```bash
rpm -U <paket.rpm>
```
> aktualisiert oder installiert eine lokale .rpm-Datei
```bash
rpm -qa
```
> listet alle installierten Pakete auf
```bash
rpm -ql <Paket>
```
> zeigt alle Dateien eines installierten Pakets
```bash
rpm -qf <Pfad-zur-Datei>
```
> zeigt, welches Paket zu einer bestimmten Datei gehört
## Versionsverwaltung 
#### `git`
> **Grundlegendes Setup & Initialisierung**
> ```bash
> git init <Pfad>
> ```
> initialisiert ein neues, leeres Git-Repository im angegebenen Verzeichnis
> ```bash
> git config --global user.name "Dein Name"
> git config --global user.email "deine.email@example.com"
> ```
> legt fest, wer die Commits erstellt (wird in jedem Commit dauerhaft gespeichert). Ohne diese Konfiguration verweigert Git das Erstellen von Commits

> **Ignorierte Dateien**
> Eine Datei namens `.gitignore` im Stammverzeichnis schließt bestimmte Dateien oder Ordner von der Versionsverwaltung aus

> **Staging & Commit**
> ```bash
> git add <Datei>
> ```
> fügt Änderungen der Staging-Area (Vorbereitungszone) hinzu
> ```bash
> git commit -m "Nachricht"
> ```
> speichert die vorbereiteten Änderungen dauerhaft mit einer Commit-Nachricht in der Historie
> ```bash
> git status
> ```
> zeigt den aktuellen Zustand des Arbeitsverzeichnisses und der Staging-Area an (welche Dateien geändert wurden, ob Änderungen vorgemerkt sind und auf welchem Branch man sich befindet)

> **Branching**
> ```bash
> git branch
> ```
> zeigt Branches (Standard: master)
> ```bash
> git branch <Branchname>
> ```
> erstellt einen neuen Branch
> -> `-d` löscht einen branch
> ```bash
> git switch <Branchname>
> ```
> (oder "git checkout Branchname") wechselt in den angegebenen Branch
>  -> `-c` erstellt den Branch und wechselt direkt dorthin

> **diff / merge**
> ```bash
> git diff
> ```
> zeigt die noch nicht gestagten Unterschiede der Arbeitsdatei im Vergleich zum letzten Commit an
> ```
> git diff <Branch> <Branch>
> ```
> Unterschiede zwischen Branches
> ```bash
> git merge <Branchname>
> ``` 
> führt die Änderungen eines anderen Branches in den aktuellen Branch zusammen

> **Interne Verweise & Pfade im `.git`-Ordner** 
> -> Git speichert Branches und Zustände als einfache Textdateien im `.git`-Verzeichnis.
> -> aktueller Branch: `cat .git/HEAD`
> -> Branch-Pointer (Commit-Hashes): Jeder Branch ist eine kleine Textdatei unter `.git/refs/heads/<Branch>`, die exakt den SHA-1-Hash des letzten Commits enthält

> **Commit wechseln**
> ```bash
> git log --oneline
> ```
> zeigt alle Commits im kompakten Einzeilen-Format an (Hash-Wert zum Wechseln)
> ```bash
> git checkout <Hash-Wert>
> ```
> Commit wechseln

> **Remote-Verwaltung**
> ```bash
> git remote -v
> ```
> zeigt alle verbundenen Remote-Repositories und deren URLs an
> ```bash
> git clone <Repository-URL> <Verzeichnis>
> ```
> kopiert ein bestehendes Git-Repository aus dem Netzwerk (z. B. GitHub, GitLab) samt vollständiger Historie in ein lokales Ziel-Verzeichnis
> -> benötigt Netzwerkzugriff, SSH-Zugangsdaten (oder HTTP) und ein installiertes Git-Tool auf dem Zielrechner

> **Git löschen**
> beim Löschen des `.git`-Ordners bleiben die Versionen der Arbeitsdateien erhalten gemäß des aktiven Branches
## Shutdown & Reboot
```bash
sudo shutdown -h now
```
> fährt System sofort herunter
> 
> -> `-h +<Zahl>` fährt System in "Zahl" Minuten herunter und schickt Mitteilung an alle angemeldeten Benutzer
> -> `-r +<Zahl>`startet System in "Zahl" Minuten neu  und schickt Mitteilung an alle angemeldeten Benutzer
> -> `-c`  bricht geplantes Herunterfahren oder Neustarten ab
> -> `<Mitteilung>` Mitteilung anhängen (am Ende des Befehls)
> 
> Alternativen:
> -> `sudo poweroff` Link zum Systemd-Befehl
> -> `sudo systemctl poweroff` moderner Systemd-Standardbefehl
> -> `sudo init 0` Klassischer Runlevel-Wechsel zum Herunterfahren
```bash
sudo reboot
```
> startet System sofort neu
## Systemkonfiguration
### LVM
#### Allgemein
> **LVM (Logical Volume Manager)**
> LVM ist eine flexible Zwischenschicht zwischen physischer Festplatte und Dateisystem. 
> -> Löst die Beschränkungen von statischen Festplatten-Partitionen auf. 
> -> Macht Speicherplatz im laufenden Betrieb erweiterbar und formbar.

> **3-Schichten-Prinzip**
> 
> 1) Physical Volume (PV) 
> -> Der rohe Speicherbaustein (z. B. eine bestehende Partition wie `/dev/sda3` oder eine physische SSD). 
> -> LVM stempelt diesen Bereich, um ihn für den Speicherpool verwalten zu können.
> 
> 2) Volume Group (VG) 
> -> Der zentrale Speicherpool (die virtuelle Gesamtfestplatte, z. B. `ubuntu-vg`). 
> -> Fasst ein oder mehrere PVs zusammen und bündelt deren Gesamtkapazität.
>     
> 3) Logical Volume (LV) 
> -> Die „virtuelle Partition“, die flexibel aus der Volume Group (VG) herausgeschnitten wird. 
> -> Bereich, auf dem am Ende die eigentlichen Daten liegen (z. B. `root` für `/` oder `swap`).

> **Dateisystem auf dem LV (z. B. EXT4)**
> -> LVM stellt nur den dynamischen Platz (das Regal) bereit. 
> -> Das Dateisystem (z. B. EXT4) baut darin das Ordnungssystem (Ordnerstrukturen, Dateien, Rechte und Journaling zur Absicherung) auf.

> **Speichererweiterung in einer VM (Ablaufkette)**

> 1. Host/Hypervisor: Virtuelle Festplatte vergrößern.
> 2. Partition (`sda3`): An das neue Ende der Festplatte ziehen.
> 3. Physical Volume: `pvresize` informiert LVM über den neuen Platz.
> 4. Volume Group: Nimmt den freien Speicher im Pool auf.
> 5. Logical Volume: `lvextend` macht das LV größer.
> 6. Dateisystem: `resize2fs` (bei EXT4) passt das Dateisystem an die neue LV-Größe an.
#### Analyse
```bash
lsblk
```
> zeigt Festplatten und Partitionen
```bash
sudo <Volume>
```
> -> `pvs` Übersicht aller Physical Volumes (PVs)
> -> `vgs` Übersicht aller Volume Groups (VGs) inkl. freiem Speicherplatz
> -> `lvs` Übersicht aller Logical Volumes (LVs)

```bash
sudo <pv/vg/lv>display
```
> -> `pv` Status- und Größeninformationen zu Physical Volumes
> -> `vg` Details zur Volume Group (inkl. Physical Extents / PE-Größen)
> -> `lv`  Details zu Logical Volumes (inkl. Pfad)
#### Erstellung
> **Schritt 1: Physical Volume initialisieren**
>```bash
>sudo pvcreate </dev/sdb> </dev/sdc>
> ```
> -> bereitet eine oder mehrere Festplatten für LVM vor

> **Schritt 2: Volume Group erstellen**
> ```bash
> sudo vgcreate <VG-Name> </dev/sdb> </dev/sdc>
> ```
> -> erstellt neue Volume Group/Pool aus einem oder mehreren Physical Volume

> **Schritt 3: Logical Volume anlegen**
> ```bash
> sudo lvcreate -L <Größe> -n <LV-Name> <VG-Name>
> ```
> -> erstellt ein LV mit fester Größe
> -> `-L 100%FREE` Zuweisung des gesamten verbliebenen Speichers
#### Dateisystem
```bash
sudo mkfs.<ext4/xfs> /dev/<VG-Name>/<LV-Name>
```
> formatiert das Logical Volume mit dem Dateisystem EXT4 oder XFS
```bash
sudo mount /dev/<VG-Name>/<LV-Name> /mnt/data
```
> hängt das LV temporär in den Ziel-Ordner ein
#### Erweiterung
```bash
sudo growpart /dev/sda 3
```
> zieht die Partition `sda3` auf das neue physische Ende der vergrößerten virtuellen Festplatte (Paket: `cloud-guest-utils`)
```bash
sudo pvresize /dev/sda3
```
> aktualisiert das Physical Volume, sodass LVM den neu gewonnenen Platz der Partition erkennt
```bash
sudo lvextend -L <Wert> /dev/<VG-Name>/<LV-Name>
```
> erweitert das LV
> -> `-r 1` (`--resizefs`) passt das Dateisystem (EXT4/XFS) automatisch mit an


```bash
sudo resize2fs /dev/<VG-Name>/<LV-Name>
```
> passt ein EXT4-Dateisystem an die neue LV-Größe an
```bash
sudo xfs_growfs /mnt/<Mountpoint>
```
> passt ein XFS-Dateisystem an (Achtung: `xfs_growfs` nimmt den Einhängepfad als Argument, z. B. `/`)
#### Wartung
```bash
sudo vgreduce <VG-Name> /dev/sdb
```
> entfernt ein leeres/ungenutztes Physical Volume aus der VG
```bash
sudo pvmove /dev/sdb
```
> verschiebt alle Datenblöcke von `/dev/sdb` auf andere verfügbare PVs innerhalb derselben VG (für Festplattentausch ohne Ausfallzeit)
```bash
sudo lvremove /dev/<VG-Name>/<LV-Name>
```
> löscht ein Logical Volume unwiderruflich (Dateisystem muss vorher unmounted sein mit `umount`)
```bash
sudo vgremove <VG-Name>
```
> löscht eine komplette Volume Group
```bash
sudo pvremove /dev/sdb
```
> entfernt den LVM-Stempel von einem Laufwerk
```bash
sudo umount /mnt/data
sudo e2fsck -f /dev/<VG-Name>/<LV-Name>
sudo lvreduce -r -L <Wert> /dev/<VG-Name>/<LV-Name>
sudo mount /dev/<VG-Name>/<LV-Name> /mnt/data
```
> schrumpft erst das Dateisystem und dann das LV (muss ausgehängt sein)
#### Persistent Mounting
```bash
blkid /dev/<VG-Name>/<LV-Name>
```
> ermittelt die eindeutige UUID des Logical Volumes
```bash
sudo nano /etc/fstab
```
> fstab-Konfiguration für dauerhaftes Einhängen beim Systemstart
> 
> **Eintrags-Format:** `UUID=xxxx-xxxx-xxxx-xxxx /mnt/data ext4 defaults 0 2`
```bash
sudo mount -a
```
> testet alle Einträge in `/etc/fstab` sofort ohne Neustart (wichtig zum Prüfen auf Syntaxfehler)
### Zeitzone
```bash
date
```
> zeigt Datum und Uhrzeit
```bash
timedatectl
```
> zeigt Uhrzeit, Datum, Zeitzone und NTP-Synchronisation
```bash
sudo timedatectl set-timezone Europe/Berlin
```
> CEST einstellen
# Scripting
## Skript-Variablen
> Zuweisung:
> ```
> <VAR>=<Wert>
> ```
> Variablennamen dürfen nur alphanumerische Zeichen oder Unterstriche enthalten und sind case-sensitiv

> Positionsparameter (Argumente beim Aufruf):
> -> werden beim Start hinter das Skript geschrieben
> ```
> ./skript.sh <Arg1> <Arg2> <Arg3>
> ```
> `$0` Name des Skripts
> `$1, $2, $3...` erstes, zweites, drittes Argument beim Aufruf 
> `$#` Anzahl der übergebenen Argumente 
> `$@`/`$*` alle übergebenen Argumente als Liste 
> `$?` Exit-Code des zuvor ausgeführten Befehls
> `$$` PID des laufenden Skripts

> Befehlssubstitution (Ergebnis eines Befehls speichern):
> ```
> <VAR>=$(<Befehl>)
> ```

> Benutzereingabe einlesen:
> ```
> read <VAR>
> ```
> liest Tastatureingabe in die Variable
> ```
> read -p "<Text: >"
> ```
> liest Eingabe mit vorgeschalteter Aufforderung
## Bash-Syntax
> Shebang-Zeile(`#!`): 
> -> Kernel liest die ersten 2 Bytes der Datei und erkennt, dass er zum Ausführen den dahinter angegebenen Interpreter starten muss
> 
> Interpreter:
> -> `#!/bin/bash` für Bash-Shell-Skripte
> -> `#!/usr/bin/env python3` für Python-Skripte

> Kommentare:
>  `#` vor einem Kommentar schreiben

>  Variablen:
>  -> Die Bash liest strikt von oben nach unten
>  -> Variablen müssen vor ihrer ersten Verwendung zugewiesen werden

> Zeilenumbruch & Semikolon:
> ->  Zeilenumbruch beendet eine Anweisung automatisch (kein `;` nötig) 
> -> Semikolon ersetzt Zeilenumbruch, um mehrere Befehle in einer Zeile zu schreiben

> Leerzeichen / Einrückungen:
> -> Leerzeichen nach einem ; oder Einrückungen dienen der Lesbarkeit und werden von der Bash ignoriert
## Bedingungslogik
#### `if`
> ```
> if [ <Bedingung> ]; then 
>   <Befehl(e)>
> else 
>   <Befehl(e)> 
> fi
> ```

> Numerische Vergleiche:
> `[ $# -eq 1 ]` "gleich"
> `[ $# -ne 1 ]` "nicht gleich"
> `[ $VAL -gt 10 ]` "größer als"
> `[ $VAL -ge 10 ]` "größer oder gleich"
> `[ $VAL -lt 10 ]` "kleiner als"
> `[ $VAL -le 10 ]` "kleiner oder gleich"
#### `for`
> ```
> for <Variable> in <Liste/Array>
> do
>   <Befehl(e)>
> done
> ```
> wiederholt Befehle für jedes Element einer Liste/eines Arrays

```bash
for username in $@
do
  echo "Hello $username!"
done
```
> Beispiel: Argumenten-Verarbeitung (`$@`)
> -> iteriert nacheinander über alle übergebenen Positionsparameter

```bash
for file in $FILES
do
  ls -lh $file
done
```
> Beispiel: Array-Verarbeitung
> -> Arrays bestehen aus einer durch Leerzeichen getrennten Zeichenkette
> -> Beispiel: FILES="/usr/sbin/accept /usr/sbin/pwck"

> erweiterte Befehle im Schleifenkontext:
> `echo -n` unterdrückt den automatischen Zeilenumbruch bei der Ausgabe
> `shift` verschiebt Positionsparameter nach links (verwirft das erste Element 
> (Beispiel:`$1`, `$2` wird zu `$1`)
#### `exit`
```bash
exit <Exit-Code>
```
> beendet Ausführung des Skripts sofort an der entsprechenden Stelle, nachfolgender Code im Skript wird ignoriert

> Exit-Codes:
> `0` Erfolg 
> `1` bis `255`: Fehler
> Best Practice: verschiedene Fehlerarten mit unterschiedlichen Codes kennzeichnen (z. B. `exit 1` für fehlende Parameter, `exit 2` für ungültige Argumente)

> Abfragen des Exit-Codes:
```bash
echo $username | grep "^[A-Za-z]*$" > /dev/null
if [ $? -eq 1 ]; then
  echo "Fehler: Ungültiges Zeichen"
  exit 2
fi
```
## Skript-Ausführung
> Ein Skript ist eine unformatierte ASCII/UTF-8-Textdatei
> Dateiendungen für Übersichtlichkeit:
> -> `.sh` für Shell-Skript
> -> `.py` für Python Skript

> Ausführungsrecht:
> `x` macht Skript zu einem ausführbaren Programm
>```
>chmod +x <Skript>.sh
>```

> Aufrufbar-Machen:
>```
><Skript>
>```
> -> Möglichkeit 1: Pfadangabe (`./<Skript.sh>`)
> -> Möglichkeit 2: Skript in ein PATH-Verzeichnis integrieren
> -> Möglichkeit 3: aktuelles Verzeichnis zu PATH hinzufügen (EXTREM GEFÄHRLICH! Erklärung: Gefahr der Pfad-Injektion/Executing beliebigen Codes, wenn man sich in fremden/schreibbaren Ordnern befindet)
> -> Möglichkeit 4: Ausführung in Shell (mit `source`)
> Skript startet keine neue Subshell, sondern wird in aktueller Shell-Sitzung ausgeführt
> Anwendung: Nötig, wenn das Skript Umgebungsvariablen setzen oder Funktionen laden soll, die nach dem Ausführen im aktuellen Terminal erhalten bleiben müssen
> Skript benötigt dafür kein Ausführungsrecht (chmod +x)
>```
>source <Skript>.sh
>```
## Skript-Beispiele
### Parameter-Prüfung (`if`/`else`)
> Das Skript prüft beim Aufruf, ob genau 1 Argument (ein Name) übergeben wurde, und gibt eine persönliche Begrüßungsmeldung aus. Falls kein oder mehr als 1 Argument übergeben wird, zeigt es eine Fehlermeldung mit der korrekten Syntax an
```bash
#!/bin/bash
# 1. Shebang: Weist den Kernel an, /bin/bash als Interpreter zu nutzen

# 2. Bedingung: Ist die Anzahl der Argumente ($#) gleich 1? (-eq = equal)
if [ $# -eq 1 ]; then
    BENUTZER=$1
    echo "Hallo $BENUTZER! Das Skript wurde erfolgreich gestartet."
else
    # Wird ausgeführt, wenn KEIN oder MEHRERE Argumente übergeben wurden:
    echo "Fehler: Bitte gib genau einen Namen an."
    echo "Aufruf: $0 <Name>"
fi
```
### Argumenten-Schleife (`for`/`shift`)
> Das Skript begrüßt beliebig viele übergebene Benutzer in einer einzigen Zeile. Es prüft zuerst, ob mindestens ein Argument übergeben wurde, nutzt `echo -n` für die Ausgabe ohne Zeilenumbruch und entfernt mit `shift` den bereits verarbeiteten ersten Namen vor der Schleife, welche die restlichen Namen ergänzt.
```bash
#!/bin/bash
# 1. Prüfen, ob KEINE Argumente übergeben wurden ($# -eq 0)
if [ $# -eq 0 ]; then
    echo "Fehler: Bitte gib mindestens einen Namen an."
    exit 1
else
    # Erste Person ausgeben (ohne Zeilenumbruch dank -n)
    echo -n "Hallo $1"
    
    # Verwirft das erste Argument ($1), $2 wird das neue $1
    shift
    
    # Schleife über alle verbleibenden Argumente ($@)
    for username in $@
    do
        echo -n ", und $username"
    done
    
    # Abschlusszeichen mit Zeilenumbruch setzen
    echo "!"
    exit 0
fi
```
### Eingabe-Validierung (`grep`/ `exit`)
> Das Skript prüft mithilfe eines regulären Ausdrucks und `grep`, ob die übergebenen Namen ausschließlich aus Buchstaben bestehen. Fehlerhafte Eingaben brechen die Schleife sofort ab und geben einen spezifischen Fehler-Code (`exit 2`) an das System zurück.
```bash
#!/bin/bash
# 1. Prüfen, ob mindestens ein Argument existiert
if [ $# -eq 0 ]; then
    echo "Fehler: Bitte gib mindestens einen Namen an."
    exit 1
else
    # 2. Alle übergebenen Namen nacheinander validieren
    for username in $@
    do
        # grep prüft auf reine Buchstaben; Ausgabe wird nach /dev/null verworfen
        echo $username | grep "^[A-Za-z]*$" > /dev/null
        
        # $? prüft den Exit-Code von grep (1 = keine Übereinstimmung / ungültig)
        if [ $? -eq 1 ]; then
            echo "FEHLER: Namen dürfen nur Buchstaben enthalten ($username ist ungültig)."
            exit 2
        else
            echo "Hallo $username!"
        fi
    done
    
    # Erfolgreicher Abschluss
    exit 0
fi
```
# Benutzerverwaltung
## Benutzer & Gruppen
### Benutzer-Typen
> Unter Linux hat jeder Benutzer eine eindeutige **UID** (User ID) und eine **GID** (Gruppen-ID). 
> Standardmäßig erhält jeder neue Benutzer eine gleichnamige primäre Gruppe. 

> **1) Superuser (`root`)** 
> -> `UID 0` / `GID 0`
> -> Unbegrenzte Systemrechte
> -> Home-Verzeichnis: `/root` (Top-Level, außerhalb von `/home`)

> **2) Reguläre Benutzer (Standard)**
> -> `UID >= 1000` (bei älteren Systemen vereinzelt ab 500)
> -> Normale Rechte (interaktiver Zugriff)
> -> Home-Verzeichnis: `/home/<Benutzername>`
> -> Standard-Shell: `/bin/bash`

> **3) System- & Service-Accounts**
> -> `UID < 1000`
> -> Für Hintergrunddienste und Programme (z. B. Webserver, Datenbanken)
> -> Besitzen aus Sicherheitsgründen keine interaktive Login-Shell (`/sbin/nologin`)
> -> Home-Verzeichnisse liegen meist außerhalb von `/home` oder existieren gar nicht
### Benutzer
```Bash
cat /etc/passwd
```
> zeigt alle Benutzer des Systems
```bash
sudo useradd <Benutzername>
```
> legt einen neuen Benutzer an
> 
> -> `-m` erstellt automatisch das Home-Verzeichnis
> -> `-M` erstellt kein Home-Verzeichnis 
> -> `-s <Shell>` legt Standard-Shell fest
> -> `-g <Gruppe/GID>` legt primäre Gruppe fest
> -> `-G <Gruppe1,Gruppe2>` fügt User sekundären Gruppen hinzu 
> -> `-u <UID>` weist eine spezifische UID zu 
> -> `-c <Kommentar>` setzt GECOS-Feld (z. B. Max Mustermann) 
> -> `-d <Pfad>` legt ein abweichendes Home-Verzeichnis fest 
> -> `-e <YYYY-MM-DD>` setzt ein Ablaufdatum für das Konto

```bash
sudo passwd <Benutzername>
``` 
> setzt oder ändert Passwort des Benutzers (als Root ohne altes Passwort einzugeben) 
> 
> -> `-l` sperrt das Benutzerkonto (lock) 
> -> `-u` entsperrt das Benutzerkonto (unlock) 
> -> `-e` erzwingt Passwortwechsel beim nächsten Login (expire) 
> -> `-d` löscht das Passwort (passwortloses Konto) 
> -> `-S` zeigt den Passwort-Status des Kontos an (L=Locked, P=Passwort)
```bash
sudo usermod <Benutzername>
```
> bearbeitet bestehendes Benutzerkonto
>
> -> `-aG <Gruppe>` fügt Benutzer einer zusätzlichen Gruppe hinzu (ohne alte Gruppen zu entfernen) -> `newgrp auditops` Gruppenmitgliedschaft in der Shell aktualisieren
> -> `-s <Shell>` ändert Shell
> -> `-l <neuer-Name>` ändert Benutzernamen
```bash
sudo userdel <Benutzername>
``` 
> löscht einen Benutzer 
> -> `-r` löscht auch Home-Verzeichnis und Mail-Spool des User
```bash
sudo chage -l <Benutzer>
```
> Passwort- und Account-Alter und Ablaufdatum

```bash
getent passwd
```
> Zentralverwaltung über Netzwerkdienste in modernen Systemen
```bash
getent passwd <Benutzer>
```
> -> fragt System-Konfiguration (`/etc/nsswitch.conf`) ab und liefert alle Benutzer oder Informationen über einen bestimmten Benutzer
### Gruppen
```bash
groups
```
> zeigt alle vorhandenen Gruppen 
```bash
groups <Benutzer>
```
> zeigt, zu welchen Gruppen ein Benutzer gehört
```bash
getent group
```
> zeigt alle Gruppen im System (Alternative: `cat /etc/group`)
```bash
getent group <Gruppe>
```
> zeigt, welche Benutzer zu einer Gruppe gehören 
```bash
sudo gpasswd -d <Benutzername> <Gruppenname>
```
> entfernt Benutzer aus Gruppe
```bash
sudo groupadd <Gruppenname>
```
> erstellt neue Gruppe
> -> `-g <GID>` weist der Gruppe eine spezifische GID zu
> -> `-f` (force) wenn Gruppe bereits existiert wird sie ignoriert und die Befehlskette läuft weiter
```bash
sudo groupmod -n <neuer Name> <alter Name>
```
> benennt bestehende Gruppe um
```bash
sudo groupdel <Gruppenname>
```
> löscht Gruppe
### Identität prüfen
```bash
id <Benutzername>
```
> zeigt UID (User-ID), GID (Gruppen-ID) und alle zugehörigen Gruppen
```bash
groups <Benutzername>
```
> listet alle Gruppen auf, in denen der Benutzer Mitglied ist
### Access-Control
> Die zentralen Konfigurationsdateien für Benutzer und Rechte liegen unter `/etc/`.
> sie dürfen niemals direkt mit einem Texteditor bearbeitet werden (Gefahr von System-Sperrung).
#### `passwd`/`shadow`/`group`
> **`/etc/passwd`**
> -> öffentlich lesbar für alle Benutzer (`644`).
> -> enthält grundlegende Benutzerinformationen (ohne echte Passwörter).
> -> Syntax:
> `USERNAME : PASSWORD : UID : GID : GECOS : HOMEDIR : SHELL`
> 
> Bedeutung der Felder:
> - `PASSWORD`: Fast immer `x` (Verweis darauf, dass das Passwort gehasht in `/etc/shadow` liegt).
> - `GECOS`: Zusatzinfos (voller Name, Raum, Telefonnummer – änderbar mit `chfn`).
> - `SHELL`: Standard-Shell (z. B. `/bin/bash` oder `/sbin/nologin`).

> **`/etc/shadow`**
> -> nur für Root lesbar (`600` / `000`).
> -> speichert gehashte Passwörter und Ablaufdaten.
> -> Syntax:
> `USERNAME : PASSWORD : LASTCHANGE : MINAGE : MAXAGE : WARN : INACTIVE : EXPDATE`
> 
> Wichtige Passwort-Formate im Feld `PASSWORD`:
> - `!` oder `!!`: Konto ist gesperrt / deaktiviert.
> - `$6$...`: Passwort gehasht mit SHA-512 (Standard auf modernen Systemen).
> - `$y$...` / `$y$j9T$...`: Passwörter mit modernen Hash-Verfahren (yescrypt).

> `/etc/group`
> -> öffentlich lesbar (`644`).
> -> listet alle Gruppen und deren zusätzliche Mitglieder auf.
> -> Syntax:
> `GROUPNAME : PASSWORD : GID : MEMBERS`
#### `sudoers`
> **`/etc/sudoers` & `/etc/sudoers.d/`**
> -> steuert, welche Benutzer mit `sudo` Admin-Rechte erlangen dürfen.
> -> Datei darf nur mit `visudo` bearbeitet werden (prüft Syntax vor dem Speichern, um Aussperren zu verhindern).
> -> Syntax-Aufbau:
> ```
> <Wer> <Wo>=(<Als_Wer>) <Welche_Befehle>
> ```
> -> Beispiele:
> `max ALL=(ALL:ALL) ALL` User max darf von jedem Host aus als jeder User jeden Befehl ausführen
> `%wheel ALL=(ALL) ALL` Alle Mitglieder der Gruppe `wheel` (unter Ubuntu: `%sudo`) haben Vollrechte 
> `alex ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx` User alex darf ohne Passwort-Eingabe Nginx neustarten

> **Visudo**
```
sudo -l
```
> zeigt alle Sudo-Rechte an, die der aktuell angemeldete Benutzer besitzt
```
sudo visudo
```
> öffnet die Hauptkonfigurationsdatei `/etc/sudoers` im abgesicherten Modus
```
sudo visudo -f /etc/sudoers.d/<Datei>
```
> erstellt/bearbeitet eine isolierte modulare Sudo-Regeldatei unter "/etc/sudoers.d/"
```
sudo -l
```
> zeigt alle Sudo-Rechte an, die der aktuell angemeldete Benutzer besitzt
#### Andere
> **`/etc/skel/`** 
> -> Skeleton-Verzeichnis (Vorlage für neue Home-Verzeichnisse). 
> -> Inhalte werden bei `useradd -m` automatisch ins Home-Verzeichnis des neuen Users kopiert

> **`/etc/login.defs`** 
> -> Zentrale Konfiguration für die Benutzererstellung (definiert z. B. UID/GID-Bereiche für Standard- und System-Users).
## Rechteverwaltung
### Struktur
```bash
ls -l
```
> zeigt Rechte an (drwxr-xr-x)
> 
> Positionen:
> 1: Typ (`-` Datei, `d` Verzeichnis, `l` Symlink, `b` Block, `c` Character, `s` Socket)
> 2-4: Besitzer (User/u)
> 5-7: Gruppe (Group/g)
> 8-10: Andere (Others/o)
> 
> Symbolische Werte:
> `r` Lesen (Read/4)
> `w` Schreiben (Write/2)
> `x` Ausführen (Execute/1)
> `-` kein Recht
### Besitzrechte
```bash
sudo chown <User> <Datei/Verzeichnis>
```
> ändert Besitzer von Datei/Verzeichnis
```bash
sudo chown <User>:<Gruppe> <Datei/Verzeichnis>
```
> ändert gleichzeitig Besitzer und Gruppe
> 
> -> `-R` Rekursiv (wendet Änderungen auf alle Unterordner und Dateien an)
```bash
sudo chgrp <Gruppe> <Datei/Verzeichnis>
```
>ändert die Gruppe
### Zugriffsrechte
> SYMBOLISCHE SCHREIBWEISE
```bash
chmod <Subjekt><Operator><Rechte> <Datei>
```
> Optionen:
> -> Subjekt: `u`(User),`g`(Group),`o`(Others),`a`(All)
> -> Operatoren:`+`(hinzufügen),`-`(entfernen), `=`(exakt setzen)
> -> Rechte: `r`(Read),`w`(Write),`x`(Execute)
> 
> `-R`für rekursiv (Rechte für alle Dateien innerhalb eines Verzeichnisses und seiner Unterverzeichnisse ändern)

> OKTALE SCHREIBWEISE
```bash
chmod <123> <Datei>
```
> Einzelrechte:
> -> `0`= ---
> -> `1`= --x
> -> `2`= -w-
> -> `4`= r--
> 
> Summen:
> -> `3`= -wx
> -> `5`= r-x
> -> `6`= rw-
> -> `7`= rwx
> 
> häufige Kombinationen: 
> -> `755` = `rwxr-xr-x` Standard für Ordner/ausführbare Skripte
> -> `644` = `rw-r--r--` Standard für normale Dateien
> -> `700` = `rwx------` Nur Besitzer hat Zugriff (z. B. `.ssh`-Ordner)
> -> `600` = `rw-------` Nur Besitzer darf lesen/schreiben (z. B. SSH-Keys)
```bash
umask
```
> zeigt aktuelle Rechte-Maske für neue Dateien/Ordner
```bash
umask <Wert>
```
> setzt neue Maske (z. B. 002)
> 
> Funktionsweise:
> -> Dateien (Basis 666): 666 - 002 = 664 (rw-rw-r--)
> -> Verzeichnisse (Basis 777): 777 - 002 = 775 (rwxrwxr-x)
### Sonderrechte
#### Finden
```bash
find / -perm -4000 -type f 2>/dev/null
```
> findet alle SUID-Dateien auf dem gesamten System
```bash
find / -perm -2000 -type f 2>/dev/null
```
> findet alle SGID-Dateien auf dem gesamten System
```bash
find / -perm -1000 -type f 2>/dev/null
```
> findet alle Sticky-Bit-Dateien auf dem gesamten System
#### SUID
> **SUID (Set User ID)**
> -> Oktalwert: `4000` | Symbolisch: `u+s` (Erscheint als `s` anstelle des Ausführungs-`x` beim Besitzer)
> -> Funktion bei Dateien: Das Programm wird mit den Berechtigungen des Dateibesitzers ausgeführt (nicht mit den Rechten des Ausführenden).
> -> Klassischer Anwendungsfall: `/usr/bin/passwd` (gehört `root`, damit normale User ihr Passwort in `/etc/shadow` ändern können).

```bash
chmod u+s <Datei>
```
> setzt SUID-Bit
#### SGID
> -> Oktalwert: `2000` | Symbolisch: `g+s` (Erscheint als `s` anstelle des Ausführungs-`x` bei der Gruppe)
> -> Funktion bei Dateien: Programm wird mit den Rechten der Dateigruppe ausgeführt. 
> -> Funktion bei Verzeichnissen: Alle neu erstellten Dateien/Unterordner innerhalb dieses Verzeichnisses erben automatisch die Gruppe des Elternverzeichnisses

```bash
chmod g+s <Datei/Verzeichnis>
```
> setzt SGID-Bit
#### Sticky Bit
> -> Oktalwert: `1000` | Symbolisch: `o+t` oder `+t` (Erscheint als `t` anstelle des Ausführungs-`x` bei Others) 
> -> Funktion bei Verzeichnissen: Benutzer können in diesem Verzeichnis zwar Dateien erstellen, aber nur ihre eigenen Dateien löschen oder umbenennen (selbst wenn andere Schreibrechte besitzen). 
> -> Klassischer Anwendungsfall: `/tmp` und `/var/tmp` (Rechte: `rwxrwxrwt` bzw. `1777`).

```bash
chmod +t <Verzeichnis>
```
> setzt Sticky Bit
### ACLs
> **Allgemein**
> ACLs erlauben die Vergabe individueller Rechte an beliebige User und Gruppen.
> Paketinstallation (falls nicht vorhanden): `acl`.
> ein "+"-Zeichen nach den Rechten bei `ls -l <Datei>` zeigt, dass ACL gesetzt sind

> **Befehle**
```bash
getfacl <Datei/Verzeichnis>
```
> ACL anzeigen
```bash
sudo setfacl -m <Option> <Datei>
```
> ACL setzen
> 
> -> `-m` modifiziert oder fügt ACL-Einträge hinzu
> -> `u:<User>:<Rechte> <Datei>` für einen bestimmten User
> -> `g:<Groupe>:<Rechte> <Datei>` für eine bestimmte Gruppe
> -> `-x u/g:<User/Group> <Datei>` ACLs löschen
> -> `-d` neu erstellte Dateien und Unterordner erben die Rechte
> -> `-b <Datei>` entfernt alle erweiterten ACLs auf einmal (`remove all`)
### Limits
#### `ulimit`
```bash
ulimit -a
```
> zeigt alle aktuellen Ressourcen-Limits der Shell an
```bash
ulimit -u
```
> zeigt Limit für die maximale Anzahl von Prozessen, die der Benutzer gleichzeitig ausführen darf
```bash
ulimit -u <Zahl>
```
> setzt temporäres Limit für offene Prozesse für die aktuelle Sitzung herab (normale User können Limits nur senken, Root kann sie erhöhen). Änderungen gelten sofort, verfallen aber beim Schließen des Terminals oder Neustart.
>
> -> `-n` maximale Anzahl offener Datei-Descriptors (open files) 
> -> `-v` maximaler virtueller Speicher 
> -> `-f` maximale Dateigröße
#### `/etc/security/limits.conf` 
> Die Datei `/etc/security/limits.conf` definiert persistente Ressourcen-Limits für Benutzer oder Gruppen, die beim Login via PAM (Pluggable Authentication Modules) greifen.

> Syntax-Aufbau:
> ```
> <domain> <type> <item> <value>
> ```
> 
> -> `domain`:
>  betroffener Benutzername, Gruppennamen (eingeleitet mit `@group`) oder `*` für alle
> 
> -> `type`:
> `soft` der Schwellenwert, der vom Benutzer gewarnt oder angepasst werden kann
> `hard` das absolute Limit, das vom Benutzer nicht überschritten werden darf 
> `-` setzt Soft- und Hard-Limit gleichzeitig
> 
> -> `item`:
>  die zu limitierende Ressource (z. B. `nproc` für Prozesse, `nofile` für offene Dateien)
>  
> -> `value`:
> der numerische Grenzwert oder `unlimited`
# Prozessverwaltung
## Analyse
### `ps`(statisch)
>(Hauptanwendung: Prozesse suchen/filtern)
```bash
ps
```
>zeigt aktive Prozesse der aktuellen Shell-Sitzung
>
> `-f` Full-Format (u.a. PPID)
> `-v` zeigt Prozentsatz des Speicherverbrauchs
> `-p <PID>` zeigt Informationen zu bestimmten Prozessen
>
```bash
ps aux
```
> zeigt alle laufenden Prozesse im System im Detail (User, PID, CPU, RAM)
```bash
pstree
```
> visualisiert Prozess-Hierarchien als Baumstruktur
### `top/htop`(dynamisch)
>(Hauptanwendung: Leistungsanalyse)
```bash
top
```
> klassische interaktive Live-Anzeige der Prozesse, standardmäßig sortiert nach Prozentsatz der beanspruchten CPU Zeit in absteigender Reihenfolge
> 
> Optionen:
> -> `-b -n1` "-b" ändert Format zu Text statt interaktiver Ansicht und "-n1" stoppt nach einem Durchlauf
> 
> Steuerung:
> -> `M` Speicherverbrauch
> -> `N` Prozess-ID
> -> `T` Laufzeit
> -> `P` Prozentsatz der CPU-Auslastung
> -> `R` rekursive Reihenfolge
> -> `L` Suchen
> -> `d` bestimmt das Zeitintervall in Sekunden zwischen den Aktualisierungen
```bash
htop
```
> interaktive Live-Anzeige aller Prozesse (benutzerfreundlicher)
> 
> Shortcuts:
> -> `F10/q` = Beenden 
> -> `F3` Prozess suchen 
> -> `F9/k` Prozess beenden 
> -> `P` nach CPU-Auslastung sortieren 
> -> `M` nach RAM-Auslastung sortieren
### `pgrep/pidof`
```bash
pgrep <Prozessname>
```
> gibt PID aus
>
> -> `-l`  gibt Prozessname mit aus
> ->  `-u <Benutzer>` gibt PID aller Prozesse eines Benutzers aus
```bash
pidof <Programmname>
```
>gibt PID eines laufenden Programms aus
## Prozess-Steuerung
### `kill`
```bash
kill <PID>
```
> sendet Signal an eine PID für ordentliches Beenden (-15)
```bash
kill -9 <PID>
```
> erzwingt sofortiges Beenden des Prozesses (-9)
```bash
kill %<Job-ID>
```
> beendet Job
```bash
killall <Prozessname>
```
> beendet alle Prozesse, die diesen Namen tragen
```bash
pkill <Muster>
```
> beendet Prozesse basierend auf Namensmustern
>
> -> `-u <Benutzer>` alle Benutzerprozesse beenden
>
> Signale:
> -> `-15` (SIGTERM) ordentliches Beenden
> -> `-9` (SIGKILL) Sofortiger Abbruch durch Kernel
> -> `-1` (SIGHUP) Konfiguration neu laden
> -> `-2` (SIGINT) Unterbrechung (entspricht Strg + C)
### `nice/renice`
> **Allgemein**
>
> Mit `nice` und `renice` wird die CPU-Priorität (Nice-Wert) von Prozessen gesteuert.
> Skala: `-20` (höchste Priorität) bis `19` (niedrigste Priorität).  Standard:`0`.
> 
> Regel: normale Nutzer können Werte nur erhöhen (netter machen). 
> Nur `root` kann Werte negativ machen oder bereits erhöhte Prioritäten wieder senken.
> 
> in `top` sieht man den Nice-Wert in der Spalte `NI`
> mit `ps` den Nice-Wert abfragen:  `ps -o pid,ni,cmd -p <PID>`

> **Befehle**
```bash
nice -n <Wert> <Prozess>
```
> Prozess direkt mit Priorität starten
```
renice -n [Wert] -p [PID]
```
> ändert den Nice-Wert der PID
## Job-Control
```bash
<Befehl> &
```
> startet Befehl im Hintergrund
```bash
jobs
```
> listet alle Hintergrund-Jobs der aktuellen Shell-Sitzung auf
```bash
Strg + z
```
> pausiert aktuellen Vordergrund-Prozess und schickt ihn in den Hintergrund
```bash
kill %<Job-ID>
```
> beendet Job
```bash
bg %<Job-ID>
```
> setzt einen pausierten Hintergrund-Prozess fort
```bash
fg %<Job-ID>
```
> holt einen Hintergrund-Prozess zurück in den Vordergrund
```bash
nohup <Befehl> &
```
> führt Befehl im Hintergrund aus, selbst wenn Terminal-Sitzung geschlossen wird
## Systemd
### `systemctl`
#### `systemctl`
```bash
systemctl <Befehl> <Optional-Dienst>
```
> Basis-Befehle:
> -> `status` zeigt aktuellen Status
> -> `start` startet Dienst sofort
> -> `stop` stoppt laufenden Dienst
> -> `restart` startet Dienst neu
> -> `reload` lädt Konfiguration eines Dienstes neu, ohne ihn zu stoppen
> -> `enable` aktiviert automatischen Start des Dienstes beim Systemstart 
> -> `enable --now` für sofortigen Start
> -> `disable` deaktiviert automatischen Start beim Systemstart
> -> `is-active` prüft, ob ein Dienst aktuell läuft
> -> `is-enabled` prüft, ob Autostart aktiv ist
> 
> -> `--failed` listet alle Dienste auf, die sich im Fehlerzustand befinden
> -> `daemon-reload` lädt Systemd-Konfigurationsdateien neu (nach Änderung)
> 
> -> `list-units --type=<Typ>` alle Einheiten eines Typs auflisten (z.B. target, socket)
> -> `list-dependencies` Abhängigkeiten eines Dienstes anzeigen
> 
> -> `mask` blockiert Dienst vollständig, sodass er auch nicht von anderen Diensten gestartet werden kann
> -> `unmask` hebt vollständige Blockierung des Dienstes auf
#### `sockets`
```bash
systemctl list-sockets
```
> listet alle aktiven Socket-Units und die zugehörigen Dienste auf
```bash
systemctl stop <Dienst>.socket
```
> stoppt Socket-Listener, um automatisches Wiederbeleben des Dienstes bei eingehenden Anfragen zu verhindern
#### `Targets`
> **systemd Targets**
> 
> Ein Target ist ein definierter Systemzustand (der moderne Ersatz für die alten SysVinit-Runlevels). Es bestimmt, welche Dienste beim Starten aktiv sind und wie sich das System verhält.
> 
> Wichtige Targets:
> -> multi-user.target: Standard für Server (Konsole und Netzwerk, ohne grafische Oberfläche)
> -> graphical.target: System mit grafischer Benutzeroberfläche
> -> rescue.target / emergency.target: eingeschränkte Modi für die Systemreparatur
```bash
systemctl get-default
```
> aktuelles Standard-Target anzeigen (z. B. `multi-user.target` für CLI, `graphical.target` für GUI)
```bash
sudo systemctl set-default <target>
```
> Standard-Target für den nächsten Boot ändern
```bash
sudo systemctl isolate <target>
```
> sofort in ein anderes Target wechseln (z. B. in den Notfall-Modus)
#### `loginctl`
```bash
loginctl list-sessions
```
> aktive Sitzungen anzeigen
```bash
loginctl list-users
```
> eingeloggte Benutzer anzeigen
```bash
loginctl show-session <Session-ID> loginctl show-user <Username>
```
> Details zu einer Sitzung oder einem Benutzer anzeigen
```bash
sudo loginctl terminate-user <Username>
```
> User-Session komplett beenden
#### Troubleshooting
> **Konfigurationen auf Syntaxfehler testen**
```bash
sudo <Dienst> <Test-Flag>
```
> universelles Schema
>
> Optionen:
> `-t` (Test) prüft die Syntax und bricht bei Fehlern ab
> `configtest` spezielles Argument bei älteren Diensten 
> `-C` / `--check` gängiger Parameter bei diversen Daemons für Dry-Runs oder Syntaxprüfungen
### `journalctl`
> Bei einem System mit `systemd` werden Meldungen normalerweise vom Dienst systemd-journald gesammelt, gespeichert und mit `journalctl` ausgelesen.
> Das Journal ist kein gewöhnliches Klartext-Logfile.

> **Speicherort des Journals**
> 
> Je nach Konfiguration können Journaldaten unter verschiedenen Verzeichnissen liegen.
> Persistente Journal-Daten liegen typischerweise unter `/var/log/journal/`.
> Flüchtige Journal-Daten können unter `/run/log/journal/`liegen.
> Ob Logs dauerhaft auf der Festplatte gespeichert werden, hängt von der `journald`-Konfiguration ab.

> **Befehle**
```bash
journalctl
```
> gesamtes Journal anzeigen
> 
> -> `-k` nur Kernel-Meldungen
> -> `-b` nur Meldungen des aktuellen Bootvorgangs (aktuell: `-b 0` vorher: `-b -1`)
> -> `-f` Journal live verfolgen
> -> `-u <Dienst>` Meldungen eines bestimmten Dienstes
> -> `-p err` Meldungen mit Fehler-Priorität oder höher
> -> `--no-pager` deaktiviert interaktiven Pager (less), sodass die Log-Ausgabe ungebremst ins Terminal geschrieben wird
> -> `--disk-usage` zeigt, wie viel Speicherplatz die gesammelten Journal-Logdateien auf der Festplatte belegen
> -> `--vacuum-time=<Zahl>d` löscht automatisch alle alten Log-Einträge, die älter als (Zahl) Tage sind
>
> -> `-n <Zahl>` letzte (Zahl) Meldungen
> -> `--since "<Zahl> minutes/hour ago"` letzter Zeitraum (Alternativ z.B. `"2026-10-23 14:00:00"`)
> -> `--until "<Zahl> minutes/hour ago"` Zeitraum bis
> 
> -> `-o <Format>` bestimmt Ausgabeformat:
> short: journalctl-Standard (Datum ohne Jahr)
> short-iso: internationale Standardformat ISO 8601 (Jahr, Monat, Tag, Uhrzeit, Zeitzone)
> cat: gibt nur reinen Nachrichtentext ohne Metadaten
> json-pretty: lesbares JSON-Objekt
> verbose: listet jedes interne Journal-Feld für jeden Eintrag auf (maximale Detailtiefe)
### Logging
#### Allgemein
> Linux-Dienste und Programme erzeugen Log-Nachrichten, um Ereignisse, Fehler und Zustandsänderungen festzuhalten.
> Klassisch übernimmt ein Syslog-Daemon wie `syslogd`, `rsyslog` oder `syslog-ng` die Verarbeitung und Speicherung dieser Nachrichten.
> Auf modernen Linux-Systemen können klassische Syslog-Daemons und `systemd-journald` zusammenarbeiten.

> **Ablauf**
>
> 1) Kernel Ring Buffer (`dmesg`): zeichnet auf, sobald der Kernel lädt – noch bevor Dateisysteme gemountet sind oder Log-Daemons laufen.
> 
> 2) `systemd-journald`: übernimmt beim Start von systemd, liest den Ring Buffer ein und sammelt ab da alle Anwendungs- und System-Logs.
>
> 3) klassisches Syslog (`/var/log/`): liest die Daten vom Journal oder Daemon ab und schreibt sie dauerhaft in Textdateien auf die Festplatte.
> 
> 4) `logrotate`: greift als Letztes ein, wenn die Dateien auf der Festplatte zu groß werden
#### Kernel Ring Buffer
> Datenstruktur fester Größe im RAM, die Kernel-Meldungen beim Booten (bevor Syslog bereit ist) und im laufenden Betrieb aufzeichnet. 
> Wächst der Speicher stetig, werden die ältesten Einträge automatisch überschrieben. 
```bash
dmesg
```
> gibt den Kernel Ring Buffer aus
```bash
dmesg | grep boot
```
> filtert Ring-Buffer-Meldungen nach Boot-Ereignissen
#### /var/log/
> `/var/log/` ist der klassische Speicherort für Logdateien

> Typische Dateien:
> 
> `/var/log/auth.log`  
> -> Authentifizierungs- und Login-Ereignisse  
> -> häufig auf Debian/Ubuntu
> 
> `/var/log/syslog`  
> -> allgemeine Systemmeldungen  
> -> häufig auf Debian/Ubuntu
> 
> `/var/log/messages`  
> -> allgemeine System- und Anwendungsnachrichten  
> -> häufig auf RHEL-artigen Systemen
> 
> `/var/log/kern.log`  
> -> Kernel-Meldungen  
> -> distributionabhängig

> **Wichtig:** 
> Die tatsächlich vorhandenen Dateien und deren Inhalt hängen von Distribution und Logging-Konfiguration ab. Es gibt daher nicht auf jedem Linux-System exakt dieselben Logdateien.
#### Log Rotation
> Logdateien können über längere Zeit sehr groß werden und dadurch den verfügbaren Speicherplatz aufbrauchen.
> Dafür wird häufig `logrotate` eingesetzt.
> 
> `logrotate` kann Logdateien:  
> -> rotieren/umbenennen  
> -> archivieren  
> -> komprimieren  
> -> nach einer bestimmten Zeit löschen

> Dabei ist:  
> -> `error.log` = aktuelles Log  
> -> `error.log.1` = vorherige Version  
> -> `error.log.2.gz` = ältere, mit gzip komprimierte Version
> 
> Die Konfiguration von `logrotate` liegt typischerweise unter:
```
/etc/logrotate.conf
/etc/logrotate.d/
```
#### Logdateien auslesen
> **Klassische Logdateien**
> 
> Klartext-Logs können direkt mit `less`, `cat`, `head` oder `tail` gelesen und mit `grep` gefiltert werden.
> 
>Logging untersuchen:
> `ls -lh /var/log/` zeigt vorhandene Logdateien
> `less /var/log/syslog` öffnet Logdatei zur Betrachtung
> `tail -f /var/log/syslog` verfolgt neue Einträge einer Logdatei live

> **Binäre Logdateien**
>
> Nicht jedes Log unter `/var/log` ist eine normale Textdatei.
> Der Inhalt dieser Dateien kann nicht klassisch ausgelesen werden, sondern mit speziellen Programmen.
> Beispiele:
```bash
last /var/log/wtmp
```
> Auslesen von speziellen Logdateien, hier erfolgreiche Login- und Logout-Ereignisse
```bash
lastb  /var/log/btmp
```
> Auswertung von fehlgeschlagenen Login-Versuchen
### Kernel-Module
> Kernel-Module sind Treiber und Kernelfunktionen, die dynamisch zur Laufzeit geladen oder entladen werden können, ohne das System neu zu starten.

> **Befehle**
```bash
lsmod
```
> zeigt alle aktuell geladenen Kernel-Module an (liest `/proc/modules`)
```bash
modinfo <Modulname>
```
> zeigt Details, Autoren, Parameter und Abhängigkeiten eines Kernel-Moduls
```bash
sudo modprobe <Modulname>
```
> lädt ein Kernel-Modul inklusive aller benötigten Abhängigkeiten in den RAM
```bash
sudo modprobe -r <Modulname>
```
> entlädt ein Kernel-Modul und dessen nicht mehr benötigte Abhängigkeiten
```bash
echo "blacklist <Modulname>" | sudo tee /etc/modprobe.d/blacklist-<Modulname>.conf
```
> dauerhaftes Deaktivieren/Blacklisten eines ungewollten Kernel-Moduls
### Kernel-Laufzeit-Parameter
> -> `/proc/sys/` Schnittstelle für Live-Kernel-Parameter im RAM
> -> `/etc/sysctl.conf` & `/etc/sysctl.d/*.conf` Dateien für dauerhafte Kernel-Einstellungen

> **Befehle**
```bash
sysctl -a
```
> listet alle verfügbaren Kernel-Parameter und deren aktuelle Werte auf
```bash
sysctl <Parameter.Name>
```
> fragt einen konkreten Kernel-Parameter ab (z. B. "sysctl net.ipv4.ip_forward")
```bash
sudo sysctl -w net.ipv4.ip_forward=1
```
> ändert einen Kernel-Parameter temporär im RAM (bis zum nächsten Reboot)
```bash
sudo sysctl -p
```
> lädt Konfigurationen aus `/etc/sysctl.conf` neu in den RAM
```bash
sudo sysctl --system
```
> lädt alle Konfigurationsdateien aus `/etc/sysctl.d/` und `/etc/sysctl.conf` neu
## Cronjobs
### Zeitsteuerung
> Zeitgesteuerte automatische Befehlsausführung im Hintergrund (Daemon: `cron`)
 
> **Konfigurationspfade** 
> `crontab -e` öffnet die benutzerspezifische Crontab 
> `/etc/crontab` systemweite Crontab (enthält Spalte für den ausführenden Benutzer)
> `/etc/cron.d/` Verzeichnis für modulare systemweite Cron-Dateien
> `/etc/cron.daily/`, `/etc/cron.hourly/`, etc. Verzeichnisse für Skripte, die automatisch in festen Intervallen laufen

> **Crontab Syntax (5 Sterne)** 
> ```
> * * * * * <Befehl/Skript>
> ```
> | | | | |
> | | | | +----- Wochentag (0-7) (0 und 7 = Sonntag)
> | | | +------- Monat (1-12)
> | | +--------- Tag des Monats (1-31) 
> | +----------- Stunde (0-23) 
> +------------- Minute (0-59)
> 
> **Syntax-Beispiele:**
> `*/15 * * * *` alle 15 Minuten
> `0 2 * * *` täglich um 02:00 Uhr nachts
> `0 8 * * 1` jeden Montag um 08:00 Uhr 
### Crontab
```bash
systemctl status cron
```
> prüft, ob der Cron-Daemon aktiv ist (auf RHEL-Systemen: "crond")
```bash
crontab -l
```
> listet alle aktiven Cronjobs des aktuellen Benutzers auf
```bash
crontab -e
```
> öffnet die Crontab des aktuellen Benutzers zum Bearbeiten
```bash
sudo crontab -u <Benutzername> -e
```
> bearbeitet die Crontab eines spezifischen Benutzers als Root
```bash
crontab -r
```
> löscht die komplette Crontab des aktuellen Benutzers (Vorsicht!)
# Datenbanken
## Allgemein
> **Datenbank (DB):** 
> Strukturierte Datensammlung, verwaltet über ein DBMS (Database Management System) wie MySQL.

> **Relationale DB (SQL):** 
> Daten liegen in Tabellen (Zeilen/Datensätze, Spalten/Attribute), verknüpft über Schlüssel (`PRIMARY KEY`, `FOREIGN KEY`).

> **MySQL:** 
> Beliebtes relationales SQL-DBMS; Standard-Port ist `3306` (TCP).
## MySQL
### Allgemein
> **Relationale DBs:**
> speichern Daten in Tabellen (Zeilen = Datensätze, Spalten = Felder).
> 
> **Schlüssel:** 
> `PRIMARY KEY` (eindeutige ID pro Zeile), `FOREIGN KEY` (Verweis auf ID einer anderen Tabelle).
> 
> **Datentypen:**
> `INT` (Ganzzahl), `VARCHAR(n)` (Text), `DECIMAL(10,2)` (exakte Zahl), `TIMESTAMP` (Zeit).
### Basics
```sql
CREATE DATABASE test_db;
USE test_db;

CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO users (name) VALUES ('Max');
SELECT * FROM users;
```
> `CREATE DATABASE test_db;` legt eine neue leere Datenbank an
> 
> `USE test_db;` wählt diese Datenbank als aktives Arbeitsziel aus
> 
> `CREATE TABLE users (...);` erstellt eine Tabelle `users` mit Spalten (`id`, `name`, `created_at`)
> 
> `INSERT INTO users (name) VALUES ('Max');` schreibt einen Datensatz in die Tabelle
> 
> `SELECT * FROM users;` liest alle Einträge aus `users` aus und zeigt sie an

```sql
SHOW DATABASES;
```
> zeigt Datenbank an

```sql
UPDATE users SET name = 'Moritz' WHERE id = 1; 
DELETE FROM users WHERE id = 1;
```
> Daten ändern / löschen

```sql
SELECT users.name, orders.amount
FROM users
JOIN orders ON users.id = orders.user_id;
```
> verbindet Tabellen über gemeinsame IDs (`JOIN` = nur passende Treffer)
### Docker
```bash
docker volume create mysql_data
docker run -d --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=<PW> \
  -e MYSQL_DATABASE=<app_db> \
  -v mysql_data:/var/lib/mysql \
  -p 127.0.0.1:3306:3306 \
  --restart unless-stopped \
  mysql:8.4
```
> Docker starten & anbinden
> 
> `-e MYSQL_ROOT_PASSWORD=...` Pflicht-Variable für das Root-Passwort beim Erststart
> `-v mysql_data:/var/lib/mysql` speichert Datenbankdateien dauerhaft im Volume (überlebt Container-Löschen)
> `-p 127.0.0.1:3306:3306`: Bindet Port `3306` nur an `localhost` (sicherer als ungeschützt nach außen)
```bash
docker exec -it mysql-db mysql -u root -p
```
> öffnet den MySQL-Client direkt im laufenden Container (Passwort eingeben)
```bash
docker stop mysql-db && docker rm mysql-db
```
> stoppt und löscht den Container; `mysql_data` bleibt erhalten
# Netzwerkverwaltung
## Theorie
### Schichtenmodell
> **1) Link Layer (Verbindungsschicht)**
> 
> -> Steuert die Kommunikation zwischen direkt angeschlossenen Geräten im selben lokalen Netzwerk.
> -> Nutzt MAC-Adressen (Media Access Control, 48 Bit) zur eindeutigen Identifizierung von Netzwerkkarten (NICs).
> -> Adressen sind nicht global gültig; Pakete bleiben im lokalen Link.

> **2) Network Layer (Netzwerkschicht)**
> 
> -> Ermöglicht die Kommunikation über lokale Netzwerkgrenzen hinaus (Routing).
> -> Nutzt global eindeutige IP-Adressen (IPv4 / IPv6).
> -> Router verbinden verschiedene Netzwerke ("Inseln") und leiten Pakete zielgerichtet weiter.

> **3) Application Layer (Anwendungsschicht)**
> 
> -> Höchste Schicht; ermöglicht es einzelnen Programmen, sich über das Netzwerk zu verbinden (z. B. Webbrowser, SSH-Client).
### IPv4 & Adressierung
> -> 32 Bit lang (ca. 4,3 Milliarden theoretische Adressen).
> 
> -> Dezimale Notation: 4 durch Punkte getrennte Zahlen von 0 bis 255
> 
> -> Subnetzmaske & CIDR: Teilt die IP in Netzwerkteil und Hostteil auf (z. B. Netzmaske `255.255.255.0` entspricht CIDR `/24`).
> 
> -> Private IPv4-Bereiche (werden im Internet nicht geroutet):
>   - `10.0.0.0/8`
>   - `172.16.0.0/12`
>   - `192.168.0.0/16`

> -> NAT / Masquerading: Übersetzt interne private IPs auf eine einzige öffentliche IP des Routers.
### IPv6 & Adressierung
> -> 128 Bit lang (löst den IPv4-Adressmangel).
> -> Hexadezimale Notation: 8 Gruppen à 4 Zeichen, getrennt durch Doppelpunkte (z. B. `2001:db8:0:abcd::7334`).
> -> Abkürzungsregeln: Führende Nullen pro Gruppe entfernbar
> -> aufeinanderfolgende Null-Gruppen einmalig durch `::` ersetzbar.

> -> Wichtige Präfixe:
>   - Global Unique Address: Weltweit gültig.
>   - Unique Local Address (ULA): Für interne Netze (beginnt mit `fc` oder `fd`).
>   - Link Local Address: Nur für den direkten lokalen Link gültig (beginnt immer mit `fe80`).

> -> Konfiguration:
>   - SLAAC (Stateless Address Autoconfiguration): Automatisierte Konfiguration direkt über den Router.
>   - DHCPv6: Zentraler Server steuert die IP-Vergabe kontrolliert.
### Routing & Hops
> -> Routing: 
> Prozess des Weiterleitens von Paketen vom Quell- zum Zielnetzwerk.
> 
> -> Routing-Tabelle: 
> Jedes Gerät speichert, welches Netz direkt erreichbar ist oder über welchen Router (Gateway) geleitet werden muss.
> 
> -> Default Route: 
> Standard-Gateway für alle Pakete, deren Ziel in keiner spezifischen Route steht.
> 
> -> Hop: Jede Zwischenstation (Router) auf dem Weg eines Pakets.
### Sockets
> -> Socket:
> Kommunikationsendpunkt für Programme über ein Netzwerk oder lokal.
> 
> -> Typen:
>   - Unix Sockets: Nur für Prozesse auf demselben Gerät.
>   - UDP Sockets: Schnell, verbindungslos, keine Zustellungsgarantie (z. B. DNS, Streaming).
>   - TCP Sockets: Verbindungsorientiert, zuverlässig, garantiert Reihenfolge (z. B. HTTP, SSH).
### Ports 
> Begriffe:
> -> IP-Adresse = Rechner im Netzwerk
> -> Port = welches Programm dort wartet
> -> localhost (127.0.0.1) = System-IP zur Kommunikation mit sich selbst

> Transport-Protokolle an Ports (TCP vs. UDP)
>
> TCP (Transmission Control Protocol):
> -> Verbindungsorientiert und zuverlässig. Vor dem Datenaustausch wird eine Verbindung aufgebaut ("Handshake"). Der Empfänger bestätigt jedes Paket; verlorene Daten werden automatisch neu gesendet. 
> -> Nutzung: Überall, wo keine Fehler passieren dürfen (Webseiten/HTTP, SSH, E-Mail, Datenbanken).
> 
> UDP (User Datagram Protocol):
> -> Verbindungslos ("Fire and Forget"). Pakete werden einfach abgeschickt, ohne zu prüfen, ob sie ankommen oder in der richtigen Reihenfolge sind. Dafür ist es extrem schnell und hat keinen Rechen-Overhead.
> -> Nutzung: DNS (Namensauflösung), Video-Streaming, VoIP (Telefonie), Online-Spiele.

> Wichtige Standard-Ports: 
> -> 22 (SSH) = verschlüsselte Fernsteuerung von Linux-Servern 
> -> 2220 = häufiger abweichender SSH-Port (Standard ist 22) 
> -> 80 (HTTP) = unverschlüsselte Webseiten 
> -> 443 (HTTPS) = verschlüsselte Webseiten 
> -> 21 (FTP) = Datei-Übertragungen 
> -> 53 (DNS) = Namensauflösung im Internet 
> -> 3306 (MySQL/MariaDB) = Datenbanken
## Ports
```bash
nc <Host> <Port>
```
> baut unverschlüsselte Text-Verbindung zu einem Port auf 
> (Beispiel: nc localhost 30000)
```bash
nc -l -p <Port> < <Datei>
```
> öffnet lokalen Port als Server (Listener) und sendet bei eingehender Verbindung automatisch den Inhalt der Datei
> 
> -> `-l` versetzt Netcat in Server-Modus
```bash
openssl s_client -connect <Host>:<Port>
```
> baut verschlüsselte (SSL/TLS) Netzwerkverbindung zu einem Port auf (funktioniert wie nc, jedoch abgesichert)
## SSL-Zertifikate
> **Self-Signed Zertifikat erstellen (Single-Command)**
> ```bash
> sudo openssl req -x509 -nodes -days <Zahl> -newkey rsa:2048 -keyout /etc/ssl/private/<Datei.key> -out /etc/ssl/certs/<Datei.crt> -subj "/CN=<common-name>"
> ```
> -> `req` Untermodul für Zertifikatsanfragen und selbstsignierte Zertifikate
> -> `-x509` erstellt selbstsigniertes Zertifikat statt einer Certificate Signing Request (CSR)
> -> `-nodes` (No DES) verhindert Passwortabfrage für den privaten Schlüssel
> -> `-days <Zahl>` legt die Gültigkeitsdauer auf x Tage fest
> -> `-newkey rsa:2048` generiert einen neuen RSA-Schlüssel mit 2048 Bit Länge
> -> `-keyout` Speicherort für den privaten Schlüssel
> -> `-out` Speicherort für das öffentliche Zertifikat
> -> `-subj "/CN=test.local"` setzt den Common Name per Parameter, ohne interaktive Abfrage
>
> **Zertifikats-Signieranfrage stellen**
> ```bash
> openssl req -new -newkey rsa:2048 -nodes -keyout /etc/ssl/private/<Datei.key> -out /etc/ssl/certs/<Datei.csr>
> ```

> **Zertifikatsdetails prüfen**
> ```bash
> openssl x509 -in /etc/ssl/certs/app.crt -text -noout
> ```
> zeigt den vollständigen Inhalt des Zertifikats lesbar an (kein Zertifikatinhalt)
> ```bash
> openssl x509 -in /etc/ssl/certs/app.crt -dates -noout
> ```
> zeigt gezielt das Start- und Ablaufdatum des Zertifikats an
> ```bash
> openssl x509 -in /etc/ssl/certs/app.crt -issuer -noout
> ```
> zeigt den Aussteller (Issuer) des Zertifikats an

> **Verschlüsselte Verbindung testen**
> ```bash
> openssl s_client -connect <Host>:<Port>
> ```
> baut eine verschlüsselte (SSL/TLS) Netzwerkverbindung zu einem Port auf (funktioniert wie `nc`, jedoch abgesichert und zeigt Details zum Zertifikat der Gegenseite)
## SSH
> SSH (Secure Shell):
> -> Netzwerkprotokoll zur verschlüsselten
> -> Fernsteuerung von Servern über die Konsole
```bash
ssh <User>@<Host>
```
> verbindet sich sicher mit einem entfernten Rechner
```bash
ssh -i <Schlüsseldatei> <User>@<Host> -p <Port>
```
> verbindet sich mittels SSH-Schlüssel statt Passwort
>  Schlüsseldatei benötigt Rechte: chmod 600 (rw-------)
```bash
ssh <User>@<Host> -p <Port> "<Befehl>"
```
> führt einen Befehl direkt auf dem entfernten Server aus und gibt die Ausgabe zurück

> allgemeine Optionen:
> -> `-i` steht für Identity und gibt Pfad zum privaten Schlüssel an
> -> `-p` steht für Port und gibt Zielport des SSH-Dienstes an
## Datenabruf
```bash
curl -I <URL>
```
> ruft Header-Informationen einer Webseite ab
```bash
wget <URL>
```
> lädt Dateien aus dem Netzwerk herunter
## Netzwerk-Analyse
### `ip`
```bash
ip a
```
> listet alle IP-Adressen (IPv4 & IPv6) aller Schnittstellen auf (Alias: `ip addr show`)
```bash
ip neighbor
```
> zeigt alle Geräte im lokalen Netzwerk (ARP-Cache), mit denen das System kürzlich direkt kommuniziert hat
```bash 
ip link show
```
> zeigt alle verfügbaren Netzwerkschnittstellen, MAC-Adressen und MTU (Maximum Transmission Unit)
```bash
sudo ip addr add <IP/CIDR> dev <Interface>
```
> weist einer Schnittstelle manuell eine IP-Adresse zu
```bash
ip route show
```
> listet die aktuelle IPv4-Routingtabelle auf
```bash
sudo ip route add default via <Router-IP>
```
> fügt eine Standardroute (Default Gateway) hinzu

> Interface:
> physischer oder virtueller Verbindungspunkt des Computers zum Netzwerk
>
> Bezeichnungen: 
> 
> -> `lo` Loopback-Interface (interner Rückkanal des Systems)
> -> `eth0` klassisches Ethernet (Legacy-Name)
> -> `eno1` Onboard-Netzwerkkarte (feste Firmware-Zuordnung)
> -> `ens33 / ens192` PCI-Express / Virtuelle Netzwerkkarte (Standard bei VMware, KVM etc.)
> -> `enp3s0` Netzwerkkarte nach PCI-Bus-Position benannt
> -> `wlan0 / wlp2s0` WLAN-Schnittstelle
> -> `tun0 / tap0` virtuelle Schnittstellen für VPN-Tunnel
> -> `docker0` virtuelle Netzwerk-Bridge für Docker-Container
### `ping`
```bash
ping -c <Anzahl> <Ziel-IP/Domain>
```
> sendet ICMP Echo Requests zur Prüfung der Erreichbarkeit
> -> `-c 3` stoppt nach 3 Paketen
```bash
ping6 -c 1 fe80::<Schnittstellenkennung>%<Interface>
```
> prüft IPv6 Link-Local-Adressen
### `nmap`
```bash 
sudo nmap -sn <IP-Adresse-Blöcke-1-bis-3>0/24
```
> Ping-Scan (Nur aktive Hosts im Subnetz finden, keine Port-Scans)
```bash
nmap <IP-Adresse>
```
> Standard-Portscan (scannt die 1.000 gängigsten Ports einer IP)
```bash
nmap -p <Ports> <IP-Adresse>
```
> spezifische Ports oder Portbereiche scannen (z.B. 1-1000 oder 22,40,80)
### DNS
#### Konfigurationsdateien
> `/etc/hosts` statische lokale Zuordnung von IP-Adressen zu Hostnamen (wird vor dem echten DNS geprüft)

> `/etc/resolv.conf` konfiguriert die vom System verwendeten DNS-Resolver und Nameserver.
#### Befehle
```bash
host <Domain>
```
> fragt IP-Adresse zu einem Domainnamen ab
```bash
dig <Domain>
```
> führt detaillierte DNS-Abfragen durch (zeigt Server-Antworten, Record-Typen wie A/AAAA etc.)
### Socket-Analyse
```bash
ss
```
> zeigt alle aktiven Verbindungen und Sockets an
> 
> -> `-t` nur TCP-Sockets 
> -> `-u` nur UDP-Sockets 
> -> `-l` nur lauschende Sockets 
> -> `-p` zeigt Prozessname/-ID (PID), die den Socket nutzt 
> -> `-n` zeigt numerische Ports statt Dienstnamen 
> -> `-4` / `-6` beschränkt auf IPv4 bzw. IPv6 
> -> `-s` zeigt eine statistische Zusammenfassung
```bash
ss -tulpn
```
> klassischer Admin-Befehl: zeigt alle lauschenden TCP- und UDP-Ports inklusive Prozessnamen und IDs numerisch an
```bash
ss -t
```
> listet alle aktiven TCP-Verbindungen auf
```bash
ss -4 -t -l
```
> zeigt alle lauschenden IPv4-TCP-Sockets an
```bash
ss -s
```
> liefert eine kompakte Zusammenfassung der Socket-Statistiken
## Container
### Docker
```bash
sudo apt update && sudo apt install -y docker.io
sudo systemctl enable --now docker
sudo usermod -aG docker $USER && newgrp docker
```
> persönliche Installation & Rechtevariante
```bash
docker run -d --name <Name> -P nginx:alpine
```
> Container einrichten
>
>`-d` (detached): läuft im Hintergrund  
> `--name test-app` gibt Container festen Namen statt Zufalls-ID 
> `-P` lässt Docker einen freien Host-Port automatisch auf den im Image freigegebenen Container-Port mappen
> `nginx:alpine` schlankes Beispiel-Image
```bash
docker ps -a
```
> zeigt alle Container inkl. gestoppter (ohne `-a` nur laufende)
```bash
docker logs <Name>
```
> zeigt die Ausgabe/Logs des Containers an
```bash
docker stop <Name>
docker rm <Name>
```
> fährt den laufenden Container herunter und löscht den gestoppten Container
```bash
docker rmi nginx:alpine
```
> löscht das heruntergeladene Image vom System (nur möglich, wenn kein Container mehr darauf basiert

```bash
docker volume create sql_data
```
> erstellt das Volume `sql_data` auf dem Host
```bash
docker run -it --name sql-work -v sql_data:/var/lib/mysql alpine sh
```
> startet einen interaktiven Container (`-it`) mit eingehängtem Volume (`-v sql_data:/var/lib/mysql`) und öffnet direkt eine Shell (`sh`) darin
```bash
docker start -ai sql-work
```
> starten wenn Container bereits läuft
# Ubuntu-Sicherheit
## Firewall
### Allgemein
> Host-basierte Paketfilter schützen das lokale System vor unerlaubten Netzwerkzugriffen.
> 
> Protokolle auf Layer 3 & 4 (Transportschicht):
> -> TCP (Transmission Control Protocol): verbindungsorientiert, garantiert Zustellung und Reihenfolge (z. B. SSH, HTTP, HTTPS, MySQL)
> -> UDP (User Datagram Protocol): verbindungslos, schnell, keine Auslieferungsgarantie (z. B. DNS, DHCP, Streaming, VPN WireGuard/OpenVPN)
> -> ICMP (Internet Control Message Protocol): Diagnose- und Steuerprotokoll (z. B. Ping, Traceroute)
> 
> Funktionsweise:
> -> Layer 3 (Netzwerk): Filterung nach Quell- und Ziel-IP-Adressen oder Subnetzen (CIDR)
> -> Layer 4 (Transport): Filterung nach Zielport und Protokoll (TCP/UDP)

> Architektur-Unterschiede:
> -> UFW (Uncomplicated Firewall): Standard auf Ubuntu/Debian. Vereinfachtes Inline-Frontend; Anweisungen werden direkt angewendet und gespeichert
> -> firewalld: Standard auf RHEL/Rocky Linux/Fedora. Arbeitet mit einem dynamischen Zonen-Konzept und trennt strikt zwischen Runtime (Laufzeit) und Permanent (Dauerhaft)

> **firewalld:**
> Das Zonen-Konzept (firewalld):
> -> Weist Netzwerkschnittstellen Sicherheitsstufen zu (z. B. `drop`, `block`, `public`, `external`, `dmz`, `work`, `home`, `trusted`, `internal`).
> -> Standard-Zone ist meist `public` (alle unbekannten eingehenden Verbindungen werden blockiert, außer explizit erlaubte).
> 
> Laufzeit vs. Permanente Konfiguration (firewalld):
> -> Ohne `--permanent`: Änderung gilt sofort im Arbeitsspeicher, geht bei Neustart/Reload verloren (gut zum schadlosen Testen)
> -> Mit `--permanent`: Änderung wird in Konfigurationsdatei geschrieben, wird aber erst nach `firewall-cmd --reload` aktiv
### `ufw`
#### Steuerung
```bash 
sudo ufw status
```
> zeigt aktuellen Firewall-Status
```bash
sudo ufw status verbose
```
> zeigt detaillierten Status inklusive Standard-Regeln (Default Policies)
```bash
sudo ufw status numbered
```
> listet alle aktiven Regeln nummeriert auf
```bash
sudo ufw enable
```
> aktiviert Firewall und übernimmt konfigurierte Regeln
```bash
sudo ufw disable
```
> deaktiviert Firewall (Regeln bleiben gespeichert)
```bash
sudo ufw reload
```
> lädt Regelsatz nach Änderungen neu
```bash
sudo ufw reset
```
> setzt alle Regeln auf den Auslieferungszustand zurück und deaktiviert die Firewall
#### Regeln verwalten
```bash 
sudo ufw allow <Port>/<Protokoll>
```
> erlaubt eingehenden Datenverkehr für einen Port
```bash 
sudo ufw deny <Port>/<Protokoll>
```
> blockiert eingehenden Datenverkehr für einen Port
```bash 
sudo ufw allow <Dienst>
```
> erlaubt vordefinierten Dienst
```bash 
sudo ufw allow from <IP-Adresse> to any port <Port>
```
> erlaubt Zugriff auf einen bestimmten Port nur für eine konkrete Quell-IP
```bash 
sudo ufw allow from <IP-Bereich/CIDR> to any port <Port>
```
> erlaubt Zugriff auf einen Port für ein gesamtes Subnetz
```bash 
sudo ufw delete <Regel-Nummer>
```
> löscht Regel anhand ihrer Nummer aus `ufw status numbered`
```bash 
sudo ufw delete allow <Port>/<Protokoll>
```
> löscht spezifische Erlaubnis-Regel direkt über die Syntax
## SSH
#### SSH-Key einrichten
> Die Authentifizierung über SSH-Schlüssel ersetzt oder ergänzt die klassische Passwort-Eingabe.
> ED25519 ist der moderne Standard-Algorithmus. Er ist extrem sicher, verarbeitet Daten schnell und erzeugt sehr kurze Schlüssel. 
> Privater Schlüssel (`id_ed25519`) bleibt auf dem lokalen (Windows) PC (`C:\Users\<Benutzer>\.ssh\`). Er darf niemals weitergegeben werden.
> Öffentlicher Schlüssel (`id_ed25519.pub`) wird auf den Linux-Server übertragen und dort in die Datei `~/.ssh/authorized_keys` eingetragen.

> 1)  SSH-Schlüssel lokal auf Windows erstellen (neues ED25519-Schlüsselpaar in Windows PowerShell) 
> Ablauf:
> -> Befehl in der Powershell ausführen und alle Abfragen (Speicherort und Passphrase) mit `Enter` bestätigen
> ```Powershell
> ssh-keygen -t ed25519
> ```

> 2) öffentlichen Schlüssel auslesen & kopieren
> Ablauf:
> -> Befehl in der Powershell ausführen und anschließend die komplette Ausgabe-Zeile kopieren
> ```Powershell
> type $env:USERPROFILE\.ssh\id_ed25519.pub
> ```

> 3) Schlüssel auf dem Linux-Server hinterlegen
> Ablauf:
> -> Auf dem Server einloggen und folgende Befehle ausführen
```bash
# 1. SSH-Verzeichnis erstellen & Rechte setzen (700 = rwx------)
mkdir -p ~/.ssh && chmod 700 ~/.ssh

# 2. Kopierten Schlüssel in die Datei authorized_keys anfügen
echo "HIER_KOPIERTEN_PUBLIC_KEY_EINFUEGEN" >> ~/.ssh/authorized_keys

# 3. Datei-Berechtigungen einschränken (600 = rw-------)
chmod 600 ~/.ssh/authorized_keys
```
> Hinweis Berechtigungen:
> OpenSSH verweigert den Login per Schlüssel, wenn der Ordner `.ssh` oder die Datei `authorized_keys` zu offene Rechte besitzen (z. B. Schreibrechte für andere Benutzer).
> `700`: Nur der Besitzer darf auf den Ordner `.ssh` zugreifen.
> `600`: Nur der Besitzer darf die `authorized_keys` lesen/schreiben.

> Login:
> ```
> ssh <user>@<server-ip>
> ```
#### Passwort-Nutzung
> Ziel: Eliminierung von Brute-Force-Angriffen durch Deaktivierung der passwortbasierten SSH-Authentifizierung und die ausschließliche Nutzung von Public-Key-Authentication

> 1) Passwort-Authentifizierung verbieten
> -> Konfigurationsdatei öffnen:
> ```
> sudo nano /etc/ssh/sshd_config.d/50-cloud-init.conf
> ```
> (Alternativ: `/etc/ssh/sshd_config`)
> 
> Wert anpassen:
> ```
> PasswordAuthentication no
> ```

> 2) Systemd Socket-Override anlegen
> Ubuntu nutzt Socket-Aktivierung für SSH. Um sicherzustellen, dass die Regeln übernommen werden, eine Ergänzung anlegen:
> ```
> sudo systemctl edit ssh.socket
> ```
> Folgenden Block einfügen:
> ```
> [Socket]
> FreeBind=yes
> ```

> 3) Dienst-Neustart
> ```
> sudo systemctl daemon-reload && sudo systemctl restart ssh.socket ssh
> ```
> Status prüfen:
> ```
> sudo systemctl status ssh.socket ssh
> ```

> 4) SSH Verifizierungstests
> Regulärer Verbindungsaufbau (mit Key):
> ```
> ssh <user>@<server-ip>
> ```
> (Ergebnis: Erfolgreicher Login ohne Passwortabfrage)
> 
> Gegenprobe (Passwort-Zwang simulieren):
> ```
> ssh -o PubkeyAuthentication=no user@<server-ip>
> ```
> (Ergebnis: `Permission denied (publickey)` – Verbindungsaufbau wird sofort verweigert)
## Fail2ban
### Installation
> 1) fail2ban installieren
> ```
> apt install fail2ban
> ```

> 2) Eigene `jail.local` anlegen
> (Originale Konfigurationsdatei vor Überschreiben bei Updates schützen):
> ```
> sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
> ```

> 3) `jail.local` bearbeiten
> Datei (in Nano) öffnen und`[sshd]` konfigurieren:
> ```
> [sshd]
> enabled = true
> port = ssh
> maxretry = 3
> findtime = 10m
> bantime = 1h
> ```

> 4) Dienst aktivieren & Status prüfen
> ```
> sudo systemctl enable fail2ban --now
> ```
> ```
> sudo fail2ban-client status sshd
> ```
### Verwaltung
```bash 
sudo fail2ban-client status
```
> zeigt Übersicht aller aktiven Jails (Schutzmodule)
```bash
sudo fail2ban-client status <Jail>
```
> zeigt Details zu einem spezifischen Jail
> 
> wichtige Jails: 
> `sshd` SSH-Zugriffe
> `recidive` sperrt wiederholte Wiederholungstäter langfristig
> `apache-auth` fehlgeschlagene Logins bei Apache Webservern
> `nginx-http-auth` fehlgeschlagene Logins bei Nginx Webservern
> `postfix` Mailserver-Schutz vor unbefugtem Relay oder Logins
```bash
sudo fail2ban-client set sshd unbanip <IP-Adresse>
```
> hebt die Sperre für eine bestimmte IP-Adresse manuell auf
## Sicherheitsupdates
### `unattended-upgrades`für VM
> Ziel: 
> Automatisierung von sicherheitsrelevanten System- und Paketaktualisierungen, um den Server ohne manuellen Eingriff auf dem aktuellen Stand zu halten.
> 
> Funktionsweise: 
> Das Tool prüft regelmäßig im Hintergrund auf neue Paketversionen, lädt diese herunter und installiert Sicherheitsupdates automatisch.
> 
> Mehrwert: 
> Schließt bekannte Sicherheitslücken zeitnah und verhindert, dass man manuelle Updates im Alltag vergisst.

> **Einrichtung & Konfiguration**

> 1) Paket installieren
> Tool wird installiert und sichergestellt, dass Benachrichtigungen aktiv sind:
> ```
> sudo apt update && sudo apt install unattended-upgrades update-notifier-common -y
> ```

> 2) automatische Updates aktivieren
> ```
> sudo dpkg-reconfigure --priority=low unattended-upgrades
> ```

> 3) Konfiguration
> ungenutzte Abhängigkeiten entfernen:
>```
>Unattended-Upgrade::Remove-Unused-Dependencies "true";
>```

> 4) automatischer Neustart
> ```
> Unattended-Upgrade::Automatic-Reboot "true";
> Unattended-Upgrade::Automatic-Reboot-Time "02:00";
> ```

> 5) Akkubetrieb-Prüfung deaktivieren
> -> da es sich um einen Server/eine VM handelt, wird verhindert, dass Updates fälschlicherweise wegen angeblichem "Akkubetrieb" pausieren:
> ```
> Unattended-Upgrade::OnlyOnACPower "false";
> ```

> 6) Testlauf
> Simuliert den Update-Prozess, um die Konfiguration auf Syntaxfehler zu prüfen:
> ```
> sudo unattended-upgrade --dry-run --debug
> ```

> Hinweise:
> 
> Enterprise-Best Practice: 
> In Testumgebungen wird oft `Automatic-Reboot "true"` (z.B. um 02:00 Uhr) genutzt. In Produktion steht dies meist auf `false`: Neustarts erfolgen stattdessen kontrolliert über Monitoring (`/var/run/reboot-required`) in definierten Wartungsfenstern.
## System-Härtung
## Rootkit-Scanning
## Log-Monitoring
# RHEL-Sicherheit
## Firewall
### Allgemein
> Host-basierte Paketfilter schützen das lokale System vor unerlaubten Netzwerkzugriffen.
> 
> Protokolle auf Layer 3 & 4 (Transportschicht):
> -> TCP (Transmission Control Protocol): verbindungsorientiert, garantiert Zustellung und Reihenfolge (z. B. SSH, HTTP, HTTPS, MySQL)
> -> UDP (User Datagram Protocol): verbindungslos, schnell, keine Auslieferungsgarantie (z. B. DNS, DHCP, Streaming, VPN WireGuard/OpenVPN)
> -> ICMP (Internet Control Message Protocol): Diagnose- und Steuerprotokoll (z. B. Ping, Traceroute)
> 
> Funktionsweise:
> -> Layer 3 (Netzwerk): Filterung nach Quell- und Ziel-IP-Adressen oder Subnetzen (CIDR)
> -> Layer 4 (Transport): Filterung nach Zielport und Protokoll (TCP/UDP)

> Architektur-Unterschiede:
> -> UFW (Uncomplicated Firewall): Standard auf Ubuntu/Debian. Vereinfachtes Inline-Frontend; Anweisungen werden direkt angewendet und gespeichert
> -> firewalld: Standard auf RHEL/Rocky Linux/Fedora. Arbeitet mit einem dynamischen Zonen-Konzept und trennt strikt zwischen Runtime (Laufzeit) und Permanent (Dauerhaft)

> **firewalld:**
> Das Zonen-Konzept (firewalld):
> -> Weist Netzwerkschnittstellen Sicherheitsstufen zu (z. B. `drop`, `block`, `public`, `external`, `dmz`, `work`, `home`, `trusted`, `internal`).
> -> Standard-Zone ist meist `public` (alle unbekannten eingehenden Verbindungen werden blockiert, außer explizit erlaubte).
> 
> Laufzeit vs. Permanente Konfiguration (firewalld):
> -> Ohne `--permanent`: Änderung gilt sofort im Arbeitsspeicher, geht bei Neustart/Reload verloren (gut zum schadlosen Testen)
> -> Mit `--permanent`: Änderung wird in Konfigurationsdatei geschrieben, wird aber erst nach `firewall-cmd --reload` aktiv
### `firewalld`
#### Steuerung
```bash
sudo firewall-cmd --state
```
> prüft, ob der `firewalld`-Dienst läuft
```bash
sudo firewall-cmd --get-active-zones
```
> zeigt aktuell genutzte Zonen und zugewiesene Netzwerkschnittstellen
```bash
sudo firewall-cmd --get-default-zone
```
> zeigt aktuelle Standard-Zone (meist `public`)
```bash
sudo firewall-cmd --set-default-zone=<Zone>
```
> ändert Standard-Zone des Systems
```bash
sudo firewall-cmd --reload
```
> lädt Konfiguration neu (übernimmt permanente Regeln im laufenden Betrieb)
```bash
sudo firewall-cmd --complete-reload
```
> bricht alle aktiven Verbindungen ab und lädt Regeln neu (Notfall-Reload)
#### Regeln verwalten
```bash
sudo firewall-cmd --list-all
```
> zeigt alle aktiven Regeln der Standard-Zone
```bash
sudo firewall-cmd --add-port=<Port>/<Protokoll>
```
> öffnet temporär einen Port (geht nach Neustart/Reload verloren)
```bash
sudo firewall-cmd --add-port=<Port>/<Protokoll> --permanent
```
> öffnet einen Port dauerhaft (erfordert anschließendes `--reload`)
```bash
sudo firewall-cmd --remove-port=<Port>/<Protokoll> --permanent
```
> entfernt eine permanente Port-Freigabe
```bash
sudo firewall-cmd --add-service=<Dienst> --permanent
```
> erlaubt vordefinierten Dienst dauerhaft
```bash
sudo firewall-cmd --remove-service=<Dienst> --permanent
```
> entfernt Dauerfreigabe eines Dienstes
```bash
sudo firewall-cmd --add-rich-rule='rule family="ipv4" source address="<IP-Adresse>" port port="<Port>" protocol="tcp" accept' --permanent
```
> erweiterte Regel (Rich Rule) für den Zugriff von nur einer bestimmten IP-Adresse