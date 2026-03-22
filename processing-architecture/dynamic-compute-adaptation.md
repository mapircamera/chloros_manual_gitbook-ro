# Adaptare dinamică a puterii de calcul

Chloros 1.1.0 introduce detectarea inteligentă a hardware-ului și selectarea automată a strategiei de procesare. Motorul de procesare se adaptează la hardware-ul dvs. — de la un Jetson Nano până la o stație de lucru cu mai multe GPU-uri — fără nicio configurare manuală.

***

## Cum funcționează

Când Chloros pornește, acesta profilează automat sistemul dvs.:

1. **Detectează sistemul de operare** — Windows sau Linux
2. **Identifică nucleele CPU și memoria RAM totală**

3.**Detectează prezența GPU** — capacitate NVIDIA CUDA, VRAM, model
4. **Identifică modelul Jetson** (dacă este cazul) — prin `/proc/device-tree/model`
5. **Verifică senzorii termici** (Jetson) — pentru procesare sensibilă la temperatură
6. **Selectează strategia de calcul optimă** — pe baza întregului hardware detectat
7. **Configurează automat numărul de worker-i, tipul de pipeline și alocarea memoriei**Rezultatul este stocat în cache, astfel încât rulările ulterioare să pornească mai repede. Dacă hardware-ul se modifică (de exemplu, se adaugă un GPU), Chloros re-profilează la următoarea lansare.***

## Strategii de calcul

Chloros selectează una dintre cele trei strategii de calcul în funcție de hardware-ul dvs.:

| Strategie | GPU necesar | Processori | Pipeline | Cel mai potrivit pentru |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`** | Da (12 GB+ VRAM sau 16 GB+ partajat) | 3-4 | `fused_gpu` | GPU-uri pentru desktop cu 12 GB+, Jetson Orin NX 16 GB, AGX Orin |
| **`GPU_SINGLE`** | Da (&lt; 12 GB VRAM) | 1-3 | `tiled_gpu` | GPU-uri entry-level, Jetson Nano, Orin Nano |
| **`CPU_PARALLEL`** | Nu | nuclee - 1 | `cpu_fallback` | Sisteme fără GPU NVIDIA |

### Tipuri de pipeline

* **`fused_gpu`** — Cale completă de procesare GPU. Toate operațiunile de debayer, corecție și indexare se execută pe GPU într-o singură trecere fuzionată. Debit maxim, dar necesită mai multă memorie VRAM.
* **`tiled_gpu`** — Cale GPU eficientă din punct de vedere al memoriei. Procesează imaginile în secțiuni pentru a se încadra în memoria limitată a GPU-ului. Debit mai mic, dar funcționează pe dispozitive cu memorie limitată.
* **`cpu_fallback`** — Procesare exclusiv pe CPU folosind paralelism multi-threaded. Se folosește când nu este disponibil un GPU NVIDIA.***

## Comportament specific platformei

| Platformă | Strategie | Workers | Pipeline | Note |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (serializat) | Mod eficient din punct de vedere al memoriei, procesează o singură imagine la un moment dat |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` (concurent) | Dispozitiv de margine recomandat — procesare GPU paralelă reală |
| **Jetson AGX Orin 64 GB** | `GPU_PARALLEL` | 4 | `fused_gpu` (concurent) | Performanță maximă la marginea rețelei |
| **Desktop cu GPU de 8 GB** | `GPU_SINGLE` | 3 | `tiled_gpu` | Performanță bună pentru desktop cu blocuri eficiente din punct de vedere al memoriei |
| **Desktop cu GPU de 12 GB+** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Performanță optimă pe desktop |
| **Sistem doar cu CPU** | `CPU_PARALLEL` | nuclee - 1 | `cpu_fallback` | Nu este necesar GPU, utilizează ThreadPool |

{% hint style="info" %}
**Memorie unificată Jetson**: dispozitivele Jetson partajează memoria GPU și CPU. Un Jetson Orin NX de 16 GB raportează ~15,3 GB de VRAM, dar aceasta este aceeași memorie RAM fizică utilizată de sistemul de operare și de procesele CPU. Chloros ține cont de acest lucru atunci când stabilește pragurile de alocare a memoriei.
{% endhint %}

***

## Alocarea dinamică a memoriei GPU

Chloros utilizează un [pipeline de procesare cu 4 thread-uri](processing-pipeline.md):

* **Thread 1** (Detectare) — Încărcarea imaginii, parsarea EXIF, detectarea țintei
* **Thread 2** (Calibrare) — Calculul calibrării reflectanței
* **Fir 3** (Procesare) — Debayer GPU, corecție vignetă, calcul index
* **Fir 4** (Export) — Scriere fișiere, încorporare metadate

Pe măsură ce firele anterioare ale canalului își finalizează activitatea (de exemplu, toate imaginile au fost detectate), alocarea lor de memorie GPU este eliberată și **redistribuită către firele active rămase**. Aceasta înseamnă că Thread 3 (etapa cu utilizare intensivă a GPU-ului) primește progresiv mai multă memorie pe măsură ce pipeline-ul avansează, îmbunătățind randamentul pentru operațiunile care necesită cea mai mare putere de calcul.

### Etape de alocare

| Etapă | Thread-uri active | Distribuția memoriei GPU |
| --- | --- | --- |
| **Inițial** | 1, 2, 3, 4 | Împărțită între toate thread-urile |
| **Mijloc-început** | 2, 3, 4 | Memoria firului 1 redistribuită |
| **Mijloc-sfârșit** | 3, 4 | Memoria firelor 1+2 merge către 3+4 |
| **Sfârșit** | 3 sau 4 | Memorie maximă pentru firul rămas |***

## Procesare Texture Aware

Metoda de debayer Texture Aware (numai Chloros+) utilizează semnificativ mai multă memorie GPU decât metoda Standard, datorită modelului de denoising AI/ML:

* Sistemele cu **&lt; 7 GB VRAM** sunt forțate să intre într-o buclă de procesare sincronă pentru modul Texture Aware (o imagine pe rând)
* Sistemele cu **7 GB+ VRAM** pot procesa Texture Aware simultan, deși cu un număr redus de procesori comparativ cu modul Standard***

## Gestionarea termică (Jetson)

Dispozitivele Jetson au restricții termice, în special în implementări închise sau aeriene. Chloros monitorizează temperaturile GPU și CPU și ajustează automat procesarea:

| Temperatură | Răspuns |
| --- | --- |
| **&lt; 70°C** | Funcționare normală — viteză maximă |
| **70°C** (Avertisment) | Reducerea dimensiunii lotului |
| **80°C** (Critic) | Limitarea agresivă — reducerea concurenței și a numărului de procesori |
| **90°C** (Oprire) | Oprire completă a procesării GPU |

Monitorizarea temperaturii utilizează `tegrastats` pe platformele Jetson. Pe sistemele desktop cu răcire adecvată, limitarea termică este rar declanșată.

***

## Gestionarea presiunii memoriei

Chloros monitorizează presiunea asupra memoriei sistemului în timpul procesării:

* **Prag de memorie**: o utilizare de 85% declanșează un comportament conservator
* **Reducere OOM**: Dacă apare un eveniment de memorie insuficientă, alocarea este redusă cu 25% (multiplicator 0,75x)
* **Revenire la pipeline**: În condiții de presiune severă asupra memoriei, pipeline-ul revine automat de la `fused_gpu` la `tiled_gpu`
* **Recomandări privind swap**: Pe Jetson, Chloros vă avertizează dacă spațiul de swap este insuficient pentru dimensiunea setului de date***

## Monitorizarea adaptării de calcul

### Ieșire de stare CLI

La începerea procesării, CLI afișează profilul hardware detectat:

```

Chloros CLI 1.1.0
Platform: Linux aarch64 (Jetson Orin NX 16GB)
Strategy: GPU_PARALLEL | Workers: 3 | Pipeline: fused_gpu
CUDA: Available | GPU Memory: 15.3 GB (shared)
```

### Diagnosticare sistem

Rulați `chloros-cli selftest` pentru a vedea un profil hardware complet și a verifica capacitățile de calcul:

```bash
chloros-cli selftest
```

Aceasta verifică disponibilitatea CUDA, memoria GPU, modelele de denoizare și conectivitatea backend-ului.

***

## Pași următori

* [Pipeline de procesare](processing-pipeline.md) — Înțelegerea arhitecturii pipeline-ului cu 4 thread-uri
* [Ghidul NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Implementare și optimizare specifică pentru Jetson
* [CLI : Linia de comandă](../CLI.md) — Referință completă CLI
