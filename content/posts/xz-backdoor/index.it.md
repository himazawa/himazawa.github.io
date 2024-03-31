---
title: "La backdoor di xz dalla prospettiva di un Security Engineer"
subtitle: "L'ennesimo attacco alla supply chain"
date: 2024-03-30T19:49:24+01:00
draft: false
featuredImage: xz.png
tags: [backdoor, CVE-2024-3094, xz, liblzma, supply-chain, security-engineering]
---

Come probablilmente già sai, `xz` è stato compromesso.
Per i non addetti ai lavori `xz` è una libreria per la compressione dei dati molto utilizzata sopratutto su Linux.

Il pacchetto è stato usato come entrypoint per l'injection di codice malevolo in `sshd`, modificandone il flusso di autenticazione.

Questa vulnerabilità, introdotta deliberatamente attraverso una backdoor, è conosciuta come [CVE-2024-3094](https://nvd.nist.gov/vuln/detail/CVE-2024-3094).
<!--more--> 

{{< admonition type=tip title="Note" open=true >}}
La situazione si sta ancora evolvendo, maggiori dettagli emergeranno nel prossimo futuro e aggiornerò il post di conseguenza.
{{< /admonition >}}

Si tratta probabilmente di un'operazione a tutti gli effetti vista la metodologia e la durata (quasi due anni), ma non sono la persona giusta per parlare di attributions,  OpSec e Threat Actors.
Nella sezione "Risorse" in fondo alla pagina ci sono link che riportano ad analisi più dettagliate.

La situazione non sembra rosea quindi sto provando a scrivere questo blogpost in modo tale da fare un pò di chiarezza.

Non voglio trattare aspetti troppo tecnici della compromissione ma voglio guardare alla issue dalla prospettiva di un Security Engineer, riassumendo cosa è andato storto e quali sono le possibili remediation a questo problema.

## Timeline
{{< admonition type=tip title="Note" open=true >}}
Controlla la sezione Risorse per i link ad una timeline più dettagliata
{{< /admonition >}}

- __2023__:
    - Un nuovo maintainer appare sul progetto `xz`
    - A new maintainer shows up in the `xz` project
- __29 Mar 2024__: 
    - Andres Freund invia un'email alla mailing list oss-security che riguarda una backdoor in `xz/liblzma`.
    Stava ottimizzando la sua infrastruttura e si è accorto che ssh era "sospettosamente" lento (parliamo di 400ms di differenza, difficilmente notabili a meno che non si stia facendo micro ottimizzazione). Qualche debug dopo si è accorto che la problmatica di performace era probabilmente causata dal codice della backdoor. L'analisi iniziale è stata fatta con l'aiuto di Florian Weimer.

    - Le distribuzioni impattate (che quindi contenevano il pacchetto `xz`) hanno cominciato a rilasciare patch di downgrade a una versione precedente

- __30 Mar 2024__: 
    - GitHub ha bloccato l'accesso al repository e ha sospeso l'account di entrambi i maintainer di `xz`

    - Un [comunicato ufficiale](https://tukaani.org/xz-backdoor/) è stato rilasciato dal maintainer del progetto


## Componenti impattate

L'estensione di questo breach è ancora poco chiaro di seguito è riportata una (parziale) lista dei componenti che contiengono la versione malevola di `xz`:

Distribuzioni:
- Arch
- [Debian Sid](https://security-tracker.debian.org/tracker/CVE-2024-3094)
- Gentoo
- [Fedora 40](https://www.redhat.com/en/blog/urgent-security-alert-fedora-41-and-rawhide-users)
- Manjaro Testing
- Parabola
- NixOS Unstable
- Slackware
- [SUSE Tumbleweed](https://news.opensuse.org/2024/03/29/xz-backdoor/)
- [Kali Linux](https://infosec.exchange/@kalilinux/112180505434870941)

Il pacchetto con la backdoor è inoltre contenuto nei seguenti package manager:
- Homebrew
- MacPorts
- pkgsrc

Al momento sappiamo che ci sono dei controlli nel codice della backdoor che riguardano [specificatamente le istanze di Linux x86_64/amd64](https://gist.github.com/thesamesam/223949d5a074ebc3dce9ee78baad9e27#design) quindi il numero dei target effettivi potrebbe essere ridotto (relativamente) ma la situazione è poco chiara, non raccomanderei di tenere un pacchetto compromesso sul sistema.

## Considerazioni

### Il comportamento di GitHub

Le ragioni dietro il blocco dei repository di `xz` è ancora un mistero per me, specialmente sapendo che con il codice disponibile è possibile anche fare ulteriori analisi e avere uno scenario più chiaro della compromissione.

Il blocco dell'accesso ai sortgenti di `xz` è un fattore che rallenterà l'arrivo delle analisi, che è un male  in una situazione time-critical come questa.


### I downgrade

La patch strategy per praticamente tutti è stata di forzare il downgrade dalle versioni `5.6.0`-`5.6.1` ad una precedente.
Qualcuno(homebrew, per esempio), ha forzato il downgrade alla versione `5.4.6`.

Questo è interessante perchè sicurametne sappiamo che c'è una backdoor sulle versioni `5.6.0`-`5.6.1`, ma sappiamo anche che l'attaccante ha lavorato sul repository per più di due anni.

La versione `5.4.6`, ad esempio, è stata anch'essa compilata dall'attaccante e non dovrebbe essere considerata safe.
Ancora una volta, grazie GitHub per aver bloccato l'accesso ai sorgenti e contribuito a far capire ancora meno la situazione.

## Come si previene questo fenomeno?


Come ho detto all'inizio del post, non voglio scendere troppo in profondità con l'analisi tecnica della backdoor, per due ragioni principali: con l'accesso al codice sorgente bloccato [solo un archivio](https://github.com/xz-mirror/xz) (al momento della scrittura) è disponibile, le informazioni sono incomplete e non vorrei davvero reversare il binario di `xz`.

Inoltre, e questo è il motivo più importante, persone con più conoscienze di me sul modo di operare dei Threat Actors ci stanno già lavorando.
Non appena verranno pubblicati i primi articoli tecnici saranno disponibili nella sezione "Risorse", in fondo alla pagina

Una cosa che però posso fare qui è mostrare il punto di vista di un Security Engineer sul problema, come avrei mitigato la situazione e quali step sono andati storti.

{{< admonition type=tip title="Note" open=true >}}
TL;DR: non esiste una reale soluzione al problema
{{< /admonition >}}


### Problemi di fiducia
![](https://imgs.xkcd.com/comics/dependency_2x.png)

`xz` è un software mantenuto (fino al 2023) da una singola persona. Successivamente un altro maintainer è arrivato ma sfortunatamente per noi, è la stessa persona che ha inviato la backdoor upstream. Questo ovviamente va in conflitto col fatto che `xz` sia un pacchetto incredbilimente popolare in tantissime distribuzioni e parte delle dipednenze di numerosi software.

Probabilmente questo è stato visto dall'attaccante come una miniera d'oro dal momento che risultava facile riuscire a farsi affidare il ruolo di maintainer e pubblicare codice malevolo.

Dovendo fare affidamento a  codice di terze parti per la supply chain, è necessario avere fiducia in qualcuno a un certo punto.
Quando si parla di Supply Chain security, le raccomandazioni sono sempre le stesse: pin degli hash e verifica delle firme.

Questo funziona fino a quando ci troviamo in scenari dove un attaccante ha compromesso la CICD della dipedenza e invia build malevole, un account è stato compromesso etc.

Ma cosa si può fare se, tutto a un tratto, un maintainer fidato si rivela malevolo?

Da utente standard, a meno che tu non voglia (e sia capace) di fare code review a ogni singolo commit di ogni singolo pezzo di software con cui interagisce il tuo sistema operativo: più o meno nulla.

Dall'altro lato, i developers e gli owner dei repsitory dovrebbero aumentare i controlli della loro supply chain includendo metriche più strette al fine di escludere pacchetti con alto rischio.

Una delle cose più ricorrenti che si sentono quando si parla si Open Source e Security sono le persone che credono che, dal momento che il codice sorgente è disponibile, magicamente risulta sicuro.

Uno dei fattori spesso trascurato è l'assunzione che avere accesso al sorgente si traduce automaticamente in un pool maggiore di occhi che cercano problemi e vulnerabilità.

L'efficacia di queste review dipende principamente dal livello di coinvolgimento della community e dall'esperienza delle persone che poi quel codice effettivamente lo leggono, e di solito non è tanto. Molti progetti ricevono pochissima attenzione e solo pochi contribuiscono attivamente ai processi di review. Come risultato, le vulnerabilità (intenzionali o meno) possono passare inosservate per lunghi periodi di tempo, creando un rischio significante per gli utenti.

Ogni volta che una discussione come questa si riapre, mi torna in mente il ["Mese di Kali" di InfosectCBR](https://blog.infosectcbr.com.au/2018/11/pitfalls-using-strcat.html), dove [Silvio Cesare](https://twitter.com/silviocesare) ha passato un mese a trovare e pubblicare vulnerabilità nei software contenuti nei repository di Kali Linux.


Di seguito riporto una lista (parziale) di fattori che potrebbero contribuire a ridurre il rischio:

### Statistiche di GitHub

Giusto per essere chiaro fin dall'inizio: No, non si può fare affidamento su queste metriche.

Esiste un mercato sulla compravendita di statistiche di GitHub come stelle, fork etc.
Puoi trovare un buon articolo qui: https://dagster.io/blog/fake-stars

### Engagement della community
Valutare sempre la dimensione e l'engagement della community che circonda il progetto.

Una community grande ed attiva può fornire occhi aggiuntivi sulle code review, i bug report etc.

`xz` aveva letteralmente 2 maintainers e uno dei due è risultato essere l'attore malevolo.

### Fondi e supporto
Condsiderare sempre l'aspetto economico del progetto e da chi sta venendo finanziato. I progetti con dei fondi dedicati tendono ad avere risorse aggiuntive per i security audit e la manutenzione del codice e soprattutto è meno probabile che vengano abbandonati.

{{< admonition type=tip title="Tip" open=true >}}
Ricorda: si vuole fare affidamento su quel codice per l'intera settimana, non solo durante il tempo libero del maintainer. I progetti con un buon supporto finanziario è molto più probabile divengano lavori a tempo pieno che solo hobby da portare avanti sporadicamente.
{{< /admonition >}}

### SDLC
Una buona porzione della valutazione dovrebbe concentrarsi sul ciclo si sviluppo del software per assicurarsi che i gate (di sicurezza e qualità più in generale) siano implementati correttamente, le pull request abbiano bisogno di approvazione e ci siano delle pratiche sane che non permettano ad un singolo contributor di inviare codice malevolo senza approvazione.

Inoltre è necessario tenere in cosiderazione che siamo umani e commettiamo errori. Passare una code review non significa automaticamente che il codice sia sicuro, come ho detto prima: non esiste una vera soluzione al problema, solo modi per abbassare la probabilità che le cose brutte accadano.

### Enterprise vs Individual
Questo è un argomento abbastanza controverso perchè ci sono progetti mantenuti da persone individuali che sono ben strutturati.
Di solito però, utilizzando cdice prodotto da (grandi) aziende renderà più probabile l'avere delle best practice di sviluppo, avere un continuo stream di fondi per mantenere il progetto in piedi e soprattuto difficilmente una grande azienda metterebbe una backdoor di proposito nel proprio codice. Di nuovo, questo aumenta solo le probabilità, non va preso come concetto assoluto ;)

### Controlli Ricorsivi
Il progetto che stai includendo probabilmente avrà anch'esso delle dipendenze, assicurati che i maintainers applichino a loro volta lo stesso scrutinio sulla loro supply chain per evitare compromissioni indirette.

## Risorse
    
- OSS-Security List: https://www.openwall.com/lists/oss-security/2024/03/29/4
- Comprehensive timeline: https://boehs.org/node/everything-i-know-about-the-xz-backdoor
- Compromise link roundup: https://shellsharks.com/xz-compromise-link-roundup
- Obfuscation Analysis: https://gynvael.coldwind.pl/?lang=en&id=782
