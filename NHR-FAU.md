## Was ist das Erlangen National High-Performance Computing Center (NHR@FAU)?

Großes Rechenzentrum in Erlangen mit vielen modernen GPUs (u.a. 304 Nvidia A100 (40/80 GB), 352 Nvidia A40, 384 Nvidia H100, ...).

Die meisten wichtigen Infos sind in der [offiziellen Dokumentation](https://doc.nhr.fau.de/clusters/overview/) zu finden. 

## Einladung

Siehe auch in der [offiziellen Doku](https://doc.nhr.fau.de/account/).

Läuft grundsätzlich über Professoren an der OTH, d.h. Studenten haben ohne Einladung keinen Zugang. 

Ansprechpartner sind hier grundsätzlich die Professoren, die für euer Projekt zuständig sind.

## Verbindungsaufbau und Node Struktur

Für Details siehe die [offizielle Dokumentation](https://doc.nhr.fau.de/access/overview/)

#### HPC-Portal

Das __[HPC-Portal](https://doc.nhr.fau.de/hpc-portal/)__ ist erreichbar unter https://portal.hpc.fau.de/. Es ist die zentrale Anlaufstelle um Nutzung und Projekte zu verwalten.

#### SSH-Keypair generieren und im HPC-Portal hinterlegen

Der erste Schritt ist es, auf dem eigenen Computer ein SSH Keypair zu erzeugen. Dieser Schritt muss für jeden Computer, der zur Verbindung genutzt werden soll, wiederholt werden.

Der Public SSH Key muss dann im HPC-Portal hinterlegt werden. 

Eine Anleitung dazu findet sich [hier](https://doc.nhr.fau.de/access/ssh-command-line/).

Bei ersten Mal am besten Verbindung mit

```
ssh csnhr.nhr.fau.de
```
testen.

#### Verbindung via SSH

Nun ist eine Verbindung zu den Clustern möglich. Deren SSH Adressen sind [hier](https://doc.nhr.fau.de/clusters/overview/) zu finden.

## Nutzung



### Slurm und Jobs

Das NHR nutzt [Slurm](https://slurm.schedmd.com/overview.html) um Rechenresourcen auf dem Cluster zu verwalten. Über Slurm kann Zugriff zu einer Node angefragt werden, die dem Nutzer dann für einen bestimmten Zeitraum exklusiv zur Verfügung steht.

Um ein Script auf einem der GPUs auszuführen (etwa um ein Modell zu trainieren), kann es mit 
```
sbatch [optionen] <jobscript>
```

Eine interaktive Session kann mit

```
salloc [optionen]
```

gestartet werden. Diese erlaubt über die Konsole direkten Zugang zur zugewiesen Node und eignet sich damit für Tests, Debugging usw.

Um in einem anderen Konsolenfenster auf einen interaktiven Job zuzugreifen kann

```
srun --jobid=<jobID> --overlap --pty /bin/bash -l
```
verwendet werden.

Wenn der interaktive job vor Ablauf der Zeit nicht mehr gebraucht wird, sollte er mit `exit` beendet werden.

Details zu diesen Befehlen gibt es [hier](https://doc.nhr.fau.de/batch-processing/batch_system_slurm/).

### Nutzung von IDEs

Um die Nutzung von IDEs zu ermöglichen ist es nötig, einen SSH-Tunnel durch die Login Node ausfzubauen. Eine Anleiung dazu gibt es in diesem Abschnitt. 

__Vorraussetzung__: SSH Verbindung über Terminal ist eingerichtet und funktioniert. 

#### 1. Im Terminal Verbindung zum Cluster aufbauen und interaktiven Slurm Job starten

Beispiel für 2 Stunden Job mit einer A40 auf dem Alex Cluster:
```
ssh alex
salloc --gres=gpu:a40:1 --time=2:00:00
```

WICHTIG: Warten bis der Job gestartet und die Verbindung zur Node im Terminal aufgebaut ist.

#### 2. In einem _zweiten_ Terminal Fenster den SSH Tunnel aufbauen

Hierzu ist die adresse der Node mit dem grade erstellten interaktiven Job nötig. Diese findet sich in der Kommandozeile im ersten Terminal Fenster: Vor jedem Befehl steht dort soetwas wie `<Nutzer>@<Node>:~$`.

Ein SSH Tunnel auf dem Port 6000 wird dann über folgender Befehl aufgebaut:
```
ssh -L 6000:<Node>:22 alex
```
Andere Ports funktionieren auch, entsprechend müssen nachfolgende Befehle dann abgeändert werden.

Beispiel für Node mit ID a0325 auf dem Alex cluster:
```
ssh -L 6000:a0325:22 alex
```

### 3. PyCharm via Toolbox
In Toolbox unter neue SSH Verbindung folgenden Command einfügen (mit entsprechenden Anpassungen):

```
ssh -p 6000 <USERNAME>@localhost -i /PFAD/ZUM/PRIVATE_KEY
```

Der Username ist der HPC-Account Name (zu finden z.B. in der Konsole oder im HPC Portal).

Der angegebene Pfad ist der lokale Pfad zu dem bei der erstmaligen Verbindung zu den NHR Servern erstellte SSH Private Key.

Standardmäßig sollte das soetwas wie `/home/<Benutzername>/.ssh/id_ed25519_nhr_fau` sein.

Andere IDEs die SSH unterstützen sollen sich ebenfalls über den obigen Befehl verbinden können.

### 4. Virtual Environment Einrichten

Im Terminal die unten angegebenen Schritte durchführen, um ein Conda environment einzurichten.

Sobald dieses im Terminal funktioniert, dieses als lokalen Interpreter in Pycharm setzen:

Unten rechts auf interpreter clicken -> add new interpreter -> add local interpreter -> Select existing, conda, path to conda setzen, environment auswählen

Den Pfad zur Conda executable findet man mit `conda info`.

Sollte es Probleme geben erstmal PyCharm neustarten, alternativ auch in PyCharm mal "reload all from disk" und "invalidate cache" nutzen.

## Setup 

### Module

Module laden mit
```
module add {module}
```
Das muss jedes mal nach dem Verbinden gemacht werden!

Für die Programmierung mit Python ist
```
module add python
```
erforderlich.

### Venv

Als Erstes die [Initialisierungsschritte in der offiziellen Doku](https://doc.nhr.fau.de/environment/python-env/) befolgen.

Nicht vergessen ein geeignetes python modul mit conda zu laden!

Ein virtual environment wird am einfachsten mit Conda erstellt.

```
conda create -n <env. name> python=<py. version>
```

Wie mit Conda gewohnt muss es dann jedes Mal vor dem Start eines Jobs oder zu beginn einer interaktiven Session aktiviert werden.

```
conda activate <my_env_name_here>
```
