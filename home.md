# Cybersecurity Project

## Introduzione
In questo report, analizzeremo un'esercitazione pratica di pentesting in cui abbiamo condotto una serie di attacchi controllati all'interno di un ambiente di test. Abbiamo utilizzato Metasploitable 3 come macchina bersaglio e Kali Linux come macchina attaccante, entrambe configurate per operare sulla stessa rete. Questa configurazione ci ha permesso di simulare un ambiente realistico in cui un attaccante può tentare di compromettere un sistema target.
Lo scopo principale di questa simulazione è accedere al sistema target e rubare quante più credenziali salvate possibile.

L'attacco è stato eseguito seguendo diverse fasi chiave:

1)**Scansione Iniziale della Rete**: In questa fase, abbiamo eseguito una scansione degli indirizzi IP della rete utilizzando strumenti come Nmap. L'obiettivo era identificare le macchine attive e raccogliere informazioni sui sistemi operativi in esecuzione e sui servizi disponibili.

2)**Exploit della Porta 445**: Identificata una macchina con la porta 445 aperta, abbiamo sfruttato un exploit noto per ottenere le credenziali smb. Questo è stato realizzato attraverso un attacco di forza bruta e l'utilizzo di un dizionario di password.

3)**Determinazione utenti locali Sam**: Utilizzo le credenziali SMB, utilizzo un' altro exploit per determinare gli utenti locali.

4)**Esecuzione di Comandi da Remoto**: Utilizzando le credenziali ottenute, abbiamo sfruttato un altro exploit per eseguire comandi da remoto e aprire una sessione Meterpreter, un potente payload di Metasploit che ci ha fornito accesso remoto al sistema.

5)**Raccolta di Hash delle Password**: Una volta ottenuto l'accesso, abbiamo utilizzato l'estensione hashdump di Meterpreter per rubare gli hash delle password locali del sistema.

6)**Utilizzo di Mimikatz**: Successivamente, abbiamo impiegato Mimikatz, uno strumento avanzato di post-exploitation, per estrarre ulteriori dati sensibili e credenziali dalla memoria del sistema compromesso.

7)**Ottenimento delle Credenziali di un Server**: Infine, grazie ai dati raccolti, siamo riusciti a ottenere le credenziali di un server.

## Scansione della rete

### a)Identificazione dell'Indirizzo IP e della Netmask
+ Aprire il terminale sulla macchina attaccante (Kali Linux).
+ Eseguire il comando ```ifconfig``` per determinare l'indirizzo IP e la netmask della macchina.
+ Risultato: L'indirizzo IP è 10.0.2.19 e la netmask è 255.255.255.0.

### b)Scansione degli Indirizzi IP nella Rete
+ Utilizzare Nmap per identificare le macchine attive nella rete.
+ Comando: ```nmap 10.0.2.0/24```.
+ Risultato: Viene identificato un host attivo con l'indirizzo IP 10.0.2.4.

### c)Scansione Completa della Macchina Target
+ Utilizzare Nmap per eseguire una scansione completa dell'host identificato.
+ Comando:```nmap -v -sS -sV -T4 -A 10.0.2.4.```
  - v: Modalità verbosa.
  - sS: (SYN stealth scan): esegue una scansione SYN stealth, che è più silenziosa e meno invasiva.
  - T4: Alta velocità di scansione.
  - A: Abilita la rilevazione del sistema operativo, versioni, script di scansione e traceroute.
  - sV (version detection): tenta di identificare la versione dei servizi in esecuzione sulle porte aperte
+ Risultato: La scansione fornisce dettagli sui servizi in esecuzione, le versioni, e ulteriori informazioni sul sistema operativo della macchina target.

##### Riferimenti
https://www.redhat.com/sysadmin/quick-nmap-inventory

## Exploit della porta 445
Adesso andremo a creare un dizionario che utilizzeremo per i nomi utente e password. Successivamente, utilizzeremo un exploit per tentare il brute force sulla porta 445.

### a)Creazione del dizionario
Utilizza il comando Kali cewl per costruire un nuovo elenco di parole dal contenuto della pagina di configurazione di metasploitable3: 
```cewl -d 0 -w metasploitable3.txt https://github.com/rapid7/metasploitable3/wiki/Configuration``` (l'opzione -d 0 indica che i collegamenti ipertestuali non devono essere seguiti)

### b)Esexuzione Exploit
**1)Aprire Metasploit**: Aprire il terminale e digitare il comando per avviare Metasploit:```msfconsole```.

**2)Caricare il modulo SMB Login Scanner**: Una volta aperto Metasploit, caricare il modulo per eseguire un attacco brute force sull'SMB (porta 445): ```use auxiliary/scanner/smb/smb_login```.

**3)Impostare l'host bersaglio (RHOSTS)**: Specificare l'indirizzo IP del sistema bersaglio: ```set RHOSTS 10.0.2.4```

**4)Impostare il file dei nomi utente**: Specificare il percorso del file che contiene i nomi utente: ```set USER_FILE /home/kali/metasploitable3.txt```.

**5)Impostare il file delle password**: Specificare il percorso del file che contiene le password: ```set PASS_FILE /home/kali/metasploitable3.txt.```

**6)Eseguire l'exploit**: Avviare l'attacco brute force utilizzando il comando: ```exploit```.

**RISULTATO**=Abbiamo ottenuto delle credenziali di accesso SMB.


##### Riferimenti:
+ https://www.geeksforgeeks.org/cewl-tool-creating-custom-wordlists-tool-in-kali-linux/
+ https://www.hackingarticles.in/password-crackingsmb/


## Determinazione utenti locali Sam
**1)Aprire Metasploit**: Aprire il terminale e digitare il comando per avviare Metasploit:```msfconsole```.

**2)Caricare il modulo SMB enumusers Scanner**: Una volta aperto Metasploit, caricare il modulo per eseguire una scansione degli utenti locali Sam: ```ausiliario/scanner/smb/smb_enumusers```.

**3)Impostare l'host bersaglio (RHOSTS)**: Specificare l'indirizzo IP del sistema bersaglio: ```set RHOSTS 10.0.2.4```

**4)Impostare il nome utente**: vagrant.

**5)Impostare la password**: vagrant

**6)Eseguire l'exploit**: Avviare l'attacco brute force utilizzando il comando: ```exploit```.

**RISULTATO**=Abbiamo ottenuto gli utenti locali SAM.

#### Unordered
A list of items in which the order of the items does not explicitly matter.

You may use any of the following symbols to denote bullets for each list item:

```markdown
* valid bullet
- valid bullet
+ valid bullet
```

For example

```markdown
+ Lorem ipsum dolor sit amet
+ Consectetur adipiscing elit
+ Integer molestie lorem at massa
+ Facilisis in pretium nisl aliquet
+ Nulla volutpat aliquam velit
  - Phasellus iaculis neque
  - Purus sodales ultricies
  - Vestibulum laoreet porttitor sem
  - Ac tristique libero volutpat at
+ Faucibus porta lacus fringilla vel
+ Aenean sit amet erat nunc
+ Eget porttitor lorem
```
Renders to:

+ Lorem ipsum dolor sit amet
+ Consectetur adipiscing elit
+ Integer molestie lorem at massa
+ Facilisis in pretium nisl aliquet
+ Nulla volutpat aliquam velit
  - Phasellus iaculis neque
  - Purus sodales ultricies
  - Vestibulum laoreet porttitor sem
  - Ac tristique libero volutpat at
+ Faucibus porta lacus fringilla vel
+ Aenean sit amet erat nunc
+ Eget porttitor lorem

And this HTML

```html
<ul>
  <li>Lorem ipsum dolor sit amet</li>
  <li>Consectetur adipiscing elit</li>
  <li>Integer molestie lorem at massa</li>
  <li>Facilisis in pretium nisl aliquet</li>
  <li>Nulla volutpat aliquam velit
    <ul>
      <li>Phasellus iaculis neque</li>
      <li>Purus sodales ultricies</li>
      <li>Vestibulum laoreet porttitor sem</li>
      <li>Ac tristique libero volutpat at</li>
    </ul>
  </li>
  <li>Faucibus porta lacus fringilla vel</li>
  <li>Aenean sit amet erat nunc</li>
  <li>Eget porttitor lorem</li>
</ul>
```

#### Ordered

A list of items in which the order of items does explicitly matter.

```markdown
1. Lorem ipsum dolor sit amet
2. Consectetur adipiscing elit
3. Integer molestie lorem at massa
4. Facilisis in pretium nisl aliquet
5. Nulla volutpat aliquam velit
6. Faucibus porta lacus fringilla vel
7. Aenean sit amet erat nunc
8. Eget porttitor lorem
```
Renders to:

1. Lorem ipsum dolor sit amet
2. Consectetur adipiscing elit
3. Integer molestie lorem at massa
4. Facilisis in pretium nisl aliquet
5. Nulla volutpat aliquam velit
6. Faucibus porta lacus fringilla vel
7. Aenean sit amet erat nunc
8. Eget porttitor lorem

And this HTML:

```html
<ol>
  <li>Lorem ipsum dolor sit amet</li>
  <li>Consectetur adipiscing elit</li>
  <li>Integer molestie lorem at massa</li>
  <li>Facilisis in pretium nisl aliquet</li>
  <li>Nulla volutpat aliquam velit</li>
  <li>Faucibus porta lacus fringilla vel</li>
  <li>Aenean sit amet erat nunc</li>
  <li>Eget porttitor lorem</li>
</ol>
```

**TIP**: If you just use `1.` for each number, Markdown will automatically number each item. For example:

```markdown
1. Lorem ipsum dolor sit amet
1. Consectetur adipiscing elit
1. Integer molestie lorem at massa
1. Facilisis in pretium nisl aliquet
1. Nulla volutpat aliquam velit
1. Faucibus porta lacus fringilla vel
1. Aenean sit amet erat nunc
1. Eget porttitor lorem
```

Renders to:

1. Lorem ipsum dolor sit amet
2. Consectetur adipiscing elit
3. Integer molestie lorem at massa
4. Facilisis in pretium nisl aliquet
5. Nulla volutpat aliquam velit
6. Faucibus porta lacus fringilla vel
7. Aenean sit amet erat nunc
8. Eget porttitor lorem

### Code

#### Inline code
Wrap inline snippets of code with `` ` ``.

```markdown
In this example, `<section></section>` should be wrapped as **code**.
```

Renders to:

In this example, `<section></section>` should be wrapped with **code**.

HTML:

```html
<p>In this example, <code>&lt;section&gt;&lt;/section&gt;</code> should be wrapped with <strong>code</strong>.</p>
```

#### Indented code

Or indent several lines of code by at least four spaces, as in:

<pre>
  // Some comments
  line 1 of code
  line 2 of code
  line 3 of code
</pre>

Renders to:

    // Some comments
    line 1 of code
    line 2 of code
    line 3 of code

HTML:

```html
<pre>
  <code>
    // Some comments
    line 1 of code
    line 2 of code
    line 3 of code
  </code>
</pre>
```


#### Block code "fences"

Use "fences"  ```` ``` ```` to block in multiple lines of code.

<pre>
``` markup
Sample text here...
```
</pre>


```
Sample text here...
```

HTML:

```html
<pre>
  <code>Sample text here...</code>
</pre>
```

#### Syntax highlighting

GFM, or "GitHub Flavored Markdown" also supports syntax highlighting. To activate it, simply add the file extension of the language you want to use directly after the first code "fence", ` ```js `, and syntax highlighting will automatically be applied in the rendered HTML. For example, to apply syntax highlighting to JavaScript code:

<pre>
```js
grunt.initConfig({
  assemble: {
    options: {
      assets: 'docs/assets',
      data: 'src/data/*.{json,yml}',
      helpers: 'src/custom-helpers.js',
      partials: ['src/partials/**/*.{hbs,md}']
    },
    pages: {
      options: {
        layout: 'default.hbs'
      },
      files: {
        './': ['src/templates/pages/index.hbs']
      }
    }
  }
};
```
</pre>

Renders to:

```js
grunt.initConfig({
  assemble: {
    options: {
      assets: 'docs/assets',
      data: 'src/data/*.{json,yml}',
      helpers: 'src/custom-helpers.js',
      partials: ['src/partials/**/*.{hbs,md}']
    },
    pages: {
      options: {
        layout: 'default.hbs'
      },
      files: {
        './': ['src/templates/pages/index.hbs']
      }
    }
  }
};
```

### Tables
Tables are created by adding pipes as dividers between each cell, and by adding a line of dashes (also separated by bars) beneath the header. Note that the pipes do not need to be vertically aligned.


```markdown
| Option | Description |
| ------ | ----------- |
| data   | path to data files to supply the data that will be passed into templates. |
| engine | engine to be used for processing templates. Handlebars is the default. |
| ext    | extension to be used for dest files. |
```

Renders to:

| Option | Description |
| ------ | ----------- |
| data   | path to data files to supply the data that will be passed into templates. |
| engine | engine to be used for processing templates. Handlebars is the default. |
| ext    | extension to be used for dest files. |

And this HTML:

```html
<table>
  <tr>
    <th>Option</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>data</td>
    <td>path to data files to supply the data that will be passed into templates.</td>
  </tr>
  <tr>
    <td>engine</td>
    <td>engine to be used for processing templates. Handlebars is the default.</td>
  </tr>
  <tr>
    <td>ext</td>
    <td>extension to be used for dest files.</td>
  </tr>
</table>
```

### Right aligned text

Adding a colon on the right side of the dashes below any heading will right align text for that column.

```markdown
| Option | Description |
| ------:| -----------:|
| data   | path to data files to supply the data that will be passed into templates. |
| engine | engine to be used for processing templates. Handlebars is the default. |
| ext    | extension to be used for dest files. |
```

| Option | Description |
| ------:| -----------:|
| data   | path to data files to supply the data that will be passed into templates. |
| engine | engine to be used for processing templates. Handlebars is the default. |
| ext    | extension to be used for dest files. |

### Links

#### Basic link

```markdown
[Assemble](http://assemble.io)
```

Renders to (hover over the link, there is no tooltip):

[Assemble](http://assemble.io)

HTML:

```html
<a href="http://assemble.io">Assemble</a>
```


#### Add a title

```markdown
[Upstage](https://github.com/upstage/ "Visit Upstage!")
```

Renders to (hover over the link, there should be a tooltip):

[Upstage](https://github.com/upstage/ "Visit Upstage!")

HTML:

```html
<a href="https://github.com/upstage/" title="Visit Upstage!">Upstage</a>
```

#### Named Anchors

Named anchors enable you to jump to the specified anchor point on the same page. For example, each of these chapters:

```markdown
# Table of Contents
  * [Chapter 1](#chapter-1)
  * [Chapter 2](#chapter-2)
  * [Chapter 3](#chapter-3)
```
will jump to these sections:

```markdown
### Chapter 1 <a id="chapter-1"></a>
Content for chapter one.

### Chapter 2 <a id="chapter-2"></a>
Content for chapter one.

### Chapter 3 <a id="chapter-3"></a>
Content for chapter one.
```
**NOTE** that specific placement of the anchor tag seems to be arbitrary. They are placed inline here since it seems to be unobtrusive, and it works.

### Images
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
