---
title: "Security Theatre? Direi quasi Security Circus"
date: 2023-02-13T20:20:00+01:00
draft: false
authors: ["himazawa"]
tags: [security theatre, infosec, rants]
categories: [general-knowledge]
featuredImage: cargocult.jpg
---

Lavorando nel campo, ho visto molte organizzazioni investire una quantita' significativa di tempo e risorse (che si traduce in persone e soldi), in misure che garantiscono poca se non zero, reale sicurezza.
<!--more--> 
Ho assistito a organizzazioni che si esibiscono in un eterno clown show nel nome della security.

Questo fenomeno e' comunemente chiamato "security theatre".

## Cosa e' il Security Theatre

Il security theatre, nella sua forma piu' semplice, e' una grande illusione. E' l'insieme di misure di sicurezza messe in piedi non perche' prevengano realmente i data breach, ma solo perche' sulla carta sembrano buoni, tranquillizzano gli stakeholders e fanno sentire tutti come se stessero davvero facendo qualcosa di utile per proteggersi.

[Lo senti? Lo senti l'odore? _compliance_ figliolo, non c'e' nient'altro al mondo che odora cosi'. ](https://www.youtube.com/watch?v=MzQPTdDwtVk).

Come esempio, prendiamo il requisito di avere password complesse che devono essere regolarmente cambiate. Questo e' il classico esempio di security theatre.

Per quanto possa sembrare una buona fa molto poco per prevenire un data breach. E non dimentichiamoci il downside principale: la cosiddetta "password fatigue", che porta i dipendenti a usare password che sono piu' facili da ricordare o sempli variazioni di quelle usate in precedenza. Del resto c'e' un motivo (molti in realta') se [Microsoft](https://techcommunity.microsoft.com/t5/microsoft-entra-azure-ad-blog/expansion-of-fido-standard-and-new-updates-for-microsoft/ba-p/3290633), [Google](https://blog.google/technology/safety-security/one-step-closer-to-a-passwordless-future/) ed [Apple](https://www.apple.com/newsroom/2022/05/apple-google-and-microsoft-commit-to-expanded-support-for-fido-standard/) vogliono abbandonare del tutto le password.


{{< figure src="https://imgs.xkcd.com/comics/password_strength.png" title="sistema la tua password policy">}}

Un altro esempio di security theatre e' il fatto di implementare sistemi che dovrebbero dare un livello aggiuntivo di sicurezza come firewall, IDS e IP (in realta' potremmo aprire un dibattito sul fatto che aumentino o diminuiscano la superficie d'attacco), per poi non aggiornali.

## Cause

Quindi, cosa possono fare le aziende per evitare questo circo infinito? Prima di tutto dovrebbero capire quali sono i veri rischi che corrono e implementare contromisure che li indirizzino in maniera adeguata. Questo deve includere in primo luogo la formazione dei dipendenti su come riconoscere e rispondere ad un attacco.

Dopotutto, la cybersecurity non e' altro che la Corsa all'Oro 2.0, ci sono moltissimi soldi sul tavolo e tutti ne vogliono una fetta, questo causa pero' difficolta' nel trovare genete realmente preparata e se per caso ci si volesse affidare a dei consulenti esterni sarebbe ancora peggio, in quanto si dipenderebbe completamente da un'entita' esterna per la propria Security Posture.

## Tirando le somme

In conclusione, il security theatre e' un problema comune nel mondo dell'IT Security. Per evitare di far parte di questo pessimo show le organizzazioni dovrebbero concentrarsi nell'implementare controlli funzionia, nel testarne regolarmente l'efficacia e nell'avere un piano di risposta comprensivo.
A, per favore, smettiamola con le policy inutili.

{{< admonition type=tip title="Tip" open=true >}}
[Qui](https://www.philvenables.com/post/ceremonial-security-and-cargo-cults) puoi trovare un articolo di Phil Venables, CISO di Google, sulla Cerimonial Security e i Cargo Cults.
{{< /admonition >}}

