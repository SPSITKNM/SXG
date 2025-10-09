Procesy a Pipes v UNIX systémoch
Obsah

Čo je proces
Štruktúra procesu v pamäti
Stavy procesu
Process Control Block (PCB)
fork() - Vytváranie procesov
wait() a waitpid()
Pipes - Medziprocesová komunikácia
Praktické príklady


Čo je proces
Proces = bežiaca inštancia programu v operačnom systéme
Rozdiel medzi programom a procesom:

Program = súbor na disku (exekuovateľný kód)
Proces = program načítaný v pamäti a vykonávaný CPU


Štruktúra procesu v pamäti
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
Vysvetlenie segmentov:

TEXT - Strojový kód programu (nemenný, read-only)
DATA - Globálne a statické premenné
HEAP - Dynamicky alokovaná pamäť (rastie smerom nadol)
STACK - Lokálne premenné, parametre funkcií, návratové adresy (rastie nahor)
KERNEL SPACE - Systémové dáta, nedostupné pre užívateľský program


Stavy procesu
Proces môže byť v niekoľkých stavoch počas svojho životného cyklu:
    ┌─────────────┐
    │    NEW      │  ← Proces sa práve vytvára
    └──────┬──────┘
           ↓
    ┌─────────────┐
    │   READY     │  ← Čaká na pridelenie CPU
    └──────┬──────┘
           ↓
    ┌─────────────┐
    │  RUNNING    │  ← Vykonáva sa na CPU
    └──┬───┬───┬──┘
       │   │   │
       ↓   ↓   ↓
   WAITING READY TERMINATED
Popis stavov:

NEW - Proces sa vytvára, OS prideľuje zdroje
READY - Proces je pripravený na vykonanie, čaká na CPU
RUNNING - Proces sa práve vykonáva na CPU
WAITING - Proces čaká na udalosť (I/O operácia, signál, atď.)
TERMINATED - Proces skončil vykonávanie


Process Control Block (PCB)
PCB = dátová štruktúra, kde OS udržuje informácie o každom procese
Štruktúra PCB
cppstruct PCB {
    pid_t pid;                   // Process ID
    pid_t ppid;                  // Parent PID
    int state;                   // READY, RUNNING, WAITING...
    int priority;                // Priorita plánovania
    void* program_counter;       // Kde sa vykonáva kód
    void* stack_pointer;         // Vrchol zásobníka
    void* registers[N];          // Hodnoty registrov CPU
    FileDescriptor* fd_table;    // Otvorené súbory
    MemoryMap* memory;           // Mapa pamäte
    // ... ďalšie údaje
};
Príklad PCB hierarchie
┌─────────────────┐
│   RODIČ PCB     │
│   PID:  1000    │ ← Jeho jedinečné ID
│   PPID: 999     │ ← ID jeho rodiča
└─────────────────┘
Po fork() vznikne kópia s vlastným PCB:
┌─────────────────┐
│  POTOMOK PCB    │
│   PID:  1001    │ ← NOVÉ jedinečné ID
│   PPID: 1000    │ ← ID rodiča (z ktorého vznikol)
└─────────────────┘
Detailný PCB potomka
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

fork() - Vytváranie procesov
Kde je uložená hodnota "0" z fork()?
POZOR: V PCB NIE JE žiadna "0"!
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
Čo sa stane pri fork()?
Kernel vykonáva tieto kroky:
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
Vizualizácia fork()
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
Kompletná vizualizácia procesu
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
Kde má značku 0?
Premenná pid v user space pamäti (nie v PCB!)
PCB obsahuje PID 1001 - skutočné ID procesu
Premenná pid v kóde obsahuje 0 - návratová hodnota fork()
getpid() číta z PCB a vracia 1001
Prečo rodič dostáva PID potomka?
cpp// Rodič dostáva PID potomka (napr. 1001) kvôli:
// 1. wait() - rodič musí vedieť na koho čaká
// 2. kill() - posielanie signálov
// 3. Sledovanie potomkov
Pseudokód kernel fork()
cppvoid kernel_fork() {
    // 1. Vytvor nový PCB
    PCB* child = create_new_process();
    child->pid = 1001;
    child->ppid = current_process->pid;
    
    // 2. Skopíruj pamäť
    copy_memory(current_process, child);
    
    // 3. KRITICKÉ: Nastav návratové hodnoty
    current_process->return_value = child->pid;   // Rodič dostane 1001
    child->return_value = 0;                      // Potomok dostane 0
    
    // 4. Obaja procesy sa teraz prebudia z fork()
}
Poznámka: 0 hovorí "nemám žiadnych vlastných potomkov"

wait() a waitpid()
Funkcia wait()

Rodič musí počkať na ukončenie potomka
wait() čaká na akéhokoľvek potomka
Zabraňuje vzniku zombie procesov

Funkcia waitpid()

Počká len na konkrétneho potomka (podľa PID)

Ako kernel nachádza potomkov?
Kernel si pamätá vzťahy rodič-potomok pomocou PPID:
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
Praktický príklad - jeden potomok
cppint main() {
    fork();   // Nemusíš ukladať PID!
    
    if (fork() == 0) {
        // POTOMOK
        printf("Potomok\n");
        exit(0);
    } else {
        // RODIČ
        wait(NULL);   // Kernel sám nájde potomka
        printf("Hotovo\n");
    }
}
Poznámka:

Kernel vie, že tento proces má potomka (podľa PPID v tabuľke)
wait() počká na toho potomka
Nemusíš špecifikovať PID

Tabuľka procesov v kernel space
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
Pseudokód vyhľadávania potomkov
cppProcess* find_children(pid_t parent_pid) {
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

Pipes - Medziprocesová komunikácia
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
cpp#include <unistd.h>

int pipe(int pipefd[2]);
Parametre:

pipefd - pole 2 integerov (file descriptors)

Po úspešnom volaní:

pipefd[0] = READ end (čítací koniec)
pipefd[1] = WRITE end (zapisovací koniec)

Návratové hodnoty:

0 pre úspech
-1 pre zlyhanie/chybu

File Descriptor Table
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
│ 4  │ PIPE WRITE end ────┼──> [PIPE BUFFER]   │
└────┴────────────────────┴──────────────────── ┘
Kernel:

Vytvoril buffer v kernel space
Pridelil 2 file descriptors (napr. 3 a 4)
Jeden smeruje na čítací koniec, druhý na zapisovací

Analógia pre lepšie pochopenie
┌──────────┐         ┌─────────────────┐         ┌──────────┐
│ KOHÚTIK  │────────>│   RÚRA/POTRUBIE │────────>│  VEDRO   │
└──────────┘         └─────────────────┘         └──────────┘
  Nalievaš               Voda tečie              Vyteká
   vodu                  jedným smerom            voda
Vlastnosti:

Voda tečie len jedným smerom (jednosmerná komunikácia)
Voda vychádza v rovnakom poradí - FIFO (First In, First Out)

Čo je File Descriptor (FD)?
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
Pri volaní read(3, buffer, 100):
Kernel:

Pozrie sa: "FD 3? Aha, to je pipe read end"
Načíta dáta z pipe bufferu
Vráti ich do tvojho buffer

Štandardné file descriptors
Každý proces má automaticky otvorené:

0 = stdin (štandardný vstup, klávesnica)
1 = stdout (štandardný výstup, obrazovka)
2 = stderr (chybový výstup, obrazovka)

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


Praktické príklady
Príklad 1: Základná komunikácia cez pipe
cpp#include <stdio.h>
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
Čo sa stalo? Krok po kroku
Krok 1: pipe() v rodičovi
RODIČ:
┌───────────────────────────┐
│ pipefd[0] = 3 (READ)      │──┐
│ pipefd[1] = 4 (WRITE)     │──┼──> [PIPE BUFFER: prázdny]
└───────────────────────────┘  │
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
Dôležité: Obaja zdieľajú rovnaké file descriptory smerujúce na rovnaký pipe buffer!
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

Domáca úloha: Sieťový simulátor s 5 smerovačmi
Celková štruktúra
[Router 1 - Generuje data]
          ↓
[Router 2 - Čísluje pakety]
          ↓
[Router 3 - Meria dĺžku]
          ↓
[Router 4 - Prevod na malé písmená]  ← NOVÝ
          ↓
[Router 5 - Prevod na veľké písmená] ← NOVÝ
          ↓
[Parent]
Zadanie
Rozšírte pôvodný 3-smerovačový systém o 2 nové smerovače:
Router 4:

Prijíma očíslované a označené pakety (napr. 3. Student cte knihu. (18))
Konvertuje text na malé písmená
Preposiela do Router 5

Router 5:

Prijíma pakety s malými písmenami
Konvertuje text na VEĽKÉ PÍSMENÁ
Posiela finálny výstup parentovi

Požiadavky:

Použite fork() na vytvorenie 5 potomkov (každý = 1 router)
Použite 4 pipes na prepojenie (Router1→R2, R2→R3, R3→R4, R4→R5, R5→Parent)
Každý router vypíše svoj PID a činnosť
Všetky procesy musia byť korektne ukončené pomocou wait()


Užitočné príkazy
Kompilácia
bash# C++
g++ program.cpp -o program

# Spustenie
./program
Debugging
bash# Zobrazenie procesov
ps aux | grep program

# Stromová štruktúra
pstree -p

# Otvorené file descriptory
lsof -p <PID>

# Sledovanie system calls
strace -f ./program

