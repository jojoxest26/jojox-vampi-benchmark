# JoJoX × VAmPI — benchmark onesto

> **VAmPI è un'app di test vulnerabile apposta.** I problemi qui sotto sono stati messi lì dal suo autore per allenarsi: non sono bug di JoJoX, e non sono bug di un'app in produzione.

[VAmPI](https://github.com/erev0s/VAmPI) (di [erev0s](https://github.com/erev0s), licenza MIT) è una REST API in Flask costruita per misurare quanto funzionano gli strumenti di sicurezza. Il suo README elenca 9 vulnerabilità volute. Abbiamo passato JoJoX sul commit [`f16052d`](https://github.com/erev0s/VAmPI/tree/f16052dce83f05847133ec98f01c5193a41de7d8) e confrontato i risultati con quella lista, **compreso quello che non abbiamo trovato**.

*English version below.*

## In breve

| | |
|---|---|
| Vulnerabilità ufficiali di VAmPI trovate | **1 su 9** |
| Altri problemi reali trovati (fuori dalla lista ufficiale) | password in chiaro, debug attivo, container come root, CI non bloccata su SHA, 2 dipendenze vulnerabili |
| Problemi totali segnalati | 40 (5 critici, 13 alti, 18 medi, 4 bassi) — di cui 31 dalle dipendenze |
| Correzioni automatiche applicate | 0 (niente qui era correggibile in modo meccanico e sicuro) |
| File analizzati / controlli | 22 / 103 |
| LLM usati | nessuno: controlli a pattern + database OSV.dev |

## Le 9 vulnerabilità ufficiali

| Vulnerabilità (dal README di VAmPI) | JoJoX | Perché |
|---|---|---|
| SQL injection | ✅ trovata | [`models/user_model.py:72`](https://github.com/erev0s/VAmPI/blob/f16052dce83f05847133ec98f01c5193a41de7d8/models/user_model.py#L72), confermata |
| Cambio password non autorizzato | ❌ | logica di autorizzazione: serve capire chi può fare cosa |
| Broken Object Level Authorization | ❌ | idem |
| Mass assignment | ❌ | dipende dal modello dati, non da un pattern |
| Esposizione dati dall'endpoint `/_debug` | ❌ | abbiamo segnalato `debug=True` in `app.py:17`, ma è un'altra cosa |
| Enumerazione utenti e password | ❌ | comportamento delle risposte, non del codice |
| RegexDoS | ❌ | **mancanza nostra**: la regex è in `api_views/users.py`, un controllo potrebbe vederla |
| Niente rate limiting | ❌ | assenza di qualcosa: difficile da provare leggendo il codice |
| JWT aggirabile per chiave debole | ❌ | **mancanza nostra**: `SECRET_KEY = 'random'` in `config.py:13` non è stata segnalata |

**Cosa ci dice questo:** 6 delle 8 mancate sono problemi di logica e di autorizzazione che uno scanner statico a pattern non può vedere in modo affidabile. Per quelle serve un test dinamico o una revisione umana. Due (RegexDoS e la chiave JWT debole) invece sono limiti nostri che possiamo chiudere.

## Cosa abbiamo trovato oltre la lista

- **Password salvate in chiaro**, critico: [`models/user_model.py:24`](https://github.com/erev0s/VAmPI/blob/f16052dce83f05847133ec98f01c5193a41de7d8/models/user_model.py#L24) e `:85`. Nessuna funzione di hashing nel file.
- **Dipendenze vulnerabili**: `PyJWT==2.6.0` e `flask==2.2.2`, con 30 avvisi OSV/GHSA in tutto. Alcuni avvisi sono alias della stessa vulnerabilità, quindi il numero di problemi distinti è più basso. `swagger-ui-bundle` non è stato controllato perché non ha una versione fissata.
- **Modalità debug di Flask attiva**: `app.py:17`.
- **Container che gira come root**: `Dockerfile:8`.
- **4 azioni GitHub non bloccate su un commit SHA**: `.github/workflows/docker-image.yml`.

Elenco completo: [`results/findings.tsv`](results/findings.tsv) · report: [`results/summary.md`](results/summary.md) · dati grezzi: [`results/data.json`](results/data.json).

## Come verificarlo

Ogni risultato punta a file e riga del commit `f16052d`: puoi aprirli e controllarli uno per uno, senza installare niente. Da quando jojox.it sarà online, basterà incollare `github.com/erev0s/VAmPI` per rifare la stessa analisi: stesso commit, stesso punteggio di partenza.

## Limiti

- JoJoX fa analisi statica (a pattern) + controllo delle dipendenze. Non esegue l'app e non fa test dinamici.
- Un risultato "heuristic" è un indizio forte, non una prova. Solo la SQL injection è segnata come "confirmed".
- Un solo repository non è un benchmark statistico: è un caso concreto, riproducibile.

Hai trovato un errore in questo confronto? [Apri una issue](../../issues).

---

## English

> **VAmPI is a deliberately vulnerable test app.** The issues below were planted by its author for training. They are not bugs in JoJoX or in any production app.

We ran JoJoX on [VAmPI](https://github.com/erev0s/VAmPI) (by erev0s, MIT) at commit `f16052d` and compared the results with the 9 vulnerabilities its README lists, including the ones we missed.

- **Official vulnerabilities found: 1 of 9** (SQL injection, `models/user_model.py:72`, confirmed).
- **Missed: 8.** Six are authorization and business-logic flaws (password change, BOLA, mass assignment, `/_debug` data exposure, user enumeration, missing rate limiting) that pattern-based static analysis can't reliably detect. Two are **our gaps**: the RegexDoS in `api_views/users.py` and the weak JWT key `SECRET_KEY = 'random'` in `config.py:13`.
- **Found beyond the list:** plaintext password storage (critical), vulnerable `PyJWT==2.6.0` and `flask==2.2.2` (30 OSV/GHSA advisories, some of them aliases of the same issue), Flask debug mode on, a container running as root, and 4 GitHub Actions not pinned to a SHA.
- 40 issues in total, 0 auto-fixes, 22 files, 103 checks, no LLM.

Every finding links to a file and line at the pinned commit, so you can check them by hand. Full list in [`results/findings.tsv`](results/findings.tsv).
