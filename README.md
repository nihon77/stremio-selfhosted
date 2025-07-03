# stremio-selfhosted  
**Stremio Stack con Mammamia, Media Flow Proxy e altro ancora**

Questo repository contiene istruzioni, configurazioni e suggerimenti per il self-hosting sul proprio NAS domestico di un'intera istanza privata di Stremio, con plugin **Mammamia**, **media-flow-proxy**, **StreamV** e altri componenti opzionali.

---

## 📢 Disclaimer (con un piccolo rant) 

> Questo progetto è a scopo puramente educativo.  
> L'utilizzo improprio di componenti che accedono a contenuti protetti da copyright potrebbe violare le leggi del tuo paese.  
> **Usa questi strumenti solo per contenuti legalmente ottenuti.**  
> L’autore non si assume responsabilità per eventuali usi illeciti.

> ### 📣 Un pensiero personale:
> Se stai usando Stremio con mille plugin e Real-Debrid, sappilo: non sei un pirata.
> 
>  **News flash: non sei un pirata, sei un leacher da salotto con le crocs ai piedi.**
> 
> Il pirata vero seedava, uploada, si faceva il port forwarding da solo e sniffava i peer con Wireshark. Tu clicchi e guardi. Comodo, eh? Ma zero gloria.
> Stai "guardando gratis" sì, ma stai succhiando banda da server altrui senza restituire niente.

> **Non dai nulla, non condividi nulla.**  

> **Zero upload, zero sharing, zero rispetto per chi ci mette storage, tempo e skill.**
> La tua banda in upload è più vuota della cartella “Download” su eMule nel 2025.
> 
> 💰 E poi ci sono quelli che bypassano la pubblicità sui siti di streaming…
> Ma lo sai che quei 2 banner schifosi sono l’unica cosa che tiene in piedi quei siti?
> Se li togli pure quelli, poi piangi perché non trovi più il film russo del 2003 sottotitolato in polacco.

> 💀 Se sei uno che si è mai lamentato per la qualità di uno stream pirata...ti meriti il buffering perpetuo.

> Se proprio vuoi vivere ai margini del sistema, almeno fallo con un po’ di dignità.
> Usa i torrent. Condividi. Seeda. Rompiti la testa sui port forwarding.
> E soprattutto: non fare il figo con gli script di qualcun altro.

> “Steal with style. Share like it’s 2006. Respect the swarm.”

---

## ✅ Requisiti  

Hai deciso di configurare una tua istanza privata di Mammamia e Media Flow Proxy, senza spendere un centesimo? Ecco cosa ti serve:

### 🖥️ Hardware e sistema operativo
- Un **PC o NAS** collegato alla rete locale.
- Una distribuzione Linux basata su Debian (es. Ubuntu).  
  > *(Le istruzioni sono per Ubuntu, ma facilmente adattabili ad altre distro.)*

### 🌍 Accesso remoto con IP dinamico + Port Forwarding

Poiché molti provider Internet assegnano un **IP pubblico dinamico**, è necessario un sistema per mantenere accessibile il tuo server anche quando l’IP cambia.
- Un **IP pubblico** (va bene anche se dinamico).
- Un account gratuito su [**duckdns.org**](https://www.duckdns.org) per creare **hostname statici** che puntano sempre al tuo NAS.
- Il tuo router deve eseguire un **Port Forwarding**:
  - Porta **80** (HTTP) → verso la **porta 8080** del tuo NAS
  - Porta **443** (HTTPS) → verso la **porta 8433** del tuo NAS

> 🔁 Questo setup è fondamentale per permettere a Nginx Proxy Manager di ottenere e rinnovare automaticamente i certificati SSL tramite Let’s Encrypt.


### 🔐 Creazione degli hostname su No-IP

Per accedere alle tue applicazioni da remoto, devi creare 3 hostname pubblici gratuiti su [**duckdns.org**](https://www.duckdns.org).

> ⚠️ Gli hostname devono essere univoci. Il mio consiglio è quello di aggiungere un identificativo personale (es. il tuo nome o una sigla) per evitare conflitti.

#### Esempi di hostname personalizzati:
- `mammamia-mario.duckdns.org`
- `mfp-mario.duckdns.org `
- `streamv-mario.duckdns.org `
Puoi ovviamente scegliere qualsiasi nome, purché sia disponibile e facile da ricordare.

Questi hostname punteranno sempre al tuo NAS anche se il tuo IP cambia.  
Il tutto è possibile installando un piccolo agente (Dynamic DNS client) che aggiorna automaticamente il record DNS.

### 🍴 Consigliato: fai un fork del repository

> ✨ E' consigliabile creare un **fork personale** di questo repository su GitHub, in modo da poterlo modificare facilmente secondo le tue esigenze.
> Per farlo ti servirà anche un account su GitHub

Per fare ciò:
1. Vai sulla pagina del repository originale
2. Clicca su **"Fork"** (in alto a destra)
3. Clona il tuo fork sul NAS:

```bash
git clone https://github.com/<il-tuo-utente>/<nome-repo>.git
cd <nome-repo>
```
---

## 🔧 Componenti del progetto

| Servizio           | Nome Servizio Docker | Porta interna | Descrizione                              |
|--------------------|----------------------|---------------|------------------------------------------|
| **[Mammamia](https://github.com/UrloMythus/MammaMia)**|mammamia       | 8080(*)          | Plugin personalizzato per Stremio        |
| **[Media Flow Proxy (MFP)](https://github.com/mhdzumair/mediaflow-proxy)**|mediaflow_proxy | 8888(*)   | Proxy per streaming video                |
| **[StreamV](https://github.com/qwertyuiop8899/StreamV)**|steamv        | 7860(*)          | Web player personalizzato (opzionale)    |
| **[Nginx Proxy Manager](https://github.com/NginxProxyManager/nginx-proxy-manager)**|npm | 8080/8443/8181 | Reverse proxy + certificati Let's Encrypt |
| **[docker-duckdns](https://github.com/linuxserver/docker-duckdns)** |duckdns-updater |—         | Aggiorna il DNS dinamicamente            |

>ℹ️ (*)Le **porte elencate (tranne quelle di Nginx Proxy Manager)** sono **interne alla rete Docker** e **non sono esposte direttamente** sulla macchina host.
Questo significa che i servizi **non sono accessibili dall’esterno se non tramite Nginx Proxy Manager**, che funge da gateway sicuro con supporto a **HTTPS e Let's Encrypt**.

---

## 🔧 Configurazione hostname statici con No-IP

Se il tuo IP pubblico è dinamico, DuckDns ti permette di associare un hostname che si aggiorna automaticamente ogni volta che il tuo IP cambia. Ecco come fare:

### 1. Registrazione e login

- Vai su [https://www.duckdns.org/](https://www.duckdns.org/)
- Clicca su **Sign In With GitHub/Google/QuelloCheVipare** e crea un account gratuito.


### 2. Creazione degli hostname (Dynamic DNS)

- Dopo il login, vi ritroverete direttamente nella **Dashboard** di DuckDns. Notate nella sezione in alto il vostro token identificatvo che utilizzeremo nel .env del container duckdns.
- Inserisci il nome host, ad esempio:

  - `mammamia-mario`
  
  Scegli un nome unico che ti permetta di riconoscerlo facilmente.
- Premi **Add Domain**.
- Nel campo **Current IP**, vedrai il tuo IP pubblico attuale (se errato, correggilo con quello giusto).

<img width="1818" alt="Screenshot 2025-07-03 at 11 23 30" src="https://github.com/user-attachments/assets/f44d2ab9-ce39-43ff-b61a-1b204a266711" />

### 3. Ripeti per gli altri due hostname

- Crea altri due hostname per:

  - `mfp-mario.duckdns.org`
  - `streamv-mario.duckdns.org`

<img width="1818" alt="Screenshot 2025-07-03 at 11 27 55" src="https://github.com/user-attachments/assets/9d70c20c-985e-410f-9638-a2bf7c7d9988" />

---

## 🌐 Configurazione DNS globale (Cloudflare, Quad9, ecc.)

Se il tuo sistema utilizza systemd-resolved (come avviene per default in Ubuntu e derivati), non modificare direttamente il file /etc/resolv.conf, perché viene gestito automaticamente.

1. Apri il file di configurazione:

```bash
sudo nano /etc/systemd/resolved.conf
```

2. Cerca la sezione [Resolve] e modifica come segue:

```text
[Resolve]
DNS=1.1.1.1 1.0.0.1
FallbackDNS=9.9.9.9
DNSStubListener=yes
```

* 1.1.1.1 = Cloudflare
* 9.9.9.9 = Quad9 (opzionale fallback)

3. Riavvia il servizio:

```bash
sudo systemctl restart systemd-resolved
```

4. Verifica che i DNS siano attivi:

```bash
resolvectl status
```

---

## 📦 Docker + Docker Compose

> Questo progetto usa **Docker** per semplificare l’installazione e l’isolamento dei servizi.

### 📥 Installazione Docker

```bash
# 🔁 Rimuovi eventuali versioni precedenti
sudo apt remove docker docker-engine docker.io containerd runc

# 🔄 Aggiorna l’elenco dei pacchetti
sudo apt update

# 📦 Installa i pacchetti richiesti per aggiungere il repository Docker
sudo apt install -y ca-certificates curl gnupg lsb-release

# 🗝️ Aggiungi la chiave GPG ufficiale di Docker
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 📥 Aggiungi il repository Docker alle fonti APT
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 🐳 Installa Docker, Docker Compose e altri componenti
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 👤 Aggiungi il tuo utente al gruppo docker
sudo usermod -aG docker $USER

# ⚠️ Per applicare il cambiamento, esegui il logout/login oppure:
newgrp

# ✅ Verifica che Docker funzioni correttamente
docker run hello-world
```

---

## 🚀 Avvio del progetto dal repository GitHub

Il progetto è contenuto in un repository GitHub che include un file `docker-compose.yml` preconfigurato. Alcuni servizi costruiranno automaticamente le immagini Docker a partire da Dockerfile remoti ospitati su GitHub.

### 🔧 Prerequisiti

Assicurati di avere:
- Docker e Docker Compose installati (vedi sezione precedente)
- Git installato (`sudo apt install git` se non lo hai)


### 📥 Clona il repository

```bash
cd ~
git clone https://github.com/tuo-utente/tuo-repo.git
cd tuo-repo
```

### 🌐 Crea la rete Docker esterna proxy
Se non l'hai già fatto, crea la rete che verrà utilizzata da Nginx Proxy Manager e dagli altri container per comunicare tra loro:

```bash
docker network create proxy
```
>🔁 Questo comando va eseguito una sola volta. Se la rete esiste già, Docker mostrerà un errore che puoi ignorare in sicurezza.

### 🛠️ Creazione dei file .env per MammaMia,Media Flow Proxy,StreamV e docker-duckdns
In ogni sotto cartella di questo progetto è presente un file .env_example con tutte le chiavi necessarie per il corretto funzionamento dei vari moduli.
Per ogni modulo copiare e rinominare il file .env_example in .env. I vari .env dovranno essere modificati in base alle vostre specifiche configurazioni.

**1. .env per MammaMia**
Per configurare il plugin MammaMia è necessario configurare il relativo file .env. Vi rimando al repo del progetto per i dettagli.

📄 Esempio: ./mammamia/.env
```text
# File .env per il plugin mammamia
TMDB_KEY=xxxxxxxxxxxxxxxx
PROXY=["http://xxxxxxx-rotate:xxxxxxxxx@p.webshare.io:80"]
FORWARDPROXY=http://xxxxxxx-rotate:xxxxxxxx@p.webshare.io:80/
```

**2. .env per Media Flow Proxy**
Per configurare il modulo Media Flow Proxy è necessario configurare il relativo file .env. Vi rimando al repo del progetto per i dettagli.

📄 Esempio: ./mfp/.env
```text
API_PASSWORD=password
TRANSPORT_ROUTES={"all://*.ichigotv.net": {"verify_ssl": false}, "all://ichigotv.net": {"verify_ssl": false}}
```

**3. .env per StreamV**
Per configurare il plugin StreamV è necessario configurare il relativo file .env. Vi rimando al repo del progetto per i dettagli.

📄 Esempio: ./mfp/.env
```text
TMDB_API_KEY="xxxxxxxxxxxxxxxx"
MFP_PSW="xxxxxxxxx"
MFP_URL="https://mfp-mario.ddns.net"
BOTHLINK=true
```

**4. .env per DuckDNS Updater**
Per configurare correttamente il client DDNS, è necessario un file .env contenente le credenziali e gli hostname associati al tuo account DuckDns.

📄 Esempio: ./duckdns-updater/.env
```text
# File .env per il client DDNS
SUBDOMAINS=mammamia,mfp,streamv
TOKEN=IL_TUO_TOKEN
TZ=Europe/Rome
```

🛑 Attenzione alla sicurezza: imposta i permessi del file .env in modo che sia leggibile solo dal tuo utente, ad esempio:

```bash
chmod 600 ./duckdns-updater/.env
```

🔁 Ricorda di sostituire:
IL_TUO_TOKEN → con il token visibile sulla Dashboard di DuckDns.
SUBDOMAINS → con i tuoi hostname specifici separati da virgole (host1,host2 ecc senza .duckdns.org).


### 🏗️ Build delle immagini e avvio dei container

Per buildare le immagini (se definite tramite build: con URL GitHub) e avviare tutto in background:

```bash
docker compose up -d --build
```
> 🧱 Il flag --build forza Docker a scaricare i Dockerfile remoti ed eseguire la build, anche se l'immagine esiste già localmente.

### 🔍 Verifica che tutto sia partito correttamente

```bash
docker compose ps
```

Puoi anche consultare i log con:

```bash
docker compose logs -f
```

🔁 Aggiornare il repository e ricostruire tutto (quando aggiorni da GitHub)
```bash
git pull
docker compose down
docker compose up -d --build
```

---

## 🔐 Configurazione degli hostname e gestione SSL con Nginx Proxy Manager (NPM)

Per rendere accessibili le tue applicazioni web da internet in modo sicuro, useremo **Nginx Proxy Manager (NPM)**. Questo tool semplifica la gestione dei proxy inversi e automatizza l’ottenimento dei certificati SSL con Let’s Encrypt.

### 1. Creazione dei tre hostname su DuckDns

Assicurati di aver creato 3 hostname statici su [**duckdns.org**](https://www.duckdns.org) che puntino al tuo IP pubblico (anche se dinamico, aggiornato tramite l’agent docker-duckdns):

- `mammamia-<tuo-id>.duckdns.org`
- `mfp-<tuo-id>.duckdns.org`
- `streamv-<tuo-id>.duckdns.org`

> 🔔 **Suggerimento:** Usa un identificativo unico (`<tuo-id>`) per evitare conflitti con altri utenti DuckDns.

### 2. Port Forwarding sul router

Per permettere il corretto funzionamento di NPM e il rinnovo automatico dei certificati SSL:

- Reindirizza la porta **80 (HTTP)** del router verso la porta **8080** del PC/NAS dove gira NPM.
- Reindirizza la porta **443 (HTTPS)** del router verso la porta **8443** del PC/NAS.

> Questo consente a Let’s Encrypt di verificare il dominio e rilasciare i certificati.

### 3. Configurazione dei proxy host in Nginx Proxy Manager

Per ogni applicazione, crea un nuovo **Proxy Host** in NPM seguendo questi passi:
- **accedi ad http://<ip-tuo-server>:8181** (al primo accesso le credenziali di default sono **Email: admin@example.com Password: changeme**. Vi verrà chiesto di modificarle)
- **Dalla barra di menu selezionate **Hosts** → **Proxy Hosts** → **Add New Proxy**
- **Domain Names:** inserisci l’hostname corrispondente (es. `mammamia-<tuo-id>.duckdns.org`)
- **Scheme:** `http`
- **Forward Hostname / IP:** il mome del servizio cosi come configurato nel docker-compose ovvero mammmia, mediaflow_proxy e streamv
- **Forward Port:** la porta interna dove l’app è in ascolto (es. `8080` per Mammamia, `8888` per mediaflow_proxy e `7860` per streamv)
- Abilita le seguenti opzioni:
  - **Block Common Exploits**
  - **Websockets Support** (se necessario)
  - **Enable HSTS** (opzionale, aumenta la sicurezza)
  ![image](https://github.com/user-attachments/assets/3ad4e778-81be-492f-9db1-3df5b51f1ed9)

- **SSL tab:** seleziona:
  - **Enable SSL**
  - **Force SSL**
  - **HTTP/2 Support**
  - Spunta **Request a new SSL certificate from Let's Encrypt**
  - Accetta i Termini di servizio di Let’s Encrypt
  - Inserisci un indirizzo email valido per la registrazione SSL
  ![image](https://github.com/user-attachments/assets/6f0ef193-45d3-48c8-a6be-7711986f7054)

- **Nel tab Advanced** aggiungete queste configurazioni :

  ```text
  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;
  proxy_pass_request_headers on;
  ```
  ![image](https://github.com/user-attachments/assets/92fea31d-bb8c-49c5-a887-f3d5486c9f7f)

Ripeti questa configurazione per ciascuno dei tre hostname con la rispettiva porta (ad esempio, `mfp-<tuo-id>.duckdns.org` → porta `8888`, ecc.).

### 4. Verifica e manutenzione

- Dopo aver configurato i proxy host, prova ad accedere agli URL pubblici via browser.
- NPM gestirà automaticamente il rinnovo dei certificati SSL.
- Assicurati che il tuo router sia sempre configurato correttamente per il port forwarding, specialmente dopo eventuali riavvii o aggiornamenti firmware.

---


