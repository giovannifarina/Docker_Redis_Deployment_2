# Laboratorio Docker: PostgreSQL, Redis & Jupyter (Cache-Aside Pattern)

Questo laboratorio è stato configurato con **Docker Compose** per sperimentare l'architettura **Cache-Aside** integrando un database relazionale (**PostgreSQL**) con una cache in-memory (**Redis**) e un ambiente interattivo di analisi (**JupyterLab**).

---

##  Architettura del Sistema

```
                  +---------------------------+
                  |  JupyterLab (/notebooks)  |
                  |     (Python 3 Kernel)     |
                  +-------------+-------------+
                                |
             +------------------+------------------+
             |                                     |
             v                                     v
   +--------------------+               +--------------------+
   |   PostgreSQL 16    |               |      Redis 7       |
   | (Relational RDBMS) |               | (In-Memory Cache)  |
   |   Port: 5432       |               |   Port: 6379       |
   | Volume: pgdata     |               |  TTL / O(1) Lookup |
   +--------------------+               +--------------------+
```

- **PostgreSQL (`postgres:16-alpine`)**: RDBMS per la persistenza sicura e la modellazione relazionale dei dati con vincoli di Primary Key.
- **Redis (`redis:7-alpine`)**: Key-Value store in memoria RAM ad altissime prestazioni per il caching delle tuple più richieste.
- **Jupyter (`jupyter/minimal-notebook`)**: Container configurato secondo le specifiche del repository `Minimal-HDFS-Deployment`, arricchito con librerie Python (`psycopg2-binary`, `sqlalchemy`, `redis`, `pandas`, `matplotlib`, `seaborn`).

---

## 📊 Dataset Selezionato

- **Dataset**: [Goodreads Books Dataset su Kaggle](https://www.kaggle.com/datasets/jealousleopard/goodreadsbooks) (autore: *jealousleopard*).
- **Perché è ideale per questo scenario?**:
  - Contiene oltre 11.000 libri con attributi realistici: `book_id`, `title`, `authors`, `average_rating`, `isbn`, `num_pages`, `ratings_count`, `publisher`.
  - Ogni record ha una **chiave primaria univoca** (`book_id`), perfetta sia per l'indicizzazione B-Tree in PostgreSQL che per il lookup con chiave singola in Redis (`book:{book_id}`).
  - Simula fedelmente scenari reali come cataloghi e-commerce, streaming multimediale o blog dove le letture puntuali per ID sono preponderanti rispetto alle scritture (*read-heavy workloads*).

---

## 🚀 Come Avviare il Laboratorio

Dalla cartella principale (`/Users/giovannifarina/Redis2`), esegui:

```bash
docker compose up -d --build
```

Questo comando:
1. Crea l'immagine personalizzata per Jupyter con i driver PostgreSQL e Redis.
2. Avvia PostgreSQL con health check automatico.
3. Avvia Redis con health check automatico.
4. Avvia JupyterLab esponendo la porta `8888`.

### Accesso a JupyterLab

Apri il browser all'indirizzo:
👉 **[http://localhost:8888](http://localhost:8888)**

*(L'accesso è configurato senza token o password, identico alla configurazione di `Minimal-HDFS-Deployment`).*

---

## 📓 Struttura del Notebook

Nel file [`notebooks/postgres_redis_cache_lab.ipynb`](notebooks/postgres_redis_cache_lab.ipynb) troverai il percorso guidato:

1. **Test Connessione**: Verifica del corretto collegamento ai container `postgres:5432` e `redis:6379`.
2. **Download & Esplorazione**: Caricamento di `books.csv` (presente nella cartella `datasets/` e scaricabile anche al volo via URL).
3. **Ingestion Relazionale**: Creazione della tabella `books` con `PRIMARY KEY (book_id)` e indice su `isbn`, seguita dall'inserimento massivo con SQLAlchemy e Pandas.
4. **Recupero da PostgreSQL**: Query puntuale `SELECT * FROM books WHERE book_id = %s;` e misurazione dei millisecondi impiegati.
5. **Pattern Cache-Aside**:
   - **Cache Miss**: Recupero della tupla da PostgreSQL, serializzazione JSON e salvataggio su Redis con TTL (Time-To-Live).
   - **Cache Hit**: Lettura ultra-rapida direttamente dalla memoria RAM di Redis.
6. **Benchmark Prestazionale**: Simulazione di 500 accessi casuali e calcolo delle metriche:
   - Tempo medio (*Mean*)
   - Mediana (*P50*)
   - 95° e 99° percentile (*P95*, *P99*)
   - Fattore di accelerazione (*Speedup*)
7. **Grafici**: Visualizzazione comparativa con Bar Chart e Boxplot (PostgreSQL vs Redis).

---

## 🛑 Arresto e Pulizia dei Container

Per arrestare i container:
```bash
docker compose down
```

Per eliminare anche i volumi con i dati salvati su PostgreSQL:
```bash
docker compose down -v
```
