**La terminazione di un programma**

128 viene usato per indicare l'incapacità di eseguire un altro programma in un sottoprocesso

##### **Esecuzione di funzioni preliminari all'uscita**

#include<stdlib.h>

int atexit( void( *function )(void) )

LA funzione richiede come argomento function l'indirizzo di una opportuna funzione di pulizia da chiamare all'uscita del programma

( La funzione di pulizia dovrà essere eseguita così void function(void) )

DIventa tuttavia possibile passare dati alla f di chiusura, vedi on_exit( void(*function)(int, void *), void *arg )

**Memoria virtuale**

consiste nell'assegnare ad ogni processo uno spazio virtuale di indirizzamento lineare in cui gli indirizzi vanno da zero ad un qualunque valore massimo.

Questo spazio non corrisponde all'effettiva posizione dei dati nella RAM e non è nenache continuo.

LA memoria viene suddivisa in pagine di dimensione fissa 4kb su 32 bit, huge page 4Mb

ciascuna pagina di memoria è associata ad un supporto. per ciascun processo il kernelsi cura di mantenere una mappa di queste corrispondenze nella cosidetta page table

In genere la memoria fisica è solo una piccola frazione della memoria virtuale, è necessario un meccanismo che permetta di trasferire le pagine che servono dal supporto su cui si trovano in memoria, questo è detto paginazione paging



se tento di accedere a una pagina che non è nella memoria reale, page fault, ma gestione trasparente da parte del kernel il processo non si accorge (solo aumneto dei tempi per swap)

Un programma C è diviso in segmenti

* text segment, contiene il codice del programma e le costanti, rimane invariato per tutto il tempo di esecuzione
* data segment, contiene i dati del programma le variabili globali e la memoria allocata dinamicamente ed è composta da
  * segmento da dati iniziatizzati del tipo int i =3
  * dati non inizializzati del tipo int vect[100]
  * heap, detto anche free store usato per allocazione dinamica di memoria, il suo limite inferiore ha una posizione fissa
* stack, viene salvato l'indice di ritorno per le chaiame a funzioni, la funzione chiamata alloca qui lo spazio per le sue variabil ilocali, questi dai vengono impilati, stacked, e le funzioni possono essere chiamate ricorsivamente
* (esiste anche environment, contiene i dati relativi alle variabili di ambiente passate al programma al suo avvio)

Le variabili il cui contenuto è allocato dinamicamente in questo modo non potranno essre usate direttamente come le altre ( cioè quelle in stack) ma l'accesso sarà possibile solo in maniera indiretta con puntatori che si sono ottenuti dalle funzioni di allocazione.

* malloc
* calloc
* realloc
* free

void *malloc(size_t size )  Alloca un'area di memoria non inizializzata

entrambe le funzioni restituiscono il puntatore alla zona di memoria allocata in caso di successo

le funzioni garantiscono che i puntatori sono allineati correttamente per tutti i tipi di dati, per esempio sono allineati a multipli di 4 byte per 32 bit e 8 byte per 64 bit

void free( void *ptr )

non chiamare due volte free!! double free bug, si suggerisce di assegnale sempre a NULL ogni puntatore su cui sia stata eseguita free immediatamente dopo l'esecuzione della funzione

memory leak : dimenticarsi di free memory: una possibile soluzione allocare invece che nello heap  è allocare nello stack usano la funzione _alloca_: non esiste free in quanto viene rilasciata automaticamente  al ritorno della funzione

Per ottener informazioni sulle modalità con cui un programma sta usando la memoria virtuale è disponibile **mincore** 

il meccanismo che previene la paginazione di parte della memoria virtuale di un processo è chiamato memory locking

la richiesta di un memory lock riduce la memoria fisica disponibile nel sistema per gli altri processi

le funzioni di sistema per bloccare e sbloccare la paginazione di singole sezioni di memoria sono mlock e munlock

int mlock( const void *addr, size_t len )

blocca l'intervallo di memoria niziante all'indirizzo addr e lungo len byte

Al ritorno di mlock tutte le pagine che contengono una parte dell'intervallo bloccato sono garantite essere in RAM e vi verranno mantenute per tutta la durata del blocco.

Funzioni che richiedono di allocare un blocco di memoria allineato ad un multiplo di una certa dimensione. questa esigenza emerge per esempio nei buffer... malloc non è più sufficiente! 
queste funzioni sono pvalloc, memalign, pvalloc

posix_memalign( void **memptr, size_t alignment, size_t t )
che alloca un buffer di memoria allineato ad un multiplo di alignment

So the result of e.g. `posix_memalign(&p, 32, 128)` will be a 128-byte chunk of memory whose start address is guaranteed to be a multiple of 32.

**La gestione delle opzioni**

touch -r riferimento.txt -m questofle.txt

#include <unistd.h>
int getopt( int argc, char * const argv[], const char *optsring)

la stringa optstring indica tutte quali sono le opzioni riconosciute separate da ":"

**VARIABILI DI AMBIENTE **

#include<stdlib.h>
#include<stdio.h>
int main(){
	char home[] = "HOME";
	char *ptr = home;
	char* aieie = getenv(ptr);
	printf(aieie);
	printf("\n");
	return 0;
}

output is /home/f126ck

char *getenv( const char *name )
	#ritorna il puntatore alla stringa contenetnte il valore della variabile di ambiente

setenv, unsetenv, putenv, clearenv

**Il passaggio di variabili e valori di ritorno nelle funzioni**

Le variabili vengono passate per valore!! si può passare puntatori, ok, ma è una _copia_ del puntatore che non viene "modificata" globalmente.

talvolta è necessario che la funzione possa restituire indietro alla funzione chiamante un valore relativo ad uno dei suoi argomenti usato anche in ingresso. per fare questo si usa **value result argument** SI PASSA INVECE DI UNA NORMALE VARIABILE UN PUNTATORE ALLA STESSA. (usata nei socket)
se  non ben gestito, causa dangling pointer. soluzione usare extern o variabile globale.

**Il passaggio di un numero variabile di argomenti**
variadic function

si usa ... cioè ellipsis, ma deve sempre avere almeno un argomento fisso!

esempio

​	int execl( const char *path, const char *arg, ... );

**Controllo di flusso non locale**

usaere goto è utile nel caso di : uscita in caso di errore

big endian : il byte che contiene i bit più significativi e poi i byte meno significativi all'indirizzo successivo

il network order è big endian, intel è little endian

Processi

qualunque processo può generarne altri

ogni processo è identificato con il suo PID.

la sequenza è generare un nuovo processo, il quale eseguirà in un passo successivo il programma desideraro.
è quello che fa la shell!

il processo iniziale è /sbin/init che viene lanciato dal kernel alla conclusione della fase di avvio, ha sempre PID = 1 e non è figlio di nessun altro processo e non può essere mai terminato
dato che c'è un "padre" si può pensare a una struttura ad albero, infatti comando pstree da bash (systemd non più init ora)

il kernel ha una tabella coi processi attivi, chiamata process table.
per ciascun processo si ha una struttura task_struct con le info relative.
a vedere, contiene anche una info sui file descriptor aperti.

La struttura del sistema permette di lanciare al posto di init qualunque altro programma, anche una shell.
si fa passando init=/bin/sh come parametro di avvio del kernel.

Scehduler viene ad una frequenza specificata da HZ del kernel, ma è possibile anche tickless, ad ogni chiamata del timerviene programmata l'esecuzione successiva sulla base di una stima.
In questo modo si può avere un risparmio energetico senza fare migliaia di context switch al secondo su macchine che non stanno facendo un cazzo

il pid è in un intero a 16 bit, massimo 32768 processi? NO, cioè si, ma se buchi( l'assegnazione è progressiva ) l'assegnazione overflow e riparte da tipo 300, coprendo i buchi di quelli conclusi, così da garantire posto per priorità maggiore.

Tutti i processi figli dello stesso processo sono detti sibling, per creare un nuovo processo uso fork.

#include<unistd.h>

pid_t fork( void )

​    // crea un nuovo processo

la memoria tra padre e figlio è **copiata ma non condivisa**

due motivi

* cient/server
* un programma vuole eseguire un altro programma, esempio tipico la shell. in questo caso il processo crea un figlio la cui unica operazione è quella di fare un exec. questa cosa si chiama spawn

export LD_LIBRARY_PATH=./  #per permettere l'uso delle librerie dinamiche

bufferizzazione, nel caso di file i dati vengoo scritti solo quando necessario, mentre se terminale ad ogni riga di a cap \n

La funzione fork ha la caratteristica di duplicare nei processi figli tutti i file descriptor dei file aperti nel processo padre, tra le quali c'è anche la posizione corrente del file (fseek??)

DNOTIFY 
Its function is essentially an extension to [filesystems](https://en.wikipedia.org/wiki/Filesystem) to notice changes to the filesystem, and report those changes to  applications. Instead of application checking for changes to filesystem, application can register to be notified by kernel when changes to  filesystem occur.
One major use is in [desktop search](https://en.wikipedia.org/wiki/Desktop_search) utilities like [Beagle](https://en.wikipedia.org/wiki/Beagle_(software)), where its functionality permits [reindexing](https://en.wikipedia.org/wiki/Index_(search_engine)) of changed files without scanning the filesystem for changes every few minutes

COnclusione di un processo
ad ogni processo figlio viene asssegnato un nuovo padre in genere init

Se il processo viene concluso in maniera anomala il programma non può specificare nessun exit status ed è il kernel che lo deve fare

processo orfano, se il figlio termina dopo il padre e a chi riporta il valore?
soluzione: il processo orfano viene adottato da init
quando un processo termina, il kernel controlla se è il padre di altri processi in esecuzione, il PPID parent PID dei child verrà impostato a 1.

init non può mai terminare, se lo fa kernel panic!!

I processi che sono terminati ma il cui stato di terminazione non è ancora stato ricevuto dal padre si chiamano processi zombie.

Legger gli stati di uscita di zombie e quindo terminarli è necessario perchè sebbene non consumino cpu o ram occupano una voce nella tabella dei processi e se ne sono tanti c'è starvation nel senso che nuovi processi non riescono ad andare in esecuzione, ricorda 32768 in precedenza.

è necessario gestire esplicitamente la gestione dei figli per evitare di riempire di zombie la tabella, una funzione usata è wait

pid_t wait( int *status )

è usata per attendere la terminazione dei figli, ma non riesco a capire se un **preciso** child è finito, allora si aggiunge 

pid_t waitpid( pid_t pid, int *status, int options )

con questa si può specificare quale pid attendere.

una delle modalità principali con ci si utilizzano i processi è quella di usarli per lanciare nuovi programmi, questo viene fatto con una delle funzioni della famiglia exec.
Quando un processo chiama exec, il PID del processo non cambia, mala funzione rimpiazza lo stack i dati e il testocon un nuovo programma letto da disco.
ci sono 6 diverse funzioni exec, 

* execl
* execv
* execle
* execlp
* execvp
* execve

ma la piuù generale è execve

#include<unistd.h>

int execve( const char *filename, char *const argv[], char *const envp[]  )

essa esegue il programma indicato da filename, passandogli argv.

in caso di successo la funzione **NON** ritorna!! ritornano solo in caso di errore

v e l stanno per vector e list.

con fork si crea un nuovo processo, con exec si lancia un nuovo programma, con exit e wait si effettua e si verifica la conclusione dei processi.

**Controllo di accesso**

il sistema associa UserID UID e GroupID GID ad ogni utente.
Tutte le operazioni del sistema vengono compiute dai processi, quindi è necessario che ad ogni processo sia identificato chi lo ha lanciato per fini di controllo permessi.

Se ho bisogno di poter impersonare un altro utente per un limitato insieme di operazioni, allora si introducono due gruppi di identificatori real ed effective.

UID effettivo e GID effettivo usati nelle verifiche dei permessi del processo e per il controllo di accesso ai file

`screen` crea terminali multipli su una console

**Priorità processi**

carateristica di sistema multitasking è il preemptive scheduling, cioè non sono i singoli processi, ma il kernel stesso a decidere quando la CPU deve essere passata ad un altro processo.
Non è affatto detto che dare ad un programma la massima priorità di esecuzione abbia risultati significativi in termini di prestazioni.
Gli stati sono

* runnable, R
* sleep S
* uninterruptible sleep D
* stopped T
* zombie Z
* killable D

Il meccanismo tradizionale UNIX è sempre stato scheduling con priorità dinamiche, in modo da assicurare che tutti i processi vadano in esecuzione, ma per real time introdotta priorità assoluta o statica, la priorità è visibile in `PR` nel comando `top`

La dimensione del time-slice ( in cui il processo esegue ) è determinato dal niceness di un processo, un valore positivo indica una time-slice più breve ma una priorità più alta.

Nice value is a user-space and priority PR is  the process's actual priority that use by Linux kernel. In linux system  priorities are 0 to 139 in which 0 to 99 for real time and 100 to 139  for users.  nice value range is -20 to +19 where -20 is highest, 0 default and +19  is lowest. relation between nice value and priority is : 

```
PR = 20 + NI
```

è stato creato un sistema modulare che permette di cambiare lo scheduler a sistema attivo!!

**Tecniche di scheduling Real Time**

* FIFO
* RR Round Robin identico al FIFO ma ogni processo esegue al massimo per una time-slice

Ogni processo può rilasciare volontariamente la CPU in modo da consentire agli altri processi di essere eseguiti, la funzione è `sched_yield`

Uno dei problemi che si pongono nei sistemi multiprocessore è infatti quello del cosiddetto ping pong. cioè se viene stoppato su un processore e rimandaro in esecuzione su uno diverso, la cache è diversa!! soluzione, CPU affiinity

architetture NUMA - Non Uniform Memory Access

LA CPU NON E' L'UNICA RISORSA, per questo esiste anche l'I/O scheduler per l'accesso al disco. LA sceltadi uno scheduler I/O si può fare in maniera generica all'avvio del kernelcon il parametro di avvio elevator, cui assegnare il nome dello scheduler, ma se ne può anche indicare uno specifico per l'accesso al singolo disco scrivendo nel file `/sys/block/<dev>/queue/scheduler` 

Casi tipici di race condition si hanno quando diversi processi accedono allo stesso file, o nell'accesso a meccanismi di intercomunicazione come la memoria condivisa.

**LA GESTIONE DI FILE E DIRECTORY**

il concetto di everything is a file è stato implementato attraverso il Virtual File System.

si hanno  4 oggetti correlati:

* filesystem
* dentry  directory entry (oggetti che il kernl usa per fare il pathname resolution)
* inode oggetto usato dal kernel per identificare un file
* file

Ogni volta che una system call specifica un pathname viene effettuata una ricerca nella dcache per ottenere immediatamante la dentry corrispondente che a sua volta ci darà tramite l'inode il riferimento al file.

