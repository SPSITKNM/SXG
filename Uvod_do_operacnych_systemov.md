# Procesy a Pipe v OS

Proces : bežiaca inštanca programu v operačnom systéme 

štruktúra procesu 

┌─────────────────────────────────┐
│      PROCES V PAMÄTI            │
├─────────────────────────────────┤
│  TEXT SEGMENT                   │  ← Strojový kód programu (read-only)
├─────────────────────────────────┤
│  DATA SEGMENT                   │  ← Globálne a statické premenné
├─────────────────────────────────┤
│  HEAP                           │  ← Dynamická pamäť (malloc, new)
│         ↓ rastie nadol          │
├─────────────────────────────────┤
│         ↑ rastie nahor          │
│  STACK                          │  ← Lokálne premenné, návratové adresy
├─────────────────────────────────┤
│  KERNEL SPACE                   │  ← Dáta jadra OS (nedostupné užívateľovi)
└─────────────────────────────────┘

proces môže moze byt v niekolkych stavoch 


    ┌─────────────┐
    │    NEW      │  ← Proces sa práve vytvára
    └──────┬──────┘
           ↓
    ┌─────────────┐
    │   READY     │  ← Čaká na CPU
    └──────┬──────┘
           ↓
    ┌─────────────┐
    │  RUNNING    │  ← Vykonáva sa na CPU
    └──┬───┬───┬──┘
       │   │   │
       ↓   ↓   ↓
   WAITING READY TERMINATED


   Proces control block == PCb

   OS udržuje pre kazdy proces pcb štrukturu ktorý vyzerá takto : struct PCB {

    pid_t pid;                   Process ID
    pid_t ppid;                  Parent PID
    int state;                   READY, RUNNING, WAITING...
    int priority;                Priorita plánovania
    void* program_counter;       Kde sa vykonáva kód
    void* stack_pointer;         Vrchol zásobníka
    void* registers[N];          Hodnoty registrov CPU
    FileDescriptor* fd_table;    Otvorené súbory
    MemoryMap* memory;           Mapa pamäte
    // ... ďalšie údaje
};


Každý proces má svoj PCB

┌─────────────────┐
│   RODIČ PCB     │
│   PID:  1000    │ ← Jeho jedinečné ID
│   PPID: 999     │ ← ID jeho rodiča
└─────────────────┘

2. Po fork() vznikne kópia s vlastným PCB

┌─────────────────┐
│  POTOMOK PCB    │
│   PID:  1001    │ ← NOVÉ jedinečné ID
│   PPID: 1000    │ ← ID rodiča (z ktorého vznikol)
└─────────────────┘

┌─────────────────────────────────────────┐
│            POTOMOK PCB                  │
├─────────────────────────────────────────┤
│ PID:  1001      ← Jeho ID v systéme     │
│ PPID: 1000      ← ID rodiča             │
│ state: RUNNING                          │
│ registers: [...]                        │
│ memory_map: 0x3000-0x4000               │
│ file_descriptors: [0, 1, 2, 3, ...]     │
│ ... atď ...                             │
└─────────────────────────────────────────┘
         ↑
   POZOR V PCB NIE JE žiadna "0"!

Kde teda je tá "0"?

Tá "0" je v pamäti procesu, nie v PCB!

┌──────────────────────────────────────────────────┐
│         USER SPACE PAMÄŤ POTOMKA                 │
├──────────────────────────────────────────────────┤
│  int main() {                                    │
│      pid_t pid;        ← Premenná v pamäti       │
│      pid = fork();     ← Tu sa uloží 0           │
│                   ↓                              │
│      Premenná pid obsahuje: 0                    │
│  }                                               │
└──────────────────────────────────────────────────┘

Rozdiel medzi PCB a pamäťou procesu

KERNEL SPACE (spravuje OS):
┌─────────────────────┐
│   PCB POTOMKA       │
│   PID:  1001        │ ← Toto je ID procesu
│   PPID: 1000        │
└─────────────────────┘
        ↑
    Nedostupné z kódu


USER SPACE (tvoj program):
┌─────────────────────┐
│   Pamäť programu    │
│                     │
│   pid_t pid = 0;    │ ← Premenná, ktorú vytvoril fork()
│                     │
└─────────────────────┘
        ↑
    Môžeš pristupovať z kódu

Čo sa stane pri for () ? 

Čo sa stane pri fork()

Kernel vytvorí nový PCB s PID 1001
Kernel nakopíruje pamäť rodiča do potomka
Kernel nastaví návratovú hodnotu funkcie fork():

V rodičovi: do registra CPU uloží 1001
V potomkovi: do registra CPU uloží 0


Keď sa proces vráti z fork(), uloží hodnotu z registra do premennej pid

Analogia 

RODIČ:
┌─────────────────┐
│ Môj dom: 1000   │ ← Môj PID
│ Syn býva: 1001  │ ← Premenná pid (adresa syna)
└─────────────────┘

POTOMOK:
┌─────────────────┐
│ Môj dom: 1001   │ ← Môj PID
│ Som syn: 0      │ ← Premenná pid (signál "som syn")
└─────────────────┘





KERNEL robí toto:

┌──────────────────────┐
│   fork() vnútorne:   │
├──────────────────────┤
│ 1. Vytvor nový PCB   │
│ 2. Priraď nový PID   │  → Potomok dostane napr. PID 1001
│ 3. Skopíruj pamäť    │
│ 4. V RODIČOVI:       │
│    return 1001;      │  ← Vráti PID potomka
│ 5. V POTOMKOVI:      │
│    return 0;         │  ← Vráti 0
└──────────────────────┘

┌─────────────────────────────┐     ┌─────────────────────────────┐
│     RODIČ                   │     │     POTOMOK                 │
│     PID = 1000              │     │     PID = 1001              │
│                             │     │                             │
│  int main() {               │     │  int main() {               │
│      pid_t pid;             │     │      pid_t pid;             │
│      pid = fork();          │     │      pid = fork();          │
│         ↓                   │     │         ↓                   │
│      pid = 1001  ← !!!!     │     │      pid = 0     ← !!!!     │
│                             │     │                             │
│      if (pid == 0) {        │     │      if (pid == 0) {        │
│         // FALSE            │     │         // TRUE ← ide sem   │
│      } else {               │     │      }                      │
│         // TRUE ← ide sem   │     │                             │
│      }                      │     │                             │
│  }                          │     │  }                          │
└─────────────────────────────┘     └─────────────────────────────┘


Vizualizácia celého procesu


PRED fork():
═══════════════════════════════════════
RODIČ (PID 1000):
  PCB: { PID: 1000, PPID: 999, ... }
  Pamäť: { int main() { pid_t pid; ... } }


PO fork():
═══════════════════════════════════════
RODIČ (PID 1000):
  PCB: { PID: 1000, PPID: 999, ... }
  Pamäť: { pid = 1001; }  ← návratová hodnota fork()
           ↑
       Uložená v USER SPACE!

POTOMOK (PID 1001):
  PCB: { PID: 1001, PPID: 1000, ... }
  Pamäť: { pid = 0; }  ← návratová hodnota fork()
           ↑
       Uložená v USER SPACE!


Kde ma značku 0 ? 

v premennej pid v user space pamati nie v pcb 

pcb obsahuje pid 1001 - skutočné id procesu

premenná pid v kode obsahuje 0 navratova hodnota fork 

getpid() --> číta z PCB ---> vracia 1001

Aký je call pri volaní fork ? 

1. Zavoláš fork()
   ↓
2. Kernel vytvorí kópiu procesu (nový PCB s novým PID)
   ↓
3. Kernel VEDOME nastaví návratovú hodnotu fork():
   - V rodičovi: vráti PID potomka (napr. 1001) == prečo ? bez pid by rodič nevedel na koho čaká u wait totožne tak pre signaly napr kill , pre sledovanie potomkov 
   - V potomkovi: vráti 0
   ↓
4. Táto návratová hodnota sa uloží do premennej:
   pid_t pid = fork();

   Ako to robí kernel ? 

Pseudokód toho, čo kernel robí vnútorne:

void kernel_fork() {

    // 1. Vytvor nový PCB
    PCB* child = create_new_process();
    child->pid = 1001;
    child->ppid = current_process->pid;
    
    // 2. Skopíruj pamäť
    copy_memory(current_process, child);
    
    // 3. KRITICKÉ: Nastav návratové hodnoty
    current_process->return_value = child->pid;   Rodič dostane 1001
    child->return_value = 0;                      Potomok dostane 0
    
    // 4. Obaja procesy sa teraz prebudia z fork()


    0 teda hovorí nemam ziadnych vlastnych potomkov 
}

Funkcia wait 


= rodič musi počkať na ukončenie potomka 

= wait čaká na akéhokoľvek potomka 



Funkcia waitpid 

počká len na toho jedného 


Kernel si pamätá vzťahy rodič poto

čo sa deje ? 

RODIČ volá wait():
│
├─> Kernel: "Hľadám potomkov s PPID = 1000"
│
├─> Našiel som: PID 1001 (PPID = 1000)
│   Je zombie? NIE, ešte beží
│
├─> Našiel som: PID 1002 (PPID = 1000)  
│   Je zombie? NIE, ešte beží
│
├─> Žiadny zombie → Uspím rodiča
│
│   [čas plynie...]
│
├─> PID 1001 volá exit(0)
│   Stáva sa ZOMBIE
│
├─> Kernel: "Zobúdzam rodiča s PID 1000"
│   Vyčistím zombie 1001
│
└─> wait() vráti: 1001

Praktický príklad : 

Jeden potomok ( nepotrebuješ PID )

int main() {
    fork();   Nemusíš ukladať PID!
    
    if (fork() == 0) {
        // POTOMOK
        printf("Potomok\n");
        exit(0);
    } else {
        // RODIČ
        wait(NULL);   Kernel sám nájde potomka
        printf("Hotovo\n");
    }
}

Kernel vie, že tento proces má potomka (podľa PPID v tabuľke)
wait() počká na toho potomka
Nemusíš špecifikovať PID

TABUĽKA PROCESOV (v kernel space):

┌──────────────────────────────────────┐
│ PID  │ PPID │ STATE    │ ...         │
├──────────────────────────────────────┤
│ 1    │ 0    │ RUNNING  │ (init)      │
│ 999  │ 1    │ RUNNING  │             │
│ 1000 │ 999  │ WAITING  │ ← RODIČ     │
│ 1001 │ 1000 │ ZOMBIE   │ ← POTOMOK   │
│ 1002 │ 1000 │ RUNNING  │ ← POTOMOK   │
│ 1003 │ 500  │ RUNNING  │             │
└──────────────────────────────────────┘
         ↑
       PPID umožňuje kernelu nájsť
       všetkých potomkov rodiča

Ako to funguje v pozadí vyhľadávania 

Process* find_children(pid_t parent_pid) {
    // ↑ Vráti ukazovateľ na Process (alebo pole procesov)
    //                    ↑ Parameter: PID rodiča
    
    Process* children = [];
    // ↑ Pole ukazovateľov na procesy (potomkov)
    
    for (each process in process_table) {
        // ↑ Iteruj cez VŠETKY procesy v systéme
        //   process_table = globálna tabuľka všetkých PCB
        
        if (process.ppid == parent_pid) {
            // ↑ Ak PPID tohto procesu == parent_pid
            //   → Tento proces JE potomok!
            
            children.append(process);
            // ↑ Pridaj ho do zoznamu potomkov
        }
    }
    
    return children;
    // ↑ Vráť pole všetkých nájdených potomkov
}

Pipe roura pre jednosmernú komunikácou medzi procesmi 

Prečo pipe existuje?

Problém: Procesy sú izolované

┌─────────────────┐         ┌─────────────────┐
│   PROCES A      │         │   PROCES B      │
│   PID: 1000     │    ?    │   PID: 1001     │
│   Pamäť: 0x1000 │────X────│   Pamäť: 0x3000 │
└─────────────────┘         └─────────────────┘

Každý proces má vlastnú pamäť. Proces A nemôže priamo čítať pamäť procesu B.

Riešenie: Pipe cez kernel

┌─────────────────┐                     ┌─────────────────┐
│   PROCES A      │                     │   PROCES B      │
│   write(fd, ..) │────┐         ┌──────│   read(fd, ..)  │
└─────────────────┘    │         │      └─────────────────┘
                       ↓         ↓
                  ┌──────────────────┐
                  │  KERNEL SPACE    │
                  │                  │
                  │  PIPE BUFFER     │
                  │  [DÁTA DÁTA]     │
                  └──────────────────┘


Kernel poskytuje zdieľaný buffer, kde:

Proces A zapisuje dáta
Proces B číta dáta
Kernel sa stará o synchronizáciu                  


Vytvorenie pipe 

#include <unistd.h>

int pipe(int pipefd[2]);

Parametre:

pipefd - pole 2 integerov (file descriptors)

Po úspešnom volaní:

pipefd[0] = READ end (čítací koniec)
pipefd[1] = WRITE end (zapisovací koniec)

nacratova hodnota je 0 and -1 


PRED pipe():
┌──────────────────────────────────┐
│  FILE DESCRIPTOR TABLE           │
├────┬─────────────────────────────┤
│ 0  │ stdin                       │
│ 1  │ stdout                      │
│ 2  │ stderr                      │
└────┴─────────────────────────────┘

PO pipe(pipefd):
┌──────────────────────────────────────────────┐
│  FILE DESCRIPTOR TABLE                       │
├────┬─────────────────────────────────────────┤
│ 0  │ stdin                                   │
│ 1  │ stdout                                  │
│ 2  │ stderr                                  │
│ 3  │ PIPE READ end  ────┐                    │
│ 4  │ PIPE WRITE end ────┼──> [PIPE BUFFER]    │
└────┴────────────────────┴─────────────────────┘


Kernel:

Vytvoril buffer v kernel space
Pridelil 2 file descriptors (napr. 3 a 4)
Jeden smeruje na čítací koniec, druhý na zapisovací

PIPE 

Čo je pipe a prečo existuje?

Problém: Procesy sú izolované

┌─────────────────┐            ┌─────────────────┐
│   PROCES A      │    ?       │   PROCES B      │
│   PID: 1000     │    X       │   PID: 1001     │
│   int x = 42;   │ nemôže     │   int y;        │
│   Pamäť: 0x1000 │ pristúpiť  │  Pamäť: 0x3000  │
└─────────────────┘            └─────────────────┘

Každý proces má vlastnú pamäť. Proces A nemôže čítať x z procesu B priamo.


┌─────────────────┐                     ┌─────────────────┐
│   PROCES A      │                     │   PROCES B      │
│                 │                     │                 │
│  write(fd, ...) │────┐         ┐──────│  read(fd, ...)  │
└─────────────────┘    │         │      └─────────────────┘
                       ↓         ↓
                  ┌──────────────────┐
                  │  KERNEL SPACE    │
                  │                  │
                  │  ┌────────────┐  │
                  │  │PIPE BUFFER │  │
                  │  │[DÁTA DÁTA] │  │
                  │  └────────────┘  │
                  └──────────────────┘

Proces A zapisuje dáta 
Proces B číta dáta 
Kernel synchronizuje                   


Analógia pre lepšie pochopenie : 

┌──────────┐         ┌─────────────────┐         ┌──────────┐
│ KOHÚTIK  │────────>│   RÚRA/POTRUBIE │────────>│  VEDRO   │
└──────────┘         └─────────────────┘         └──────────┘
  Nalievaš               Voda tečie              Vyteká
   vodu                  jedným smerom            voda

Voda tečie len jedným smerom ( jednosmerná komunikácia )

Voda vychádza v rovnakom poradi FIFO - first in first out 

Syntax - vytvorenie pipe

#include <unistd.h>

int pipe(int pipefd[2]);

Parametre : 

    pipefd - pole 2 integerov ( file descriptors )

Po úspešnom volaní:

pipefd[0] = READ end (tzv čítací koniec)
pipefd[1] = WRITE end (zapisovací koniec)

Návratové hodnoty sú teda 0 pre úspech, - 1 pre zlyhanie / chybu 

príklad vytvorenia : 

int pipefd[2];

if (pipe(pipefd) == -1) {
    perror("pipe");
    exit(1);
}

printf("Pipe vytvorená!\n");
printf("pipefd[0] = %d (READ koniec)\n", pipefd[0]);    napr. 3
printf("pipefd[1] = %d (WRITE koniec)\n", pipefd[1]);   napr. 4

Čo je to teda File Descriptor (FD) 

File descriptor je malé celé číslo (integer), ktoré reprezentuje otvorený "súbor":

┌────────────────────────────────────────┐
│  Tvoj program (USER SPACE)             │
│                                        │
│  int fd = 3;                           │
│      ↓                                 │
│  "Mám lístok číslo 3"                  │
└────────────────────────────────────────┘
         │
         ↓
┌────────────────────────────────────────┐
│  KERNEL                                │
│                                        │
│  FD 3 → ukazuje na pipe read end      │
└────────────────────────────────────────┘

read(3, buffer, 100);

Kernel:

Pozrie sa: "FD 3? Aha, to je pipe read end"
Načíta dáta z pipe bufferu
Vráti ich do tvojho buffer

Štandardné file descriptors
Každý proces má automaticky otvorené:

0 = stdin   (štandardný vstup, klávesnica)
1 = stdout  (štandardný výstup, obrazovka)
2 = stderr  (chybový výstup, obrazovka)

Čo sa stane pri pipe() vnútorne?

PRED pipe():

PROCES:
┌──────────────────────────────────┐
│  FILE DESCRIPTOR TABLE           │
├────┬─────────────────────────────┤
│ 0  │ stdin                       │
│ 1  │ stdout                      │
│ 2  │ stderr                      │
└────┴─────────────────────────────┘

PROCES:
┌──────────────────────────────────────────────┐
│  FILE DESCRIPTOR TABLE                       │
├────┬─────────────────────────────────────────┤
│ 0  │ stdin                                   │
│ 1  │ stdout                                  │
│ 2  │ stderr                                  │
│ 3  │ PIPE READ end  ────┐                    │
│ 4  │ PIPE WRITE end ────┤                    │
└────┴────────────────────┴─────────────────── ┘
                           │
                           ↓
                    ┌──────────────┐
                    │ KERNEL SPACE │
                    │              │
                    │ PIPE BUFFER  │
                    │  [........]  │
                    │   64 KB max  │
                    └──────────────┘


Kernel:

Vytvoril buffer v kernel space (~64 KB)
Pridelil 2 file descriptors (3 a 4)
FD 3 → čítací koniec bufferu
FD 4 → zapisovací koniec bufferu

Vnútorná štruktúra pipe

USER SPACE:                  KERNEL SPACE:
┌─────────────┐             ┌──────────────────────────┐
│ pipefd[0]=3 │────────────>│  READ ENDPOINT           │
│             │             │         ↓                │
│             │             │    ┌──────────┐          │
│             │             │    │  BUFFER  │          │
│             │             │    │ (FIFO)   │          │
│             │             │    │ ~64 KB   │          │
│             │             │    └──────────┘          │
│             │             │         ↑                │
│ pipefd[1]=4 │────────────>│  WRITE ENDPOINT          │
└─────────────┘             └──────────────────────────┘


Buffer vlastnosti:

Veľkosť: ~64 KB (65536 bytov)
Typ: FIFO (First In, First Out) - fronta
Umiestnenie: Kernel space (nedostupný priamo z programu)
Atomické operácie (kernel zabezpečuje synchronizáciu)

Základná komunikácia - Jednoduchý príklad

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

int main() {
    int pipefd[2];
    
    // 1. Vytvor pipe
    if (pipe(pipefd) == -1) {
        perror("pipe");
        exit(1);
    }
    
    printf("Pipe vytvorená: READ=%d, WRITE=%d\n", 
           pipefd[0], pipefd[1]);
    
    // 2. Fork - vytvor potomka
    pid_t pid = fork();
    
    if (pid == 0) {
        // ===== POTOMOK - ČITATEĽ =====
        printf("[POTOMOK] Zatvárám write end\n");
        close(pipefd[1]);  // Nepotrebujem písanie
        
        char buffer[100];
        printf("[POTOMOK] Čakám na dáta...\n");
        ssize_t n = read(pipefd[0], buffer, 100);
        
        printf("[POTOMOK] Prijal som %zd bytov: %s\n", n, buffer);
        
        close(pipefd[0]);
        exit(0);
        
    } else {
        // ===== RODIČ - ZAPISOVATEĽ =====
        printf("[RODIČ] Zatvárám read end\n");
        close(pipefd[0]);  // Nepotrebujem čítanie
        
        char msg[] = "Ahoj potomok!";
        printf("[RODIČ] Posielam správu...\n");
        write(pipefd[1], msg, strlen(msg) + 1);
        
        printf("[RODIČ] Správa odoslaná\n");
        close(pipefd[1]);
        
        wait(NULL);  // Počkaj na potomka
        printf("[RODIČ] Hotovo\n");
    }
    
    return 0;
}


Čo sa stalo ? Krok po kroku 

Krok 1: pipe() v rodičovi

RODIČ:
┌───────────────────────────┐
│ pipefd[0] = 3 (READ)      │──┐
│ pipefd[1] = 4 (WRITE)     │──┼──> [PIPE BUFFER: prázdny]
└───────────────────────────┘  ---> 

Krok 2: fork()

RODIČ:                          POTOMOK:
┌───────────────────────────┐  ┌───────────────────────────┐
│ pipefd[0] = 3 (READ)      │  │ pipefd[0] = 3 (READ)      │
│ pipefd[1] = 4 (WRITE)     │  │ pipefd[1] = 4 (WRITE)     │
└───────────────────────────┘  └───────────────────────────┘
        │                              │
        └──────────────┬───────────────┘
                       ↓
                [PIPE BUFFER: prázdny]

Dôležité: Obaja zdieľajú rovnaký file descritptor smerujúce na rovnaký pipe buffer!


Krok 3: Zatváranie nepoužívaných koncov


RODIČ:                          POTOMOK:
┌───────────────────────────┐  ┌───────────────────────────┐
│ close(pipefd[0]) ✗        │  │ close(pipefd[1]) ✗        │
│                           │  │                           │
│ pipefd[1] = 4 (WRITE) ────┼─>│                           │
└───────────────────────────┘  │ pipefd[0] = 3 (READ) ─────┤
                                └───────────────────────────┘
                                        ↓
                                 [PIPE BUFFER]

Krok 4: Rodič píše


RODIČ:
write(4, "Ahoj potomok!", 14);

[PIPE BUFFER]:
┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
│A│h│o│j│ │p│o│t│o│m│o│k│!│\0│
└─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴──┘
 ↑                            ↑
 write                      14 bytov

Krok 5: Potomok číta

 POTOMOK:
read(3, buffer, 100);

Buffer potomka:
"Ahoj potomok!\0"

[PIPE BUFFER]:
[prázdny - dáta vyčítané]


--------- Celková štruktúra domácej prípravy ----------

[Router 1 - Generuje data]
          ↓
[Router 2 - Čísluje pakety]
          ↓
[Router 3 - Měří délku]
          ↓
[Router 4 - převod na malá písmena]  ← NOVÝ
          ↓
[Router 5 - převod na velká písmena] ← NOVÝ
          ↓
[Parent]

Kontrola funkcionality zadania   

./main data.txt 8

Overenie paralelneho behu 

Meranie času čas je < 15 s keby bezali seriovo bolo by to >30 kvoli usleep 100000

time ./main data.txt 100 > /dev/null

Malička poznámka : 

výstup tvojho programu sa „prelieva“ cez pipe do grepu, ktorý prečíta každý riadok zo štandardného vstupu a vypíše iba tie obsahujúce reťazec „zpracoval“

./main data.txt 12 | grep -E "(SUDÁ|LICHÁ|R4|R5)"

Manualne overenie spravnosti routingu 

./main data.txt 12 | grep -E "(SUDÁ|LICHÁ|R4|R5)"

Parelélne spracovanie pre demonštráciu súbežnosti 

./main data.txt 25

Štatistika a bilancia


./main data.txt 20  

podla dlzky parne neparne 

./main data.txt 15 | grep -E "(SUDÁ|LICHÁ)"

rozvetvit do dvoch paralelnych vetvi 

./main data.txt 20 | grep -E "(R4|R5)" | head -15

spracovanie textu odlišnym sposobom 

./main data.txt 10 | grep -E "(R4 \(VELKÁ\)|R5 \(malá\))"

opatovne spojenie v rod procese 

./main data.txt 15 | grep "Přijat paket"

logovanie kolko paketov spracovala ktora vetva 

./main data.txt 50 | grep -E "(zpracoval|statistika)" -A2
