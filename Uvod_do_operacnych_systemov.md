# Úvod do operačných systémov

Princípy **procesov, PCB, `fork()`, `wait()`** a **pipe** v operačnom systéme.
Diagramy sú vysádzané ako bloky kódu, aby si zachovali štruktúru.

---

## Proces — bežiaca inštancia programu

Štruktúra procesu v pamäti:

```text
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
```

### Stavový diagram procesu

```text
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
```

### Process Control Block (PCB)

OS udržuje pre každý proces PCB štruktúru, ktorá vyzerá takto:

```c
struct PCB {
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
```

Každý proces má svoj PCB:

```text
┌─────────────────┐
│   RODIČ PCB     │
│   PID:  1000    │ ← Jeho jedinečné ID
│   PPID: 999     │ ← ID jeho rodiča
└─────────────────┘
```

Po `fork()` vznikne kópia s vlastným PCB:

```text
┌─────────────────┐
│  POTOMOK PCB    │
│   PID:  1001    │ ← NOVÉ jedinečné ID
│   PPID: 1000    │ ← ID rodiča (z ktorého vznikol)
└─────────────────┘
```

```text
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
   POZOR: V PCB NIE JE žiadna "0"!
```

**Kde teda je tá "0"?** Tá "0" je v pamäti procesu, nie v PCB:

```text
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
```

Rozdiel medzi PCB a pamäťou procesu:

```text
KERNEL SPACE (spravuje OS):        USER SPACE (tvoj program):
┌─────────────────────┐            ┌─────────────────────┐
│   PCB POTOMKA       │            │   Pamäť programu    │
│   PID:  1001        │            │                     │
│   PPID: 1000        │            │   pid_t pid = 0;    │ ← vytvoril fork()
└─────────────────────┘            └─────────────────────┘
        ↑                                  ↑
    Nedostupné z kódu                 Prístupné z kódu
```

---

## Čo sa stane pri `fork()`

1. Kernel vytvorí nový PCB s PID 1001.
2. Kernel nakopíruje pamäť rodiča do potomka.
3. Kernel nastaví návratovú hodnotu funkcie `fork()`:
   - v **rodičovi**: do registra CPU uloží 1001 (PID potomka)
   - v **potomkovi**: do registra CPU uloží 0

Keď sa proces vráti z `fork()`, uloží hodnotu z registra do premennej `pid`.

### Analógia

```text
RODIČ:                             POTOMOK:
┌─────────────────┐                ┌─────────────────┐
│ Môj dom: 1000   │ ← Môj PID      │ Môj dom: 1001   │ ← Môj PID
│ Syn býva: 1001  │ ← pid          │ Som syn: 0      │ ← pid ("som syn")
└─────────────────┘                └─────────────────┘
```

Čo robí kernel vnútorne:

```text
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
```

```text
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
```

### Vizualizácia celého procesu

```text
PRED fork():
═══════════════════════════════════════
RODIČ (PID 1000):
  PCB:   { PID: 1000, PPID: 999, ... }
  Pamäť: { int main() { pid_t pid; ... } }

PO fork():
═══════════════════════════════════════
RODIČ (PID 1000):
  PCB:   { PID: 1000, PPID: 999, ... }
  Pamäť: { pid = 1001; }   ← návratová hodnota fork() (v USER SPACE)

POTOMOK (PID 1001):
  PCB:   { PID: 1001, PPID: 1000, ... }
  Pamäť: { pid = 0; }      ← návratová hodnota fork() (v USER SPACE)
```

### Kde sa nachádza hodnota 0?

- v premennej `pid` v user-space pamäti, **nie** v PCB
- PCB obsahuje PID 1001 — skutočné ID procesu
- premenná `pid` v kóde obsahuje 0 — návratovú hodnotu `fork()`
- `getpid()` číta z PCB → vracia 1001

### Postup pri volaní `fork()`

1. Zavoláš `fork()`.
2. Kernel vytvorí kópiu procesu (nový PCB s novým PID).
3. Kernel vedome nastaví návratovú hodnotu `fork()`:
   - v rodičovi: PID potomka (napr. 1001) — bez neho by rodič nevedel, na koho
     čaká pri `wait()`, ani komu poslať signál (`kill`)
   - v potomkovi: 0
4. Táto návratová hodnota sa uloží do premennej: `pid_t pid = fork();`

Pseudokód toho, čo kernel robí vnútorne:

```c
void kernel_fork() {
    // 1. Vytvor nový PCB
    PCB* child = create_new_process();
    child->pid  = 1001;
    child->ppid = current_process->pid;

    // 2. Skopíruj pamäť
    copy_memory(current_process, child);

    // 3. KRITICKÉ: nastav návratové hodnoty
    current_process->return_value = child->pid;  // Rodič dostane 1001
    child->return_value = 0;                     // Potomok dostane 0

    // 4. Obaja procesy sa teraz prebudia z fork()
    //    0 v potomkovi hovorí: "nemám žiadnych vlastných potomkov"
}
```

---

## Funkcia `wait()`

Rodič musí počkať na ukončenie potomka. `wait()` čaká na **akéhokoľvek**
potomka; `waitpid()` počká len na jedného konkrétneho.

Kernel si pamätá vzťahy rodič–potomok. Čo sa deje, keď rodič volá `wait()`:

```text
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
├─> Žiadny zombie → uspím rodiča
│
│   [čas plynie...]
│
├─> PID 1001 volá exit(0) → stáva sa ZOMBIE
│
├─> Kernel: "Zobúdzam rodiča s PID 1000", vyčistím zombie 1001
│
└─> wait() vráti: 1001
```

### Praktický príklad — jeden potomok (nepotrebuješ PID)

```c
int main() {
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
```

Kernel vie, že tento proces má potomka (podľa PPID v tabuľke), `wait()` počká
na toho potomka — nemusíš špecifikovať PID.

### Tabuľka procesov (v kernel space)

```text
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
       PPID umožňuje kernelu nájsť všetkých potomkov rodiča
```

Vyhľadávanie potomkov v pozadí:

```c
Process* find_children(pid_t parent_pid) {
    Process* children = [];               // pole ukazovateľov na potomkov

    for (each process in process_table) { // iteruj cez VŠETKY procesy
        if (process.ppid == parent_pid) { // ak PPID == parent_pid → potomok
            children.append(process);
        }
    }

    return children;
}
```

---

## Pipe (rúra) — jednosmerná komunikácia medzi procesmi

### Prečo pipe existuje

**Problém: procesy sú izolované.**

```text
┌─────────────────┐            ┌─────────────────┐
│   PROCES A      │    ?       │   PROCES B      │
│   PID: 1000     │    X       │   PID: 1001     │
│   int x = 42;   │ nemôže     │   int y;        │
│   Pamäť: 0x1000 │ pristúpiť  │   Pamäť: 0x3000 │
└─────────────────┘            └─────────────────┘
```

Každý proces má vlastnú pamäť. Proces A nemôže priamo čítať pamäť procesu B.

**Riešenie: pipe cez kernel.**

```text
┌─────────────────┐                     ┌─────────────────┐
│   PROCES A      │                     │   PROCES B      │
│   write(fd, ..) │────┐         ┌──────│   read(fd, ..)  │
└─────────────────┘    │         │      └─────────────────┘
                       ↓         ↓
                  ┌──────────────────┐
                  │  KERNEL SPACE    │
                  │  PIPE BUFFER     │
                  │  [DÁTA DÁTA]     │
                  └──────────────────┘
```

Kernel poskytuje zdieľaný buffer: proces A zapisuje, proces B číta, kernel sa
stará o synchronizáciu.

### Analógia

```text
┌──────────┐         ┌─────────────────┐         ┌──────────┐
│ KOHÚTIK  │────────>│   RÚRA/POTRUBIE │────────>│  VEDRO   │
└──────────┘         └─────────────────┘         └──────────┘
  nalievaš vodu       voda tečie jedným smerom    vyteká voda
```

Voda tečie len jedným smerom (jednosmerná komunikácia) a vychádza v rovnakom
poradí — **FIFO** (first in, first out).

### Vytvorenie pipe

```c
#include <unistd.h>

int pipe(int pipefd[2]);
```

Parameter `pipefd` je pole 2 integerov (file descriptorov). Po úspešnom volaní:

- `pipefd[0]` = READ end (čítací koniec)
- `pipefd[1]` = WRITE end (zapisovací koniec)

Návratová hodnota: 0 pri úspechu, -1 pri chybe.

```text
PRED pipe():                          PO pipe(pipefd):
┌──────────────────────┐              ┌──────────────────────────────────┐
│  FILE DESCRIPTOR TAB │              │  FILE DESCRIPTOR TABLE           │
├────┬─────────────────┤              ├────┬─────────────────────────────┤
│ 0  │ stdin           │              │ 0  │ stdin                       │
│ 1  │ stdout          │              │ 1  │ stdout                      │
│ 2  │ stderr          │              │ 2  │ stderr                      │
└────┴─────────────────┘              │ 3  │ PIPE READ end  ────┐        │
                                      │ 4  │ PIPE WRITE end ────┼─> BUFF │
                                      └────┴────────────────────┴────────┘
```

Kernel vytvoril buffer v kernel space (~64 KB) a pridelil 2 file descriptory
(napr. 3 a 4) — jeden na čítací koniec, druhý na zapisovací.

### File descriptor (FD)

File descriptor je malé celé číslo, ktoré reprezentuje otvorený „súbor":

```text
┌────────────────────────────────────────┐
│  Tvoj program (USER SPACE)             │
│  int fd = 3;   → "Mám lístok číslo 3"  │
└────────────────────────────────────────┘
         ↓
┌────────────────────────────────────────┐
│  KERNEL                                │
│  FD 3 → ukazuje na pipe read end       │
└────────────────────────────────────────┘
```

Pri `read(3, buffer, 100)` kernel pozrie: „FD 3? To je pipe read end",
načíta dáta z pipe bufferu a vráti ich do tvojho bufferu.

**Štandardné file descriptory** — každý proces ich má automaticky otvorené:

- `0` = stdin (štandardný vstup, klávesnica)
- `1` = stdout (štandardný výstup, obrazovka)
- `2` = stderr (chybový výstup, obrazovka)

### Vnútorná štruktúra pipe

```text
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
```

Vlastnosti bufferu: veľkosť ~64 KB (65 536 bajtov), typ FIFO, umiestnenie v
kernel space, atomické operácie (kernel zabezpečuje synchronizáciu).

---

## Základná komunikácia — jednoduchý príklad

```c
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

    printf("Pipe vytvorená: READ=%d, WRITE=%d\n", pipefd[0], pipefd[1]);

    // 2. Fork — vytvor potomka
    pid_t pid = fork();

    if (pid == 0) {
        // ===== POTOMOK — ČITATEĽ =====
        printf("[POTOMOK] Zatváram write end\n");
        close(pipefd[1]);   // nepotrebujem písanie

        char buffer[100];
        printf("[POTOMOK] Čakám na dáta...\n");
        ssize_t n = read(pipefd[0], buffer, 100);
        printf("[POTOMOK] Prijal som %zd bajtov: %s\n", n, buffer);

        close(pipefd[0]);
        exit(0);

    } else {
        // ===== RODIČ — ZAPISOVATEĽ =====
        printf("[RODIČ] Zatváram read end\n");
        close(pipefd[0]);   // nepotrebujem čítanie

        char msg[] = "Ahoj potomok!";
        printf("[RODIČ] Posielam správu...\n");
        write(pipefd[1], msg, strlen(msg) + 1);
        printf("[RODIČ] Správa odoslaná\n");
        close(pipefd[1]);

        wait(NULL);   // počkaj na potomka
        printf("[RODIČ] Hotovo\n");
    }

    return 0;
}
```

### Kroky vykonania — krok po kroku

**Krok 1: `pipe()` v rodičovi**

```text
RODIČ:
┌───────────────────────────┐
│ pipefd[0] = 3 (READ)      │──┐
│ pipefd[1] = 4 (WRITE)     │──┼──> [PIPE BUFFER: prázdny]
└───────────────────────────┘
```

**Krok 2: `fork()`** — obaja zdieľajú rovnaké file descriptory smerujúce na
rovnaký pipe buffer.

```text
RODIČ:                          POTOMOK:
┌───────────────────────────┐  ┌───────────────────────────┐
│ pipefd[0] = 3 (READ)      │  │ pipefd[0] = 3 (READ)      │
│ pipefd[1] = 4 (WRITE)     │  │ pipefd[1] = 4 (WRITE)     │
└───────────────────────────┘  └───────────────────────────┘
        │                              │
        └──────────────┬───────────────┘
                       ↓
                [PIPE BUFFER: prázdny]
```

**Krok 3: zatváranie nepoužívaných koncov**

```text
RODIČ:                          POTOMOK:
┌───────────────────────────┐  ┌───────────────────────────┐
│ close(pipefd[0]) ✗        │  │ close(pipefd[1]) ✗        │
│ pipefd[1] = 4 (WRITE) ────┼─>│ pipefd[0] = 3 (READ) ─────┤
└───────────────────────────┘  └───────────────────────────┘
                                        ↓
                                 [PIPE BUFFER]
```

**Krok 4: rodič píše** — `write(4, "Ahoj potomok!", 14);`

```text
[PIPE BUFFER]:
┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬──┐
│A│h│o│j│ │p│o│t│o│m│o│k│!│\0│
└─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴──┘
```

**Krok 5: potomok číta** — `read(3, buffer, 100);` → buffer potomka obsahuje
`"Ahoj potomok!\0"`, pipe buffer je prázdny (dáta vyčítané).

---

## Celková štruktúra projektu (routing)

```text
[Router 1 — generuje dáta]
          ↓
[Router 2 — čísluje pakety]
          ↓
[Router 3 — meria dĺžku]
          ↓
[Router 4 — prevod na malé písmená]
          ↓
[Router 5 — prevod na veľké písmená]
          ↓
[Parent]
```

### Kontrola funkcionality zadania

```bash
./main data.txt 8
```

### Overenie paralelného behu

Čas by mal byť < 15 s; keby routre bežali sériovo, bolo by to > 30 s kvôli
`usleep(100000)`.

```bash
time ./main data.txt 100 > /dev/null
```

### Ďalšie kontroly

Výstup programu sa „prelieva" cez pipe do `grep`, ktorý prečíta každý riadok
zo štandardného vstupu a vypíše iba tie obsahujúce daný reťazec.

```bash
# manuálne overenie routingu
./main data.txt 12 | grep -E "(SUDÁ|LICHÁ|R4|R5)"

# paralelné spracovanie (demonštrácia súbežnosti)
./main data.txt 25

# štatistika a bilancia
./main data.txt 20

# podľa dĺžky: párne / nepárne
./main data.txt 15 | grep -E "(SUDÁ|LICHÁ)"

# rozvetvenie do dvoch paralelných vetiev
./main data.txt 20 | grep -E "(R4|R5)" | head -15

# spracovanie textu odlišným spôsobom
./main data.txt 10 | grep -E "(R4 \(VELKÁ\)|R5 \(malá\))"

# opätovné spojenie v rodičovskom procese
./main data.txt 15 | grep "Přijat paket"

# logovanie: koľko paketov spracovala ktorá vetva
./main data.txt 50 | grep -E "(zpracoval|statistika)" -A2
```
