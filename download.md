---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Descărcare

Descărcați cea mai recentă versiune a Chloros pentru a începe procesarea imaginilor multispectrale.

### Cerințe de sistem

| Cerință          | Minim                         | Recomandat                     |
| -------------------- | ------------------------------- | ------------------------------- |
| **Sistem de operare** | Windows 10 (64 biți)             | Windows 11 (64 biți)             |
| **Procesor**        | Intel Core i5 sau echivalent     | Intel Core i7 sau superior         |
| **Memorie (RAM)**     | 8 GB                             | 16 GB sau mai mult                    |
| **Placă grafică**    | Compatibilă cu DirectX 11           | GPU NVIDIA cu 4 GB+ VRAM       |
| **Stocare**          | 6 GB spațiu liber                  | SSD cu 10 GB+ spațiu liber       |
| **Afișaj**          | 1920x1080                       | 2560x1440 sau mai mare             |
| **Internet**         | Necesar pentru activarea licenței | Necesar pentru activarea licenței |

{% hint style=&quot;info&quot; %}
**Accelerare GPU**: Utilizatorii Chloros+ cu GPU-uri NVIDIA (4 GB+ VRAM) pot utiliza accelerarea CUDA pentru o procesare semnificativ mai rapidă. Utilizatorii Chloros+ beneficiază, de asemenea, de procesare multi-threaded pentru viteză maximă.
{% endhint %}

***

## Descărcați Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Descărcați Chloros aici</a>

### Ultima versiune stabilă

**Chloros Instalator pentru Windows*** **Versiune**: 1.0.4
* **Data lansării**: 5 ianuarie 2026
* **Dimensiune fișier (descărcare)**: 1,8 GB
* **Dimensiune fișier (instalat)**: 5,7 GB
* **Tip fișier**: .exe (Instalator Windows)

#### **Pași de instalare:**

1. Descărcați fișierul `CHLOROS INSTALLER - CURRENT VERSION.exe`
2. Faceți dublu clic pe programul de instalare pentru a începe instalarea
3. Urmați instrucțiunile asistentului de instalare
4. Alegeți directorul de instalare (implicit: `C:\Program Files\[USER]\Chloros\`)
5. Finalizați instalarea și lansați Chloros, Chloros (Browser) sau Chloros CLI
6. Conectați-vă cu contul dvs. [MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (sau continuați cu versiunea gratuită)

{% hint style=&quot;success&quot; %}
Programul de instalare adaugă automat `chloros-cli` la PATH-ul sistemului dvs. pentru accesul la linia de comandă.
{% endhint %}

***

## Resurse suplimentare

### Python SDK

Pentru dezvoltatori și fluxuri de lucru automatizate, instalați Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Documentație**: [API: Python SDK](api-python-sdk.md)**Cerințe**: Chloros Desktop trebuie să fie instalat, Chloros+ este necesară autentificarea cu licență***

## Ce este inclus

Instalarea Chloros include:

* ✅ **Chloros** - Interfață grafică completă
* ✅ **Chloros (Browser)** - Interfață web pentru sisteme cu specificații inferioare
* ✅ **Chloros CLI** - Interfață cu linie de comandă (necesită licență Chloros+)
* ✅ **Chloros SDK** - Python API (necesită licență Chloros+)
* ✅ **Profiluri cameră** - Șabloane cameră MAPIR preconfigurate***

## Treceți la Chloros+

Deblocați funcții avansate cu un abonament Chloros+:

* 🚀 **Procesare multi-thread** - Procesați imagini în paralel
* ⚡ **Accelerare GPU (CUDA)** - Profitați de puterea GPU NVIDIA
* 💻 **Acces CLI** - Automatizați cu instrumente de linie de comandă
* 🐍 **Python SDK** - Acces programatic API
* 📱 **Dispozitive multiple** - Utilizați pe 2-10+ dispozitive (în funcție de plan)
* 🧮 **Formule personalizate** - Creați indici multispectrali personalizați

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Vizualizați planurile și prețurile Chloros+</a></p>***

## Ajutor pentru instalare

### Depanare

**Instalarea eșuează cu mesajul de eroare:**

* Asigurați-vă că aveți drepturi de administrator
* Dezactivați temporar software-ul antivirus
* Verificați dacă îndepliniți cerințele minime de sistem

**Aplicația nu pornește:**

* Încercați versiunea Chloros (browser)
* Verificați dacă Windows 10/11 (64 biți) este instalat
* Actualizați driverele grafice
* Verificați Windows Event Viewer pentru detalii despre eroare
* Contactați serviciul de asistență cu jurnalele de erori

**Probleme cu activarea licenței:**

* Asigurați-vă că conexiunea la internet este activă
* Verificați datele de autentificare la [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Verificați dacă firewall-ul nu blochează Chloros
* Consultați [Chloros+ Autentificare](chloros+-login.md) pentru instrucțiuni detaliate

### Obținerea asistenței

Aveți nevoie de ajutor cu instalarea sau configurarea?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Website**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentație**: [Noțiuni introductive](./)
* ❓ **Întrebări frecvente**: [Întrebări frecvente](faq.md)***

## Jurnal de modificări

<details>

<summary>Versiunea 1.0.4</summary>

#### **Data lansării**: 5 ianuarie 2026**Funcții noi*** **Comutator imagine/metadate**: s-a adăugat un comutator în browserul de fișiere pentru a vizualiza metadatele imaginii selectate într-un tabel în loc de grila de imagini
* **Cursor de zoom pentru grila de imagini**: nou cursorul UI pentru ajustarea dimensiunii miniaturilor (acceptă și CTRL + rotița mouse-ului)
* **Butoane de exportare a grilei de imagini**: butoane în rândul de sus pentru a comuta miniaturile de la JPG la exporturi procesate (ținte, reflectanță, index, LUT)
* **Fila Hartă**: nouă hartă 2D interactivă care afișează marcajele de localizare GPS ale imaginilor
  * Suportă Google Maps și ESRI map tiles (selectează automat cel mai bun serviciu de map tiles în funcție de disponibilitatea nivelului de zoom)
  * Previzualizare miniatură la trecerea mouse-ului peste marcajele de pe hartă

**Remedierea erorilor*** Suport îmbunătățit pentru instalarea Chloros pe computere care nu sunt în limba engleză

</details>

<details>

<summary>Versiunea 1.0.3</summary>

#### **Data lansării**: 20 decembrie 2025**Funcții noi*** Lansare inițială

**Îmbunătățiri*** Lansare inițială

**Remedierea erorilor*** Lansare inițială

**Probleme cunoscute*** Lansare inițială

</details>***

## Acord de licență**Software proprietar** - Copyright (c) 2025 MAPIR Inc.

Utilizarea, distribuirea sau modificarea neautorizată este interzisă.

**Versiune gratuită**: Disponibilă pentru uz personal și comercial, cu limitări ale funcționalităților.**Chloros+**: Licență pe bază de abonament pentru funcționalități avansate și implementări comerciale.
