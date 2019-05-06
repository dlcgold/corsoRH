# LEZIONE 1

### INTRO

La Shell è un programma che prende un'input da tastiera e lo trasferisce all'OS.
Solitamente preinstallata nelle distribuzioni Linux si ha BASH, *Bourne Again SHell*.
Quando si usa una GUI si necessita di un programma, **l'emulatore di terminale** per
interagire con la shell. Quando la shell è pronta a ricevere l'input si ha lo **shell prompt**:
```shell
[me@linuxbox ~]$
```
Se alla fine si ha il simbolo \# si indica che si hanno i permessi di *superuser*:
```shell
[me@linuxbox ~]#
```
Nella shell si ha la *history* dei vecchi comandi (solitamente fino a 1000) utilizzando le frecce su e giù.
Con le frecce destra e sinistra ci si sposta col cursore nel comando.
Vediamo qualche comando base d'esempio:

* Per ora e data corrente:
  ```shell
  [me@linuxbox ~]$ date
  Thu Mar 8 15:09:41 EST 2018
  ```
* Per il calendario:
  ```shell
  [me@linuxbox ~]$ cal
  March 2018 
  Su Mo Tu We Th Fr Sa
               1  2  3
   4  5  6  7  8  9  10
  11 12 13 14 15 16 17
  18 19 20 21 22 23 24
  25 26 27 28 29 30 31
  ```
* Per vedere l'uso di memoria sui dischi e sulle varie partizione:
  ```shell
  [me@linuxbox ~]$ df
  File system    1K-blocks       Used  Available Use% Mounted on
  dev               4014968         0   4014968   0% /dev 
  run               4023532      9784   4013748   1% /run
  /dev/sda28      131449884 112330780  12412112  91% /
  tmpfs             4023532     36680   3986852   1% /dev/shm
  ```
* Per vedere l'uso della RAM:
  ```shell
  [me@linuxbox ~]$ free
                total        used        free      shared     buff/cache  available
  Mem:        8047068     2504188     1657424      357836     3885456     4839276
  Swap:             0           0           0

  ```
* Per terminare la sessione uso o Ctrl-d o:
  ```shell
  [me@linuxbox ~]$ exit
  ```
Inoltre si hanno delle sessioni di temrinale che girano sempre dietro la gui e sono accessibili con Ctrl-Alt + i tasti da F1 a F6; generalmente con
Alt-F7 si torna alla GUI.

### NAVIGAZIONE

Nei sistemi UNIX i file sono organizzati in una *hierical directory structure*, 
quindi in una struttura ad albero. La prima cartella è quindi chiamata *root directory*
e contiene files e altre sottocartelle. I sistemi UNIX e UNIX-Like hanno un unico albero per tutti i device i quali vengono in caso montati in un determinato punto dell'albero a seconda del volere di chi gestisce il sistema.

Per vedere in che posizione dell'labero ci troviamo (ovvero in che cartella ci troviamo) si ha:
```shell
[me@linuxbox ~]$ pwd
/home/me
```
Inoltre all'avvio ci troveremo nella cartella */home*, diversa per ciascun account. Infatti dentro */home* si hanno le varie sottocartelle degli utenti (una per ogni utente).
Per vedere cosa contiene una cartella, elencando files e directories uso:
```shell
[me@linuxbox ~]$ ls
Desktop Documents file.txt Music Pictures Public Templates Videos
```
Specificando il percorso possiamo vedere il contenuto di una determinata cartella
```shell
[me@linuxbox ~]$ ls Documents
document.txt other
```
per elencare anche file e directory nascoste uso:
```shell
[me@linuxbox ~]$ ls -a Documents
document.txt other .test .test.d
```
possiamo specificare più directories:
```shell
[me@linuxbox ~]$ ls ~ Documents
/home/me:
Desktop Documents file.txt Music Pictures Public Templates Videos
/home/me/Documents:
document.txt other
```
possiamo anche richiedere informazioni sui file:
```shell
[me@linuxbox ~]$ ls -l Documents
drwxr-xr-x 2 me me 4096 30 apr 11.37 other
-rw-r--r-- 2 me me 4096 30 apr 11.37 document.txt
```

Il percorso di file o di una directory può essere specificato in due modi:
1) **con il PERCORSO ASSOLUTO**, ovvero indicando ogni cartella e sotto cartella 
   fino a quella desiderata, per esempio */home/Documents/*. Si ricorda che il primo "/" indica la cartella di root
2) **con il PERCORSO RELATIVO**, che invece da partiren dalla cartella di root parte dalla cartella corrente. Si hanno anche "." per indicare la cartella corrente e 
  ".." per indicare quella precedente, quindi:
  ```shell
  [me@linuxbox ~]$ pwd
  /home/me
  [me@linuxbox ~]$ ls
  Documents
  [me@linuxbox ~]$ cd ./Documents
  [me@linuxbox Documents]$ pwd
  /home/me/Documents
  [me@linuxbox Documents]$ cd ..
  [me@linuxbox ~]$ pwd
  /home/me
  [me@linuxbox ~]$ cd Documents
  [me@linuxbox Documents]$ pwd
  /home/me/Documents
  ```
<p align="center">
  <img width = "600" height="200" src="img/cd.png"/>
</p>

In UNIX i nomi di file e directory sono *case sensitive* e non si usano le estensioni per determinare il contenuto di un file. Per comodità **NON SI USANO SPAZI NEI NOMI DI FILES E DIRECTORIES**

