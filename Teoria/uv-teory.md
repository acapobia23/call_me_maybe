# uv — Guida rapida

## Cos'è uv

`uv` è un package manager e project manager per Python, sviluppato da [Astral](https://astral.sh/) (gli stessi creatori di `ruff`). È scritto in Rust ed è pensato per sostituire in un solo strumento diversi tool che di solito si usano insieme in Python:

- `pip` (installazione pacchetti)
- `venv` (creazione ambienti virtuali)
- `pip-tools` (lock delle dipendenze)
- `pipx` (esecuzione tool isolati)
- `poetry` (gestione progetto e `pyproject.toml`)

### Perché usarlo

- **Velocità**: risolve e installa dipendenze molto più velocemente di pip (spesso 10-100x), grazie alla cache globale e al resolver scritto in Rust.
- **Un solo tool**: non serve più destreggiarsi tra `venv`, `pip freeze`, `requirements.txt` scritti a mano.
- **Lock file automatico**: `uv.lock` fissa le versioni esatte di ogni dipendenza (dirette e transitive), garantendo build riproducibili su qualsiasi macchina.
- **Gestione automatica del venv**: con `uv run` non devi nemmeno attivare manualmente l'ambiente virtuale, uv lo sincronizza da solo prima di eseguire.
- **Gestione versioni Python**: uv può scaricare e gestire installazioni di Python diverse, senza bisogno di `pyenv`.

### Concetti chiave

| Concetto | Cosa fa |
|---|---|
| `pyproject.toml` | File di configurazione del progetto: nome, versione, dipendenze |
| `uv.lock` | Lock file con le versioni esatte risolte — **va sempre committato su git** |
| `.venv/` | Ambiente virtuale locale, creato automaticamente — **non va committato** |
| workspace | Più pacchetti Python gestiti insieme, con un solo venv e un solo lock file condiviso |

### Applicazione vs Libreria

- Se stai scrivendo un **programma** che esegui direttamente (`uv run main.py`), non serve altro che un `pyproject.toml` + i tuoi file `.py`.
- Se stai scrivendo una **libreria** che deve essere importata (`import mio_pacchetto`), serve una struttura a pacchetto (flat layout o src layout) con `__init__.py`.
- Se hai un'app che dipende da una libreria interna (es. un modulo SDK scritto da te), la soluzione corretta è un **workspace uv**: un solo ambiente virtuale e un solo lock file condivisi tra i pacchetti membri, così non rischi conflitti di versione tra le dipendenze dell'app e quelle della libreria.

---

## Cheat Sheet — comandi uv

### Installazione

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv --version
```

### Creare un progetto

```bash
uv init nome_progetto       # crea struttura base (pyproject.toml, main.py, ecc.)
cd nome_progetto
```

### Ambiente virtuale

```bash
uv venv                     # crea .venv/ nella cartella corrente
source .venv/bin/activate   # attivazione manuale (opzionale con uv run)
deactivate                  # disattiva l'ambiente
```

### Gestire le dipendenze

```bash
uv add nome_pacchetto            # aggiunge una dipendenza e aggiorna pyproject.toml + uv.lock
uv add nome_pacchetto==1.2.3     # con versione specifica
uv add --dev pytest ruff         # dipendenza solo di sviluppo
uv remove nome_pacchetto         # rimuove una dipendenza
uv add -r requirements.txt       # importa da un vecchio requirements.txt
```

### Sincronizzare l'ambiente

```bash
uv sync                     # installa/allinea tutto secondo pyproject.toml + uv.lock
uv lock                     # rigenera solo il lock file, senza installare
```

### Eseguire codice

```bash
uv run main.py              # sincronizza l'ambiente ed esegue lo script
uv run python                # apre una shell python nel venv del progetto
uv run pytest                # esegue un tool installato come dipendenza dev
```

### Gestione versioni Python

```bash
uv python list               # elenca le versioni Python disponibili/installabili
uv python install 3.12        # installa una versione specifica di Python
uv venv --python 3.12         # crea il venv con quella versione
```

### Workspace (più pacchetti collegati)

```toml
# pyproject.toml alla root del workspace
[tool.uv.workspace]
members = ["package_a", "package_b"]

[tool.uv.sources]
package-a = { workspace = true }
```

```bash
uv sync                     # risolve e installa TUTTI i membri del workspace insieme
uv run main.py              # esegue dalla root, con accesso a tutti i pacchetti membri
```

### File da versionare su git

```
✅ pyproject.toml
✅ uv.lock
✅ .python-version
❌ .venv/          (in .gitignore, rigenerabile con `uv sync`)
```

---

## Riferimenti

- Documentazione ufficiale: https://docs.astral.sh/uv/