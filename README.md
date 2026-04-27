# Docker Schummelzettel

![Docker](https://img.shields.io/badge/docker-ready-blue)
![License](https://img.shields.io/badge/license-Apache%202.0-green)

Dies ist mein persönlicher Schummelzettel für [Docker](https://www.docker.com/) unter Linux. Andere Plattformen wurden nicht getestet.

## Inhaltsverzeichnis

- [Installation](#installation)
- [Docker Hub](#docker-hub)
- [Docker Befehle](#docker-befehle)
- [Images](#images)
- [Lizenz](#lizenz)

---

## Installation

Installationsanleitung für Linux:
https://docs.docker.com/engine/install/

Installationsanleitung für Ubuntu aus dem Repository:
https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository

---

## Docker Hub

Auf der Webseite [Docker Hub](https://hub.docker.com) sind viele Images vorhanden. 

**Warnung:** Man verwende nur offizielle oder verifizierte Images (zB mit "official" Badge auf Docker Hub), da Images Schadcode enthalten können.

---

## Docker Befehle

Die Grundstruktur von `docker run` ist wie folgt:

```bash
docker run [OPTIONEN] <IMAGE> [BEFEHL] [ARGUMENTE]
```
Bedeutung der Bestandteile:
* `docker run` startet einen neuen Container aus einem Image.
* `[OPTIONEN]`: Zum Beispiel `-it` für interaktiv, oder `--rm` um den Container nach Beendigung zu löschen.
* `<IMAGE>` ist der Name des Docker-Images, zum Beispiel `python:3.12-slim` oder `ubuntu:24.04`.
* `[BEFEHL]`: Das Programm, das im Container ausgeführt wird, zum Beispiel `python` oder `bash`.
* `[ARGUMENTE]`: Argumente, die an das Programm übergeben werden, zum Beispiel `-c "print('Hallo Wien!')"` für Python.

### Mögliche Optionen in `[OPTIONEN]`
* `-it`: Interaktiv und Terminal, nötig für Eingaben.
* `--rm`: Container wird nach dem Beenden automatisch gelöscht.
* `-d`: detached mode, also Container im Hintergrund starten.
* `-v <host_path>:<container_path>`: BIND-MOUNTS erstellen, um Dateien zwischen Host und Container teilen.
* `-v <name_of_volume>:<container_path>`: VOLUMES erstellen. Dieser Ordner wird von Docker verwaltet.
* `-p <host_port>:<container_port>`: Ports weiterleiten (für Webserver etc.).
* `--name <mein_name>`: Damit bekommt der Container den gewünschten Namen.

### Weitere Befehle

```bash
# Container starten.
docker run <IMAGE>

# Container mit Tag meinTag (zB 3.12-slim bei python oder 24.04 bei ubuntu) starten.
docker run <IMAGE>:meinTag

# Container starten mit Digest meinDigest, um genau Version zu fixieren.
docker run <IMAGE>@meinDigest

# Container starten und nach Stopp automatisch löschen.
docker run --rm <IMAGE>

# Container mit Umgebungsvariablen meineENV1 und meineENV2 starten:
docker run -e meineENV1=ABC -e meineENV2=DEF <IMAGE>

# Zeigt an, welche Container gerade laufen.
docker ps

# Zeigt an, welche Container gerade laufen oder beendet wurden.
docker ps -a

# Container stoppen.
docker stop <CONTAINER>

# Beendeten Container wieder starten.
# Option -ai: interaktiv und an Terminal anhängen
docker start -ai <CONTAINER>

# Container umbenennen:
docker rename <ALTER_NAME> <NEUER_NAME>

# Container löschen.
docker rm <CONTAINER>

# Alle gestoppten Container löschen.
docker container prune

# Logs des Containers betrachten. 
docker logs <CONTAINER>

# Programm/Befehl auf gestartetem Container ausführen.
docker exec -it <CONTAINER> <BEFEHL>

# Alle lokal verfügbaren Images anzeigen:
docker image ls

# Image löschen.
docker image rm <IMAGE>

# Image aus einer Registry (standardmäßig Docker Hub) herunterladen oder aktualisieren.
docker pull <IMAGE>

# Dateien aus dem Container kopieren.
docker cp <CONTAINER>:<PFAD_IM_CONTAINER> <PFAD_HOST>

# Dateien in den Container kopieren.
docker cp <PFAD_HOST> <CONTAINER>:<PFAD_IM_CONTAINER>
```

### Beispiele
```bash
# Bash-Terminal starten und Container danach löschen.
docker run --rm -it ubuntu:24.04 bash

# Einen Python-Befehl ausführen.
docker run python:3.12-slim python -c "print('Hallo Wien!')"

# Interaktive Python-Konsole starten.
docker run -it python:3.12-slim python

# Vom Container meinUbuntu den Ordner meinOrdner
# in den Host-Ordner meinOrdnerHost kopieren.
docker cp meinUbuntu:/home/ubuntu/meinOrdner /home/marco/meinOrdnerHost/
```

---

## Images

Man speichere das Dockerfile mit Namen `Dockerfile` im Root-Verzeichnis des Projekts mit dem beispielhaften Inhalt, wobei die Datei `requirements.txt` im Root-Verzeichnis und `main.py` im Ordner `Quellordner` vorhanden sein müssen:

```text
# Basis-Image, auf dem aufgebaut wird.
FROM ubuntu:24.04

# Metadaten über das Image (optional)
LABEL maintainer="Marco Zank"
LABEL version="0.1"
LABEL description="Ein einfaches Beispielimage"

# Arbeitsverzeichnis im Container festlegen
WORKDIR /app

# Abhängigkeiten installieren.
# Befehl
#        rm -rf /var/lib/apt/lists/*
# ist, um temporäre Paketlisten zu löschen. Image wird kleiner.
RUN apt-get update && apt-get install -y \
    vim \
    python3 \
    python3-venv \
    python3-pip \
    && rm -rf /var/lib/apt/lists/*

# Virtuelle Python-Umgebung erstellen.
RUN python3 -m venv /opt/venv

# Umgebungsvariable setzen, damit virtuelle Python-Umgebung verwendbar ist.
ENV PATH="/opt/venv/bin:$PATH"

COPY requirements.txt requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Dateien vom Host ins Image kopieren
# Der zweite COPY-Befehl sorgt dafür, dass Abhängigkeiten gecacht bleiben, 
# wenn sich nur der Code ändert.
COPY ./Quellordner /app

# Standardbefehl beim Starten des Containers, wenn beim Start kein Befehl angegeben wird.
CMD ["python3", "main.py"]
```

**Hinweis:** Man kann obiges Dockerfile vereinfachen, wenn man beispielsweise `FROM python:3.12-slim` verwendet, da dann keine virtuelle Python-Umgebung notwendig ist.

Man gehe ins Root-Verzeichnis des Projekts. Der Befehl
```bash
docker build -t meinimage .
```
für das Dockerfile namens `Dockerfile` *oder*
```bash
docker build -f <DOCKERFILE> -t meinimage .
```
erzeugt aus dem aktuellen Ordner `.` das Image `meinimage`, welches Docker *intern* speichert. 


```bash
# Export des Images.
docker save -o meinimage.tar meinimage

# Import des Images.
docker load -i meinimage.tar
```

---

## Lizenz

Dieses Projekt ist unter der Apache License 2.0 lizenziert. Siehe [LICENSE](LICENSE) für Details.

Weitere Informationen zur Lizenz finden Sie hier: https://www.apache.org/licenses/LICENSE-2.0

---
