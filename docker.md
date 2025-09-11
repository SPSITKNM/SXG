# Docker Základy a Architektúra

## Architektúra Dockeru

- **Klient – démon**: `docker` klient posiela príkazy démonovi `dockerd`, ktorý ich vykonáva.
- **Docker Daemon** (`dockerd`): backendová služba, ktorá spravuje objekty ako kontajnery, imagy, volume atď.
- **Docker CLI**: príkazový riadok na interakciu s Dockerom.
- **Docker Registry**: sklad pre Docker imagy – napr. Docker Hub.

## Objekty Dockeru

### Image

- Nemenná šablóna aplikácie (blueprint).
- Obsahuje OS vrstvu, aplikáciu a jej závislosti.
- Príklad vytvorenia: `docker build -t myapp .`

### Container

- Spustiteľná inštancia Docker imagu.
- Má vlastný súborový systém, procesy, siete.
- Je izolovaný a ľahký.
- Vzniká pomocou: `docker run <image>`

### Dockerfile

- Skript, ktorý automatizuje tvorbu Docker imagov.

### Volumes

- Persistentné úložisko dát pre kontajnery.
- Sleduj **lifecycle**, anonymné volume, blind mounts.
- Syntax: `--mount`, `-v` pre volume mountovanie.

## Práca s Dockerom

### Základné príkazy

```bash
docker run hello-world          # spustí testovací kontajner
docker images                   # vypíše dostupné imagy
docker ps -a                    # všetky kontajnery
docker logs <id>                # logy kontajnera
docker stop <id>                # zastavenie kontajnera
docker start <id>               # spustenie kontajnera
docker rmi <image_id>           # zmazanie imagu
```

### Detached mode

```bash
docker run -d redis             # spustenie v pozadí
```

### Port Binding

```bash
docker run -p <host_port>:<container_port> <image>
# napr: docker run -p 6000:6379 redis
```

- Bez bindovania nie je port kontajnera prístupný zvonka.
- Príklad výstupu: `0.0.0.0:6000->6379/tcp` znamená host port 6000 je spojený s container portom 6379.

## Porovnanie s Virtual Machines

| Feature             | Docker Container                   | Virtual Machine                    |
|---------------------|------------------------------------|------------------------------------|
| Kernel              | Zdieľaný s hostom                  | Vlastný kernel                     |
| Veľkosť             | Pár MB                             | GB                                 |
| Spustenie           | Sekundy                            | Minúty                             |
| Izolácia            | Procesová (namespace/cgroups)      | Plná VM úroveň                     |

## Technológie na pozadí

- **Namespaces** – izolácia procesov, sietí, filesystemu atď.
- **Control groups (cgroups)** – obmedzenie zdrojov pre kontajnery (CPU, RAM...).
- **Union File System (napr. OverlayFS)** – spája vrstvy do jedného pohľadu.
  - Výhoda: sťahujú sa len rozdielne vrstvy medzi verziami.

## Motivácia použiť Docker

- Štandardizovaný, ľahký, izolovaný balíček aplikácie.
- Prenositeľnosť medzi systémami (vývoj, test, produkcia).
- Jednoduchá správa závislostí.
- Rýchle buildovanie a nasadenie.

## Analógie

- **Docker Hub** = kuchárska kniha (recepty)
- **Docker Image** = konkrétny recept
- **Docker Container** = hotové jedlo

# Docker: Rozdiel medzi `docker run`, `docker images` a `docker start`

## 🔹 docker run

- **Vytvára a spúšťa nový kontajner** na základe daného imagu.
- Ak image nie je lokálne, automaticky sa stiahne z Docker Hubu.
- Pomocou parametrov vieš hneď definovať:
  - `-p <host>:<container>` – mapovanie portov
  - `-d` – spustenie v pozadí (detached mode)
  - `--name` – pomenovanie kontajnera
  - `-v` – mountovanie volume
  - `-e` – nastavenie environmentálnych premenných

### Príklad:
```bash
docker run -d -p 8080:80 --name webserver nginx
```
➡️ Vytvorí a spustí nový kontajner `webserver` z imagu `nginx`.

---

## 🔹 docker images

- **Zobrazí zoznam všetkých imagov**, ktoré sú uložené lokálne.
- Nepretraktuje kontajnery, len samotné imagy (blueprinty).
- Image môžeš zmazať pomocou `docker rmi <image_id>`.

### Príklad:
```bash
docker images
```

#### Výstup:
```
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
nginx        latest    abc123def456   2 weeks ago     133MB
```

---

## 🔹 docker start

- **Spúšťa už existujúci kontajner**, ktorý bol predtým vytvorený cez `docker run`.
- Zachová všetky pôvodné atribúty kontajnera (porty, mounty, názvy...).
- Vhodné na opätovné spustenie po vypnutí.

### Príklad:
```bash
docker start webserver
```

#### Interaktívne so shellom:
```bash
docker start -ai webserver
```

---

## 🧠 Zhrnutie:

| Príkaz         | Význam |
|----------------|--------|
| `docker run`   | Vytvorí a spustí **nový kontajner** s definovanými parametrami |
| `docker images`| Vypíše všetky **lokálne dostupné imagy** |
| `docker start` | Spustí **už existujúci** (zastavený) kontajner |
- **Docker Engine** = kuchár, ktorý varí podľa receptu

## 🧩 Príkazy

### Spustenie Shellu v kontajneri
```bash
docker exec -it <container_id_or_name> /bin/bash
```
- `-i` (interactive): udrží štandardný vstup otvorený – môžeme písať do terminálu
- `-t` (tty): priradí pseudo-terminál (simuluje terminálové rozhranie)

> Týmto sa dostávame do interaktívneho prostredia kontajnera. Každý kontajner má svoj izolovaný virtuálny súborový systém.

### Zobrazenie environmentálnych premenných
```bash
printenv
```

---

## 🚀 `docker run`

- Vytvorí a spustí nový kontajner zo zadaného imagu
- Príklad:
```bash
docker run -d -p 27017:27017 mongo
```
- `-d`: detached mód (beží na pozadí)
- `-p`: prepája porty `host:container`

---

## 🌐 Docker Networks

- Docker automaticky vytvára izolované siete
- Kontajnery v tej istej sieti spolu komunikujú cez **meno kontajnera**, nepotrebujú `localhost` ani port
- Príkazy:
```bash
docker network ls
docker network create <nazov_siete>
```

### Využitie v praxi:
- Kontajnery (napr. MongoDB a Mongo Express) komunikujú cez mená
- Aplikácie mimo tejto siete (napr. host Node.js) používajú `localhost:port`

---

## ⚙️ Docker Compose vs Docker Run

| Docker Compose                         | Docker Run                             |
|---------------------------------------|----------------------------------------|
| .yaml súbor pre definovanie služieb   | Príkazy v CLI                          |
| Jednoduchšia správa viacerých služieb | Vhodné pre jednoduché testovanie       |
| Verzia zistíš príkazom:               |                                        |
```bash
docker compose version
```

---

## 📝 Zhrnutie

- `docker exec -it`: interaktívny prístup ku kontajneru
- `docker run`: spustenie kontajnera s parametrami
- Docker networks: izolované prostredie pre komunikáciu kontajnerov
- Docker Compose: YAML-based orchestrácia kontajnerov


# 🐳 Docker + Flask + Compose – praktický prehľad a koncepty

## 🧠 Základná myšlienka
JavaScript aplikácia alebo Flask backend sa pripája na databázu, zobrazí údaje a následne sa všetko kontajnerizuje cez `Dockerfile` a spustí pomocou `docker compose`.

---

## 📦 Docker Compose – Prečo?

- **Lokálny vývoj**
- **Demo open-source projektov**
- **Nasadenie na jednom serveri**

---

## 🔐 Environmentálne premenne vs Secrets

- **Nevýhoda env premenných:** bezpečnostné riziko (napr. pri logovaní alebo výpisoch)
- **Docker secrets:** oddelený mechanizmus na bezpečné ukladanie citlivých údajov

---

## ⚙️ Override projektu v Compose

```bash
docker compose --project-name projects -f mongo-services.yaml up -d
```

➡️ Prepíše predvolený názov projektu

---

## 🧾 JSON vs YAML

- `.json` = **key-value** formát, kde kľúč MUSÍ byť v úvodzovkách
```json
{
  "name": "The ultimate Docker Course",
  "price": 149,
  "tags": ["docker", "flask"]
}
```

- YAML je čitateľnejší, ale jeho **parsing je pomalší** než JSON

---

## 🗂️ Flask aplikácia – základ

```bash
python -m venv .venv
source .venv/bin/activate
pip freeze > requirements.txt
```

- `.venv` = izolované prostredie
- `requirements.txt` = zoznam závislostí
- `>` = presmerovanie výstupu do súboru

---

## 🛰️ Testovanie lokálne

```bash
curl 127.0.0.1:5000/about
```

➡️ Dostaneme späť hardkodovanú verziu z `Flask` aplikácie

```bash
flask --app app run
```

- `--app app` → súbor `app.py`
- `run` → spustenie vývojového web servera

---

## 🐳 Dockerfile – Ako vyzerá?

```dockerfile
FROM python:3.12.4-alpine3.20
RUN apk --no-cache add curl

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY app.py .

CMD ["gunicorn", "--bind", "0.0.0.0:8080", "app:app"]
```

- `alpine` → malý image (optimalizácia veľkosti)
- `WORKDIR` → ako `cd /app`
- `COPY` a `RUN` sú optimalizované na využitie cache
- `CMD` → čo sa spustí pri štarte kontajnera

---

## 🧱 VM vs Container

| Vlastnosť             | VM                    | Container              |
|------------------------|-----------------------|------------------------|
| Kernel                | Vlastný               | Zdieľaný s hostom      |
| Veľkosť               | Väčšia                | Menšia (napr. Alpine)  |
| Izolácia              | Plná                  | Procesová              |

- **Containery sú izolované od hostiteľa**
- **Porty sú dostupné vo vnútri virtuálnej siete**

---

## 🧪 Debugging

```bash
docker exec -it <container_id> /bin/bash
```

---

## 📅 Fun facts

- `docker-compose`: vznikol v **2014**
- `docker-compose v2`: **2020**
- v2 **ignoruje top-level version** vo `docker-compose.yaml`

---

## 🔁 Vývojový cyklus v Dockeri

1. Vytvorenie aplikácie (napr. Flask, JS)
2. Príprava `Dockerfile`
3. Príprava `docker-compose.yaml`
4. Build a spustenie
5. Testovanie a ladenie
6. Deployment

---


# 🐳 Docker – Od Aplikácie po Kontajner

Tento materiál vysvetľuje rozdiely medzi Dockerfile a Compose súborom, krok po kroku od vytvorenia aplikácie až po jej spustenie ako kontajner.

---

## 🧱 1. Vytvorenie vlastnej aplikácie

Najprv si vytváraš vlastnú aplikáciu – napríklad Flask API, Node.js server, Ruby Sinatra app alebo Java Spring Boot.  
Toto je obyčajný kód, ktorý funguje aj mimo Dockera.

---

## 📦 2. Dockerfile – ako vytvoriť *image* z aplikácie

**Dockerfile** je inštrukčný súbor, ktorý hovorí Dockeru, **ako z tvojej aplikácie vytvoriť image**.  
Tento image obsahuje všetko potrebné na spustenie: kód, runtime, knižnice a konfigurácie.

### 🔧 Príklad `Dockerfile`:
```Dockerfile
FROM python:3.12-alpine
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

- `FROM` – základný image, napr. Python
- `COPY` – skopíruje tvoje súbory do image
- `RUN` – inštaluje závislosti
- `CMD` – spustí aplikáciu

👉 **Výsledok:** Image s tvojou appkou pripravený na spustenie.

---

## ⚙️ 3. docker-compose.yaml – ako spustiť *container* z image-u

`docker-compose.yaml` je súbor, ktorý definuje **ako spustiť tvoju aplikáciu ako kontajner** (alebo viacero kontajnerov – napr. appka + databáza).

### 🧩 Rozdiel oproti Dockerfile:
- **Dockerfile** = *ako postaviť image*
- **Compose file** = *ako spustiť kontajner(y)* z image + prepojiť ich

### 🔧 Príklad `docker-compose.yaml`:
```yaml
version: "3"
services:
  web:
    build: .
    ports:
      - "8080:8080"
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: example
```

- `build: .` – použije Dockerfile z aktuálneho priečinka
- `ports` – premapuje porty (host:container)
- `image:` – môže použiť hotový image z Docker Hubu

---

## 🧠 Zhrnutie

| Krok | Súbor | Úloha |
|------|-------|--------|
| 1. | Tvoja appka (napr. `app.py`) | Logika aplikácie |
| 2. | `Dockerfile` | Ako z appky spraviť image |
| 3. | `docker-compose.yaml` | Ako spustiť kontajner(y) z image |

---

💡 **Poznámka:** Docker Compose umožňuje ľahko definovať viacero služieb, ich sieťovanie, premenné prostredia a persistentné volume.

---

# 🔄 Bind Mount vs Docker Volume

## 🗂️ Bind Mount

```yaml
volumes:
  - ./mydata:/var/lib/postgresql/data
```

- `./mydata` je konkrétny priečinok **na hostiteľskom počítači**
- Čokoľvek zapíše PostgreSQL do `/var/lib/postgresql/data` sa **fyzicky objaví** v `./mydata` na hostovi

### ✅ Výhody:
- Máš **plnú kontrolu nad súbormi**
- Môžeš do priečinka **vstupovať priamo z host systému**

### ⚠️ Nevýhody:
- Závisí od **presnej štruktúry cesty**
- Môže nastať problém pri **migrácii na iný systém alebo tím**

---

## 📦 Docker Volume

```yaml
volumes:
  - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:
```

- `postgres-data` je **Docker volume**, ktorý Docker spravuje **automaticky**
- Fyzicky sa môže ukladať do:
  ```
  /var/lib/docker/volumes/postgres-data/_data
  ```
  ale ty túto cestu **nešpecifikuješ priamo**

### ✅ Výhody:
- **Bezpečne prenosné**
- Docker sa stará o celý **životný cyklus (lifecycle)**: zálohy, obnovy, mazanie
- Vhodné pre **produkciu**

### ⚠️ Nevýhody:
- **Nemáš jednoduchý priamy prístup** k dátam mimo kontajnera

---

## 📝 Zhrnutie:
| Typ             | Prístup k dátam | Prenositeľnosť | Vhodné pre      |
|------------------|------------------|------------------|------------------|
| Bind Mount       | Priamy           | Nižšia           | Vývoj, ladenie   |
| Docker Volume    | Nepriamy (cez Docker) | Vysoká     | Produkcia        |

# 🐳 Docker Compose Setup with Flask, PostgreSQL and NGINX

## 📦 Package Snapshot

To capture the current Python environment:
```bash
pip freeze > requirements.txt
```

---

## 🧱 Services Overview

### 🔁 Postgres
- Acts as the backend database.
- Secrets are used to inject the password securely.
- Attached to the **private** network.

### 🌐 NGINX
- Acts as a **reverse proxy** or **load balancer**.
- Exposed on port `8080`.
- Uses a custom config from the `nginx_config`.
- Attached to **public** network.
- Uses **health check dependency** for Flask container (`condition: service_healthy`).

### ⚙️ Flask
- Main web application.
- Built from `Dockerfile.dev` inside `./flask` directory.
- Uses env vars, bind mounts, secrets, configs.
- Connected to **private** and **public** networks.
- Includes a proper health check via `/about`.

---

## 🚫 Depends On – Static Dependency Warning

It’s not ideal to statically assume:
```yaml
depends_on:
  - postgres
```
Why?
- Flask may **crash** if DB isn't ready.
- Instead, a **healthcheck** mechanism is used:
```yaml
depends_on:
  flask:
    condition: service_healthy
    restart: true
```

Flask has a **30s timeout** while waiting for DB, so using `healthcheck` is more reliable.

---

## 🔐 Network Isolation Strategy

- NGINX is on **public** network → exposed to internet.
- Flask and PostgreSQL are on **private** network → hidden and secured.
- Ensures **PostgreSQL is not publicly accessible**.

---

## ✅ Healthcheck Configuration

Example for Flask:
```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:8000/about"]
  interval: 5s
  retries: 5
  start_period: 15s
  timeout: 5s
```

### Healthcheck Meaning:
- Any `curl` result in **200-399** is considered **healthy**.
- Codes **300-399** indicate redirects (client-side actions required).

---

## 🧪 Testing Internal Networking (SSH into container)

```bash
docker exec -it <container_name> sh
```

### Test Connectivity:
```bash
nc -vz flask 8000
```

- `nc` = netcat
- `-v` = verbose
- `-z` = zero-I/O mode (connection test only)
- `flask` = container name
- `8000` = target port

---

## 🔐 NGINX + Let’s Encrypt (TLS)

> 🔐 Secure NGINX setup using Let's Encrypt is recommended for HTTPS.

---

## 🧾 Docker Compose Snippet

```yaml
services:
  nginx:
    image: nginx:latest
    ports:
      - 8080:8080
    configs:
      - source: nginx_config
        target: /etc/nginx/nginx.conf
    networks:
      - public
    depends_on:
      flask:
        condition: service_healthy
        restart: true

  flask:
    image: flask:latest
    build:
      context: ./flask
      dockerfile: Dockerfile.dev
    env_file:
      - flask/dev.env
    secrets:
      - api_key
      - source: api_key
        target: /api_key.txt
    configs:
      - source: my_config
        target: /config-dev-v2.yaml
    environment:
      - APP_VERSION=0.1.0
      - APP_TOKEN=${APP_TOKEN}
      - FLASK_DEBUG=1
      - FLASK_APP=./app.py
      - DB_HOST=postgres
      - DB_DATABASE=mydb
      - DB_USER=myuser
    volumes:
      - ./flask/config-dev.yaml:/config-dev.yaml
      - ./flask/my-data:/data
      - flask-data:/data
      - ./flask:/app
    networks:
      - private
      - public
    depends_on:
      - postgres
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/about"]
      interval: 5s
      retries: 5
      start_period: 15s
      timeout: 5s

  postgres:
    image: postgres
    environment:
      POSTGRES_USER: myuser
      POSTGRES_DB: mydb
      POSTGRES_PASSWORD_FILE: /run/secrets/pg_password
    secrets:
      - pg_password
    volumes:
      - postgres-data:/var/lib/postgressql/data
    networks:
      - private

secrets:
  api_key:
    file: flask/api_key.txt
  pg_password:
    file: /home/tomasmucha/Plocha/pg_password.txt

configs:
  my_config:
    file: ./flask/config-dev.yaml
  nginx_config:
    file: /home/tomasmucha/Plocha/nginx.conf

volumes:
  flask-data:
  postgres-data:

networks:
  public:
  private:
    driver: bridge
    ipam:
      config:
        - subnet: "10.0.0.0/19"
          gateway: "10.0.0.1"
```


# Load Balancing Algorithms

## 🔍 Purpose
Load balancers help distribute incoming traffic across multiple servers to **prevent any single server from becoming overloaded**.

---

## ⚙️ Static Load Balancing Algorithms

These algorithms **ignore the state** of any particular server (e.g., CPU usage, load, or current connections).  
They follow **fixed rules** and are **independent of real-time server performance**.

### ✅ Advantages
- Simple and efficient to implement
- Require no monitoring of server state

### 📘 Common Static Algorithms

#### 1. Round Robin
Each incoming request is assigned to the next server in a cyclic order.

Example:
- Request 1 → Server A  
- Request 2 → Server B  
- Request 3 → Server C  
- Request 4 → Server A ...

Best used when all servers have **identical specs**.

#### 2. Weighted Round Robin
Similar to Round Robin, but each server is given a **weight**.

Example:
- If Server A has weight 2, it receives twice as many requests as Server B (with weight 1).

Useful when one server has **better performance specs** or is hosting **critical data**.

#### 3. Hashing Methods
Assigns requests to servers based on calculated hash values.

- **IP Hash**: Hash of client IP or combination of client & server IPs
- **URL Hash**: Hash of the request URL
- **Five Tuple Hash**: Combines
  - Source IP
  - Source Port
  - Destination IP
  - Destination Port
  - IP Protocol

#### 4. Random Algorithm
Requests are assigned randomly to any server.  
Does not follow any pattern.

---

## 🔄 Dynamic Load Balancing Algorithms

These algorithms consider the **current state of the servers** before making a routing decision.

### 📘 Common Dynamic Algorithms

#### 1. Least Connection
Directs the request to the server with the **fewest active connections**.

#### 2. Weighted Least Connection
Same as Least Connection, but considers **server specs** via manual **weights**.

#### 3. Resource-Based Algorithm
Uses real-time metrics like:
- CPU usage
- Memory load
- Network/disk I/O

🔧 Requires a **special monitoring agent** on each server.

Best used when requests **require varying amounts of resources**.

---

## 🩺 Health Checks
Most load balancers include **health checks**.  
If a server fails the health check, it is temporarily removed from rotation.

---

## 🧠 Summary

| Algorithm                | Type     | State-Aware | Notes                              |
|--------------------------|----------|-------------|------------------------------------|
| Round Robin              | Static   | ❌          | Cycles requests evenly             |
| Weighted Round Robin     | Static   | ❌          | Manual weights for stronger nodes |
| IP/URL/Five Tuple Hash   | Static   | ❌          | Consistent request routing         |
| Random                   | Static   | ❌          | Unpredictable but simple           |
| Least Connection         | Dynamic  | ✅          | Chooses least busy server          |
| Weighted Least Connection| Dynamic  | ✅          | Considers server power             |
| Resource-Based           | Dynamic  | ✅          | Monitors real-time load            |


# Docker Networking – Poznámky

## Bridge Network

- Je to **virtuálne zariadenie**, ktoré pripája viacero lokálnych sietí (LAN).
- **Bridge môže rozdeliť lokálnu sieť** na menšie segmenty (subnety).
- Docker používa `bridge` ako **predvolenú sieť** pre kontajnery, ktoré nemajú explicitne špecifikovanú sieť.
- Príkaz: `docker network inspect bridge` — zobrazí detaily o bridge sieti.
- Keď sa **pripojíš do kontajnera**, často musíš použiť `sh` alebo `bash`, aby si interagoval s jeho shellom.
- Niektoré nástroje ako `curl` môžu v kontajneroch zobraziť chybu **„dns is not supported“**, ak sieť nie je správne nastavená.

---

## Host Network

- Kontajner **zdieľa sieťový stack hosta** — beží ako keby bol priamo na host systéme.
- Výhoda: **vyšší výkon** (napr. žiadne NAT pre sieť).
- Použitie: `--network host`
- Nevýhoda: nie je izolovaný od hosta — zdieľajú IP adresu.

---

## None Network

- Kontajner **nie je pripojený k žiadnej sieti**.
- Slúži na úplnú izoláciu — žiadny prístup na internet alebo k iným kontajnerom.
- Použitie: `--network none`

---

## Zhrnutie

| Typ siete  | Prístup k hostovi | Izolácia | Výkon | Využitie                                  |
|------------|-------------------|----------|--------|-------------------------------------------|
| Bridge     | Nie                | Áno      | 🟡      | Predvolené, flexibilné pre väčšinu prípadov |
| Host       | Áno               | Nie       | 🟢      | Pri potrebe maximálneho výkonu             |
| None       | Nie                | Áno      | 🔴      | Izolované výpočty, bezpečnosť             |


--- 

# 🐳 Docker Logs & Debugging – Poznámky

## 📥 Prístup do kontajnera
```bash
docker exec -it <container_id> bash
```
- Interaktívne vojdeš do bežiaceho kontajnera

```bash
ls -la
```
- `-l` = long list (podrobnosti)
- `-a` = zobrazí aj skryté súbory

---

## 📋 Docker Logs – štandardný výstup

- Docker logy idú na `stdout` a `stderr`
- Môžeme ich čítať pomocou príkazu:

```bash
docker logs <container_id_or_name>
```

---

## 🕒 Logovanie podľa času

```bash
docker logs --tail 5 <id>
```
- Posledných 5 riadkov

```bash
docker logs --since 2m <id>
```
- Logy od času mínus 2 minúty (podobne `h`, `s`, `d`, ...)

```bash
docker logs --since 15m --until 5m <id>
```
- Logy **od 15 minút dozadu do 5 minút dozadu**

```bash
docker logs --since 2023-10-01T00:00:00 <id>
```
- Absolútny timestamp

---

## 🔍 Filtrovanie logov

```bash
docker logs <id> | grep "GET"
```
- Vyfiltruje všetky riadky obsahujúce "GET"

---

## 🔄 Sledovanie logov v reálnom čase

```bash
docker logs -f <id>
```

```bash
docker logs -f --tail 50 <id>
```
- Sleduj živé logy, začni poslednými 50 riadkami

---

## ⚙️ Logging Driver

- Mechanizmus, ako sa logy ukladajú a spracovávajú
- Predvolený: `json-file`

---

## 🏃 Spúšťanie kontajnerov

```bash
docker run --rm -d --name logdemo logdemo:1.0.0
```

- `--rm`: kontajner sa zmaže po ukončení
- `-d`: detached mód
- `--name`: meno kontajnera
- `logdemo:1.0.0`: image

---

## 🧷 Overwrite ENTRYPOINT

```bash
docker run -it --rm --entrypoint /bin/sh logdemo:1.0.0
```
- Prepisuje `ENTRYPOINT` a otvorí shell
- Umožní ti prechádzať obsah image

---

## 🔍 Inšpekcia kontajnera

```bash
docker inspect <name> | grep Created
```
- Zistíš, kedy bol kontajner/image vytvorený

---

## 🧹 Vymazanie kontajnerov

```bash
docker container prune
```
- Vymaže **všetky zastavené** kontajnery

---

## 📎 Práca s kontajnerom

```bash
docker exec <id> cat /app/app.py
```
- Spustí príkaz `cat` v kontajneri (zobrazí obsah súboru)

```bash
docker attach <name>
```
- Pripojíš sa k bežiacemu kontajneru

---

## ⏸️ Pozastavenie a obnovenie

```bash
docker pause <id>
docker unpause <id>
```
- Pozastaví / obnoví procesy bežiace v kontajneri


# Docker príkaz `docker stats`

## 🧾 Popis

Príkaz `docker stats`:

- **Zobrazuje živé štatistiky** (v reálnom čase) o všetkých bežiacich Docker kontajneroch.

## 🔍 Využitie

Slúži na monitorovanie systémových zdrojov, ktoré využívajú kontajnery:

- **CPU %** – Využitie procesora
- **MEM USAGE / LIMIT** – Spotreba a limit pamäte
- **MEM %** – Percentuálne využitie pamäte
- **NET I/O** – Sieťový prenos (prijaté / odoslané dáta)
- **BLOCK I/O** – Prístup k disku (čítanie / zápis)
- **PIDS** – Počet procesov bežiacich v kontajneri

## 📌 Príklady použitia

```bash
# Zobrazenie štatistík všetkých bežiacich kontajnerov
docker stats

# Zobrazenie štatistík pre konkrétny kontajner
docker stats <container_id_or_name>
```

## ❗ Poznámka

- Príkaz `docker stats` **nepozastavuje ani neobnovuje** procesy v kontajneri. Slúži **výhradne na monitorovanie**.

# What is a docker swarn 

