# Ghid folosinta Org Chart Dumagas

## 1) Cum funcționează, pe scurt

Organigrama citește un singur fișier: `data/orgchart.tsv` (fișier cu tab între coloane).

Fiecare linie = un nod (om, container de echipă sau un element special din triunghi).

Legăturile se fac cu `id` și `parentId`.

La încărcare, codul pune automat rapoartele directe ale CEO în două benzi: **Administrative Business Units** și **Productive Business Units**, după coloana `businessLine`.

> ⚠️ Nu editați în TSV nodurile “BU_ADMIN” și “BU_PROD”. Sunt create automat de cod.  
> Triunghiul Consiliului de Administrație (CA) se face tot automat din rândurile marcate cu `boardVertex`.

---

## 2) Coloanele din fișier (ce sunt, dacă sunt obligatorii, ce fac)

| Coloană              | Obligatori u?                        | Ce pui                                           | Ce face în aplicație                                                                 |
|----------------------|--------------------------------------|--------------------------------------------------|--------------------------------------------------------------------------------------|
| `id`                 | ✅ Da                                 | Un identificator unic (ex: `7`, `L1_3B`)         | Este “numele intern” al nodului. Trebuie să fie unic în tot fișierul.               |
| `parentId`           | ➕ dacă nu e nod special din triunghi | id-ul șefului / containerului părinte            | Construiește arborele. Dacă pui greșit, nodul apare în altă parte sau deloc.        |
| `name`               | ✅ Da pentru persoane                 | Numele (ex: “Roxana Danciu”)                     | Primul rând din card.                                                                |
| `title`              | Opțional (recomandat)                | Funcția (ex: “Transport Manager”)                | Al doilea rând din card.                                                             |
| `department`         | Opțional                             | “Administrative” / “Productive” sau text liber   | Folosit în căutare; nu schimbă poziționarea.                                         |
| `location`           | Opțional                             | Oraș sau țară                                    | Linie în card (și în tooltip).                                                       |
| `photo`              | Opțional                             | Cale imagine (ex: `assets/7.jpg`)                | Poza rotundă. Dacă lipsește, afișăm inițiale.                                        |
| `boardVertex`        | Doar pentru triunghi                 | `top-left`, `top-right`, `bottom`                | Marchează acționarii (stânga/dreapta sus) și CEO (jos).                              |
| `boardAttach`        | Opțional                             | `left`, `right` (poți pune ambele: `left,right`) | Trasează linie punctată din triunghi spre persoană (ex: Emil). Vizibilă doar când persoana e vizibilă. |
| `businessLine`       | Pentru rapoarte directe ale CEO      | `administrative` sau `productive` (minuscul)     | Decide în ce bandă (BU) intră sub CEO. Pentru restul organizației e dedus din părinte. |
| `buOrder`            | Opțional                             | Număr întreg (ex: 1, 2, 3…)                      | Controlează ordinea rapoartelor directe ale CEO (mai mic = mai la stânga).          |
| `email`              | Opțional                             | Email                                            | Linie în card (și la acționari).                                                     |
| `phone`              | Opțional                             | Telefon                                          | Linie în card.                                                                       |
| `nodeType`           | Pentru “containere” (layere)         | Scrie `layer`                                    | Marchează un rând ca **container de echipă** (nu persoană).                          |
| `layer`              | Alternativ la `nodeType`             | Scrie `layer`                                    | E suficient să pui `layer` în oricare dintre `nodeType`/`layer`.                     |
| `order`              | Ignorat momentan                     | —                                                | Momentan nu e folosit de cod (ordinea containerelor se face alfabetic după `name`). |

🧠 **Reține:** pentru **LAYERE** pui `nodeType=layer` (sau `layer=layer`) și **NU** completați email/phone/poză pentru ele (sunt “cutiuțe” de grupare, nu oameni). La layere afișăm doar numele și **“total în unitate”**.

---

## 3) Rețete (pas cu pas)

### A) Adaug o persoană nouă în organizație
1. Alege un `id` unic (ex: `L1_3B6`).  
2. Decide unde trebuie să apară:  
   - Dacă persoana e **raport direct la CEO** → `parentId = 1` (id-ul CEO) și **obligatoriu** `businessLine = administrative` sau `productive` (ca să intre în banda corectă).  
   - Dacă persoana e într-o echipă **sub un layer** → `parentId = id-ul layerului` (ex: `L1_3B`).  
3. Completează `name`, `title`, opțional `location`, `email`, `phone`, `photo`.  
4. Salvează `orgchart.tsv`. La refresh, apare la locul potrivit.

### B) Mut pe cineva la alt șef / altă echipă
- **Schimbă DOAR** `parentId` spre noul părinte.  
- Dacă devine **raport direct la CEO**, pune corect și `businessLine` (altfel rămâne în banda implicită).

### C) Șterg pe cineva
- Șterge linia.  
- Dacă avea subordonați, aceia vor “urca” la părinte? **Nu** – vor rămâne fără părinte și **NU** vor mai apărea.  
  Mută-le `parentId` înainte să ștergi, dacă vrei să-i păstrezi.

### D) Adaug un layer (container de echipă)
- Creează o linie nouă:  
  - `id` unic (ex: `L1_3B`)  
  - `parentId` = **managerul** care va conține layerul (ex: `3` pentru Monica)  
  - `name` = numele sub-structurii (ex: “FINANCIAR CONTABIL”)  
  - `nodeType = layer` (sau `layer = layer`)  
  - (opțional) `businessLine` la fel ca managerul (nu e obligatoriu)  
- Apoi pentru oamenii din acel layer pui `parentId = id-ul` acestui layer.  
- Layerele apar ca niște cutiuțe; sub ele se afișează oamenii. La layere scrie **“total în unitate: X”**.

### E) Un manager are două layere (ca Emil sau Monica)
- Faci **două rânduri layer** sub manager (ex: `L1_2A` și `L1_2B` cu `parentId = 2`).  
- Oamenii din fiecare grup merg sub **containerul lor** (`parentId = L1_2A` sau `L1_2B`).

### F) Adaug poză
- Pui fișierul în `assets/` (ex: `assets/claudia.jpg`).  
- În `photo` scrii **exact** calea: `assets/claudia.jpg`.  
- **Important:** pe server (GitHub Pages/alt server), imaginile trebuie să fie din **același domeniu** ca pagina ca să meargă exportul PNG/PDF.

### G) Linia punctată din triunghi (ex: Emil)
- În rândul persoanei pune `boardAttach = left` (sau `right`).  
- Linia se vede **doar dacă** persoana e vizibilă în organigramă (adică ai deschis ramura până la ea).

### H) Reordonez rapoartele directe ale CEO
- În rândurile cu `parentId = 1` (CEO), pune `buOrder` cu numere (1,2,3…).  
  **Număr mai mic = mai la stânga.**  
- Pentru restul nivelurilor, ordinea e **alfabetică** după `title` apoi `name`.

---

## 4) Ce se întâmplă când editezi o coloană (pe scurt)

- **`id`** – dacă îl schimbi, trebuie să actualizezi și **toate `parentId`**-urile care se refereau la el.  
- **`parentId`** – mută nodul în alt loc din arbore.  
- **`name`/`title`/`location`/`email`/`phone`/`photo`** – schimbă ce se vede în card; **nu** modifică poziția.  
- **`boardVertex`** – pune nodul în triunghi (doar acționari și CEO): `top-left`, `top-right`, `bottom`. Nu pune altor oameni.  
- **`boardAttach`** – arată legătură **punctată** din triunghi către persoană (`left`/`right`).  
- **`businessLine`** – contează doar dacă nodul este **raport direct al CEO** (decide banda: Administrative/Productive). Pentru restul, moștenim automat.  
- **`buOrder`** – sortează rapoartele **directe** ale CEO.  
- **`nodeType/layer = layer`** – marchează rândul ca **container** (nu numărăm aceste noduri ca rapoarte). Pe card scrie “total în unitate”.  
- **`order`** – ignorat de codul curent.

---

## 5) Greșeli tipice & cum le eviți

- ❌ `id` duplicat → noduri dispar sau se amestecă. **Să fie unic.**  
- ❌ `parentId` spre un `id` care nu există → nodul **nu apare**.  
- ❌ Uitați tab-urile: salvați **TSV (tab delimited)**, nu CSV cu virgule.  
- ❌ Căi poze greșite (ex: `assets\poza.jpg` pe Windows). Folosiți slash `/`: `assets/poza.jpg`.  
- ❌ Setări greșite de capitalizare: `businessLine` trebuie să fie **exact** `administrative` sau `productive` (minuscul).  
- ❌ Editat în Excel și salvat cu separatori ciudați → verificați să fie **tab** între coloane.

---

## 6) Checklist înainte de a salva

- [ ] Toate `id`-urile sunt **unice**.  
- [ ] Fiecare `parentId` **există** în coloana `id` (sau e gol doar pentru nodurile din triunghi/CEO).  
- [ ] Dacă e raport direct la CEO → are `businessLine` = `administrative` sau `productive`.  
- [ ] Layerele au `nodeType = layer` (sau `layer = layer`).  
- [ ] Pozele există în `assets/` și căile sunt corecte.  
- [ ] Fișierul e salvat ca **TSV (tab delimited)**.
