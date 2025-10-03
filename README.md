# NTP_synchronization

# Sincronizzazione Oraria NTP tra Ubuntu (Master) e Raspberry Pi (Client) su Newtwork locale 

Guida passo passo per configurare un PC Ubuntu come server NTP (master) e una Raspberry Pi come client NTP sulla stessa rete locale, senza (o con) necessità che i client vadano su Internet.

---

## 🖥️ 1. CONFIGURAZIONE DEL SERVER NTP SU UBUNTU (MASTER)


### 1.1 Installazione del server NTP

```bash
sudo apt update
sudo apt install ntp
```

## 1.2 Configurazione del server

Apri il file di configurazione:

```bash
sudo nano /etc/ntp.conf
```

Le righe pool predefinite possono essere lasciate (così il tuo server si sincronizza con l’orario da Internet), oppure puoi rimuoverle/commentarle se vuoi un orologio "isolato" solo locale.

Aggiungi:

`restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap`

modificando la subnet (192.168.1.0) se necessario 

Assicurati che sia presente anche questa riga (consigliata come prima restrict):

`restrict default kod nomodify notrap nopeer noquery`

Salva e chiudi il file.

## 1.3 Riavvia il servizio

```bash
sudo systemctl restart ntp
```

## 1.4 Trova l’IP del server Ubuntu

```bash
hostname -I
```
Annota l’indirizzo IP mostrato, ad esempio 192.168.1.10.

## 1.5 Verifica lo stato di NTP

```bash
ntpq -p
```

Se non viene messo nemmeno un pool nella configurazione otterrai: `No association ID's returned` questo succede quando il demone ntpd non ha nessun server/peer configurato a cui associarsi per prendere il tempo e usa il sul suo hardware clock in questo caso.

In caso invece che siano stati utilizzati pool dovresti vedere un elenco di server/pool.


## 🍓 2. CONFIGURAZIONE DELLA RASPBERRY PI COME CLIENT NTP


## 2.1 Installazione di NTP

```bash
sudo apt update
sudo apt install ntp
```

## 2.2 Configurazione del client

Apri il file di configurazione della Raspberry:

```bash
sudo nano /etc/ntp.conf
```

In alcuni casi è presente in `/etc/ntpsec/ntp.conf`

Se vuoi, puoi commentare (con #) o rimuovere tutte le altre righe pool.

Aggiungi: 

`server 192.168.1.10 iburst`

Sostituisci 192.168.1.10 con l’IP effettivo ottenuto dal comando hostname -I sul pc master (Ubuntu)

Salva e chiudi il file.

## 2.3 Riavvia il servizio

```bash
sudo systemctl restart ntp
```

## 2.4 Verifica la sincronizzazione

```bash
ntpq -p
```
Dovresti vedere una riga simile a questa con iP del master dove hai installato NTP:

remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
*192.168.1.10   ...       2 u   50   64  377    0.612   -0.325   0.008



## 🔒 Note importanti

Firewall: Assicurarsi che sul master Ubuntu sia aperta la porta UDP 123 se è abilitato un firewall:

```bash
sudo ufw allow 123/udp
```

Forzare sincronizzazione manuale (sul client Raspberry, opzionale):

```bash
sudo ntpd -gq
```

Verifica sempre con:

```bash
ntpq -p
```
