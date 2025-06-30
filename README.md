# stremio-selfhosted  
**Stremio Stack con Mammamia, Media Flow Proxy e altro ancora**

Questo repository contiene istruzioni, configurazioni e suggerimenti per il self-hosting sul proprio NAS domestico di un'intera istanza privata di Stremio, con plugin **Mammamia**, **media-flow-proxy**, **StreamV** e altri componenti opzionali.

---

## 📢 Disclaimer  

> Questo progetto è a scopo puramente educativo.  
> L'utilizzo improprio di componenti che accedono a contenuti protetti da copyright potrebbe violare le leggi del tuo paese.  
> **Usa questi strumenti solo per contenuti legalmente ottenuti.**  
> L’autore non si assume responsabilità per eventuali usi illeciti.

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
- Un account gratuito su [**No-IP.com**](https://www.noip.com/) per creare **hostname statici** che puntano sempre al tuo NAS.
- Il tuo router deve eseguire un **Port Forwarding**:
  - Porta **80** (HTTP) → verso la **porta 8080** del tuo NAS
  - Porta **443** (HTTPS) → verso la **porta 8433** del tuo NAS

> 🔁 Questo setup è fondamentale per permettere a Nginx Proxy Manager di ottenere e rinnovare automaticamente i certificati SSL tramite Let’s Encrypt.


### 🔐 Creazione degli hostname su No-IP

Per accedere alle tue applicazioni da remoto, devi creare 3 hostname pubblici gratuiti su [No-IP.com](https://www.noip.com/).

> ⚠️ Gli hostname devono essere univoci. Il mio consiglio è quello di aggiungere un identificativo personale (es. il tuo nome o una sigla) per evitare conflitti.

#### Esempi di hostname personalizzati:
- `mammamia-mario.ddns.net`
- `mfp-mario.ddns.net`
- `streamv-mario.ddns.net`
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

| Servizio           | Porta interna | Descrizione                              |
|--------------------|---------------|------------------------------------------|
| **[Mammamia](https://github.com/UrloMythus/MammaMia)**       | 8080(*)          | Plugin personalizzato per Stremio        |
| **[Media Flow Proxy (MFP)](https://github.com/mhdzumair/mediaflow-proxy)** | 8888(*)   | Proxy per streaming video                |
| **[StreamV](https://github.com/qwertyuiop8899/StreamV)**        | 7860(*)          | Web player personalizzato (opzionale)    |
| **[Nginx Proxy Manager](https://github.com/NginxProxyManager/nginx-proxy-manager)** | 8080/8443/8181 | Reverse proxy + certificati Let's Encrypt |
| **No-IP DUC (Docker)** | —         | Aggiorna il DNS dinamicamente            |

>ℹ️ (*)Le **porte elencate (tranne quelle di Nginx Proxy Manager)** sono **interne alla rete Docker** e **non sono esposte direttamente** sulla macchina host.
Questo significa che i servizi **non sono accessibili dall’esterno se non tramite Nginx Proxy Manager**, che funge da gateway sicuro con supporto a **HTTPS e Let's Encrypt**.

---

## 🔧 Configurazione hostname statici con No-IP

Se il tuo IP pubblico è dinamico, No-IP ti permette di associare un hostname che si aggiorna automaticamente ogni volta che il tuo IP cambia. Ecco come fare:

---

### 1. Registrazione e login

- Vai su [https://www.noip.com/](https://www.noip.com/)
- Clicca su **Sign Up** e crea un account gratuito.
- Verifica la tua email e accedi con le credenziali create.

### 2. Creazione degli hostname (Dynamic DNS)

- Dopo il login, clicca su **Dashboard** → **DDNS & Remote Access** → **No-Ip Hostnames** → **Create Hostname**.
- Inserisci il nome host, ad esempio:

  - `mammamia-mario e selezionate un dominio come ad esempio .ddns.net`
  
  Scegli un nome unico che ti permetta di riconoscerlo facilmente.

- Nel campo **Record Type** lascia selezionato **DNS Host (A)**.
- Nel campo **IPv4 Address**, vedrai il tuo IP pubblico attuale (se errato, correggilo con quello giusto).
- Premi **Create Hostname**.
<img width="1811" alt="Screenshot 2025-06-28 at 19 15 26" src="https://github.com/user-attachments/assets/437f32d3-7db1-40b4-b864-4fa33a072625" />

### 3. Ripeti per gli altri due hostname

- Crea altri due hostname per:

  - `mfp-mario.ddns.net`
  - `streamv-mario.ddns.net`
<img width="1811" alt="Screenshot 2025-06-28 at 19 20 04" src="https://github.com/user-attachments/assets/d432cfdb-7763-4f48-8f92-51622acd4f16" />

> ⚠️ **Attenzione:** Se vedi una scritta gialla accanto a un hostname su No-IP, significa che **l'aggiornamento automatico dell'indirizzo IP non è attivo**.  
> Andremo successivamente a configurare correttamente il client No-IP (DUC) per mantenerlo aggiornato.

> 📩 **Nota importante:** No-IP, nel piano gratuito, richiede il **rinnovo manuale mensile degli hostname**.  
> Riceverai un'email ogni 30 giorni per confermare **gratuitamente** che desideri mantenere attivo ciascun hostname.  
> Se **non li rinnovi**, gli hostname verranno disattivati e **non saranno più raggiungibili**.

### 4. Creazione del gruppo ddnskey su No-IP
Per aggiornare automaticamente gli hostname tramite client come noip-updater, è consigliato non usare direttamente la tua password dell’account, ma creare un gruppo chiamato ddnskey e generare una password dedicata all’aggiornamento IP.

🧭 Passaggi:

Vai su **Dashboard** → **DDNS & Remote Access** → **DDNS Keys**.
Clicca su "Add Group" (Aggiungi gruppo).
Dai al gruppo il nome: ddnskey_streamio (ad esempio).
Associa al gruppo gli hostname che vuoi aggiornare (es. mammamia-xxx.ddns.net, mfp-xxx.ddns.net, ecc.).
<img width="1667" alt="Screenshot 2025-06-30 at 17 47 52" src="https://github.com/user-attachments/assets/e6e7d3ad-7526-422e-8044-9fff95d4b88c" />

Inserisci una nuova password sicura per questo gruppo (diversa da quella del tuo account principale) cliccando su **Add DDNS Key** e successivamente **Generate DDNS Key**.
Salva.<img width="1666" alt="Screenshot 2025-06-30 at 17 54 11" src="https://github.com/user-attachments/assets/8bee49a0-8ebf-46e3-8d8e-0ba80039131a" /><img width="1666" alt="Screenshot 2025-06-30 at 17 54 11" src="https://github.com/user-attachments/assets/1344ee82-c504-4d2d-8c44-a248cd72e11a" />


📌 Questa password sarà quella da inserire nel .env per il container noip-updater.

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

### 🛠️ Creazione dei file .env per MammaMia,Media Flow Proxy,StreamV e NoIp-Duc
In ogni sotto cartella di questo progetto è presente un file .env_example con tutte le chiavi necessarie per il corretto funzionamento dei vari moduli.
Per ogni modulo copiare e rinominare il file .env_example in .env. I vari .env dovranno essere modificati in base alle vostre specifiche configurazioni.

**1. .env per MammaMia**
Per configurare il plugin MammaMia è necessario configurare il relativo file .env. Vi rimando al repo del progetto per i dettagli.

📄 Esempio: ./mammamia/.env
```bash
# File .env per il plugin mammamia
TMDB_KEY=xxxxxxxxxxxxxxxx
PROXY=["http://xxxxxxx-rotate:xxxxxxxxx@p.webshare.io:80"]
FORWARDPROXY=http://xxxxxxx-rotate:xxxxxxxx@p.webshare.io:80/
```

**2. .env per Media Flow Proxy**
Per configurare il modulo Media Flow Proxy è necessario configurare il relativo file .env. Vi rimando al repo del progetto per i dettagli.

📄 Esempio: ./mfp/.env
```bash
API_PASSWORD=password
TRANSPORT_ROUTES={"all://*.ichigotv.net": {"verify_ssl": false}, "all://ichigotv.net": {"verify_ssl": false}}
```

**3. .env per StreamV**
Per configurare il plugin StreamV è necessario configurare il relativo file .env. Vi rimando al repo del progetto per i dettagli.

📄 Esempio: ./mfp/.env
```bash
TMDB_API_KEY="661c66b02d7ac1df5c797f3c992bafa5"
MFP_PSW="testmm123"
MFP_URL="https://mfp-w3studio.ddns.net"
BOTHLINK=true
```

**4. .env per NoIp-Duc**
Per configurare correttamente il client DDNS (come noip-updater), è necessario un file .env contenente le credenziali e gli hostname o gruppi associati al tuo account No-IP.

📄 Esempio: ./noip-updater/.env
```bash
# File .env per il client DDNS con DDNS Key
NOIP_USERNAME=DdnsKeyUser
NOIP_PASSWORD=DdnsKeyPass
NOIP_HOSTNAMES=all.ddnskey.com
```

🛑 Attenzione alla sicurezza: imposta i permessi del file .env in modo che sia leggibile solo dal tuo utente, ad esempio:

```bash
chmod 600 ./noip_updater/.env
```

🔁 Ricorda di sostituire:
DdnsKeyUser → con l'indirizzo email del tuo account No-IP.
DdnsKeyPass → con la password associata al gruppo ddnskey.
NOIP_HOSTNAMES → con i tuoi hostname specifici separati da virgole (host1.ddns.net,host2.ddns.net) oppure all.ddnskey.com che vuol dire tutti gli hostname del gruppo.


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

### 1. Creazione dei tre hostname su No-IP

Assicurati di aver creato 3 hostname statici su [No-IP.com](https://www.noip.com/) che puntino al tuo IP pubblico (anche se dinamico, aggiornato tramite l’agent No-IP):

- `mammamia-<tuo-id>.ddns.net`
- `mfp-<tuo-id>.ddns.net`
- `streamv-<tuo-id>.ddns.net`

> 🔔 **Suggerimento:** Usa un identificativo unico (`<tuo-id>`) per evitare conflitti con altri utenti No-IP.

### 2. Port Forwarding sul router

Per permettere il corretto funzionamento di NPM e il rinnovo automatico dei certificati SSL:

- Reindirizza la porta **80 (HTTP)** del router verso la porta **8080** del PC/NAS dove gira NPM.
- Reindirizza la porta **443 (HTTPS)** del router verso la porta **8443** del PC/NAS.

> Questo consente a Let’s Encrypt di verificare il dominio e rilasciare i certificati.

### 3. Configurazione dei proxy host in Nginx Proxy Manager

Per ogni applicazione, crea un nuovo **Proxy Host** in NPM seguendo questi passi:
- **accedi ad http://<ip-tuo-server>:8181** (al primo accesso le credenziali di default sono **Email: admin@example.com Password: changeme**. Vi verrà chiesto di modificarle)
- **Dalla barra di menu selezionate **Hosts** → **Proxy Hosts** → **Add New Proxy**
- **Domain Names:** inserisci l’hostname corrispondente (es. `mammamia-<tuo-id>.ddns.net`)
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

  ```bash
  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;
  proxy_pass_request_headers on;
  ```
  ![image](https://github.com/user-attachments/assets/92fea31d-bb8c-49c5-a887-f3d5486c9f7f)

Ripeti questa configurazione per ciascuno dei tre hostname con la rispettiva porta (ad esempio, `mfp-<tuo-id>.ddns.net` → porta `8001`, ecc.).

### 4. Verifica e manutenzione

- Dopo aver configurato i proxy host, prova ad accedere agli URL pubblici via browser.
- NPM gestirà automaticamente il rinnovo dei certificati SSL.
- Assicurati che il tuo router sia sempre configurato correttamente per il port forwarding, specialmente dopo eventuali riavvii o aggiornamenti firmware.

---


