---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Întrebări frecvente

<details>

<summary>Pot procesa imagini de la camere care nu sunt de marca MAPIR cu Chloros?</summary>

Nu, Chloros acceptă doar procesarea imaginilor de la camerele MAPIR — din familiile Survey3 și LATTICE. Vă rugăm să consultați lista [modelelor de camere compatibile](supported-cameras.md) pentru mai multe informații. Oferim procesarea imaginilor de la alte camere pe MAPIR Cloud; consultați lista completă [aici](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Chloros suportă camerele LATTICE?</summary>

Da. Chloros 1.2.0 suportă modulele de cameră LATTICE M3C și M3M de la un capăt la altul: **control în timp real**— descoperiți, conectați, previzualizați și capturați din fila „Camere” a interfeței grafice, `chloros-cli lattice` sau din Python SDK, inclusiv matrice sincronizate cu mai multe camere cu sincronizare temporală PTP — și**prelucrare radiometrică completă** a capturilor (brut → debayering → radianță → reflectanță → indice). Consultați [Camerele acceptate](supported-cameras.md) și [ghidul LATTICE](lattice/README.md).

</details>

<details>

<summary>Pot să-mi calibrez imaginile pentru reflectanță fără o țintă de calibrare?</summary>

**Survey3:** Nu. Fără o imagine a țintei de calibrare capturată în același timp cu imaginile care nu sunt ale țintei, nu veți putea corela valorile pixelilor imaginii cu un procent cunoscut de reflectanță. Dacă, de asemenea, nu includeți înregistrarea de la un senzor de lumină MAPIR, spectrul luminii ambientale nu va fi măsurat, iar rezultatele privind reflectanța nu vor fi precise.**LATTICE:** Da. Reflectanța poate fi raportată la iradianța descendentă măsurată de un senzor de lumină DAQ în locul unui panou (ρ = π·L/E). Atunci când *există* o țintă în cadru care a trecut testul de asigurare a calității, aceasta devine referința absolută în mod implicit (`--reflectance-source auto`). O singură excepție: „Reflexia F988 este calibrată folosind un panou de reflexie din scenă: banda se află în afara intervalului calibrat al senzorului de lumină DAQ, astfel încât Chloros aplică cea mai recentă captură a panoului și o menține între observările panoului.” Consultați [Ținte de calibrare](calibration-targets.md).

</details>

<details>

<summary>Am nevoie de un senzor de lumină DAQ?</summary>

Nu pentru radianță: exporturile de radianță LATTICE provin din calibrarea radiometrică din fabrică a fiecărei camere și nu necesită nici un senzor DAQ, nici o țintă. Pentru **reflectanță**aveți nevoie de o referință pentru lumina ambientală — fie o măsurătoare a luminii descendente efectuată de un senzor de lumină DAQ, fie o țintă de calibrare în cadru. Un senzor DAQ vă permite să obțineți valori calibrate ale**reflectanței fără a plasa panouri în scenă**. Fișierele `.daq` înregistrate sunt corelate automat cu imaginile dvs. pe baza marcajului temporal. Consultați [Țintele de calibrare](calibration-targets.md) și [Referința CLI](reference/cli-reference.md).

</details>

<details>

<summary>Pot folosi Chloros cu un asistent AI (Claude, ChatGPT etc.)?</summary>

Da — acest manual și fișierele CLI/SDK sunt concepute special pentru acest scop:

* Indexul complet al manualului este disponibil la `https://mapir.gitbook.io/chloros/llms.txt`, pentru ca asistenții AI să poată descoperi fiecare pagină.
* Codul Markdown brut al fiecărei pagini este disponibil la adresa paginii respective cu litere mici, urmată de `.md` (de exemplu, `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [Referința CLI](reference/cli-reference.md) și [Referința SDK](reference/sdk-reference.md) sunt scrise pentru a fi utilizate de modelele LLM: indicatori exacți, valori implicite, semantica ieșirii și comenzi care pot fi copiate și lipite.

Consultați [Asistenții AI](ai-assistants.md) pentru a afla cum să direcționați asistentul către Chloros.

</details>

<details>

<summary>Unde sunt salvate fișierele de ieșire procesate?</summary>

Produsele sunt salvate în folderul proiectului, grupate pe cameră și apoi pe format de fișier:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **folder-cameră** — `LATT-<sensor>-<lens>-F<filter>` pentru LATTICE, `<model>_<filter>` (de exemplu, `Survey3N_RGN`) pentru Survey3
* **dosar-format** — `tiff16`, `tiff8`, `png8`, `jpg8` sau `tiff32`
* **dosare de produse** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (întotdeauna sub `tiff32`), `<INDEX>_Index_Images/`**Fișierele exportate păstrează numele fișierului sursă — folderul identifică produsul, nu sufixul numelui de fișier.**Cu CLI, folderul proiectului este creat lângă folderul de intrare, cu excepția cazului în care se specifică `-o`. Rețineți că o execuție `chloros-cli process` care a solicitat produse, dar nu a scris niciunul, afișează `Processing finished but wrote no image products.` și**iese cu un cod de ieșire diferit de zero**, astfel încât scripturile să o poată detecta. Consultați [Formate de imagine de ieșire](output-image-formats.md) și [Referința CLI](reference/cli-reference.md).

</details>

<details>

<summary>Pot edita imaginile înainte de procesare în Chloros?</summary>

Nu. Chloros presupune că datele de intrare nu au fost modificate. Nu modificați numele fișierelor.

</details>

<details>

<summary>Pot seta camerele mele MAPIR și Survey3 pe expunere automată și să procesez imaginile în Chloros?</summary>

Nu. Seturile de date de imagine Survey3 trebuie să aibă o expunere fixă/blocată, deci fără viteză automată a obturatorului sau ISO automat. Toate imaginile de la același model de cameră trebuie să aibă viteză de obturare și ISO (expunere) identice.

Camerele LATTICE nu au această restricție: Chloros controlează expunerea în timp real (Smart AE), iar fiecare captură înregistrează expunerea și amplificarea utilizate efectiv, pe care pipeline-ul radiometric le ia în calcul.

</details>

<details>

<summary>Poate Chloros să proceseze sau să analizeze imagini ortomozice?</summary>

Nu. Sunt acceptate doar imagini individuale de la camera MAPIR, nu imagini asamblate, cum ar fi o hartă ortomozaică.

</details>

<details>

<summary>Cum pot accelera etapa de detectare a țintelor din Chloros?</summary>

În tabelul browserului de fișiere, preselectarea imaginilor țintă din coloana din dreapta va indica programului Chloros să caute ținte de calibrare numai în acele imagini, accelerând considerabil procesarea.

</details>

<details>

<summary>Dacă voi încărca imaginile mele în <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> ar trebui să le procesez în Chloros înainte de încărcare?</summary>

Dacă intenționați să încărcați imaginile pe platforma noastră de procesare online [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), nu editați imaginile înainte de încărcare. Cloud va efectua toate aceleași operațiuni de procesare și multe altele.

</details>

<details>

<summary>Va suporta vreodată MAPIR funcția X? Mi-aș dori foarte mult ca MAPIR să ofere funcția X.</summary>

Suntem întotdeauna interesați să primim feedback cu privire la produsele noastre. Dacă descoperiți o problemă cu produsele noastre sau aveți o sugestie privind modul în care le putem îmbunătăți, vă rugăm să [NE CONTACTAȚI](https://www.mapir.camera/community/contact) pentru a ne împărtăși opiniile dumneavoastră. Cea mai mare parte a activității noastre de cercetare și dezvoltare se bazează pe ascultarea celor mai importante nevoi ale clienților noștri.

</details>

<details>

<summary>Chloros este disponibil pentru Linux?</summary>

Da! Chloros 1.2.0 suportă Linux amd64 (x86_64) și arm64 (NVIDIA Jetson JetPack 6) prin intermediul pachetelor `.deb`. CLI și Python SDK sunt pe deplin acceptate pe Linux, inclusiv controlul în timp real al camerei LATTICE și al senzorului DAQ. Nu există o interfață grafică (GUI) pentru Linux — toată interacțiunea se realizează prin intermediul [CLI](CLI.md) sau [Python SDK](api-python-sdk.md). Consultați [Prezentarea generală a Linux](linux/linux-overview.md) pentru detalii.

</details>

<details>

<summary>Pot rula Chloros pe NVIDIA Jetson?</summary>

Da! Chloros suportă platformele NVIDIA Jetson, inclusiv Jetson Nano, Orin Nano, Orin NX și AGX Orin care rulează JetPack 6. Chloros detectează automat modelul dvs. de Jetson și optimizează strategia de procesare a acestuia. Consultați [Ghidul NVIDIA Jetson](linux/nvidia-jetson-guide.md) pentru instrucțiuni de configurare și implementare.

</details>

<details>

<summary>Chloros se optimizează automat pentru hardware-ul meu?</summary>

Da! Chloros include [Adaptarea dinamică a calculului](processing-architecture/dynamic-compute-adaptation.md), care detectează automat CPU-ul, GPU-ul, memoria RAM și (pe Jetson) senzorii termici. Apoi selectează strategia optimă de procesare — de la `GPU_PARALLEL` pe sistemele cu memorie mare, la `GPU_SINGLE` pe dispozitivele cu resurse limitate, până la `CPU_PARALLEL` pe sistemele fără GPU NVIDIA. Nu este necesară nicio configurare manuală.

</details>

<details>

<summary>Ce este pipeline-ul de procesare cu 4 fire?</summary>

Chloros utilizează o arhitectură de tip pipeline cu 4 fire de execuție pentru utilizatorii Chloros+: Firul 1 (Detecție) încarcă imaginile și detectează țintele de calibrare, Firul 2 (Calibrare) calculează calibrarea reflectanței, Firul 3 (Prelucrare) efectuează debayeringul accelerat de GPU și calculul indicelui, iar Firul 4 (Export) scrie fișierele de ieșire. Mai multe imagini pot fi procesate simultan în fire de execuție diferite pentru un randament maxim. Consultați [Procesul de prelucrare](processing-architecture/processing-pipeline.md) pentru detalii.

</details>

<details>

<summary>Cum pot rula diagnosticul pentru instalarea mea Chloros?</summary>

Utilizați comanda `selftest` pentru a rula un test de funcționare în 7 pași: versiune, disponibilitatea porturilor, pornirea backend-ului, conectivitatea API (`/api/test`), informații despre sistem (`/api/system-info` — GPU/CUDA/PyTorch), prezența modelului de denoizare și disponibilitatea CUDA + denoizorului:

```bash
chloros-cli selftest
```

Acest lucru este deosebit de util pe sistemele Linux/Jetson pentru a verifica configurarea GPU-ului și a CUDA.

</details>
