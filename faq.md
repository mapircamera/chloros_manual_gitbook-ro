---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Întrebări frecvente

<details>

<summary>Pot procesa imagini de la camere care nu sunt de marca MAPIR cu Chloros?</summary>

Nu, Chloros acceptă doar procesarea imaginilor de la camerele MAPIR. Vă rugăm să consultați lista [modelelor de camere compatibile](supported-cameras.md) pentru mai multe informații. Oferim procesarea imaginilor de la alte camere pe MAPIR Cloud; consultați lista completă [aici](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Pot calibra imaginile pentru reflectanță fără o țintă de calibrare?</summary>

Nu. Fără o imagine a țintei de calibrare capturată în momentul captării imaginilor fără țintă, nu veți putea corela valorile pixelilor imaginii cu un procent de reflectanță cunoscut. Dacă nu includeți și jurnalul de la un senzor de lumină MAPIR, spectrul luminii ambientale nu va fi măsurat, iar rezultatele de reflectanță nu vor fi precise.

</details>

<details>

<summary>Pot edita imaginile înainte de procesarea în Chloros?</summary>

Nu. Chloros presupune că datele de intrare nu au fost modificate. Nu modificați numele fișierelor.

</details>

<details>

<summary>Pot seta camerele mele MAPIR și Survey3 pe expunere automată și să procesez imaginile în Chloros?</summary>

Nu. Seturile de date de imagine Survey3 trebuie să aibă o expunere fixă/blocată, deci fără viteză automată a obturatorului sau ISO automat. Toate imaginile de la același model de cameră trebuie să aibă viteză a obturatorului și ISO (expunere) identice.

</details>

<details>

<summary>Chloros poate procesa sau analiza imagini ortomozice?</summary>

Nu. Sunt acceptate doar imagini individuale de la camera MAPIR, nu imagini asamblate, cum ar fi o hartă ortomozică.

</details>

<details>

<summary>Cum pot accelera etapa de detectare a țintelor din Chloros?</summary>

În tabelul browserului de fișiere, preselectarea imaginilor țintă din coloana din dreapta va indica Chloros să caute ținte de calibrare numai în acele imagini, accelerând considerabil procesarea.

</details>

<details>

<summary>Dacă voi încărca imaginile mele în <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> ar trebui să le procesez în Chloros înainte de încărcare?</summary>

Dacă intenționați să încărcați pe platforma noastră de procesare online [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), nu editați imaginile înainte de încărcare. Cloud va efectua aceeași procesare și multe altele.

</details>

<details>

<summary>MAPIR va suporta vreodată funcția X? Mi-aș dori foarte mult ca MAPIR să ofere X.</summary>

Suntem întotdeauna interesați să primim feedback cu privire la produsele noastre. Dacă descoperiți o problemă cu produsele noastre sau aveți o sugestie privind modul în care le putem îmbunătăți, vă rugăm să ne [CONTACTAȚI](https://www.mapir.camera/community/contact) pentru a ne împărtăși opiniile. Cea mai mare parte a activității noastre de cercetare și dezvoltare este ghidată de ascultarea celor mai importante nevoi ale clienților noștri.

</details>

<details>

<summary>Chloros este disponibil pentru Linux?</summary>

Da! Chloros 1.1.0 suportă Linux amd64 (x86_64) și arm64 (NVIDIA Jetson JetPack 6) prin intermediul pachetelor `.deb`. CLI și Python SDK sunt complet suportate pe Linux. Nu există o interfață grafică pentru Linux — toată interacțiunea se face prin [CLI](CLI.md) sau [Python SDK](api-python-sdk.md). Consultați [Prezentarea generală a Linux](linux/linux-overview.md) pentru detalii.

</details>

<details>

<summary>Pot rula Chloros pe NVIDIA Jetson?</summary>

Da! Chloros 1.1.0 acceptă platformele NVIDIA Jetson, inclusiv Jetson Nano, Orin Nano, Orin NX și AGX Orin care rulează JetPack 6. Chloros detectează automat modelul dvs. Jetson și optimizează strategia de procesare a acestuia. Consultați [Ghidul NVIDIA Jetson](linux/nvidia-jetson-guide.md) pentru instrucțiuni de configurare și implementare.

</details>

<details>

<summary>Chloros se optimizează automat pentru hardware-ul meu?</summary>

Da! Chloros 1.1.0 include [Adaptarea dinamică a calculului](processing-architecture/dynamic-compute-adaptation.md) care detectează automat CPU-ul, GPU-ul, memoria RAM și (pe Jetson) senzorii termici. Apoi selectează strategia optimă de procesare — de la `GPU_PARALLEL` pe sistemele cu memorie mare la `GPU_SINGLE` pe dispozitivele cu resurse limitate și până la `CPU_PARALLEL` pe sistemele fără GPU NVIDIA. Nu este necesară nicio configurare manuală.

</details>

<details>

<summary>Ce este pipeline-ul de procesare cu 4 thread-uri?</summary>

Chloros 1.1.0 utilizează o arhitectură de tip pipeline cu 4 thread-uri pentru utilizatorii Chloros+: Firul 1 (Detectare) încarcă imagini și detectează țintele de calibrare, Firul 2 (Calibrare) calculează calibrarea reflectanței, Firul 3 (Procesare) efectuează debayering accelerat de GPU și calculul indexului, iar Firul 4 (Export) scrie fișierele de ieșire. Mai multe imagini pot fi în fire diferite simultan pentru un randament maxim. Consultați [Procesarea Pipeline](processing-architecture/processing-pipeline.md) pentru detalii.

</details>

<details>

<summary>Cum pot rula diagnosticarea pe instalarea mea Chloros?</summary>

Utilizați comanda `selftest` pentru a rula 7 diagnostice de sistem, inclusiv verificarea versiunii, disponibilitatea porturilor, pornirea backend-ului, conectivitatea API, informații despre sistem, modele de denoizare și disponibilitatea CUDA:

```bash
chloros-cli selftest
```

Acest lucru este deosebit de util pe sistemele Linux/Jetson pentru a verifica configurarea GPU și CUDA.

</details>
