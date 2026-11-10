# **`dig` i `nslookup`** LXC (ubuntu 26.04) no fucionen
Aquest error es dona amb els template de LXC per ubuntu 26.04.

Si **`dig` i `nslookup` no funcionen per la consola xterm.js de Proxmox**, mentre que `echo` i altres ordres sí que funcionen, no sembla un problema general de xterm.js.
Poden ser per:
1. Problemes amb la sortida estàndard o errors de resolució DNS de les comandes `dig`  i `nslookup`.

### 1. Executa aquestes proves des de xterm.js

### 2. La causa: AppArmor d'Ubuntu 26.04

Hi ha un cas pràcticament idèntic al teu al fòrum oficial de Proxmox.

Ubuntu 26.04 incorpora perfils AppArmor per a `dig` i `nslookup` que poden bloquejar l'accés a la consola TTY de Proxmox. Això fa que les ordres acabin sense mostrar cap resultat, tot i funcionar perfectament per SSH. [Proxmox Support Forum](https://forum.proxmox.com/threads/ubuntu-26-04-lxc-cant-resolve-dns.187146/?utm_source=chatgpt.com)

### 3. Solució proposada

Aplicar una correcció específica sense desactivar tot AppArmor.

Dins del contenidor LXC, com a root, executa:

```bash
mkdir -p /etc/apparmor.d/abstractions/consoles.d
```

Crea el fitxer de compatibilitat:

```bash
echo '/dev/tty[0-9]* rw,' > /etc/apparmor.d/abstractions/consoles.d/pve-container-console
```

Després reinicia el contenidor des del node Proxmox:

Aquesta solució incorpora una regla que permet als perfils AppArmor accedir als dispositius TTY de la consola del contenidor. És la solució que es va confirmar al fòrum de Proxmox. [Proxmox Support Forum](https://forum.proxmox.com/threads/command-dig-not-working-in-container-in-ve-9-1-1.184461/?utm_source=chatgpt.com)

### 4. Comprova que funciona

Torna a obrir xterm.js i executa:

```bash
dig google.com
```

I després:

```bash
nslookup google.com
```

Hauries de veure les respostes DNS normalment.

**Important:** no és recomanable desinstal·lar AppArmor ni desactivar-ne tots els perfils. Aquesta correcció és molt més específica i conserva la resta de proteccions.

Referència: [Discussió i solució al fòrum oficial de Proxmox](https://forum.proxmox.com/threads/command-dig-not-working-in-container-in-ve-9-1-1.184461/?utm_source=chatgpt.com)

