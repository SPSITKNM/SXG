# KVM Virtualizácia – Príkazy, Validácia a Tuned

## Overenie podpory virtualizácie

```bash
lscpu | grep -i virtualization
```
> Overí, či CPU podporuje virtualizáciu (napr. VT-x, AMD-V).

```bash
lsmod | grep kvm
```
> Zistí, či sú načítané moduly jadra pre virtualizáciu (`kvm`, `kvm_intel` alebo `kvm_amd`).

```bash
modinfo kvm | head
```
> Zobrazí základné informácie o module `kvm`.

---

## Validácia QEMU hosta

```bash
sudo virt-host-validate qemu
```
> Overí, či je systém vhodný na hostovanie virtualizovaných systémov pomocou QEMU/KVM.

---

## Tuned – optimalizačný nástroj

Tuned je nástroj, ktorý automaticky aplikuje profily výkonu podľa použitia systému (napr. virtualizácia, desktop, server).

### Spustenie Tuned a overenie aktívneho profilu

```bash
sudo systemctl enable --now tuned
```

```bash
tuned-adm active
```
> Zistí, ktorý profil je momentálne aktívny.

```bash
tuned-adm list
```
> Vypíše všetky dostupné profily.

### Prepnúť na virtualizačný profil hosta

```bash
sudo tuned-adm profile virtual-host
```

### Verifikácia systému

```bash
sudo tuned-adm verify
```

---

## Virtuálne siete

```bash
sudo virsh net-list --all
```
> Zobrazí všetky siete spravované pomocou `libvirt`.

---

## Prihlasovacie údaje pre KVM (Debian VM na Rails PC)

- **Názov PC**: `debian`
- **Root používateľ**:
  - Meno: `root`
  - Heslo: `0000`
- **Nový používateľ**:
  - Meno: `new_user`
  - Heslo: `0000`
