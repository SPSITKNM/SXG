# VirtualBox Guide - Základy a Praktické Tipy

## Čo je VirtualBox?

VirtualBox je bezplatný nástroj na virtualizáciu, ktorý umožňuje spúšťať viacero operačných systémov súčasne na jednom počítači. Ideálne pre testovanie, vývoj a učenie sa nových systémov.

## Inštalácia VirtualBox

### Windows/macOS
1. Stiahnite z [virtualbox.org](https://www.virtualbox.org)
2. Spustite inštalátor a postupujte podľa pokynov
3. Nainštalujte VirtualBox Extension Pack pre rozšírené funkcie

### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install virtualbox virtualbox-ext-pack
```

### Linux (CentOS/RHEL/Fedora)
```bash
sudo yum install VirtualBox
# alebo
sudo dnf install VirtualBox
```

## Vytvorenie novej virtuálnej mašiny

### 1. Spustenie VirtualBox
```bash
virtualbox          # Spustenie grafického rozhrania
VBoxManage --version # Kontrola verzie z príkazového riadku
```

### 2. Vytvorenie VM cez GUI
1. **New** → Zadajte názov VM
2. **Type**: Linux/Windows/macOS
3. **Version**: Ubuntu 64-bit / Windows 10 / atď.
4. **Memory**: Minimálne 2GB (2048 MB)
5. **Hard disk**: Create virtual hard disk now
6. **File type**: VDI (VirtualBox Disk Image)
7. **Storage**: Dynamically allocated
8. **Size**: Minimálne 20GB

## Základné nastavenia VM

### Systémové nastavenia
```
Settings → System:
- Base Memory: 2048-4096 MB (závisí od vašej RAM)
- Processors: 2-4 CPU (závisí od vášho procesora)
- Enable VT-x/AMD-V: Povolené
- Enable PAE/NX: Povolené
```

### Display nastavenia
```
Settings → Display:
- Video Memory: 128 MB
- Enable 3D Acceleration: Áno
- Graphics Controller: VMSVGA/VBoxVGA
```

### Storage nastavenia
```
Settings → Storage:
- Controller IDE: Pridať CD/DVD disk s ISO
- Controller SATA: Virtuálny hard disk
```

### Network nastavenia
```
Settings → Network:
- Adapter 1: NAT (pre internet)
- Adapter 2: Host-only (pre komunikáciu s hostom)
```

## Príkazový riadok (VBoxManage)

### Základné príkazy

#### Zobrazenie informácií
```bash
VBoxManage list vms                    # Zoznam všetkých VM
VBoxManage list runningvms             # Bežiace VM
VBoxManage showvminfo "VM_Name"        # Detaily o VM
VBoxManage list ostypes                # Podporované OS typy
```

#### Vytvorenie VM z príkazového riadku
```bash
# Vytvorenie novej VM
VBoxManage createvm --name "Ubuntu_VM" --ostype "Ubuntu_64" --register

# Nastavenie pamäte
VBoxManage modifyvm "Ubuntu_VM" --memory 2048

# Vytvorenie virtuálneho disku
VBoxManage createhd --filename "Ubuntu_VM.vdi" --size 20480 --format VDI

# Pridanie SATA kontroléra
VBoxManage storagectl "Ubuntu_VM" --name "SATA Controller" --add sata --controller IntelAhci

# Pripojenie disku
VBoxManage storageattach "Ubuntu_VM" --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium "Ubuntu_VM.vdi"

# Pridanie IDE kontroléra pre CD/DVD
VBoxManage storagectl "Ubuntu_VM" --name "IDE Controller" --add ide --controller PIIX4

# Pripojenie ISO súboru
VBoxManage storageattach "Ubuntu_VM" --storagectl "IDE Controller" --port 1 --device 0 --type dvddrive --medium "/path/to/ubuntu.iso"
```

#### Spustenie a zastavenie VM
```bash
# Spustenie VM
VBoxManage startvm "Ubuntu_VM"                    # S GUI
VBoxManage startvm "Ubuntu_VM" --type headless    # Bez GUI (server mód)

# Zastavenie VM
VBoxManage controlvm "Ubuntu_VM" poweroff         # Tvrdé vypnutie
VBoxManage controlvm "Ubuntu_VM" acpipowerbutton  # Jemné vypnutie
VBoxManage controlvm "Ubuntu_VM" pause            # Pozastavenie
VBoxManage controlvm "Ubuntu_VM" resume           # Obnovenie
```

#### Snímky (Snapshots)
```bash
# Vytvorenie snímky
VBoxManage snapshot "Ubuntu_VM" take "Before_Install" --description "Clean system"

# Zoznam snímok
VBoxManage snapshot "Ubuntu_VM" list

# Obnovenie snímky
VBoxManage snapshot "Ubuntu_VM" restore "Before_Install"

# Vymazanie snímky
VBoxManage snapshot "Ubuntu_VM" delete "Before_Install"
```

#### Klónovanie VM
```bash
# Úplné klónovanie
VBoxManage clonevm "Ubuntu_VM" --name "Ubuntu_VM_Clone" --register

# Klónovanie so snímkami
VBoxManage clonevm "Ubuntu_VM" --name "Ubuntu_VM_Clone" --mode all --register
```

## Guest Additions

### Čo sú Guest Additions?
Špeciálne ovládače a nástroje pre lepšiu integráciu medzi hostom a guest systémom.

### Výhody:
- Lepšia grafická podpora
- Zdieľanie schránky
- Drag & drop súborov
- Zdieľané priečinky
- Automatické zmeny rozlíšenia

### Inštalácia Guest Additions

#### Linux Guest:
```bash
# Pripojenie Guest Additions CD
sudo mkdir /mnt/cdrom
sudo mount /dev/cdrom /mnt/cdrom

# Inštalácia
sudo apt update
sudo apt install build-essential dkms linux-headers-$(uname -r)
sudo /mnt/cdrom/VBoxLinuxAdditions.run

# Reštart
sudo reboot
```

#### Windows Guest:
1. Devices → Insert Guest Additions CD image
2. Spustite VBoxWindowsAdditions.exe
3. Postupujte podľa inštalátora
4. Reštartujte systém

## Zdieľané priečinky

### Nastavenie zdieľaného priečinka
```bash
# Vytvorenie zdieľaného priečinka
VBoxManage sharedfolder add "Ubuntu_VM" --name "shared" --hostpath "/host/path" --automount

# Odstránenie zdieľaného priečinka
VBoxManage sharedfolder remove "Ubuntu_VM" --name "shared"
```

### Prístup k zdieľanému priečinku v Linux Guest
```bash
# Pripojenie manuálne
sudo mkdir /mnt/shared
sudo mount -t vboxsf shared /mnt/shared

# Automatické pripojenie (pridať do /etc/fstab)
shared /mnt/shared vboxsf defaults,uid=1000,gid=1000 0 0

# Pridanie používateľa do skupiny vboxsf
sudo usermod -a -G vboxsf $USER
```

## Sieťové nastavenia

### Typy pripojení:

#### 1. NAT (Network Address Translation)
- **Použitie**: Základný internet prístup
- **Vlastnosti**: Guest môže pristupovať na internet, ale nie je dostupný z vonku
```bash
VBoxManage modifyvm "Ubuntu_VM" --nic1 nat
```

#### 2. NAT Network
- **Použitie**: Viacero VM môže komunikovať medzi sebou a s internetom
```bash
# Vytvorenie NAT siete
VBoxManage natnetwork add --netname "NatNetwork" --network "192.168.15.0/24" --enable

# Pripojenie VM k NAT sieti
VBoxManage modifyvm "Ubuntu_VM" --nic1 natnetwork --nat-network1 "NatNetwork"
```

#### 3. Bridged Adapter
- **Použitie**: VM sa správa ako fyzický počítač v sieti
```bash
VBoxManage modifyvm "Ubuntu_VM" --nic1 bridged --bridgeadapter1 "eth0"
```

#### 4. Host-only Adapter
- **Použitie**: Komunikácia len medzi hostom a guest systémami
```bash
VBoxManage modifyvm "Ubuntu_VM" --nic1 hostonly --hostonlyadapter1 "vboxnet0"
```

## Užitočné tipy a triky

### Performance optimalizácia

#### Nastavenia pre lepší výkon:
```bash
# Povolenie hardvérovej virtualizácie
VBoxManage modifyvm "Ubuntu_VM" --hwvirtex on --vtxvpid on

# Nastavenie grafického kontroléra
VBoxManage modifyvm "Ubuntu_VM" --graphicscontroller vmsvga --vram 128

# Nastavenie počtu CPU
VBoxManage modifyvm "Ubuntu_VM" --cpus 2

# Povolenie PAE
VBoxManage modifyvm "Ubuntu_VM" --pae on
```

#### Optimalizácia disku:
```bash
# Kompresácia VDI súboru
VBoxManage modifyhd "Ubuntu_VM.vdi" --compact

# Zmena typu disku na fixed size (rýchlejší)
VBoxManage clonehd "Ubuntu_VM.vdi" "Ubuntu_VM_fixed.vdi" --variant Fixed
```

### Správa VM cez príkazový riadok

#### Export/Import VM
```bash
# Export VM do OVA súboru
VBoxManage export "Ubuntu_VM" --output "ubuntu_vm.ova"

# Import OVA súboru
VBoxManage import "ubuntu_vm.ova"
```

#### Informácie o systéme
```bash
# Informácie o hoste
VBoxManage list systemproperties

# Informácie o sieťových adaptéroch
VBoxManage list hostonlyifs
VBoxManage list bridgedifs
```

### Riešenie bežných problémov

#### Problém s bootovaním:
```bash
# Kontrola boot poradia
VBoxManage modifyvm "Ubuntu_VM" --boot1 dvd --boot2 disk --boot3 none --boot4 none
```

#### Problém so zvukom:
```bash
# Nastavenie audio kontroléra
VBoxManage modifyvm "Ubuntu_VM" --audio alsa --audiocontroller ac97
```

#### Problém s USB:
```bash
# Povolenie USB 2.0/3.0 (vyžaduje Extension Pack)
VBoxManage modifyvm "Ubuntu_VM" --usb on --usbehci on --usbxhci on
```

## Automatizácia s bash skriptami

### Skript na vytvorenie VM:
```bash
#!/bin/bash
VM_NAME="Ubuntu_Dev"
ISO_PATH="/path/to/ubuntu.iso"
VM_PATH="$HOME/VirtualBox VMs"

# Vytvorenie VM
VBoxManage createvm --name "$VM_NAME" --ostype "Ubuntu_64" --register --basefolder "$VM_PATH"

# Konfigurácia
VBoxManage modifyvm "$VM_NAME" --memory 2048 --cpus 2 --vram 128
VBoxManage modifyvm "$VM_NAME" --nic1 nat --graphicscontroller vmsvga
VBoxManage modifyvm "$VM_NAME" --hwvirtex on --vtxvpid on --pae on

# Storage
VBoxManage createhd --filename "$VM_PATH/$VM_NAME/$VM_NAME.vdi" --size 20480
VBoxManage storagectl "$VM_NAME" --name "SATA" --add sata --controller IntelAhci
VBoxManage storageattach "$VM_NAME" --storagectl "SATA" --port 0 --device 0 --type hdd --medium "$VM_PATH/$VM_NAME/$VM_NAME.vdi"

# ISO
VBoxManage storagectl "$VM_NAME" --name "IDE" --add ide
VBoxManage storageattach "$VM_NAME" --storagectl "IDE" --port 1 --device 0 --type dvddrive --medium "$ISO_PATH"

echo "VM $VM_NAME bola vytvorená!"
```

## Užitočné príkazy pre údržbu

### Čistenie a údržba:
```bash
# Zoznam médií
VBoxManage list hdds
VBoxManage list dvds

# Odstránenie nepoužívaných médií
VBoxManage closemedium disk "path/to/unused.vdi" --delete

# Aktualizácia VirtualBox
sudo apt update && sudo apt upgrade virtualbox
```

### Backup a obnovenie:
```bash
# Backup konfigurácie VM
VBoxManage export "Ubuntu_VM" --output "backup_$(date +%Y%m%d).ova"

# Zoznam všetkých VM s cestami
VBoxManage list vms -l | grep "Config file"
```

---

## Odporúčania pre výučbu

1. **Začnite s GUI** - študenti si najprv osvojte grafické rozhranie
2. **Postupne pridávajte CLI** - príkazový riadok až keď ovládajú základy
3. **Používajte snímky** - pred každým experimentom
4. **Zdieľané priečinky** - pre jednoduchý prenos súborov
5. **Dokumentujte nastavenia** - pre jednoduché replikovanie prostredí

**Bezpečnostné upozornenie:** VM nie sú 100% izolované - nepoužívajte pre nebezpečné aktivity!
