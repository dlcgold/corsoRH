# INTRODUZIONE

L'architettura del calcolatore si basa sull'archietttura di Von Neumann,
dotato di un'unità di memoria, un'alu etc...
Le periferiche sono attaccate mediante un bus, nel dettaglio il bus PCI ora,
prima era il bus ISA. 

### la virtualizzazione

Un computer diventa un simulatore di altre macchine mediante la virtualizzazione.
La macchina viene divisa in varie macchine logiche, virtuali. Si usa un hypervisor o
un software per la virtualizzazione. Si sfrutta l'accelerazione hardware. KVM e Qemu 
sono totalmente open. Si usa la virtualizzazione per questioni soprattutto di costo.

### stack di rete

Lo stack (pila) di rete è diviso in livelli, con uno strato fisico, collegamento, rete, trasporto, sessione, presentazione e applicazione, questo nel livello ISO/OSI.

*(prendere immagine da slide)*

Dal livello di trasporto in giù siamo nel "livello mezzi" in su siamo nel "livello degli host". Un messaggio scende lo stack fino al livello fisico, viene trasferito altrove e
riscala lo stack. Ora si usa il modello TCP/IP, con livello fisico, collegamento, rete, trasporto e applicazione e il modello UDP (*vedere appunti reti per differenze*).

### sistema operativo

È ciò che ci permette di usare il software su una macchina. Si hanno diversi livelli di software.
Il primo è il BIOS, che inizializza l'hardware etc... Si ha poi un bootloader che effettivamente carica l'os. Ora si ha uefi che permette un controllo maggiore del primo
strato software, dando la possibilità di dare comandi mediante la shell uefi. 
Noi approfondiremo GNU/Linux, GNU è la parte di userland (shell, software etc...)
mentre Linux è il kernel (dove c'è lo stack di rete, il controllo driver, il multitasking etc...).
Il kernel sta le applicazioni e l'hardware del pc, astraendo la comunicazione.
Si ha uno scheduler che passa tra le varie app permettendo il multitasking

### shell

È un'interfaccia utente di tipo testuale. Vedremo bash. 

*slide coi comandi base*

*slide coi comando pro*

