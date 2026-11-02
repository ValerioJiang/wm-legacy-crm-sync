# Roadmap step-by-step per Claude Code — wm-legacy-crm-sync

Come usarla: ogni milestone è un prompt da incollare in Claude Code. Claude Code segue il workflow ADLC descritto in `CLAUDE.md` (Understand → Explore → Design → Execute → Verify → Close) e si ferma ai due checkpoint: dopo Understand e dopo il piano. Non passare alla milestone successiva finché la Verify non è pulita e hai capito tutto quello che è stato fatto.

---

## Passo 0 — Preparazione (lo fai tu, a mano)

1. Crea su GitHub il repository pubblico `wm-legacy-crm-sync` e clonalo.
2. Metti nella root il file `CLAUDE.md`.
3. Registrati con un IBM ID e scarica **webMethods Service Designer** dalla pagina download IBM. Installalo e verifica che il Microservices Runtime locale parta e che riesci ad aprire la console di amministrazione.
4. Iscriviti su IBM Training al percorso gratuito di livello Basic su webMethods Integration e fallo in parallelo alle prime milestone.

---

## M0 — Setup e conoscenza dei componenti

```
Leggi CLAUDE.md. Avvia la milestone M0 con il workflow ADLC.
Obiettivo: preparare la struttura del repository e un documento di riferimento sui componenti webMethods.
- Crea la struttura di cartelle descritta in CLAUDE.md (vuota, con README brevi per cartella).
- Scrivi docs/components.md: per ogni componente della tabella in CLAUDE.md, 5-10 righe su cosa fa, concetti chiave e link alla documentazione ufficiale IBM che hai verificato. Niente dettagli che non hai trovato nelle fonti.
- Scrivi docs/setup.md: come installo e avvio Service Designer e il Microservices Runtime, e come i package finiscono nel repo Git.
- Verifica nelle fonti ufficiali se esiste un'immagine container ufficiale del Microservices Runtime e come si ottiene; scrivi il risultato in docs/adlc/M0/02-explore.md.
Fermati dopo 01-understand.md per la mia conferma.
```

**Cosa devi saper spiegare dopo M0:** differenza tra Integration Server e Microservices Runtime, cos'è un package, cosa fanno UM, Terracotta, API Gateway, BPM e MWS.

---

## M1 — Legacy DB, mock CRM, docker-compose

```
Avvia la milestone M1 con il workflow ADLC.
Obiettivo: i due sistemi da integrare, avviabili con un comando.
- db/: schema Postgres di un gestionale clienti/polizze legacy (nomi di colonne "vecchi", campi codificati, date come stringhe, alcuni dati sporchi), più seed con circa 200 record e una colonna o tabella per tracciare le modifiche da sincronizzare.
- mock-crm/: servizio REST (FastAPI) con endpoint upsert e get dei clienti, chiave esterna per l'idempotenza, e possibilità di simulare errori 500, timeout e 409 tramite configurazione.
- infra/docker-compose.yml con Postgres e mock CRM; il Microservices Runtime per ora gira in locale da Service Designer.
- Test automatici del mock CRM.
Fermati dopo 01-understand.md e dopo 03-plan.md per la mia approvazione.
```

---

## M2 — Sincronizzazione legacy → CRM su Integration Server (la milestone più importante)

```
Avvia la milestone M2 con il workflow ADLC.
Obiettivo: sincronizzazione dei clienti modificati dal legacy al CRM su Integration Server.
Componenti: Integration Server/MSR, Adapter for JDBC, document type, servizi flow.
Nel piano:
- definisci i document type canonici in LcsCommon e il mapping campo per campo legacy → canonico → CRM (tabella in docs);
- definisci gli adapter service JDBC in LcsLegacy;
- definisci il servizio di orchestrazione in LcsCrmSync: lettura modifiche, mapping, chiamata REST al CRM, aggiornamento dello stato, try/catch con SEQUENCE, retry con backoff sugli errori temporanei, scarto e log sugli errori permanenti, idempotenza;
- definisci il logging comune e il formato degli errori.
Gli step di Designer sono [VALERIO]: per ognuno scrivi una guida passo-passo (step flow, pipeline in/out, mapping) in docs/guides/. Tu prepari tutto il resto e, quando committo, fai review dei package.
Fermati dopo 01-understand.md e dopo 03-plan.md.
```

**Cosa devi saper spiegare dopo M2:** pipeline e document type, try/catch con SEQUENCE, come hai gestito retry e idempotenza, differenze rispetto a come l'avevi fatto in MuleSoft.

---

## M3 — Test con Unit Test Framework

```
Avvia la milestone M3 con il workflow ADLC.
Obiettivo: test automatici dei servizi di M2 con il Unit Test Framework incluso in Service Designer, con mock dell'adapter JDBC e della chiamata al CRM.
Casi minimi: mapping corretto, dato sporco scartato, errore 500 con retry, 409 gestito come idempotente.
Verifica prima nella documentazione ufficiale come si definiscono ed eseguono i test, anche da riga di comando, e scrivilo in 02-explore.md.
Gli step in Designer sono [VALERIO] con guida.
```

---

## M4 — CI/CD con GitHub Actions e container

```
Avvia la milestone M4 con il workflow ADLC.
Obiettivo: pipeline GitHub Actions che a ogni push esegue i test del mock CRM e, se tecnicamente possibile con la licenza e le immagini disponibili (vedi M0), costruisce un'immagine del Microservices Runtime con i package e lancia i test di M3.
Se l'esecuzione di MSR in CI non è possibile, la pipeline fa solo i controlli possibili (struttura dei package, lint, test del mock) e docs/cicd.md spiega come sarebbe la pipeline completa con Deployer/Asset Build Environment o con container.
Nessuna credenziale o licenza nel repository.
```

---

## M5 — API REST API-first e design delle policy di API Gateway

```
Avvia la milestone M5 con il workflow ADLC.
Obiettivo: package LcsApi con API REST per consultare lo stato di sincronizzazione e rilanciare un cliente.
- Parti da uno spec OpenAPI in api/ e genera i servizi in Integration Server (step [VALERIO] con guida).
- docs/api-gateway-design.md: come pubblicherei questa API su webMethods API Gateway (policy di autenticazione, throttling, logging, versioning). Etichettalo come documento di design: non abbiamo un'istanza di API Gateway.
```

---

## M6 — Batch su flat file

```
Avvia la milestone M6 con il workflow ADLC.
Obiettivo: package LcsBatch che importa un file notturno a lunghezza fissa (aggiornamenti anagrafici da un sistema esterno) nel DB legacy.
- Flat file schema/dizionario (WmFlatFile), validazione, scrittura via JDBC, report degli scarti.
- Schedulazione tramite lo scheduler di Integration Server.
- Genera file di esempio validi e con errori.
Step in Designer [VALERIO] con guida.
```

---

## M7 — Asincrono pub/sub

```
Avvia la milestone M7 con il workflow ADLC.
Obiettivo: rendere asincrona la sincronizzazione: un servizio pubblica un evento "cliente modificato", un trigger lo consuma e chiama LcsCrmSync.
In 02-explore.md verifica nelle fonti ufficiali se Universal Messaging è ottenibile e avviabile in Docker con la nostra licenza di valutazione.
- Se sì: aggiungi UM a docker-compose e configura IS per usarlo come messaging provider, con trigger, retry e gestione dei messaggi falliti.
- Se no: implementa il pattern in Integration Server senza UM e scrivi docs/universal-messaging-design.md su come cambierebbe con UM (canali, trigger, retry, dead letter).
```

---

## M8 — Design di BPM, My webMethods Server e Terracotta

```
Avvia la milestone M8 con il workflow ADLC.
Obiettivo: documenti di design, non implementazione.
- docs/bpm-design.md: un processo di approvazione di un cliente sospetto (dato anomalo dalla sincronizzazione) con task umano: modello del processo, step automatici, task su My webMethods Server, escalation.
- docs/terracotta-design.md: dove servirebbe una cache distribuita in questa architettura (es. lookup di decodifica in cluster di IS) e come la configurerei.
Ogni documento dichiara in apertura che è un design non implementato. Cita la documentazione ufficiale per ogni concetto.
```

---

## M9 — Chiusura

```
Avvia la milestone M9 con il workflow ADLC.
Obiettivo: rendere il repository presentabile.
- README: problema, architettura (diagramma Mermaid), come avviarlo, cosa è implementato e cosa è solo design, dichiarazione che è un progetto di studio su licenza di valutazione.
- docs/mulesoft-to-webmethods.md: per ogni scelta, come l'avrei fatta in MuleSoft e come l'ho fatta in webMethods.
- Proponimi una riga onesta per il CV che descriva il progetto.
La Verify la fa un subagent che legge il repo come un recruiter tecnico e segnala qualsiasi affermazione esagerata.
```

---

## Ordine consigliato e tempi indicativi

M0 → M1 → M2 → M3 → M4 sono il nucleo: dopo M4 hai già un progetto solido da mostrare. M5–M8 aggiungono copertura sui componenti dell'annuncio. Prima di ogni colloquio rileggi i paragrafi "Cosa devi saper spiegare".
