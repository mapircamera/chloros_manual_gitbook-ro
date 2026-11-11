---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Întrebări frecvente

<details>

<summary>Pot procesa cu Chloros imagini de la camere care nu sunt de marcMAPIR?</summary>

Nu, Chloros acceptă doar procesarea imaginilor de la camerele MAPIR — familiile Survey3 și LATTICE. Vă rugăm să consultați lista [modelelor de camere compatibile](supported-cameras.md) pentru mai multe informații. Oferim însă procesarea imaginilor de la alte camere prin intermediul MAPIR Cloud; consultați lista completă [aici](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Chloros acceptă camerele LATTICE?</summary>

Da. Chloros 1.3.0 suportă modulele de cameră LATTICE M3C și M3M de la un capăt la altul: **control în timp real**— descoperire, conectare, previzualizare și captură din fila „Camerele” a interfeței grafice de utilizare (GUI), `chloros-cli lattice`, sau din SDK SDK, inclusiv matrice sincronizate cu mai multe camere cu sincronizare temporală PTP — și**prelucrare radiometrică completă** a capturilor (raw → debayered → radianță → reflectanță → indice). Consultați [Camerele suportate](supported-cameras.md) și [ghidul LATTICE](lattice/README.md).

</details>

<details>

<summary>Pot să-mi calibrez imaginile pentru reflectanță fără o țintă de calibrare?</summary>

**Survey3:** Nu. Fără o imagine a țintei de calibrare capturată în același timp cu imaginile care nu sunt ale țintei, nu veți putea corela valorile pixelilor imaginii cu un procent cunoscut de reflectanță. Dacă, de asemenea, nu includeți înregistrarea de la un senzor de lumină MAPIR, spectrul luminii ambientale nu va fi măsurat, iar rezultatele privind reflectanța nu vor fi precise.

**LATTICE:** Da. Reflectanța poate fi raportată la iradianța descendentă măsurată de un senzor de lumină DAQ în locul unui panou (ρ = π·L/E). Atunci când o țintă din cadru care a trecut de controlul calității *este* prezentă, aceasta devine referința absolută în mod implicit (`--reflectance-source auto`), cu excepția cazului în care trece cu un indicator, situație în care ținta și lanțurile DAQ concurează pe baza scorului măsurat. O singură excepție: „Reflectanța F988 este calibrată folosind un panou de reflectanță din cadru: banda se află în afara intervalului calibrat al senzorului de lumină DAQ, așa că Chloros ul aplică cea mai recentă captură a panoului și o menține între observările panoului.” Consultați [Ținte de calibrare](calibration-targets.md).

</details>

<details>

<summary>Am nevoie de un senzor de lumină DAQ?</summary>

Nu pentru radianță: exporturile de radianță LATTICE provin din calibrarea radiometrică din fabrică a fiecărei camere și nu necesită nici un senzor DAQ, nici o țintă. Pentru **reflectanță**aveți nevoie de o referință pentru lumina ambientală — fie o măsurătoare a luminii descendente efectuată de un senzor de lumină DAQ, fie o țintă de calibrare în cadru. Un senzor DAQ vă permite să obțineți valori calibrate ale**reflectanței fără a plasa panouri în cadru**. Fișierele `.daq` înregistrate sunt corelate automat cu imaginile dvs. pe baza marcajului temporal. Consultați [Țintele de calibrare](calibration-targets.md) și [Referința CLI](reference/cli-reference.md).

</details>

<details>

<summary>Pot folosi Chloros cu un asistent AI (Claude, ChatGPT etc.)?</summary>

Da — acest manual și CLI-ul/SDK-ul sunt concepute special pentru acest scop:

* Indexul complet al manualului este disponibil la adresa `https://mapir.gitbook.io/chloros/llms.txt`, pentru ca asistenții AI să poată descoperi fiecare pagină.
* Codul Markdown brut al fiecărei pagini este disponibil la URL-ul paginii cu litere mici, la care se adaugă `.md` (de exemplu, `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [Referința CLI](reference/cli-reference.md) și [Referința SDK](reference/sdk-reference.md) sunt redactate pentru utilizarea de către modelele LLM: indicatori exacți, valori implicite, semantica ieșirii și comenzi care pot fi copiate și lipite.

Consultați [Asistenții AI](ai-assistants.md) pentru a afla cum să configurați asistentul să utilizeze Chloros.

</details>

<details>

<summary>Unde se salvează fișierele de ieșire procesate?</summary>

Produsele sunt salvate în folderul proiectului, grupate pe cameră și apoi pe format de fișier:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **folder-cameră** — `LATT-<sensor>-<lens>-F<filter>` pentru LATTICE, `<model>_<filter>` (de exemplu, `Survey3N_RGN`) pentru Survey3
* **folder-format** — `tiff16`, `tiff8`, `png8`, `jpg8` sau `tiff32`
* **foldere de produse** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (întotdeauna sub `tiff32`), `<INDEX>_Index_Images/`
* **`clamp_masks/`** — numai pentru datele multispectrale LATTICE: măști per pixel lângă folderele de produse, vezi mai jos**Fișierele exportate păstrează numele fișierului sursă — folderul identifică produsul, nu sufixul numelui fișierului.**Cu ajutorul CLI-ului, folderul de proiect este creat în folderul dvs. de proiect implicit (`Documents/MAPIR/Chloros Projects`, dacă nu l-ați modificat), cu excepția cazului în care specificați `-o`; un `-o` relativ este considerat relativ față de folderul din care executați comanda. Rețineți că o execuție `chloros-cli process` care a solicitat produse, dar nu a scris niciunul, afișează `Processing finished but wrote no image products.` și**se închide cu un cod de ieșire diferit de zero**, astfel încât scripturile să o poată detecta. Consultați [Formate de imagine de ieșire](output-image-formats.md) și [Referința CLI](reference/cli-reference.md).

</details>

<details>

<summary>Ce sunt folderele clamp_masks și fișierele *_clampmask.tif?</summary>

Noutate în versiunea 1.3.0. Produsele de radianță și reflectanță ale unei camere multispectrale LATTICE M3C efectuează o separare a benzilor 3×3, iar acolo unde se produce o valoare negativă într-o bandă, Chlorose decupează acea valoare la zero. Un produs cu cel puțin un pixel decupat primește o mască `uint8` cu o singură bandă care marchează pixelii și benzile care au fost decupate (bitul 0 = canalul R al produsului, bitul 1 = G, bitul 2 = B). Măștile sunt salvate într-un dosar `clamp_masks/` **lângă** dosarele produselor — de exemplu, `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif` — iar eticheta XMP a produsului `Chloros:UnmixClampMask` denumește masca acestuia (sau menționează `none`).

Acestea nu sunt imagini ale scenei: importul propriu al Chloros le omite, iar deoarece nu se află niciodată într-un dosar de produs, puteți transmite un dosar de produs către Pix4D sau Metashape așa cum este. Consultați [Măști de fixare](output-image-formats.md#clamp-masks).

</details>

<details>

<summary>De ce afișează Chloros mesajul „Dosarul proiectului este prea adânc pentru acest computer”?</summary>

Aceasta este o problemă specifică sistemului Windows. Windows vine cu opțiunea pentru căi lungi dezactivată, iar în acest caz nicio cale de fișier nu poate depăși 260 de caractere. Produsele Chloros se află la aproximativ 130 de caractere sub folderul de proiect, așa că Chloros refuză un folder de proiect mai adânc înainte de procesare — în loc să ruleze și să nu scrie nimic, așa cum se putea întâmpla înainte de versiunea 1.3.0. Mesajul indică cel mai lung folder de proiect care se încadrează.

Remediați problema în oricare dintre următoarele moduri:

* Utilizați un folder de proiect mai scurt (**Directorul de lucru** din Setările proiectului sau `-o` / `chloros-cli set-project-folder` pentru interfața CLI).
* Activați căile lungi din Windows și reporniți Chloros. Opțiunea **Enable Windows long paths** (activată implicit) din programul de instalare Chloros face acest lucru; rulați din nou programul de instalare dacă ați debifat-o.

Consultați [Pornirea procesării](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

</details>

<details>

<summary>Pot edita imaginile înainte de procesarea în Chloros?</summary>

Nu. Chloros presupune că datele de intrare nu au fost modificate. Nu modificați numele fișierelor.

</details>

<details>

<summary>Pot seta camerele mele MAPIR Survey3 pe expunere automată și să procesez imaginile în Chloros?</summary>

Nu. Seturile de date cu imagini Survey3 trebuie să aibă o expunere fixă/blocată, deci fără viteză automată a obturatorului sau ISO automat. Toate imaginile realizate cu același model de cameră trebuie să aibă viteză a obturatorului și ISO (expunere) identice.

Camerele LATTICE nu au această restricție: Chloros controlează expunerea în timp real (Smart AE), iar fiecare captură înregistrează expunerea și amplificarea utilizate efectiv, aspecte de care ține cont procesul de prelucrare radiometrică.

</details>

<details>

<summary>Chloros poate procesa sau analiza imagini ortomozaice?</summary>

Nu. Sunt acceptate doar imaginile individuale ale camerelor MAPIR, nu imaginile asamblate, cum ar fi o hartă ortomozaică.

</details>

<details>

<summary>Cum pot accelera etapa de detectare a țintelor în Chloros?</summary>

În tabelul din browserul de fișiere, preselectarea imaginilor țintă din coloana din dreapta va indica aplicației Chloros să caute țintele de calibrare numai în acele imagini, accelerând considerabil procesarea.

</details>

<details>

<summary>Dacă voi încărca imaginile mele în <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> ar trebui să le procesez în Chloros înainte de încărcare?</summary>

Dacă intenționați să încărcați imaginile pe platforma noastră de procesare online [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), nu editați imaginile înainte de încărcare. Cloud va efectua toate aceleași operații de procesare și multe altele.

</details>

<details>

<summary>MAPIR va suporta vreodată funcția X? Mi-aș dori foarte mult ca MAPIR să ofere funcția X.</summary>

Suntem întotdeauna interesați să primim feedback cu privire la produsele noastre. Dacă descoperiți o problemă legată de produsele noastre sau aveți o sugestie privind modul în care le putem îmbunătăți, vă rugăm să [NE CONTACTAȚI](https://www.mapir.camera/community/contact) pentru a ne împărtăși opiniile dumneavoastră. Cea mai mare parte a activității noastre de cercetare și dezvoltare se bazează pe ascultarea celor mai importante nevoi ale clienților noștri.

</details>

<details>

<summary>Chloros este disponibil pentru Linux?</summary>

Da! Chloros 1.3.0 suportă Linux amd64 (x86_64) și arm64 (NVIDIA Jetson JetPack 6) prin intermediul pachetelor `.deb`. CLI-ul și SDK-ul Python sunt pe deplin suportate pe Linux, inclusiv controlul în timp real al camerelor LATTICE și al senzorilor DAQ. Nu există o interfață grafică (GUI) pentru Linux — toată interacțiunea se realizează prin [CLI](CLI.md) sau [SDK Python](api-python-sdk.md). Consultați [Prezentarea generală Linux](linux/linux-overview.md) pentru detalii.

</details>

<details>

<summary>Pot rula Chloros pe NVIDIA Jetson?</summary>

Da! Chloros suportă **familia Orin** de dispozitive NVIDIA Jetson — Orin Nano, Orin NX și AGX Orin — care rulează JetPack 6. Modelele mai vechi (Jetson Nano, TX2, Xavier NX) nu pot rula JetPack 6 și nu sunt suportate de pachetul actual. Chloros detectează automat modelul dvs. de Jetson și optimizează strategia de procesare a acestuia. Pachetul Jetson utilizează versiunea CUDA a PyTorch din JetPack 6, astfel încât procesarea se desfășoară pe GPU-ul dispozitivului Jetson. Consultați [Ghidul NVIDIA Jetson](linux/nvidia-jetson-guide.md) pentru instrucțiuni de configurare și implementare.

</details>

<details>

<summary>Chloros se optimizează automat pentru hardware-ul meu?</summary>

Da! Chloros include [Adaptarea dinamică a calculului](processing-architecture/dynamic-compute-adaptation.md) care detectează automat CPU-ul, GPU-ul, memoria RAM și (pe Jetson) senzorii termici. Apoi selectează strategia optimă de procesare — de la `GPU_PARALLEL` pentru GPU-urile cu memorie mare, la `GPU_SINGLE` pentru cele mai mici, până la `CPU_PARALLEL` pentru sistemele fără un GPU CUDA pe care PyTorch îl poate utiliza. Nu este necesară nicio configurare manuală.

</details>

<details>

<summary>Ce este pipeline-ul de procesare cu 4 fire?</summary>

Chloros utilizează o arhitectură cu 4 fire de execuție în serie pentru utilizatorii Chloros+: Firul 1 (Detectare) încarcă imaginile și detectează țintele de calibrare, Firul 2 (Calibrare) calculează calibrarea reflectanței, Firul 3 (Prelucrare) efectuează debayeringul accelerat de GPU și calculul indicelui, iar Firul 4 (Export) scrie fișierele de ieșire. Mai multe imagini pot fi procesate simultan în fire diferite pentru un randament maxim. Consultați [Conducta de procesare](processing-architecture/processing-pipeline.md) pentru detalii.

</details>

<details>

<summary>Cum pot rula testele de diagnosticare pe instalarea mea Chloros?</summary>

Utilizați comanda `selftest` pentru a rula un test de funcționare în 7 pași: versiune, disponibilitatea porturilor, pornirea backend-ului, conectivitatea API (`/api/test`), informații despre sistem (`/api/system-info` — GPU/CUDA/PyTorch), prezența modelului de denoizare și disponibilitatea CUDA + denoizorului:

```bash
chloros-cli selftest
```

Acest lucru este deosebit de util pe sistemele Linux pentru a verifica configurarea GPU-ului și a CUDA. Pe un Jetson, `CUDA: False` înseamnă de obicei că lipsesc bibliotecile CUDA și cuDNN ale JetPack 6: instalați-le cu `sudo apt install nvidia-jetpack`.

</details>
