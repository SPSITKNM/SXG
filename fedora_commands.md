# 🐧 Linux (Fedora) – Príkazový riadok Cheatsheet

## 📁 Navigácia v systéme

```
pwd = printne aktuálny adresár 
ls = vypíše obsah adresára 
ls -lAh = detailny výpis vrátane skrytých súborov 
cd /cesta/k/adresaru = prechod do adresára 
cd .. = o úroveň vyššie 
cd = návrat do domáceho adresára 
clear = vyčisti terminál 
```

## 🗃️ Práca so súbormi

```
touch sobor.txt = vytvorí prázdny súbor / textak
mkdir novy_adresar = vytvori adresar
rm subor.txt = odstranenie 
cp zdroj ciel = kopirovanie súboru = takisto ked daš cp existujuci subor neexistujuci tak sa neexistujuci vytvori ako kopia existujuceho, s rovnakym obsahom ako je ten existujuci 
mv zdroj ciel = premiestni premenuje 
cat subor.txt = vypíše obsah
```

## 🔐 Práva a vlastníctvo

```
chmod = change mode 

chmod +x skript.sh == pridá spustiteľné právo 
chmod 755 subor 
```

## 📦 Správa balíčkov (Fedora – DNF)

```
sudo dnf update = aktualizácia systemu 
sudo dnf install nazov = inštalácia balíčka 
sudo dnf remove nazov = odinštalovanie 
dnf search nazov = hladanie balíčka 
dnf list installed = zoznam nainštalovanych balíčkov 
```

## 💽 Správa diskov a systém

```
df -h = vyuzitie diskov 
du -sh * = velkost suborov a priečinkov 
```

## 👤 Používatelia a práva

```
whoami = aktualny user 
id = uid 
sudo su = prechod na roota 
adduser meno = vytvorenie noveho usera 
passwd meno = zmena hesla usera 
```

## 🌐 Sieť a internet

```
ip a = zobrazenie ip adries 
ss -tuln 
```

## 🐳 Základ Docker príkazov

```
docker ps -a = zoznam kontajnerov 
docker images = zoznam images 
docker run hello-world 
```
