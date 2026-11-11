# Adaptare dinamică a puterii de calcul

Chloros
1.3.0 utilizează detectarea hardware-ului și selectarea automată a strategiei de procesare. Motorul de procesare se adaptează la hardware-ul dvs. — de la un Jetson Orin Nano până la o stație de lucru cu mai multe GPU-uri — fără nicio configurare manuală.

***

## Cum funcționează

La pornire, Chloros
realizează un profil al sistemului dumneavoastră:

1. **Detectează sistemul de operare** — Windows
sau Linux
2. **Identifică nucleele CPU și memoria RAM totală**

3.**Detectează prezența GPU-ului** — compatibilitate NVIDIA CUDA, VRAM, model
4. **Identifică modelul Jetson** (dacă este cazul) — prin `/proc/device-tree/model`
5. **Verifică senzorii termici** (Jetson) — pentru procesare adaptată la temperatură
6. **Selectează strategia de calcul** — pe baza întregului hardware detectat
7. **Configurează automat numărul de procesori de lucru, tipul de pipeline și alocarea memoriei**

Profilul detectat este stocat în cache pentru sesiunea curentă, atât în memorie, cât și pe disc, astfel încât execuțiile ulterioare să pornească mai repede:

| Platformă | Profil stocat în cache |
| --- | --- |
| **Linux
/ Jetson** | `~/.config/chloros/system_config.json` (acceptă `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Ștergeți acel fișier pentru a forța o nouă detectare — utilă după adăugarea unui GPU sau a mai multă memorie RAM. Chloros
efectuează, de asemenea, o nouă detectare automată atunci când cache-ul a fost scris de o versiune mai veche incompatibilă.

***

## Strategii de calcul

Chloros
selectează una dintre cele trei strategii de calcul în funcție de hardware-ul dvs.:

| Strategie | Selectată când | Worker-i | Executor | Pipeline |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| GPU CUDA care raportează**12 GB+ VRAM**(pe memoria unificată Jetson, necesită, de asemenea, 12 GB+ RAM partajată total) | `min(4, ⌊(VRAM − 0.5GB) × 0.9 ÷ 3.5GB⌋)`, minimum 2 —**limitat la 2 pe Jetson** | Procese de lucru (contexte CUDA separate) | `fused_gpu` |
| **`GPU_SINGLE`**| GPU CUDA cu**2–12 GB VRAM**| 3 (suprapunere I/O; accesul la GPU serializat printr-un semafor).**1 (secvențial) pe dispozitivele Jetson cu mai puțin de 12 GB RAM** | Procese de lucru (contexte CUDA separate); secvențial în cadrul procesului pe dispozitivele Jetson cu memorie RAM redusă | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Fără GPU CUDA sau cu mai puțin de 2 GB VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Exemple concrete ale formulei de calcul a numărului de procesori de lucru `GPU_PARALLEL`: 12 GB VRAM → 2 procesori de lucru, 16 GB → 3 procesori de lucru, 20 GB+ → 4 procesori de lucru. Fiecărui worker i se alocă 3,5 GB, necesarul estimat pentru o sarcină Texture Aware, în limita a 90% din VRAM-ul rămas după o rezervă de 0,5 GB.

{% hint style="info" %}
**Strategiile GPU necesită un PyTorch care poate utiliza CUDA.** O mașină al cărei PyTorch nu dispune de un dispozitiv CUDA utilizabil primește `CPU_PARALLEL`, indiferent de GPU-ul său. Pachetul Jetson 1.3.0 utilizează versiunea PyTorch pentru CUDA din JetPack 6, astfel încât un Jetson se califică odată ce bibliotecile CUDA și cuDNN ale JetPack sunt instalate. Un Jetson cu mai puțin de 12 GB RAM procesează întotdeauna o singură imagine la un moment dat, în timpul procesării (înregistrat ca `GPU_SINGLE`).
{% endhint %}

Strategiile GPU rulează un grup de procese de lucru, fiecare fiind un proces separat al sistemului de operare cu propriul context CUDA, pe care Chloros
îl inițiază prin relansarea propriului program backend instalat (nu prin `multiprocessing` din Python
, a cărui pornire a proceselor de lucru nu funcționează în interiorul aplicației pachetizate). Strategia CPU utilizează un grup de fire de execuție (`ThreadPoolExecutor` din Python
). Chloros
nu utilizează niciun cadru distribuit de la terți (cum ar fi Ray).

### Tipuri de pipeline

* **`fused_gpu`** — Cale completă de procesare pe GPU. Operațiunile de debayer, corecție și indexare se execută pe GPU într-o singură trecere fuzionată. Debit maxim, necesită cea mai mare cantitate de VRAM.
* **`tiled_gpu`** — Cale GPU eficientă din punct de vedere al memoriei. Procesează imaginile în secțiuni (tiles) pentru a se încadra în memoria limitată a GPU-ului. Debit mai mic, dar funcționează pe dispozitive cu memorie limitată.
* **`cpu_fallback`** — Procesare exclusiv pe CPU folosind paralelism multithread. Se utilizează atunci când nu este disponibil un GPU NVIDIA și ca soluție de rezervă de ultimă instanță atunci când ambele trasee GPU eșuează.

Lanțul de soluții de rezervă la rulare este întotdeauna `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Suprascrierea manuală a strategiei

Setați variabila de mediu `CHLOROS_STRATEGY` pentru a forța o strategie specifică — o soluție de rezervă avansată pentru situațiile în care detectarea automată alege o opțiune nepotrivită pentru situația dvs. (de exemplu, menținerea GPU-ului liber pentru alte sarcini):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Variabila este recunoscută fără a ține cont de majuscule și minuscule; orice valoare care nu corespunde uneia dintre cele trei denumiri este ignorată, iar detectarea automată se desfășoară în mod normal. Chiar și în cazul unei suprascrieri, Chloros
alege în continuare numărul de lucrători pentru dvs.:

| Suprascriere | Numărul de lucrători utilizat |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

O suprascriere de tip `GPU_SINGLE` sau `GPU_PARALLEL` pe o mașină fără dispozitiv CUDA utilizabil este refuzată, cu o linie `[STRATEGY]` în jurnal, iar execuția utilizează `CPU_PARALLEL`.

Este de preferat să setați această opțiune pentru fiecare comandă în parte, mai degrabă decât permanent, astfel încât execuțiile normale să continue să se adapteze automat.

***

## Comportament specific platformei

| Platformă | Strategie | Lucrători | Pipeline | Note |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (secvențial) | Mod eficient din punct de vedere al memoriei, câte o imagine pe rând |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (secvențial) | Memoria RAM partajată sub 12 GB impune procesarea secvențială |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (concurent) | Dispozitiv de margine recomandat — limitat la 2 procesori de lucru pe Jetson |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (concurent) | Performanță maximă la marginea rețelei (de asemenea, limitat la 2 procesori de către Jetson) |
| **Desktop cu GPU de 8 GB** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 procesori suprapun operațiunile de I/O, în timp ce un semafor serializează accesul la GPU |
| **Desktop cu GPU de 12 GB+** | `GPU_PARALLEL` | 2-4 | `fused_gpu` (concurente) | Performanță optimă pe desktop: 12 GB → 2 procesori, 16 GB → 3, 20 GB+ → 4 |
| **Sistem doar cu CPU** | `CPU_PARALLEL` | nuclee fizice − 1 (min. 2) | `cpu_fallback` | Nu este necesar un GPU, utilizează un pool de thread-uri |

{% hint style="info" %}
**Memorie unificată Jetson**: dispozitivele Jetson partajează memoria GPU-ului și a procesorului. Un Jetson Orin de 16 GB raportează ~15,3 GB de VRAM, dar aceasta este aceeași memorie RAM fizică utilizată de sistemul de operare și de procesele procesorului. De aceea, dispozitivele Jetson de 16 GB+ se califică pentru `GPU_PARALLEL` la fel ca un GPU de desktop de 12 GB+, dar sunt limitate la 2 procesori de lucru — GPU-ul, procesele de lucru și contextele CUDA ale fiecărui proces de lucru utilizează toate același pool partajat.
{% endhint %}

### Bugetul GPU în funcție de VRAM (GPU-uri discrete)

Pe gazdele x86_64 cu un GPU NVIDIA discret, VRAM-ul detectat determină, de asemenea, cât din capacitatea de procesare a plăcii poate fi alocată și cât de mari pot deveni loturile:

| VRAM detectată | Plafonul bugetului GPU | Multiplicatorul dimensiunii lotului |
| --- | --- | --- |
| **8 GB+** | 90% | ×2,0 |
| **6–8 GB** | 85% | ×1,75 |
| **3,5–6 GB** | 80% | ×1,5 |
| **2–3,5 GB** | 75% | ×1,25 |
| **Sub 2 GB** | 70% | ×1,0 |

GPU-urile discrete rezervă doar 0,5 GB pentru sistem, deoarece nu partajează memoria RAM a sistemului. Profilele Jetson rezervă mult mai mult și au o limită mai mică — consultați [Ghidul NVIDIA Jetson](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Alocarea dinamică a memoriei GPU

Chloros
utilizează un [flux de procesare cu 4 fire](processing-pipeline.md):

* **Firul 1** (Detectare) — Încărcarea imaginii, analizarea datelor EXIF, detectarea țintei
* **Firul 2** (Calibrare) — Calculul calibrării reflectanței
* **Firul 3** (Prelucrare) — Debayer GPU, corectarea vignetării, calculul indexului
* **Firul 4** (Export) — Scrierea fișierelor, încorporarea metadatelor

Firurile 1, 2 și 4 consumă puțină putere de procesare GPU; Firul 3 este cel care solicită cel mai mult resursele GPU. Pe măsură ce firele anterioare din pipeline se finalizează, bugetul lor de GPU este **redistribuit către firele active rămase**, astfel încât Firul 3 primește progresiv mai multă memorie pe măsură ce procesul avansează.

### Etape de alocare

| Etapă | Fire active | Distribuția memoriei GPU |
| --- | --- | --- |
| **Inițial** | 1, 2, 3, 4 | Împărțită între toate firele, cea mai mare parte revenind firului 3 |
| **Mijloc-inițial** | 2, 3, 4 | Cota firului 1 este redistribuită |
| **Etapa intermediară-finală** | 3, 4 | Cota firelor 1 și 2 este alocată firelor 3 și 4 |
| **Etapa finală** | 3 sau 4 | Ultimul fir activ primește alocarea maximă |

Cifrele sunt guvernate de două reguli:

* Un thread care este **singurul** activ primește alocarea maximă prevăzută de profilul său.
* Când sunt active mai multe sarcini *grele* pe GPU, alocarea de bază a fiecărei sarcini grele este împărțită între ele (niciodată sub minimul configurat).

Valoarea utilizată efectiv în timpul rulării este cea **mai mică** dintre alocarea profilului platformei și recomandarea în timp real a monitorului de memorie GPU, astfel încât o placă ocupată are întotdeauna prioritate față de un profil optimist.***

## Procesare sensibilă la textură

Debayerul Texture Aware (**numai Chloros
+** — `--debayer texture-aware`) rulează un model de denoizare AI/ML care necesită aproximativ 1,75 GB de VRAM în FP16 per copie, astfel încât utilizează mult mai multă memorie GPU decât metoda standard:

* Sistemele cu **mai puțin de 7 GB VRAM**procesează Texture Aware într-o**buclă sincronă, câte o imagine pe rând** — nu pot încăpea mai multe copii ale modelului, iar un grup de procesori ar genera doar conflicte
* Sistemele cu **peste 7 GB VRAM** pot procesa Texture Aware simultan, deși cu un număr redus de worker-i în comparație cu metoda Standard
* Pe **Jetson**, Texture Aware este întotdeauna alocat unui singur worker
* În **lipsa unui GPU CUDA utilizabil**, execuția folosește debayer-ul Standard, iar backend-ul înregistrează o linie `[DEBAYER] … needs a CUDA GPU …` care explică motivul
* Pe dispozitivele Jetson cu consum redus de energie (Nano, Orin Nano), CLI
-ul aplică o limită de frecvență a GPU-ului atunci când solicitați Texture Aware — consultați [Ghidul NVIDIA Jetson](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)

***

## Gestionarea termică (Jetson)

Dispozitivele Jetson au constrângeri termice, în special în cazul implementărilor în spații închise sau la bordul aeronavelor. Chloros
monitorizează senzorii de temperatură integrați ai dispozitivelor Jetson și scalează automat dimensiunile loturilor:

| Temperatură | Răspuns |
| --- | --- |
| **&lt; 70 °C** | Funcționare normală — viteză maximă |
| **70 °C** (Avertisment) | Dimensiunea lotului se reduce progresiv (de la 100% la 50% între 70 °C și 80 °C) |
| **80 °C** (Critic) | Limitarea agresivă a vitezei (50 % → 0 % între 80 °C și 90 °C) |
| **90 °C** (Oprire) | Oprirea completă a procesării GPU |

Pe sistemele desktop cu răcire adecvată, limitarea termică este rar declanșată.

***

## Gestionarea presiunii asupra memoriei

Chloros
monitorizează continuu memoria GPU în timpul procesării și reacționează la trei niveluri.

**Dimensiunea lotului.** Un lot începe cu 8 imagini înmulțite cu multiplicatorul platformei din tabelele de mai sus. Chloros
verifică apoi VRAM-ul liber, rezervă 20% din acesta pentru overhead-ul propriu al PyTorch și presupune aproximativ 100 MB de memorie GPU per imagine de 12 MP — lotul este cel mai mic dintre cele două valori: limita derivată din memorie sau valoarea de bază a platformei. Nu scade niciodată sub 1.

**Reducere preventivă.**Peste**85% utilizare a VRAM-ului**, dimensiunile loturilor sunt reduse înainte ca ceva să eșueze.**Reducerea alocării pe fir de execuție.** Pe măsură ce utilizarea în timp real crește, bugetul de GPU al fiecărui fir de execuție este redus: ×0,75 peste 80% utilizare, ×0,5 peste 90%. Intervalele de monitorizare sunt 70% (conservator), 85% (limită normală de funcționare) și 95% (risc de OOM).**Retragere și recuperare în caz de OOM.** Dacă totuși are loc un eveniment de epuizare a memoriei:

* dimensiunea lotului este **redusă la jumătate** și redusă din nou la jumătate la fiecare eveniment OOM consecutiv — fiecare lot ulterior reușit reduce această penalizare cu un pas
* alocările firelor active sunt reduse la 70% din valoarea lor curentă, iar alocatorul trece la strategia sa conservatoare, relaxându-se din nou după o serie de alocări reușite
* sub presiune severă, pipeline-ul revine de la `fused_gpu` la `tiled_gpu` și, ca ultimă soluție, la `cpu_fallback`

**Memoria RAM a gazdei (Jetson).** Înainte de procesare, CLI
-ul estimează memoria maximă a gazdei pe baza numărului de imagini și a modului de debayer și avertizează dacă memoria RAM plus spațiul de swap bazat pe fișiere este probabil insuficient, afișând comenzile exacte pentru a adăuga spațiu de swap — consultați [Ghidul NVIDIA Jetson](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).

***

## Monitorizarea adaptării de calcul

### Diagnosticul sistemului

`chloros-cli selftest` este cea mai rapidă modalitate de a confirma ceea ce vede stratul de calcul:

```bash
chloros-cli selftest
```

Cele 7 verificări ale sale acoperă versiunea, disponibilitatea porturilor, pornirea backend-ului, `/api/test`, informațiile despre sistem, prezența modelului de denoizare și disponibilitatea CUDA + denoizorului. Verificarea 5 afișează direct linia de hardware:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Verificarea 7 afișează `CUDA: <bool>, Denoiser: <bool>` — ambele trebuie să fie adevărate pentru ca Texture Aware să poată fi utilizat.

### Jurnalele backend-ului

Strategia și numărul de procesori sunt alese în backend la începutul fiecărei rulări — nu există un banner CLI
care să le anunțe. Când ceva se comportă neașteptat (o cale GPU care revine la o soluție de rezervă, o eroare OOM, un denoiser care nu se încarcă), jurnalul backend pentru acea sesiune este locul unde apare:

| Platformă | Locația jurnalului |
| --- | --- |
| **Linux
/ Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (un fișier per lansare) |
| **Linux
, backend pornit prin CLI** | de asemenea `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Progres în timp real

În timpul unei rulări, CLI
-ul afișează progresul în timp real pentru fiecare thread (Detectare, Analizare, Procesare, Exportare), transmis prin Server-Sent Events — o modalitate practică de a afla dacă Thread 3 reprezintă gâtul de sticlă. Consultați [Procesarea pipeline-ului](processing-pipeline.md).

***

## Pași următori

* [Conducta de procesare](processing-pipeline.md) — Înțelegerea arhitecturii conductei cu 4 fire
* [Ghidul NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Implementare și optimizare specifice pentru Jetson
* [CLI
: Linia de comandă](../CLI.md) — Ghidul CLI

* [Referință CLI
](../reference/cli-reference.md) — Lista completă a comenzilor pentru versiunea 1.3.0
