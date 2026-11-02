# M0 — Understand

Fase 1 del workflow ADLC (vedi `CLAUDE.md`). Questo documento è l'input di `02-explore.md`. **Checkpoint umano: si prosegue solo dopo conferma di Valerio.**

## Obiettivo della milestone

Preparare la struttura del repository e un materiale di riferimento sui componenti webMethods, in modo che le milestone successive partano da una base ordinata e da una comprensione condivisa dei termini (IS, MSR, package, UM, Terracotta, API Gateway, BPM, MWS).

## Deliverable

| # | Deliverable | Criterio di done |
| --- | --- | --- |
| 1 | Struttura cartelle da `CLAUDE.md` (`db/`, `mock-crm/`, `packages/` con le 5 sottocartelle Lcs*, `infra/`, `.github/workflows/`, `docs/`) | Ogni cartella esiste con un `README.md` breve che dice cosa conterrà e in quale milestone |
| 2 | `docs/components.md` | Per ciascuno dei 13 componenti della tabella in `CLAUDE.md`: 5-10 righe (cosa fa, concetti chiave) e almeno un link alla documentazione ufficiale IBM effettivamente aperto e verificato. Nessun dettaglio non trovato nelle fonti |
| 3 | `docs/setup.md` | Passi per installare Service Designer, avviare il Microservices Runtime locale, aprire la console di amministrazione, e come i package di Designer finiscono nel repo Git (percorso dei package, cosa committare e cosa no) |
| 4 | `docs/adlc/M0/02-explore.md` | Contiene, con fonti, se esiste un'immagine container ufficiale del Microservices Runtime, come si ottiene e con quale licenza; più lo stato del repo e i rischi |
| 5 | `CLAUDE.md` sezione "Stato" | M0 spuntata in fase Close |

## Vincoli

- Regole di onestà di `CLAUDE.md`: nessun nome di servizio, proprietà o versione inventato; ciò che non è verificato nella documentazione IBM non entra in `components.md`.
- Documentazione in italiano, nomi tecnici e commit in inglese, Conventional Commits.
- Nessuna credenziale né file di licenza nel repo.
- Il repo è pubblico fin da ora: ogni documento deve già rispettare la dichiarazione di progetto di studio.

## Fuori scope di M0

- Qualsiasi codice: niente schema DB, mock CRM, docker-compose, package IS (M1 e M2).
- Installazione effettiva di Service Designer: la fa Valerio a mano (Passo 0 della roadmap); M0 documenta solo la procedura.
- Decidere ora se Universal Messaging girerà in Docker (M7) o se MSR girerà in CI (M4): in M0 si raccoglie solo l'informazione sull'immagine container.

## Componenti webMethods coinvolti

In M0 nessun componente viene usato a runtime. Vengono **descritti** tutti e 13 quelli della tabella in `CLAUDE.md`. Quelli da approfondire di più, perché servono già in M1-M2:

- Integration Server / Microservices Runtime: differenza tra i due, cos'è un package e come è fatto su disco.
- Designer / Service Designer: contenuto del bundle, versione disponibile per download, licenza di valutazione.
- Adapter for JDBC: solo a livello di concetti (connection, adapter service, notification).

## Cosa deve saper spiegare Valerio alla fine di M0

Differenza tra IS e MSR; cos'è un package; a cosa servono UM, Terracotta, API Gateway, BPM e MWS. `components.md` va scritto con questo come test di qualità: se una voce non permette di rispondere a queste domande in un colloquio, non è finita.

## Domande aperte per Valerio (rispondere prima di Explore)

1. Service Designer è già installato? Se sì, quale versione e percorso di installazione: `setup.md` deve riportare valori reali, non ipotetici.
2. I package IS vanno committati nella cartella `packages/` del repo come copia, oppure il repo deve puntare direttamente alla cartella `packages` del MSR locale (es. con il Git locale di Designer)? La scelta cambia `setup.md`.
3. Il mock CRM sarà in FastAPI come dice la roadmap (M1): lo confermo ora così i README delle cartelle lo dicono già.

## Passi successivi

Alla conferma di questo documento: fase Explore (`02-explore.md`) con verifica della documentazione IBM e della disponibilità dell'immagine container MSR, poi piano `03-plan.md` per approvazione.
