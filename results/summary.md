# Case study JoJoX — erev0s/VAmPI

https://github.com/erev0s/VAmPI · 1352 ★ · licenza/license MIT · commit `f16052d` · 2026-11-15

## Numeri / Numbers

| | Prima / Before | Dopo / After |
|---|---|---|
| Punteggio / Score | 9/100 | 9/100 |
| Critici / Critical | 5 | 5 |
| Alti / High | 13 | 13 |
| Medi / Medium | 18 | 18 |
| Bassi / Low | 4 | 4 |

File analizzati / Files analyzed: 22 · file corretti / files fixed: 0 · controlli / checks: 103

## I problemi più interessanti / The most interesting issues

### 1. Un valore inserito dall'utente viene incollato direttamente dentro una query al database
*EN: A value entered by the user is pasted directly into a database query*

`models/user_model.py:72` · critico / critical · `sql-injection` · ✋ da sistemare a mano / manual fix

**Prima / Before**
```python
        if vuln:  # SQLi Injection
            user_query = f"SELECT * FROM users WHERE username = '{username}'"
            query = db.session.execute(text(user_query))
```

> Una query SQL è costruita concatenando o interpolando direttamente una stringa invece di usare parametri. Se quel testo arriva (anche indirettamente) da un utente, può alterare la query stessa — SQL injection.
>
> *EN: A SQL query is built by concatenating or interpolating a string directly instead of using parameters. If that text comes (even indirectly) from a user, it can alter the query itself — SQL injection.*

### 2. Le password sembrano non essere protette
*EN: Passwords don't seem to be protected*

`models/user_model.py:24` · critico / critical · `plaintext-password-storage` · ✋ da sistemare a mano / manual fix

**Prima / Before**
```python
        self.email = email
        self.password = password
        self.admin = admin
```

> Un campo password viene salvato così com'è arrivato dalla richiesta, e nel file non compare nessuna funzione di hashing (bcrypt, argon2, scrypt). Se il database viene compromesso, tutte le password sono immediatamente utilizzabili.
>
> *EN: A password field is saved exactly as it arrived in the request, and no hashing function (bcrypt, argon2, scrypt) appears in the file. If the database is compromised, every password is immediately usable.*

### 3. Un'azione di terze parti nella CI non è bloccata su una versione precisa (commit SHA)
*EN: A third-party CI action isn't pinned to a precise version (commit SHA)*

`.github/workflows/docker-image.yml:16` · medio / medium · `gha-unpinned-action` · ✋ da sistemare a mano / manual fix

**Prima / Before**
```yaml
        name: Set up QEMU
        uses: docker/setup-qemu-action@v2
      -
```

> Il workflow usa un'azione di terze parti referenziata per tag o branch (es. "@v1", "@main") invece che per commit SHA. Chi la mantiene — o chi ne compromette l'account — può spostare quel riferimento su un commit diverso in qualsiasi momento, cambiando silenziosamente cosa gira nella CI, con accesso ai suoi segreti. Un commit SHA punta sempre allo stesso identico codice.
>
> *EN: The workflow uses a third-party action referenced by tag or branch (e.g. "@v1", "@main") instead of by commit SHA. Whoever maintains it — or whoever compromises their account — can move that reference to a different commit at any time, silently changing what runs in the CI, with access to its secrets. A commit SHA always points to the exact same code.*

## Cosa resta da fare a mano / What's left to fix by hand

- Le password sembrano non essere protette (*Passwords don't seem to be protected*) — critico / critical × 2
- Dipendenze con vulnerabilità note (da aggiornare) (*Dependencies with known vulnerabilities (to upgrade)*) — critico / critical × 2
- Un valore inserito dall'utente viene incollato direttamente dentro una query al database (*A value entered by the user is pasted directly into a database query*) — critico / critical × 1
- Dipendenze con vulnerabilità note (da aggiornare) (*Dependencies with known vulnerabilities (to upgrade)*) — alto / high × 13

## Note / Notes

- Stessa pipeline del link pubblico su jojox.it (download, controlli a pattern + dipendenze): chi incolla lo stesso link allo stesso commit vede lo stesso punteggio "prima". / Same pipeline as the public link on jojox.it.
- Il "dopo" applica le correzioni automatiche di JoJoX in locale: nessuna pull request viene aperta sul repository originale. / The "after" applies JoJoX's automatic fixes locally; no pull request is opened on the original repository.
- erev0s/VAmPI è un progetto di terzi (licenza MIT): citalo e linkalo nel post. / It's a third-party project: credit and link it in the post.
