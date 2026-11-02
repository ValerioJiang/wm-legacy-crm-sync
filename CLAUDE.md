# CLAUDE.md — wm-legacy-crm-sync

## Scopo del repository

Progetto portfolio per imparare IBM webMethods partendo da un caso reale che conosco bene: la sincronizzazione tra un gestionale legacy e un CRM (lo stesso problema risolto in MuleSoft su Unipol).

Obiettivi, in ordine di importanza:
1. **Imparare davvero** Integration Server, abbastanza da sostenere un colloquio tecnico. Le parti core le scrivo io in Designer; tu mi guidi, prepari tutto il resto e fai review.
2. Avere un repository pubblico credibile da collegare al CV.
3. Documentare il passaggio da MuleSoft a webMethods (`docs/mulesoft-to-webmethods.md`).

## Regole di onestà (non negoziabili)

- Il README dichiara che è un **progetto di studio/portfolio** costruito con webMethods Service Designer (licenza di valutazione, uso non commerciale).
- Mai scrivere nel README, nei commit o nei documenti che il progetto è in produzione o che rappresenta esperienza su clienti.
- I componenti non provati (BPM, My webMethods Server, Terracotta, eventualmente API Gateway e Universal Messaging) compaiono **solo come documenti di design**, etichettati come tali.
- Se non sei sicuro di un dettaglio di webMethods (nomi di servizi built-in, struttura file, versioni), **verificalo nella documentazione ufficiale IBM** o chiedimelo. Non inventare nomi di servizi `pub.*`, proprietà o opzioni.

## Componenti webMethods: cosa sono e cosa usiamo

| Componente | Cosa fa (alto livello) | Equivalente MuleSoft (analogia) | Uso in questo progetto |
| --- | --- | --- | --- |
| **Integration Server (IS)** | Runtime dove girano i servizi di integrazione. Unità di deploy = *package*. Servizi *flow* (linguaggio visuale: INVOKE, MAP, BRANCH, LOOP, REPEAT, SEQUENCE per try/catch, EXIT) e servizi Java. I dati viaggiano nella *pipeline*; le strutture sono *document type*. | Mule runtime + flow + evento/payload | **Sì, è il cuore del progetto** |
| **Microservices Runtime (MSR)** | Versione leggera di IS pensata per container, stessi package. È incluso in Service Designer. | Mule runtime standalone / Runtime Fabric | **Sì**, è il runtime che usiamo davvero |
| **Designer / Service Designer** | IDE Eclipse per sviluppare servizi, document type, test. Service Designer = bundle gratuito (Designer + MSR + JDBC Adapter + Unit Test Framework + integrazione Git locale). | Anypoint Studio | **Sì**, lo uso io |
| **Adapter for JDBC** | Connessioni e *adapter service* verso database (select, insert, update, stored procedure) e *adapter notification* per catturare cambiamenti. | Database connector | **Sì**, legge il DB legacy |
| **Flat file (WmFlatFile)** | Schemi e dizionari per file CSV, delimitati o a lunghezza fissa. | DataWeave su CSV/fixed width | **Sì**, nel batch |
| **REST API descriptor / OpenAPI** | Esposizione di servizi IS come API REST, anche generate da uno spec OpenAPI (API-first). | APIkit + RAML/OAS | **Sì** |
| **Unit Test Framework** | Test dei servizi con mock dei servizi invocati. | MUnit | **Sì** |
| **Universal Messaging (UM)** | Broker di messaggistica (pub/sub e JMS). IS lo usa come *messaging provider*; i *trigger* sottoscrivono i messaggi e invocano servizi, con retry e gestione errori. | Anypoint MQ / JMS | **Se riusciamo a farlo girare in Docker**; altrimenti solo design + pattern simulato in IS |
| **API Gateway** | Gateway davanti alle API: policy di sicurezza, throttling, logging, routing. | API Manager + gateway policy | **Solo documento di design** delle policy (salvo accesso a un'istanza) |
| **Terracotta** | Cache distribuita / BigMemory e clustering di IS (cache condivise tra nodi, stato). | Object Store (persistente/distribuito) | **Solo design** |
| **BPM (Process Engine)** | Processi di business modellati in BPMN in Designer, eseguiti sul Process Engine di IS, con step automatici e task umani. | Nessun equivalente diretto in Mule | **Solo design** |
| **My webMethods Server (MWS)** | Portale web per amministrazione, monitoraggio dei processi e inbox dei task umani BPM. | Runtime Manager / Monitoring (solo in parte) | **Solo design** |
| **Deployer / Asset Build Environment** | Strumenti per build e deploy dei package tra ambienti (CI/CD tradizionale webMethods). | Maven plugin + CI | Citato nel design CI/CD; noi usiamo container + GitHub Actions |

## Architettura del progetto (target)

- `db/` — Postgres che simula il **gestionale legacy** (schema + seed con dati realistici, anche sporchi).
- `mock-crm/` — piccolo servizio REST che simula il **CRM** (FastAPI o Node), con errori simulabili (500, timeout, 409).
- `packages/` — package Integration Server:
  - `LcsCommon` — document type canonici, logging, gestione errori comune
  - `LcsLegacy` — connessione JDBC e adapter service verso il legacy
  - `LcsCrmSync` — orchestrazione della sincronizzazione legacy → CRM (mapping, retry, idempotenza)
  - `LcsApi` — API REST API-first
  - `LcsBatch` — import notturno da flat file
- `infra/` — `docker-compose.yml` (Postgres, mock CRM, MSR, eventualmente UM).
- `.github/workflows/` — CI.
- `docs/` — architettura, ADR, design dei componenti non provati, guida MuleSoft → webMethods, documenti ADLC.

## Chi fa cosa

- **Io (Valerio)**: creo in Designer i servizi flow, i document type e gli adapter service, perché devo saperli fare a mano. Li committo.
- **Tu (Claude Code)**:
  - prepari infrastruttura, DB, mock CRM, Docker, CI, script, documentazione;
  - per ogni servizio flow mi scrivi una **guida passo-passo per Designer** (step, pipeline in/out, mapping) invece di scrivere tu i file `flow.xml`;
  - fai **review** dei file dei package che committo (struttura, naming, gestione errori), segnalando dubbi senza riscriverli;
  - puoi scrivere servizi **Java** IS solo quando serve davvero e dopo averlo proposto nel piano.
- Non modificare a mano `flow.xml` / `node.ndf` generati da Designer, salvo mia richiesta esplicita.

## Workflow ADLC (Agentic Development Lifecycle)

Ogni milestone segue sei fasi. Ogni fase produce un documento in `docs/adlc/<milestone>/` che è l'input della successiva.

1. **Understand** → `01-understand.md`: obiettivo, vincoli, cosa è fuori scope, componenti webMethods coinvolti. **Checkpoint umano: mi fermo e confermo.**
2. **Explore** → `02-explore.md`: cosa c'è già nel repo, cosa dice la documentazione ufficiale sui componenti coinvolti (con link), rischi e incognite.
3. **Design** → `03-plan.md` (skill *write plan* / *prepare plan*): piano con ogni decisione già presa. Ogni step ha: obiettivo, dipendenze, file da creare/modificare e file da **non** toccare, interfacce (input/output, document type), criteri di done verificabili, note/stato. Gli step che faccio io in Designer sono marcati `[VALERIO]`. **Checkpoint umano: approvo il piano.**
4. **Execute** → (skill *execute plan*): esegui solo gli step approvati, nell'ordine. Se uno step fallisce **due volte**, fermati e scrivi cosa hai provato; niente tentativi a caso.
5. **Verify** → `05-verify.md` (skill *double check*): la verifica la fa un **subagent separato** che non ha eseguito il lavoro. Controlla i criteri di done uno per uno, esegue test e build, segnala ciò che non torna. Chi esegue non giudica il proprio lavoro.
6. **Close** → (skill *update docs*): aggiorna README, `docs/architecture.md`, `docs/mulesoft-to-webmethods.md`, e la sezione "Stato" in fondo a questo file.

## Convenzioni

- Commit piccoli, messaggi in inglese, Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`).
- Documentazione in italiano; codice, nomi di servizi e commit in inglese.
- Naming servizi IS: `<package>.<area>:<serviceName>` in camelCase (es. `LcsCrmSync.customer:syncCustomer`).
- Nessuna credenziale nel repo: `.env.example` con valori finti, `.env` in `.gitignore`.

## Stato

- [ ] M0 — Setup e conoscenza dei componenti
- [ ] M1 — Legacy DB, mock CRM, docker-compose
- [ ] M2 — Sync legacy → CRM su Integration Server
- [ ] M3 — Test con Unit Test Framework
- [ ] M4 — CI/CD con GitHub Actions e container
- [ ] M5 — API REST API-first + design policy API Gateway
- [ ] M6 — Batch su flat file
- [ ] M7 — Asincrono pub/sub (Universal Messaging o pattern simulato)
- [ ] M8 — Design BPM, MWS, Terracotta
- [ ] M9 — README finale e guida MuleSoft → webMethods
