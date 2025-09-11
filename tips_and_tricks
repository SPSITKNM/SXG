# Poznámky k SSH a ECDSA

## ECDSA – Eliptic Curve Digital Signature Algorithm

- Modernejší a bezpečnejší než RSA / DSA
- **Security strength** – udáva sa v bitoch, napr. 128-bitová bezpečnosť znamená, že útočník musí skúsiť 2^128 možností (brute-force)
- **Čím viac bitov, tým ťažšie prelomiť**
- Príklad:  
  - RSA 2048-bit → ~112-bit security  
  - ECDSA 521-bit → ~256-bit security

### Výhody ECDSA-521:
- Vysoká bezpečnosť
- Rýchle podpisovanie a overovanie
- Efektivita (malý kľúč, malý podpis)
- Kompatibilita s modernými systémami (GitLab, OpenSSH)

### Nevhodné použitie:
- Staršie systémy / servery (nižšia kompatibilita)
- Vysoká kompatibilita: **RSA** (aj keď slabší)

---

## Kryptografické algoritmy

### RSA
- Založený na faktorizácii veľkých prvočísel
- Výhody: vysoká kompatibilita
- Nevýhody: pomalý, potrebuje dlhé kľúče

### ECDSA
- Eliptické krivky + diskrétny logaritmus
- Výhody: kratšie kľúče, vyššia bezpečnosť, rýchlejší podpis/verifikácia

### ED25519
- Eliptické krivky + moderná optimalizácia pre rýchlosť a bezpečnosť

---

## Overenie inštalácie OpenSSH klienta

```bash
ssh -V
```

---

## Vytvorenie ECDSA-521 SSH kľúča

```bash
ssh-keygen -t ecdsa -b 521 -C "tvoje_meno@tvoj_email"
```

- `-t` = typ kľúča (rsa, ecdsa, ed25519, dsa)
- `-b` = bitová dĺžka (521 → krivka nistp521)
- `-C` = komentár (nepovinné)

### Kroky:

1. Spusti príkaz `ssh-keygen -t ecdsa -b 521 -C "tvoje_meno@tvoj_email"`
2. Zadaj súbor pre uloženie (napr. `~/.ssh/id_ecdsa`)
3. Zadaj passphrase = heslo pre ochranu súkromného kľúča

---

## Asymetrická kryptografia

- Dva kľúče: súkromný (tajný), verejný (verejný)
- Funguje: čo zašifruješ jedným, dešifruješ len druhým
- **Použitie:**
  - Autentifikácia (GitLab, SSH)
  - Bezpečnosť
  - Eliminácia hesiel (sú slabé, často opakované)

### Analógia:
- Verejný kľúč = zámok
- Súkromný kľúč = kľúč

---

## Zobrazenie verejného kľúča

```bash
cat ~/.ssh/id_ecdsa.pub
```

- `cat` = zobrazenie obsahu
- `~/.ssh/id_ecdsa.pub` = cesta k verejnému kľúču

---

## Pridanie kľúča do GitLab

- Preferences → SSH Keys → Add Key

---

## Nastavenie SSH konfigurácie podľa repo `ssh-config`

```bash
cd
mkdir -p .ssh/shared
```

```bash
grep -qxF 'include shared/config' .ssh/config || echo -e '\n\ninclude shared/config' >> .ssh/config
```

```bash
cd .ssh/shared
git clone git@gitlab.railsformers.com:railsformers/ssh-config.git .
```

---

## Prvé pripojenie na SSH

```bash
ssh -T git@gitlab.railsformers.com
```

- `ssh` = secure shell
- `-T` = zakáže interaktívny shell (GitLab shell nepodporuje)

### Prvé pripojenie:
- Výzva: “Are you sure you want to continue connecting?”
- Dôvod: nový server → ešte nie je v `known_hosts`
- Odtlačok hostiteľského kľúča → overenie → pridať do `~/.ssh/known_hosts`

---

## Ako to funguje?

- Server pošle verejný kľúč
- SSH vypíše fingerprint → potvrdíš áno
- Uloží sa do `known_hosts`
- Zabraňuje MITM útokom (podvrhnutý server)

---

## SSH OVERALL

- **SSH (Secure Shell)** slúži na bezpečné vzdialené pripojenie medzi dvoma zariadeniami.
- **Architektúra:** klient-server
- **Pripojenie:**  
  ```bash
  ssh používateľ@adresa_servera
  ```
### Autentifikácia

Existujú dve hlavné možnosti overenia:

- **Klasické heslo** – zadáva sa pri každom prihlásení.
- **SSH kľúč (key pair)** – bezpečnejší spôsob:
  - **Private key** – ostáva u teba (na tvojom zariadení)
  - **Public key** – nahráva sa na server alebo napr. do GitLabu

### Test pripojenia 

```bash
ssh -T git@gitlab.com	
```
## Čo nerobiť, keď sa jedná o produkčný server ? 

Nevypínať žiadne procesy ani služby 
- kill procesov, vypnutie databázy atď
- Může to priamo ovplyvniť dostupnosť samotnej služby 

Neresetovať server ani služby bez výslovného pokynu

- Príkazy ako : reboot, shutdown, systemctl restart su abonded. 

Needituj konfiguračné súbory 

- žiadne zmeny v .conf, .env, database.yml, crontab 

Nevykonávat zapisove prikazy bez povolenie 

- rm, mv, cp, touch, nano, vim 

Nepracuj s DS bez konzultácie 

- žiadne DROP, UPDATE, DELETE alebo ALTER 

Zapamataj si : cd - = vracia na posledne ulozene miesto 

docker ps = 

docker compose 







