# 🐳 Docker Notes

My personal Docker reference, written so that months later I can read it top to bottom and remember **why** things work, not just the commands.

**How to read this:** each layer starts with the **problem**, then the **intuition** (the mental picture), then **hands-on** commands, then **what confused me** and the answer. If I only have 2 minutes, I jump to the cheat sheet at the bottom.

---

## 📚 Contents

- [Layer 0: Why Docker at all?](#layer-0-why-docker-at-all)
- [Layer 1: Images & Containers](#layer-1-images--containers)
- [Layer 2: Background Containers & Ports](#layer-2-background-containers--ports)
- [Layer 3: Volumes & Bind Mounts (Storage)](#layer-3-volumes--bind-mounts-storage)
- [Layer 4: Dockerfile (Build My Own Image)](#layer-4-dockerfile-build-my-own-image)
- [Layer 5: Docker Compose (Many Containers, One File)](#layer-5-docker-compose-many-containers-one-file)
- [Common Errors & Fixes](#common-errors--fixes)
- [Practice Exercises](#practice-exercises)
- [Cheat Sheet](#cheat-sheet)

---

## Layer 0: Why Docker at all?

### The problem
Want to try PostgreSQL? Without Docker: `brew install`, set up PATH, start a service, versions clash with something else, and later I try to uninstall it and leftovers remain. My Mac slowly gets messy.

### The intuition
Docker puts software in a **sealed box**. Everything the software needs is **inside the box**. When I'm done, I **throw the box away** and my Mac is exactly as it was.

> **Docker = try anything, use it, throw it away. No installing, no cleanup.**

### Is it a virtual machine?
Not really.
- A **VM** runs a whole separate operating system: heavy, slow to start.
- A **container** only packs the app + its libraries and shares the core of the OS: light, starts in about 1 second.
- On a Mac, Docker Desktop secretly runs **one** small Linux VM, and all my containers share it. I never have to think about it.

---

## Layer 1: Images & Containers

### The 3 words everything is built on

| Word | Mental picture | What it really is |
|---|---|---|
| **Image** | A **recipe** / an installer `.dmg` | A read-only package: app + everything it needs. Downloaded once. |
| **Container** | The **dish** cooked from the recipe | A **running** copy of an image. The actual sealed box. |
| **Registry** (Docker Hub) | The **App Store** | Where images live online. `docker pull` / `docker run` downloads from here. |

### The key idea
- **Image = template. Container = the running thing.**
- One image → I can make **many** containers from it.
- Delete a container → the image is still there → start a fresh container in seconds.

```
Docker Hub ──pull──► Image (on my Mac) ──run──► Container (running box)
                                         ──run──► Container (another box)
```

### Hands-on

```bash
docker run hello-world          # first test: prints a message and exits

docker run -it ubuntu bash      # a shell inside Ubuntu, without installing Ubuntu
#   inside: cat /etc/os-release, ls, apt update
#   type `exit` to leave

docker images                   # images I have downloaded
docker ps                       # containers running right now
docker ps -a                    # ALL containers, including stopped ones

docker rm <id>                  # delete a container (first 3–4 chars of the ID are enough)
docker rmi ubuntu               # delete an image
```

**What `docker run -it ubuntu bash` actually did:**
1. Looked for the `ubuntu` image on my Mac → not there → **pulled** it from Docker Hub
2. **Created a container** from it
3. Ran `bash` inside it
4. `-i` (keep input open) + `-t` (proper terminal) = I can type inside

### The "try and throw away" trick ⭐
```bash
docker run -it --rm python:3.13 python
```
- `--rm` = **delete the container automatically when I exit**
- I get Python 3.13 without installing it. Swap in `node`, `golang`, `redis`, anything.

### Image tags = versions
```
python:3.13      postgres:17      nginx   (no tag → means "latest")
──┬─── ─┬──
 name  version (tag)
```
I can run `postgres:15` and `postgres:17` at the same time, with no conflict. For anything I care about, **pin a version** instead of relying on `latest`.

### ⚠️ The one catch
**Files created inside a container disappear when the container is deleted.**
Fine for experiments, bad for databases. → Fixed in Layer 3.

---

## Layer 2: Background Containers & Ports

### The problem
In Layer 1, containers ran in my terminal and died when I exited. Real tools (databases, web servers) need to:
1. **Keep running in the background**
2. **Be reachable from my Mac** (browser, DB tools, my code)

### Running in the background: `-d`
```bash
docker run -d --name myweb nginx
```
- `-d` = **detached**: runs in the background and gives my terminal back
- `--name myweb` = a readable name, so I don't copy IDs like `a3f9c2...`

### Ports: the intuition
A container is a sealed box, **including its network**. Nginx inside listens on port 80, but that's port 80 **inside the box**. My Mac can't see it.

`-p` **cuts a door** in the box: traffic to a port on my Mac goes to a port inside the box.

```
   My Mac                           Container
 localhost:8080  ───── door ─────►  port 80 (nginx)

              -p 8080:80
                 ──┬─ ┬─
       my Mac's port   port inside the box
```

> **Read `-p` as: MY MAC : THE BOX** (left = outside, right = inside)

```bash
docker run -d -p 8080:80 --name myweb nginx
# open http://localhost:8080 → Nginx welcome page. Never installed Nginx.
```

### Settings: `-e` (environment variables)
Many images need settings, passed with `-e KEY=value`:
```bash
docker run -d \
  --name mypg \
  -e POSTGRES_PASSWORD=secret \
  -p 5432:5432 \
  postgres:17
```
Postgres **refuses to start** without `POSTGRES_PASSWORD`. Each image's Docker Hub page lists its settings.

Connect to it:
```bash
docker exec -it mypg psql -U postgres        # use psql inside the box
# or from my Mac with any tool: host localhost, port 5432, user postgres, pw secret
```

### Managing background containers
```bash
docker ps                   # is it running?
docker logs myweb           # what has it printed?
docker logs -f myweb        # follow logs live (Ctrl+C stops following, not the container)

docker exec -it myweb bash  # go INSIDE a running container (type exit to leave; it keeps running)

docker stop myweb           # stop (container still exists)
docker start myweb          # start it again
docker rm -f myweb          # stop + delete in one go
```

### `run` vs `exec` (easy to mix up)
| | What it does |
|---|---|
| `docker run` | **Creates a NEW** container from an image |
| `docker exec` | Runs a command in an **EXISTING, RUNNING** container |

### 🧪 The experiment that leads to Layer 3
```bash
docker exec -it mypg psql -U postgres -c "CREATE TABLE fruits (name text); INSERT INTO fruits VALUES ('mango');"
docker rm -f mypg
# start the same postgres command again...
docker exec -it mypg psql -U postgres -c "SELECT * FROM fruits;"
# → ERROR: relation "fruits" does not exist. Data died with the container.
```

---

## Layer 3: Volumes & Bind Mounts (Storage)

### The problem
A container's files live and die with the container. Delete the box → data gone.

### The intuition
Put the data **outside** the box, and plug it in. The box gets thrown away; the data stays.

Both kinds use the same flag, and it reads the same way as ports:

> **`-v OUTSIDE : INSIDE`**

There are **2 kinds**, and the difference is simply **who owns the storage**:
- **Named volume** → Docker owns it (hidden storage). For **data** like databases.
- **Bind mount** → I own it (a normal folder on my Mac). For **my code/files**.

---

### 3a. Named Volume: Docker-managed storage (like a USB drive)

```
   Container (temporary)              Volume (permanent)
 ┌──────────────────────────┐       ┌──────────────┐
 │ /var/lib/postgresql/data │ ────► │   pgdata     │
 └──────────────────────────┘ plug  │ survives rm  │
                                    └──────────────┘
```

```bash
docker run -d \
  --name mypg \
  -e POSTGRES_PASSWORD=secret \
  -p 5432:5432 \
  -v pgdata:/var/lib/postgresql/data \
  postgres:17
```
- `pgdata` = a **name** I pick. Docker creates the volume if it doesn't exist.
- `/var/lib/postgresql/data` = where Postgres 17 keeps its files **inside the box** (the image's Docker Hub page says which path to use).

**The experiment again, with a volume:** create the fruits table → `docker rm -f mypg` → run the **same command with the same `-v pgdata:...`** → `SELECT * FROM fruits;` → **mango is still there.** ✅
New box, same data, because the data was never in the box.

```bash
docker volume ls               # list volumes
docker volume inspect pgdata   # details
docker volume rm pgdata        # ⚠️ deletes the DATA permanently
docker volume prune            # ⚠️ deletes ALL volumes not used by a container
```
Deleting a container **never** deletes its named volume. I must delete volumes on purpose. (Safe, but old ones pile up, so check `docker volume ls` sometimes.)

---

### 3b. Bind Mount: share a folder from my Mac (a window)

**This part confused me at first, so here it is slowly.**

**Step 1: the problem.** A container **cannot see any file on my Mac**. It's sealed.
```bash
docker run --rm ubuntu ls /stuff     # ❌ No such file or directory
```

**Step 2: the idea.** A bind mount = **a window in the box's wall that looks into one folder on my Mac**.
It is **not a copy**. It is **the same folder, seen from two places**.

```
     MY MAC                            CONTAINER (box)
 /Users/kundan/Developer/Docker  ◄═══ same folder ═══►  /stuff
        note.txt                                         note.txt
```

**Step 3: the flag.**
```
-v "$PWD":/stuff
   ──┬───  ──┬──
     │       └── what the folder is CALLED inside the box (any name I like)
     └────────── a real folder on my Mac ($PWD = "the folder I'm in right now")
```

**Step 4: what I tried and saw** (from `/Users/kundan/Developer/Docker`):
```bash
echo "hello from mac" > note.txt

# box reads my Mac's file
docker run --rm -v "$PWD":/stuff ubuntu cat /stuff/note.txt
# → hello from mac

# box writes a file → it appears on my Mac (even after the box is deleted)
docker run --rm -v "$PWD":/stuff ubuntu bash -c "echo 'hi from container' > /stuff/reply.txt"
cat reply.txt
# → hi from container

# live: inside a box in terminal 1, create a file on the Mac in terminal 2,
# then run `ls /stuff` in terminal 1 → the file is there instantly
docker run -it --rm -v "$PWD":/stuff ubuntu bash
```

**Step 5: why it's useful. My code is on my Mac, and the tools are in the box.**
```bash
echo 'print("i ran inside docker")' > hello.py
docker run --rm -v "$PWD":/app python:3.13 python /app/hello.py
# Python isn't installed on my Mac, but Python in the box runs my Mac's file.
```

### Reading that command piece by piece (this tripped me up)
```
docker run  --rm  -v "$PWD":/app  python:3.13  python /app/hello.py
            ────  ──────────────  ───────────  ───────────────────
              1         2              3                4
```
| # | Piece | Meaning |
|---|---|---|
| 1 | `--rm` | Delete the box when done |
| 2 | `-v "$PWD":/app` | **The mount only.** My current Mac folder → appears as `/app` in the box |
| 3 | `python:3.13` | **The image** (which box to start). NOT part of the mount! |
| 4 | `python /app/hello.py` | **The command** to run inside the box |

**What confused me:**
- ❓ *Is `python:3.13` part of the mount?* → **No.** Pieces are separated by spaces. The mount is only `"$PWD":/app`.
- ❓ *Why are there two `:`?* → **Same symbol, different jobs.** In `-v` it means *Mac folder : box folder*. In `python:3.13` it means *image name : version*.
- ❓ *What is `/app`?* → **Just a name I choose** for the folder inside the box. `/code`, `/xyz` work too. **Rule: the name in the mount (piece 2) must match the path in the command (piece 4).**
  ```bash
  docker run --rm -v "$PWD":/xyz python:3.10 python /xyz/hello.py   # works the same
  ```

**Extras:**
- `:ro` on the end = read-only (the box can read my files but not change them): `-v "$PWD":/app:ro`
- `-w /app` = set the working directory inside the box (like `cd /app` first)

---

### Named volume vs bind mount

| | Named volume | Bind mount |
|---|---|---|
| Looks like | `-v pgdata:/path` | `-v "$PWD":/path` or `-v ~/folder:/path` |
| Who owns the files | Docker (hidden) | Me (normal Mac folder, visible in Finder) |
| Use for | **Data** (databases) | **My code / files** I edit |
| Speed on Mac | Faster | A bit slower for heavy I/O |

**How Docker tells them apart:** if the left side starts with `/`, `~`, `.`, or `$PWD` it's a **path** → bind mount. A plain word is a **name** → volume.

> **Rule of thumb: named volume for data, bind mount for code.**

---

## Layer 4: Dockerfile (Build My Own Image)

### The problem
With a bind mount, my code **stays on my Mac**. The box only **borrows** it. If I send someone the command, it won't work for them because they don't have my folder.

### The intuition: packing a lunchbox 🍱
A **Dockerfile** is a list of **packing instructions**:
1. Take a box that already has Python in it
2. Put my files inside
3. Stick a note on the lid: "when opened, run this"

After packing, the box carries **everything**: Python + my code + libraries + what to run. Anyone can run it with one command, with no `-v` and no setup.

```
Dockerfile ──docker build──► Image ──docker run──► Container
 (recipe /                 (packed               (box opened
  packing list)             lunchbox)             and running)
```

---

### 4a. Simplest example: one Python file

```
Docker/
├── Dockerfile     ← exact name: capital D, no extension
└── hello.py
```

**`Dockerfile`:**
```dockerfile
FROM python:3.13-slim
WORKDIR /app
COPY hello.py .
CMD ["python", "hello.py"]
```

| Line | Meaning |
|---|---|
| `FROM python:3.13-slim` | Start from a box that already has Python 3.13 (`slim` = smaller version) |
| `WORKDIR /app` | Inside the box, create folder `/app` and `cd` into it |
| `COPY hello.py .` | Copy `hello.py` **from my Mac** into the box (`.` = current folder = `/app`) |
| `CMD ["python", "hello.py"]` | The note on the lid: when the box starts, run `python hello.py` |

```bash
docker build -t myhello .          # pack the lunchbox (build the image)
docker images                      # myhello is in the list, next to nginx/python
docker run --rm myhello            # open it → i ran inside docker
docker run --rm -it myhello bash   # look inside: pwd → /app, ls → hello.py
```

**Before vs now:**
```bash
# Bind mount: borrowing the file from my Mac
docker run --rm -v "$PWD":/app python:3.13 python /app/hello.py

# Own image: everything is already packed inside
docker run --rm myhello
```

---

### 4b. COPY: only what I name goes in

```
COPY  hello.py   .
      ───┬────   ┬
       from     to
      (Mac)    (box: current folder = /app)
```
If my folder has 200 files, `COPY hello.py .` puts **only `hello.py`** in the box, with the **same name**. The other 199 stay on my Mac.

```dockerfile
COPY hello.py .                       # one file
COPY hello.py utils.py config.json .  # several files
COPY hello.py main.py                 # copy + RENAME (then CMD must say main.py)
COPY src/ ./src/                      # a folder
COPY . .                              # EVERYTHING in the folder → /app (most common in real projects)
```

**`.dockerignore`:** when using `COPY . .`, keep junk out. Create `.dockerignore` next to the Dockerfile (works like `.gitignore`):
```
.DS_Store
.venv
.git
__pycache__
qdrant_storage
```

---

### 4c. The image is a COPY, so edits need a rebuild ⚠️

The image holds a **snapshot made at build time**. Edit `hello.py` → `docker run` still shows the **old** output.
```bash
docker build -t myhello .    # repack
docker run --rm myhello      # now shows the new version
```

| | Bind mount (`-v`) | Dockerfile `COPY` |
|---|---|---|
| Code lives | On my Mac (shared live) | Inside the image (a copy) |
| Edit → see change | Instantly | Must **rebuild** |
| Good for | Developing / trying things | Packaging / sharing / shipping |

---

### 4d. RUN vs CMD ⭐

> **`RUN` = while PACKING (build time, once, result saved in the image)**
> **`CMD` = every time the box is OPENED (run time, every start)**

🍱 Lunchbox: `RUN` = "put rice in the box" (done once while packing). `CMD` = "eat the rice" (done every time someone opens it).

```dockerfile
FROM python:3.13-slim
WORKDIR /app
RUN echo "I was created during BUILD" > build_note.txt
COPY hello.py .
CMD ["python", "hello.py"]
```
```bash
docker build --progress=plain -t myhello .   # RUN executes now; you can see it in the output
                                             # (--progress=plain = show full output)
docker run --rm myhello     # CMD runs → i ran inside docker
docker run --rm myhello     # CMD runs again. RUN does NOT run again
docker run --rm myhello cat build_note.txt   # → I was created during BUILD (RUN's result is saved in the image)
```

**My test question and answer:**
```dockerfile
FROM python:3.13-slim
RUN echo "A"
CMD ["echo", "B"]
```
Build once, run 5 times → **A prints once** (during build), **B prints 5 times** (once per run).
(Building again with no changes → A usually doesn't print at all, because the step is `CACHED`.)

| | RUN | CMD |
|---|---|---|
| Happens during | `docker build` | `docker run` |
| How often | Once per build | Every container start |
| Result | **Saved into the image** | Just runs the program |
| How many allowed | As many as needed | **Only one** (if two, the last wins) |
| Used for | Installing things, setup | Starting my app |
| Override? | No | Yes: `docker run myhello ls` runs `ls` instead |

**Why it matters:** `pip install` goes in **RUN** (install once, saved). If it were in CMD, it would re-download libraries **every single start**.

---

### 4e. The `.` in `docker build -t myapi .` ⭐

**What confused me:** "I'm naming it `myapi`, so why the dot?"
The dot is **not part of the name**. It's a separate piece:
```
docker build  -t myapi  .
              ───┬────  ┬
                 │      └── WHERE are my files?  "." = the folder I'm in now
                 └── NAME of the image
```

**Why Docker needs it:** `COPY main.py .` says "take main.py **from my Mac**", but my Mac has thousands of folders. Which `main.py`? The `.` answers it: **"take files from THIS folder."**
That folder is called the **build context**. It's the table with all the ingredients, and Docker can **only** pack from that table.

```
fastapi-app/          ← the "." (Docker can only see inside here)
├── main.py           ← COPY main.py finds this
├── requirements.txt
└── Dockerfile        ← docker build reads this
```

| Command | Meaning |
|---|---|
| `docker build -t myapi .` | Build from **this** folder |
| `docker build -t myapi fastapi-app` | Build from the `fastapi-app` folder (also works) |
| `docker build -t myapi` | ❌ Error: no folder given |

> **Habit: `cd` into the folder with the Dockerfile, and always end with ` .`**

---

### 4f. A real web app: FastAPI

Same 4 lines as before, plus **2 new things**: installing libraries (`RUN pip install`) and a port.

```
fastapi-app/
├── main.py
├── requirements.txt
└── Dockerfile
```

**`main.py`**
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def home():
    return {"message": "Hello from FastAPI inside Docker!"}
```

**`requirements.txt`**
```
fastapi
uvicorn
```
(fastapi = the framework, uvicorn = the server that actually runs it. Neither is on my Mac, and they don't need to be.)

**`Dockerfile`**
```dockerfile
FROM python:3.13-slim
WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY main.py .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```
FROM python:3.13-slim                → box with Python
WORKDIR /app                         → make /app, go inside
COPY requirements.txt .              → copy the LIBRARY LIST in
RUN pip install -r requirements.txt  → install libraries (BUILD time, once)
COPY main.py .                       → copy MY CODE in
CMD [...]                            → start the server (every RUN)
```

**The CMD line:**
```
uvicorn  main:app  --host 0.0.0.0  --port 8000
         ───┬────  ──────┬──────   ─────┬─────
            │            │              └── listen on port 8000 inside the box
            │            └── accept visitors from OUTSIDE the box
            └── in file main.py, use the variable named app
```

**Why `--host 0.0.0.0`?** By default the server only accepts visitors from **inside its own box**. My browser is on my Mac, **outside** the box. `0.0.0.0` = "accept visitors from anywhere". Without it, the page won't load even with a correct `-p`.

```bash
docker build -t myapi .
docker run --rm -p 8000:8000 myapi
# http://localhost:8000       → {"message": ...}
# http://localhost:8000/docs  → FastAPI's automatic docs page
```

---

### 4g. Layer caching: why `requirements.txt` is copied FIRST ⭐

Every Dockerfile line = a **layer**. Docker **remembers (caches)** each layer. On rebuild, it reuses layers **until it reaches a line whose files changed**, and redoes everything from there down.

```
COPY requirements.txt .               ← unchanged → CACHED ⚡
RUN pip install -r requirements.txt   ← unchanged → CACHED ⚡  (no waiting for pip!)
COPY main.py .                        ← main.py changed → REDO
CMD [...]                             ← redo
```
My code changes **often** and libraries change **rarely**, so: **library list → install → my code.**
If I wrote `COPY . .` before `pip install`, every tiny code edit would re-run the whole install. 🐢

Test it: change the message in `main.py` → `docker build -t myapi .` → see `CACHED` on the pip step, and the build takes about a second.

---

### 4h. Using uv instead of pip (to try later)

The `python` image has pip but **not uv**, so first get uv into the box:
```dockerfile
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/uv
```
Normally `COPY` takes files from my Mac. `COPY --from=<image>` takes a file **from another image**. The uv team publishes an image containing the `uv` program, and I copy just that one file into my box.

**Way 1: quick swap (keep `requirements.txt`)**
```dockerfile
FROM python:3.13-slim
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/uv
WORKDIR /app

COPY requirements.txt .
RUN uv pip install --system -r requirements.txt

COPY main.py .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```
`--system` = install into the box's main Python. (No virtualenv needed, because the box is already isolated.)

**Way 2: proper uv project (`pyproject.toml` + `uv.lock`)**
`uv.lock` = **exact** versions, so the box gets exactly what my Mac has.
```bash
mkdir fastapi-uv && cd fastapi-uv
uv init --bare              # creates only pyproject.toml
uv add fastapi uvicorn      # creates uv.lock (+ a local .venv)
cp ../fastapi-app/main.py .
```
`.dockerignore` (**important**: my Mac's `.venv` has Mac builds, and the box is Linux):
```
.venv
```
```dockerfile
FROM python:3.13-slim
COPY --from=ghcr.io/astral-sh/uv:latest /uv /bin/uv
WORKDIR /app

COPY pyproject.toml uv.lock ./
RUN uv sync --locked --no-install-project

COPY main.py .

CMD ["uv", "run", "uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```
- `--locked` = fail if `uv.lock` is out of date (no surprise versions)
- `--no-install-project` = install only the libraries now; my code comes next (keeps caching working)
- `uv run` = run uvicorn from the `.venv` that `uv sync` made inside the box

| | Way 1 | Way 2 |
|---|---|---|
| Library list | `requirements.txt` | `pyproject.toml` + `uv.lock` |
| Same exact versions as my Mac | Not guaranteed | ✅ Yes |
| Good for | Quick experiments | Real uv projects |

---

## Layer 5: Docker Compose (Many Containers, One File)

### The problem
App + database = long commands to type and remember every time:
```bash
docker build -t myapi .
docker run -d --name mypg -e POSTGRES_PASSWORD=secret -v pgdata:/var/lib/postgresql/data postgres:17
docker run -d --name api -p 8000:8000 myapi
```
Plus, with plain `docker run` like this, the app **can't even find** the database (see "the `@db` idea" below).

### The intuition
**Compose = my `docker run` commands written down neatly in one file.**
The file is **`compose.yaml`**. Then:
```bash
docker compose up      # start everything
docker compose down    # stop + remove everything
```
Nothing new to learn conceptually. It's the same `-p`, `-e`, `-v` from Layers 2–4, just in YAML.

---

### 5a. One service: just my FastAPI app

```yaml
services:
  api:
    build: .
    ports:
      - "8000:8000"
```
```
docker build -t myapi .          →   build: .
docker run -p 8000:8000 myapi    →   ports:
                                       - "8000:8000"
```
| Line | Meaning |
|---|---|
| `services:` | The list of containers I want |
| `api:` | A name **I** choose for this container |
| `build: .` | Build from the Dockerfile in this folder (same `.` as `docker build`!) |
| `ports:` | Same as `-p` |

⚠️ **YAML = spaces for indentation, never tabs.** Indentation shows ownership: `build` and `ports` are indented under `api`, so they belong to `api`.

```bash
docker compose up --build     # --build = rebuild my image first (use after code changes)
# Ctrl+C to stop
docker compose down
```

---

### 5b. Two services: FastAPI + Postgres

**`requirements.txt`**
```
fastapi
uvicorn
psycopg[binary]
```
(`psycopg` = the library Python uses to talk to Postgres)

**`main.py`**
```python
import os

import psycopg
from fastapi import FastAPI

app = FastAPI()
DATABASE_URL = os.environ["DATABASE_URL"]


@app.get("/")
def home():
    return {"message": "Hello from FastAPI inside Docker!"}


@app.post("/fruits/{name}")
def add_fruit(name: str):
    with psycopg.connect(DATABASE_URL) as conn:
        conn.execute("CREATE TABLE IF NOT EXISTS fruits (name text)")
        conn.execute("INSERT INTO fruits VALUES (%s)", (name,))
    return {"added": name}


@app.get("/fruits")
def list_fruits():
    with psycopg.connect(DATABASE_URL) as conn:
        conn.execute("CREATE TABLE IF NOT EXISTS fruits (name text)")
        rows = conn.execute("SELECT name FROM fruits").fetchall()
    return {"fruits": [row[0] for row in rows]}
```
The DB address comes from the environment variable `DATABASE_URL`, which is set in `compose.yaml`.

**`compose.yaml`**
```yaml
services:
  api:
    build: .
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql://postgres:secret@db:5432/postgres
    depends_on:
      - db

  db:
    image: postgres:17
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

| compose.yaml | Same as |
|---|---|
| `build: .` | `docker build .` (my own image) |
| `image: postgres:17` | Ready-made image (`docker run postgres:17`) |
| `ports:` | `-p` |
| `environment:` | `-e` |
| `volumes:` (inside a service) | `-v` |
| `volumes: pgdata:` (at the bottom, top level) | Declares the named volume so Compose creates it |
| `depends_on: - db` | **New:** start `db` before `api` |

---

### 5c. The `@db` idea: how containers find each other ⭐⭐

```
postgresql://postgres:secret@db:5432/postgres
             ───┬──── ──┬─── ┬─ ──┬─
              user  password │  port
                             └── HOST = "db"   ← why not localhost??
```

- **Inside a container, `localhost` means THAT container itself.** If the `api` box looks for Postgres at `localhost`, it searches **inside the api box**, and Postgres isn't there.
- Compose puts all services on a **shared private network** where each one can find the others **by service name**. I named the database service `db`, so its address is `db`.

```
┌──────────── Compose private network ────────────┐
│                                                 │
│   ┌─────────┐      "db:5432"      ┌─────────┐   │
│   │   api   │ ──────────────────► │   db    │   │
│   └─────────┘                     └─────────┘   │
│        ▲                                        │
└────────┼────────────────────────────────────────┘
         │  ports: "8000:8000"
   My Mac (browser)
```

- `db` has **no `ports:`**. My Mac doesn't need to reach it directly. Only `api` talks to it, through the private network.
- Add `ports: - "5432:5432"` to `db` only if I want to open the DB from my Mac (TablePlus, DBeaver…).

> **Rule: Mac → container uses `localhost:<mac port>`. Container → container uses `<service name>:<box port>`.**

---

### 5d. What I ran and saw

```bash
docker compose up --build
```
- Logs from **both** services in one terminal, color-coded by name
- `http://localhost:8000/docs` → **POST /fruits/{name}** → Try it out → `mango` → Execute (then `apple`)
- `http://localhost:8000/fruits` → `{"fruits": ["mango", "apple"]}` ✅

**Volume proof:** `Ctrl+C` → `docker compose down` → `docker compose up` → `/fruits` still shows mango and apple.

---

### 5e. How to write a compose file myself (the method)

**Step 1:** Think "what would the `docker run` command be?"
**Step 2:** Translate each flag:
```
docker run  --name x  -p 8080:80  -e KEY=val  -v data:/path  image:tag
               │         │            │            │             │
services:      ▼         │            │            │             │
  x:  ◄────────┘         │            │            │             │
    image: image:tag ◄───┼────────────┼────────────┼─────────────┘   (or build: . for my own)
    ports:               │            │            │
      - "8080:80"  ◄─────┘            │            │
    environment:                      │            │
      KEY: val  ◄─────────────────────┘            │
    volumes:                                       │
      - data:/path  ◄──────────────────────────────┘
```
**Step 3:** Check 3 things:
1. Named volume used? → declare it under a top-level `volumes:` at the bottom
2. Does one service talk to another? → use the **service name** as the host, **not** `localhost`
3. Does one need the other started first? → `depends_on:`

**Template:**
```yaml
services:
  <name>:
    image: <image:tag>        # OR build: .
    ports:
      - "<mac>:<box>"
    environment:
      <KEY>: <value>
    volumes:
      - <volume-name-or-./path>:<path-in-box>
    depends_on:
      - <other-service>

volumes:
  <volume-name>:
```
In compose files, bind mounts can use relative paths like `./site:/usr/share/nginx/html` (no `$PWD` needed).

---

### 5f. Compose commands

```bash
docker compose up               # start everything (logs in terminal, Ctrl+C stops)
docker compose up -d            # start in background
docker compose up --build       # rebuild my image first (after code changes!)

docker compose ps               # what's running
docker compose logs -f          # follow logs of all services
docker compose logs -f api      # logs of one service

docker compose exec db psql -U postgres    # go inside a service (like docker exec)

docker compose down             # stop + remove containers (volume/data KEPT)
docker compose down -v          # ⚠️ also delete volumes → DATA GONE
```
In compose commands, use the **service name** (`api`, `db`), not container IDs.

---

## Common Errors & Fixes

| Error / symptom | Cause | Fix |
|---|---|---|
| `port is already allocated` | Something on my Mac already uses that port | Change only the **left** side: `-p 8081:80` |
| Page won't load, but `-p` looks right | Server listens only inside its box | Start it with `--host 0.0.0.0` |
| Port 5000 busy on Mac | AirPlay Receiver uses 5000 | Use another Mac port, e.g. `-p 5001:5000` |
| Edited code but output is the same | Image is a copy from build time | Rebuild: `docker build ...` / `docker compose up --build` |
| `failed to read dockerfile` | Wrong folder / forgot the `.` | `cd` into the Dockerfile's folder; end with ` .` |
| `No such file or directory` inside box | Folder not mounted, or path name mismatch | Check `-v X:/app` matches the `/app/...` used in the command |
| App can't reach the DB in Compose | Used `localhost` | Use the **service name** (`db`) as the host |
| `yaml: line X: ...` | Indentation wrong / tabs | Use spaces only; align children under parents |
| Data disappeared | No volume, or ran `down -v` / `volume rm` | Use a named volume; don't use `-v` with `down` unless intended |
| Postgres container exits immediately | Missing password | Add `POSTGRES_PASSWORD` |
| Weird errors with `COPY . .` in uv project | Mac `.venv` copied into Linux box | Add `.venv` to `.dockerignore` |

---

## Practice Exercises

For each one: `mkdir exN && cd exN`, write `compose.yaml` **without looking**, then `docker compose up` / `docker compose down`.

**Ex 1: Nginx serving my own page (easy)**
Goal: `http://localhost:8080` shows my HTML. Image `nginx`, listens on **80**, serves files from `/usr/share/nginx/html`. Create `site/index.html`. Hint: bind mount.

<details><summary>Solution</summary>

```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"
    volumes:
      - ./site:/usr/share/nginx/html:ro
```
</details>

**Ex 2: Qdrant (easy)**
Image `qdrant/qdrant`, port **6333**, stores data at `/qdrant/storage`. Use a **named volume**. Check: `http://localhost:6333/dashboard`

<details><summary>Solution</summary>

```yaml
services:
  qdrant:
    image: qdrant/qdrant
    ports:
      - "6333:6333"
    volumes:
      - qdrant_data:/qdrant/storage

volumes:
  qdrant_data:
```
</details>

**Ex 3: Postgres + Adminer, a DB web UI (medium, 2 services)**
Postgres `postgres:17` (password + named volume, **no ports**). Adminer image `adminer`, listens on **8080**. Log in at `http://localhost:8080`: System = PostgreSQL, Server = **?**, user `postgres`, password = mine.
Think: what is Postgres's address **from Adminer's point of view**?

<details><summary>Solution</summary>

```yaml
services:
  db:
    image: postgres:17
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data

  adminer:
    image: adminer
    ports:
      - "8080:8080"
    depends_on:
      - db

volumes:
  pgdata:
```
Server = `db` (the service name, not localhost).
</details>

**Ex 4: Everything together (challenge)**
Add Adminer to the `fastapi-app` compose file. Add fruits at `localhost:8000/docs`, then see the `fruits` table in Adminer at `localhost:8080`.

---

## Cheat Sheet

### The 5 layers in one breath
1. **Image** = recipe, **container** = running box. `docker run image`.
2. Box is sealed. `-d` = background, `-p MAC:BOX` = door for network, `-e` = settings.
3. Box files die with it. `-v name:/path` = Docker-owned data, `-v "$PWD":/path` = window into my Mac folder.
4. **Dockerfile** packs my code into my own image. `RUN` = at build (once), `CMD` = at start (every time). `docker build -t name .` where `.` = "my files are here".
5. **Compose** = all the `docker run` commands in one `compose.yaml`. Services find each other **by service name**.

### Commands
```bash
# Run
docker run -it --rm image bash                                   # try & throw away
docker run -d --name x -p 8080:80 -e KEY=val -v data:/path image:tag
docker run --rm -v "$PWD":/app -w /app python:3.13 python file.py

# Look
docker ps / docker ps -a / docker images / docker logs -f x

# Interact
docker exec -it x bash

# Lifecycle
docker stop x / docker start x / docker rm -f x / docker rmi image

# Build
docker build -t name .
docker build --progress=plain -t name .    # see full output

# Volumes
docker volume ls / docker volume rm name / docker volume prune

# Clean everything unused
docker system prune

# Compose
docker compose up --build / up -d / ps / logs -f [svc] / exec svc bash / down / down -v
```

### Dockerfile
```dockerfile
FROM image:tag                  # starting box
WORKDIR /app                    # create + cd into folder
COPY requirements.txt .         # library list first (caching!)
RUN pip install -r requirements.txt   # BUILD time, once, saved
COPY . .                        # then my code
EXPOSE 8000                     # just a note about the port (doesn't open anything; -p does)
CMD ["cmd", "arg"]              # RUN time, every start
```

### compose.yaml
```yaml
services:
  api:
    build: .
    ports: ["8000:8000"]
    environment:
      DATABASE_URL: postgresql://postgres:secret@db:5432/postgres
    depends_on: [db]
  db:
    image: postgres:17
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:
```

### The 4 colons / symbols that confuse me
| Where | Means |
|---|---|
| `-p 8080:80` | Mac port : box port |
| `-v "$PWD":/app` | Mac folder (or volume name) : box folder |
| `python:3.13` | image name : version |
| `docker build -t name .` | `.` = build context (where my files are), **not** part of the name |

---

## 🗺️ Progress

- [x] Layer 1: Images, containers, run, ps, rm
- [x] Layer 2: Background (`-d`), ports (`-p`), env (`-e`), logs, exec
- [x] Layer 3: Named volumes & bind mounts (`-v`)
- [x] Layer 4: Dockerfile (single file, COPY, RUN vs CMD, build context, FastAPI, caching)
- [x] Layer 5: Docker Compose (FastAPI + Postgres, service-name networking)
- [ ] Try: uv inside Docker (Layer 4h)
- [ ] Practice: Compose exercises 1–4
