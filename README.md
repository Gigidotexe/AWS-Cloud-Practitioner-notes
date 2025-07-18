## 🟩 Amministrazione, Sicurezza & Governance

---

### Modello di responsabilità condivisa

**Che cosa significa davvero?**  Il Cloud come un condominio: AWS è il proprietario della struttura (muri, impianti, portineria), tu invece decidi chi può entrare nel tuo appartamento, quali porte restano chiuse e dove conservare i documenti sensibili.

* **IaaS** – Ricevi un appartamento vuoto (EC2, VPC). Devi arredarlo, mettere l’allarme e fare le pulizie (patch).
* **PaaS** – Trovi cucina e bagno installati (RDS, Fargate). Tu porti il cibo e scegli il menu (dati, schema).
* **SaaS** – Entri in hotel (WorkMail). L’albergo si occupa di tutto, tu solo delle valigie (config del servizio).

---

### IAM in pratica

*Solo chi serve, solo quando serve.*  <br>
Crea un **ruolo di revisione** che dura poche ore: l’auditor può guardare ma non toccare. <br>
Integra **MFA** per accedere e usa **tag** come “Dipartimento=Finanza” nelle policy **ABAC** per limitare in modo elegante i permessi.

---

### CloudTrail

Ogni azione (CLI, console, SDK) finisce in un log JSON dentro S3.  <br>
“chi ha eliminato quel bucket?”, puoi risalire a utente, orario, IP. <br>
Collegalo ad **Athena** per interrogare i log con SQL (es. “quante chiamate S3‑DeleteObject negli ultimi 30 giorni?”).

---

### Config

Config scatta fotografie delle risorse. <br>
Se ieri un bucket era privato e oggi è pubblico, la timeline lo evidenzia.  <br>
Scrivi una **regola di conformità** che segnala subito i bucket senza cifratura: avrai alert prima che diventi un problema.

---

### CAF

Il percorso cloud non è solo tecnologia. Le sei prospettive **Business, People, Governance, Platform, Security, Operations** ricordano che servono strategia, cultura, controlli finanziari, architettura solida, sicurezza capillare e operazioni continue.

---

<br>

## 🟧 Compute & Networking

---

### EC2 (Istanze)

Con EC2 parti da un modello (AMI) e scegli la famiglia: “T” per test economici, “C” quando conta la CPU, “R” per database mastodontici. <br>
Se il traffico varia, l’**Auto Scaling Group** aggiunge o rimuove istanze seguendo metriche CloudWatch.  <br>
Prenota capacità con **Reserved Instances** per servizi stabili o sfrutta gli **Spot** per carichi che sopportano interruzioni.

---

### Lambda (codice)

Carichi una funzione, definisci l’evento (un upload su S3, un messaggio su SQS, un cron EventBridge) e paghi solo i millisecondi di esecuzione. <br>
Con **Provisioned Concurrency** eviti i ritardi del cosiddetto “cold‑start”.

---

### Container (scatole portatili)

Con **ECS** basta un file JSON per descrivere un servizio, con **EKS** hai Kubernetes gestito: AWS mantiene il control‑plane, tu pensi alle applicazioni. <br>
Con **Fargate**  non crei nemmeno le VM sottostanti.

---

### VPC (rete)

Una VPC è  LAN privata nel cloud.  <br>
Definisci un **blocco CIDR** – per esempio `10.0.0.0/16` – e lo dividi in *subnet*: <br>
pubbliche (hanno route verso Internet Gateway) per i load balancer <br>
private per app e database.  <br>
Ogni subnet vive in una singola AZ.  <br>
Usa **Security Group** (firewall “a stato”) sulle risorse e **NACL** (firewall “lista” sulla subnet) per filtri grossolani.

Un **NAT Gateway** fornisce uscita NAT verso internet.  <br>
Aggiungi un **Gateway Endpoint** se vuoi parlare con S3 rimanendo nella rete AWS.  <br>
Per far dialogare poche VPC si usa il **Peering** e per tante VPC si usa il **Transit Gateway**.  <br>
**Direct Connect** per connessione diretta dal tuo data‑center in maniera totalmente privata (connessione diretta via fibra).

---

### Elastic Load Balancing (distributore di traffico)
Elastic Load Balancing riceve le richieste degli utenti e le smista automaticamente sulle istanze sane in background scalando in modo elastico quando il numero di connessioni cresce aggiungendo capacità senza intervento manuale. <br>

Offre quattro famiglie di bilanciatori: <br>
**ALB** (Application LB) – livello 7, routing per path/host, WebSocket, HTTP/2. <br>
**NLB** (Network LB) – livello 4, milioni di TPS, supporta TCP/UDP. <br>
**Gateway LB** – pensato per appliance di sicurezza (firewall, IDS). <br>
**CLB** (Classic) – legacy, usalo solo per workload storici. <br>

Può inviare log di accesso a S3 per analisi o fatturazione.

---

### Route 53 (DNS e routing)

Route 53 gestisce il DNS e distribuisce il traffico dove conviene: <br>
l’utente in Italia finisce sull’ELB (Elastic Load Balancer) di Milano grazie alla *latency policy*, ma se quel server non risponde il failover lo sposta su Dublino.

---

<br>

## 🔴 Storage & Content Delivery

---

### S3 (Bucket)

Amazon S3 (Simple Storage Service) è il servizio di storage (serverless) a oggetti di AWS. <br>
Permette di salvare file (oggetti) in bucket, accessibili via internet o rete privata, in modo scalabile e sicuro. <br>
È usato per archiviazione generale, backup, siti web statici, log, big data, ML e altro. <br>

Tipi di S3: <br>
**Standard** è la classe base. Dati usati spesso, alta disponibilità e replica su 3 AZ. Nessun vincolo di tempo o costi extra. <br>
**S3 Standard-IA (Infrequent Access)** per file consultati raramente. Più economica, ma con minimo 30 giorni di conservazione e costo per ogni accesso. <br>
**S3 One Zone-IA** come la Standard-IA ma i dati sono salvati in una sola AZ. Costa meno, ma è più rischiosa. <br>
**S3 Glacier** archiviazione a lungo termine. I file sono recuperabili in minuti o ore. Molto economica, adatta a backup mensili o log vecchi. <br>
**S3 Glacier Deep Archive** la più economica. Per dati archiviati per anni. Il recupero richiede ore e c’è un vincolo minimo di 180 giorni. <br>

---

### CloudFront (di fronte)

Una richiesta parte dall’utente, arriva alla Edge Location più vicina. <br>
Se l’oggetto è in cache, la risposta è quasi istantanea; altrimenti CloudFront lo pesca dall’origin e lo conserva. <br>
Usa i *signed URL* per distribuire contenuti privati senza esporre S3 al pubblico.

---

### Storage Gateway (porta)

AWS Storage Gateway è il servizio ibrido che fa da ponte fra gli ambienti on‑premises e lo storage cloud di AWS: <br>
attraverso una semplice appliance virtuale o hardware, le tue applicazioni locali continuano a parlare NFS, SMB o iSCSI mentre i dati vengono archiviati in S3, trasformati in snapshot EBS o conservati come nastri virtuali in Glacier, con cache locale per mantenere la bassa latenza e cifratura automatica in transito e a riposo.

---

<br>

## 🔵 Database

---

*ACID* (Atomicità, Coerenza, Isolamento, Durabilità) vale per RDS/Aurora: transazioni SQL, rollback affidabili.
*BASE* (Basic Available, Soft‑state, Eventual) descrive DynamoDB: altissima disponibilità, replica multi‑regione e coerenza forte opzionale.

* DynamoDB (serverless non relazionale): chiave‑partizione = velocità. Usa **On‑Demand** per carichi imprevedibili, **Provisioned + Autoscaling** per base stabile.  Le **Global Tables** replicano in più regioni senza replica custom.
* Aurora: storage distribuito in 6 copie su 3 AZ. La funzione **Backtrack** regredisce il DB di minuti/ore senza restore da backup.
* Neptune: pensato per grafi (relazioni molti‑a‑molti) con linguaggi Gremlin/SPARQL.
* ElastiCache: Redis in‑memory dà letture microsecondo. Con *Global Datastore* replichi le chiavi oltre oceano.

---

<br>

## 🟠 Analytics & Streaming

---

**Kinesis** serve per lo streaming in tempo reale e permette di raccogliere, trasformare e analizzare flussi di eventi (log, clickstream, metriche IoT, transazioni) con latenze di pochi secondi. <br>

**Kinesis Data Streams** è il cuore del servizio: tu definisci uno stream, suddiviso in shard (unità di throughput). I produttori inviano record con una chiave di partizione; i consumer li leggono in tempo reale o in replay (retention fino a 7 giorni, estendibile). <br> 
Se noti rallentamenti, aggiungi shard e il throughput raddoppia. <br>

**Kinesis Data Firehose** semplifica l’ingest: riceve i dati, li converte (opzionalmente con Lambda) e li consegna “serverless” a destinazioni come S3, OpenSearch, Redshift o Splunk.

---

<br>

## 🟠 Region, Avaiability Zone, Local Zone & Edge Location

---

**Regions**
Grandi aree geografiche isolate fra loro (p. es. eu‑central‑1 a Francoforte, us‑east‑1 in Virginia). <br>
Ogni Regione ospita più zone di disponibilità e rappresenta il confine entro cui vengono replicati i dati e applicate le policy di residenza. <br>

**Availability Zones**
Data center o campus di data center fisicamente separati ma connessi da reti private ad altissima velocità dentro la stessa Regione. <br>
Progettate per eliminare singoli punti di guasto: distribuendo istanze e servizi su più AZ ottieni alta disponibilità e fail‑over automatico. <br>

**Local Zones**
Estensioni di una Regione in città metropolitane non ancora coperte da un’intera Regione. Offrono calcolo, storage e servizi di database più vicini agli utenti finali (latenza a singola cifra di millisecondi) e si collegano alla “parent Region” per il piano di controllo e la replica dei dati.

Le **Region** sono città‑capoluogo: isolate l’una dall’altra, con leggi sui dati specifiche. <br>
Dentro ci sono le **Availability Zone** (quartieri alimentati da diverse centrali elettriche). <br>
Distribuendo le VM su due AZ ottieni un’app capace di sopravvivere a un incendio in uno dei data‑center.  <br>
**Local Zone** porta il cloud nelle metropoli, **Wavelength** sotto le antenne 5G, **Outposts** addirittura nel tuo rack. <br>

**Edge location** (cache)
Sono i nodi periferici della rete globale AWS in cui vengono erogati servizi a bassissima latenza, in particolare: <br>
Amazon CloudFront distribuisce contenuti statici e dinamici (file, API, video) servendoli dalla edge più vicina all’utente, riducendo tempi di download e alleggerendo l’origine. <br> 
AWS Shield e WAF operano a livello edge, bloccando DDoS e traffico malevolo prima che raggiunga la tua VPC. <br>
Route 53 e Global Accelerator sfruttano la stessa infrastruttura edge per risoluzione DNS rapida e instradamento ottimizzato del traffico. <br>

---

<br>

## 🟦 Piani di supporto semplificati

---

Piano **Basicr** è incluso gratuitamente con ogni account. Offre l’accesso all’AWS Management Console, alla documentazione ufficiale, ai white‑paper e ai forum pubblici, senza però permettere l’apertura di casi tecnici a un team dedicato. È l’opzione adatta a sperimentazioni iniziali o progetti personali che non necessitano di assistenza diretta da parte di ingegneri AWS. <br>

Piano **Developer** rappresenta il primo gradino a pagamento. Consente di inviare ticket illimitati per problematiche di “how‑to” e domande tecniche, con un tempo di prima risposta entro 24 ore per i casi di bassa severità e entro 12 ore per quelli normali. Il contatto avviene esclusivamente via console e la consulenza architetturale è limitata, ma è perfetto per ambienti di sviluppo e test dove un downtime non comporta impatti immediati sul business. <br> 

Piano **Business** è pensato per applicazioni in produzione. Garantisce supporto 24/7 via console, telefono e chat, con tempi di risposta fino a un’ora per i casi critici (severità Urgent) e accesso a tutti i controlli avanzati di Trusted Advisor. Include l’assistenza di ingegneri del supporto che offrono best practice, guidance sulle architetture, monitoraggio proattivo dei limiti di servizio e, se necessario, escalation immediata ai team interni AWS. <br>

Piano **Enterprise On‑Ramp** colma il divario tra Business ed Enterprise completo. Prevede una prima risposta in 30 minuti per i casi Urgent, un pool di Senior Cloud Support Engineers specializzati e revisioni mensili dell’architettura dei workload critici. È ideale per organizzazioni che stanno passando da un singolo progetto a più workload di produzione ad alta disponibilità ma non richiedono ancora il livello di coinvolgimento continuo previsto dal piano Enterprise. <br>

Piano **Enterprise** è il livello più avanzato. Ogni cliente riceve un Technical Account Manager (TAM) dedicato che agisce da consulente di fiducia, pianificando revisioni periodiche, ottimizzazioni dei costi e prove di resilienza. I tempi di risposta scendono fino a quindici minuti per le emergenze a impatto aziendale e sono inclusi servizi proattivi come Infrastructure Event Management per l’assistenza prima, durante e dopo lanci critici o migrazioni su larga scala. <br>

---

<br>

## 🔴 Backup & Ripristino

---

**AWS Backup** centralizza snapshot e retention; **Elastic Disaster Recovery** replica server completi su AWS per recovery in pochi minuti. 

---
