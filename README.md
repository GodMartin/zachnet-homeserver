# Zachnet Home Server Project

Toto je dokumentace k mému soukromému domácímu serveru, který běží na Raspberry Pi. Cílem projektu bylo vytvořit multifunkční platformu pro hostování soukromých služeb, které jsou bezpečně přístupné odkudkoli na světě, a to vše bez nutnosti mít veřejnou IP adresu.

---

## 1. Hardware

* **Server:** Raspberry Pi 3 Model B (armhf, 32-bit)
* **Úložiště:** Softwarový RAID 1 (mirroring) vytvořený pomocí `mdadm`, který se skládá ze dvou externích USB disků. Systémový disk je microSD karta.
* **Síť:** Připojeno 100Mb/s Ethernet kabelem k routeru O2 Smart Box.

---

## 2. Software (Server)

* **Operační systém:** Raspberry Pi OS (Debian 12 "Trixie")
* **Webový server (Pro Hub):** Apache2
* **Souborový server (Lokální):** Samba
* **Souborový server (Vzdálený):** FileBrowser
* **Mediální server:** Plex Media Server
* **Blokátor reklam:** AdGuard Home
* **Vzdálený přístup:** Cloudflare Tunnel (`cloudflared`) a OpenSSH server.

---

## 3. Síťová konfigurace a služby

Toto je jádro celého projektu. Místo nebezpečného otevírání portů (Port Forwarding) na routeru je celý server připojen k internetu pomocí **Cloudflare Tunnel**.

### Cloudflare Tunnel (`cloudflared`)

Služba `cloudflared` běží přímo na Raspberry Pi a vytváří trvalý, šifrovaný tunel do sítě Cloudflare. Všechny mé veřejné domény jsou směrovány tímto tunelem na lokální služby běžící na Pi.

Konfigurace tunelu (`config.yml`):
```yaml
tunnel: 3e1b4697-2c02-47a9-baa5-ae27b16fed73
credentials-file: /etc/cloudflared/3e1b4697-2c02-47a9-baa5-ae27b16fed73.json

ingress:
  # Hlavní Hub (Apache)
  - hostname: zachnet.cz
    service: http://localhost:80
  
  # Druhý web (Apache VirtualHost)
  - hostname: zachnet.online
    service: http://localhost:80

  # Webový NAS (FileBrowser)
  - hostname: nas.zachnet.cz
    service: http://localhost:8080

  # Panel AdGuardu
  - hostname: adguard.zachnet.cz
    service: http://localhost:8081

  # Vzdálený terminál
  - hostname: ssh.zachnet.cz
    service: ssh://localhost:22

  # Koncové pravidlo
  - service: http_status:404
````

### Přehled hostovaných služeb

| Veřejná adresa | Lokální cíl (Port) | Software | Účel |
| --- | --- | --- | --- |
| `https://zachnet.cz` | `localhost:80` | Apache2 | Hlavní rozcestník (Hub) |
| `https://nas.zachnet.cz` | `localhost:8080` | FileBrowser | Vzdálený přístup k souborům (NAS) |
| `https://ssh.zachnet.cz` | `localhost:22` | OpenSSH + CF Access | Vzdálený terminál (SSH) v prohlížeči |
| `https://adguard.zachnet.cz`| `localhost:8081` | AdGuard Home | Administrace blokátoru reklam |
| `http://10.0.1.106:32400` | `localhost:32400` | Plex Media Server | Lokální mediální server (Netflix) |
| `10.0.1.106` (Lokálně) | `port 53` | AdGuard Home | DNS server pro blokování reklam v síti |
| `\\RASPBERRYPI\NAS` | `port 445` | Samba | Rychlý lokální přístup k disku (disk Z:) |

### Zabezpečení

  * **Žádné otevřené porty** na routeru.
  * Veškerý webový provoz (`zachnet.cz`, `nas.zachnet.cz`...) je chráněn **Cloudflare WAF a DDoS ochranou** (zdarma).
  * Přístup k citlivým službám (jako `ssh.zachnet.cz`) je řízen přes **Cloudflare Zero Trust (Access)**, který vyžaduje přihlášení a ověření e-mailem (One-Time-PIN).

<!-- end list -->
