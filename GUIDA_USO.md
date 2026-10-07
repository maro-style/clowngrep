# Guida all'uso di ClownGrep

Questa guida spiega **come scegliere tra le due modalità di analisi** di ClownGrep e come usare l'opzione **3 — SCEGLI PER FILE** quando file con la stessa estensione hanno strutture diverse.

## Prima regola: cosa rappresenta un "record" nel file?

ClownGrep può cercare i domini in due modi diversi:

- **Modalità 1 — RIGHE / dataleak:** considera ogni riga come un record indipendente.
- **Modalità 2 — BLOCCHI:** considera più righe consecutive come un unico record, fino alla successiva riga vuota.

La scelta non cambia il dominio cercato: cambia **quanto contesto viene salvato quando viene trovato un match**.

L'opzione **3 — SCEGLI PER FILE** non è un terzo motore di analisi: è un modo per decidere **se usare RIGHE o BLOCCHI su ciascun `.txt`/`.log` separatamente**, invece di applicare una sola scelta a tutta l'estensione.

---

## Modalità 1 — RIGHE / dataleak

### Come funziona

ClownGrep legge il file riga per riga. Se una riga contiene un dominio presente in `clienti.csv`, quella riga viene associata al cliente o alla terza parte corrispondente e salvata nell'output.

Le righe vicine che non contengono un dominio target **non vengono incluse automaticamente**.

### Quando usarla

Usa questa modalità quando ogni record è già contenuto in una sola riga.

È normalmente la scelta giusta per:

- CSV;
- SQL dump;
- JSON o JSONL con record su una sola riga;
- liste di credenziali nel formato `host:user:password`;
- log lineari;
- TXT in cui ogni riga rappresenta un elemento distinto;
- XLSX/XLSM, dopo la conversione interna delle celle in righe testuali.

### Esempio

File sorgente:

```text
user=mario | host=portal.example.com | password=abc123
user=luca | host=unrelated.example | password=xyz987
```

Se `example.com` è configurato in `clienti.csv`, ClownGrep salva:

```text
user=mario | host=portal.example.com | password=abc123
```

La seconda riga non viene salvata.

### Vantaggio

L'output è compatto e contiene solo le righe effettivamente interessanti.

### Limite

Se le informazioni appartenenti allo stesso record sono distribuite su più righe, potresti perdere il contesto.

---

## Modalità 2 — BLOCCHI

> Disponibile solo per `.txt` e `.log`.

### Come funziona

ClownGrep considera una o più righe consecutive come un unico blocco. Una **riga vuota** indica la fine del blocco e l'inizio del successivo.

Se almeno una riga del blocco contiene un dominio target, ClownGrep salva **l'intero blocco**, comprese le righe che non contengono direttamente il dominio.

### Quando usarla

Usala quando ogni record occupa più righe ed è separato dal record successivo tramite una riga vuota.

È utile, ad esempio, per dump o log strutturati così:

```text
URL: https://portal.example.com
Username: mario@example.com
Password: abc123
Browser: Chrome
Machine: DESKTOP-01

URL: https://unrelated.example
Username: user@test.example
Password: qwerty
Browser: Firefox
Machine: DESKTOP-02
```

Se `example.com` è un dominio target, ClownGrep salva tutto il primo blocco:

```text
URL: https://portal.example.com
Username: mario@example.com
Password: abc123
Browser: Chrome
Machine: DESKTOP-01
```

Questo permette di conservare username, password, browser, host e altre informazioni correlate anche se il dominio compare solo nella prima riga.

### Vantaggio

Mantiene il contesto completo del record.

### Limite importante

La modalità blocchi dipende dalle **righe vuote**. Se un file non contiene separatori vuoti tra i record, una porzione molto grande del file potrebbe essere trattata come un unico blocco.

Per questo motivo non è adatta ai normali CSV, SQL dump o file con un record per riga.

---

## Opzione 3 — SCEGLI PER FILE

> Disponibile solo per `.txt` e `.log`.

### Perché esiste

Due file con la stessa estensione possono avere strutture completamente diverse. Per esempio:

- `credentials.txt` può contenere un record completo su ogni riga;
- `browser_dump.txt` può contenere record di cinque o sei righe separati da una riga vuota.

Se scegliessi una sola modalità per tutti i `.txt`, uno dei due file potrebbe essere analizzato con la struttura sbagliata.

### Come funziona

Nel menu dell'estensione scegli:

```text
3) SCEGLI PER FILE
```

Dopo aver completato le scelte per tutte le estensioni, ClownGrep apre una sezione dedicata:

```text
=== MODALITA' PER SINGOLO FILE ===

File: credentials.txt (120.0MB)
1) RIGHE   -> ogni riga e' un record indipendente
2) BLOCCHI -> record multi-riga separati da righe vuote

> **Attenzione:** usa la modalita' BLOCCHI solo quando i record sono separati da righe vuote. Se il file non contiene questi separatori, ClownGrep puo' interpretare l'intero file come un unico blocco.
h) guida con esempi
s) salta solo questo file
Modalita' per questo file [1/2/h/s]:
```

Puoi quindi scegliere una modalità diversa per ciascun file. `s` in questa fase salta **solo quel file**, non tutta l'estensione.

### Esempio

Hai tre file `.txt`:

```text
01_credentials.txt       -> un record per riga
02_browser_dump.txt      -> record multi-riga
03_notes.txt             -> non vuoi analizzarlo
```

Scegli `3` per `.txt`, poi:

```text
01_credentials.txt  -> 1
02_browser_dump.txt -> 2
03_notes.txt        -> s
```

In questo modo ClownGrep costruisce il piano di elaborazione prima di svuotare `outputs/` e avvia la scansione solo sui file selezionati.

---

## Regola rapida

| Struttura del file | Modalità consigliata |
|---|---|
| Un record per riga | `1` — RIGHE / dataleak |
| Un record su più righe, separato da una riga vuota | `2` — BLOCCHI |
| CSV | `1` |
| SQL dump | `1` |
| JSON/JSONL lineare | `1` |
| XLSX/XLSM | `1` |
| TXT con lista `host:user:password` | `1` |
| TXT/LOG con schede multi-riga separate da righe vuote | `2` |
| Più TXT/LOG con strutture diverse | `3` — scegli RIGHE/BLOCCHI file per file |

In breve:

```text
UN RECORD = UNA RIGA        -> 1
UN RECORD = PIU' RIGHE      -> 2
STESSA ESTENSIONE, STRUTTURE MISTE -> 3
```

---

## Il comando `h`

Quando ClownGrep chiede la modalità, puoi digitare:

```text
h
```

per visualizzare direttamente nel terminale una spiegazione con esempi.

Per `.txt` e `.log` il menu è simile a:

```text
1) RIGHE / dataleak -> salva solo le righe che contengono un dominio target
2) BLOCCHI          -> salva l'intero record multi-riga che contiene il dominio
                       (i blocchi devono essere separati da righe vuote)
3) SCEGLI PER FILE  -> chiedi RIGHE/BLOCCHI separatamente per ogni file
h) guida con esempi
s) salta tutti i file di questa estensione
```

Per gli altri formati viene proposta solo la modalità a righe.

---

## Esempio pratico: stesso contenuto, risultato diverso

Input:

```text
URL: portal.example.com
User: mario
Password: abc123

URL: another.example
User: luca
Password: qwerty
```

### Con modalità 1

Viene salvata soltanto la riga contenente il dominio:

```text
URL: portal.example.com
```

### Con modalità 2

Viene salvato tutto il record:

```text
URL: portal.example.com
User: mario
Password: abc123
```

Questa è la differenza fondamentale tra le due modalità.

---

## File Excel

Per `.xlsx` e `.xlsm` ClownGrep usa sempre la modalità a righe. Ogni riga del foglio viene convertita temporaneamente in una rappresentazione testuale che include il nome del foglio e il numero di riga, quindi viene applicato il normale matching dei domini.

Se `openpyxl` non è disponibile e sono presenti file Excel, ClownGrep crea automaticamente una virtual environment locale `.clowngrep_venv`, installa la dipendenza e si riavvia usando quell'ambiente. Il Python gestito da Homebrew non viene modificato.

---

## Suggerimento operativo

Se hai un TXT o LOG e non sai quale modalità scegliere, apri qualche record del file:

- se ogni risultato utile è completo sulla stessa riga, usa **1**;
- se il dominio è su una riga ma username, password o altri campi sono nelle righe subito sotto, usa **2**;
- se non ci sono righe vuote tra i record, usa **1** oppure normalizza prima il file;
- se nella stessa estensione hai file lineari e file multi-riga, usa **3** e scegli la modalità file per file.
