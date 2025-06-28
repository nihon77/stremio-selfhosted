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

### 🌍 Accesso remoto con IP dinamico
- Un **IP pubblico** (anche dinamico va bene).
- Un account su [**No-IP.com**](https://www.noip.com/) per ottenere **hostname statici gratuiti**.

### 🔐 Hostname da creare su no-ip
- `mammamia-<inserite qualcosa se no magari esiste già>.ddns.net`
- `mfp-<inserite qualcosa se no magari esiste già>.ddns.net`
- `streamv-<inserite qualcosa se no magari esiste già>.ddns.net`

Questi hostname punteranno sempre al tuo NAS anche se il tuo IP cambia.  
Il tutto è possibile installando un piccolo agente (Dynamic DNS client) che aggiorna automaticamente il record DNS.

---

## 🔧 Componenti del progetto

| Servizio           | Porta interna | Descrizione                              |
|--------------------|---------------|------------------------------------------|
| **Mammamia**       | 5000          | Plugin personalizzato per Stremio        |
| **Media Flow Proxy (MFP)** | 3000   | Proxy per streaming video                |
| **StreamV**        | 4000          | Web player personalizzato (opzionale)    |
| **Nginx Proxy Manager** | 80/443/81 | Reverse proxy + certificati Let's Encrypt |
| **No-IP DUC (Docker)** | —         | Aggiorna il DNS dinamicamente            |

---

### 📦 Docker + Docker Compose

> Questo progetto usa **Docker** per semplificare l’installazione e l’isolamento dei servizi.

#### 📥 Installazione Docker

```bash
# 🔁 Rimuovi eventuali versioni precedenti
sudo apt remove docker docker-engine docker.io containerd runc

# 🔄 Aggiorna l’elenco dei pacchetti
sudo apt install docker-compose-plugin

# 📦 Installa i pacchetti richiesti per aggiungere il repository Docker
sudo usermod -aG docker $USER

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
---