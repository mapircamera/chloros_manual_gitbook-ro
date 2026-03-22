# Marcaje pe hartă

Fila „Hartă” afișează imaginile pe o hartă 2D interactivă, pe baza coordonatelor GPS ale acestora. Aceasta oferă o imagine de ansamblu geografică a sesiunii de captură și vă ajută să vizualizați acoperirea spațială. De asemenea, este utilă la importarea inițială a imaginilor pentru a elimina rapid imaginile pe care nu trebuie să le procesați.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Accesarea filei Hartă

1. Deschideți sau creați un proiect în Chloros
2. Importați imagini care conțin metadate GPS
3. Faceți clic pe fila **Hartă** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> din bara laterală din stânga
4. Harta va afișa marcaje la locația GPS a fiecărei imagini

{% hint style="info" %}
**GPS necesar**: Pe hartă vor apărea doar imaginile cu coordonate GPS încorporate în metadatele EXIF. Asigurați-vă că aparatul foto are GPS-ul activat în timpul capturării.
{% endhint %}

***

## Reglarea imaginilor din fila Hartă

Fila **Hartă**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> are aceeași funcție de adăugare  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  și de eliminare  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  ca și fila [**File Browser**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . De asemenea, afișează aceeași listă de fișiere de proiect, dar cu anteturi de coloane diferite:

### Nume fișier

* Numele original al fișierului din aparatul foto
* Păstrează convenția de denumire a aparatului foto (de ex., IMG\_0001.RAW)

### Latitudine

* Latitudinea imaginii

### Longitudine

* Longitudinea imaginii

### Altitudine

* Altitudinea imaginii

{% hint style="info" %}
Dacă faceți clic pe anteturile coloanelor din tabel, datele din rânduri vor fi sortate
{% endhint %}

***

## Marcaje de imagine

Fiecare imagine cu date GPS este reprezentată de un marcaj pe hartă:

### Afișarea marcajelor

* Marcajele indică coordonatele GPS exacte în care a fost capturată fiecare imagine
* Marcajele grupate pot fi grupate împreună atunci când se micșorează imaginea
* Măriți imaginea pentru a vedea locațiile individuale ale imaginilor

{% hint style="success" %}
SUPER-ZOOM: Când atingeți nivelul maxim de zoom de la furnizorul de plăci de hartă, placa este apoi mărită la zoom suplimentar, permițându-vă să vedeți marcajele care sunt apropiate unele de altele.
{% endhint %}

### Previzualizare la trecerea mouse-ului

* **Treceți mouse-ul** peste orice marcaj pentru a vedea o previzualizare miniatură a acelei imagini
* Acest lucru permite identificarea vizuală rapidă fără a părăsi vizualizarea hărții
* Util pentru localizarea imaginilor specifice într-o sesiune de captură de mari dimensiuni

***

## Furnizori de secțiuni de hartă

{% hint style="success" %}
**Selecție automată**: Chloros alege automat serviciul de plăci care oferă cel mai bun nivel de zoom pentru locația dvs. curentă pe hartă. Puteți comuta manual între furnizori, dacă doriți.
{% endhint %}

Fila Hartă acceptă doi furnizori de plăci pentru imaginile de fundal ale hărții:

### Google Maps

* Imagini standard din satelit și hărți de la Google
* Cele mai potrivite pentru acoperire globală generală

### ESRI

* Imagini din satelit și aeriene de la ESRI ArcGIS
* Oferă adesea imagini cu rezoluție mai mare în anumite regiuni

***

## Tipuri de plăci de hartă

Puteți alege tipul stratului de hartă (de la stânga la dreapta):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Teren

Afișează profiluri de altitudine și plăci de hartă cu detalii (drumuri etc.)

### Hartă

Afișează plăci de hartă standard (lățime de bandă mai mică) cu detalii (drumuri etc.)

### Satelit

Afișează plăci de hartă satelitare detaliate (lățime de bandă mai mare)

### Hibrid

Afișează plăci de hartă satelitare cu detalii adăugate (drumuri etc.)

***

## Navigare pe hartă

### Comenzi de zoom

* **Mărire/micșorare**: Utilizați rotița mouse-ului sau butoanele de zoom
* **Ecran complet**: Afișați harta pe ecran complet

### Comenzi de panoramare

* **Panoramare**: Faceți clic și trageți pentru a vă deplasa pe hartă***

## Cazuri de utilizare

### Vizualizarea traseului de zbor

* Vizualizați zona de acoperire a sesiunilor de captură cu drona
* Identificați lacunele în acoperirea imaginii
* Verificați executarea traseului de zbor

### Revizuirea studiului la sol

* Vedeți distribuția spațială a capturilor de la sol
* Localizați imaginile țintă de calibrare în raport cu zona de studiu
* Planificați locații suplimentare de captură

### Controlul calității

* Identificați rapid imaginile capturate în locații neașteptate
* Verificați precizia GPS-ului în întregul set de date
* Corelați locațiile imaginilor cu notele de teren

***

## Depanare

### Nu apar marcaje

**Cauze posibile:**

* Imaginile nu conțin metadate GPS
* GPS-ul a fost dezactivat pe cameră în timpul captării
* Datele EXIF au fost eliminate de un software extern

**Soluție**: Verificați dacă GPS-ul este activat pe aparatul foto și reimportați fișierele originale

### Marcaje în locație greșită

**Cauze posibile:**

* GPS-ul aparatului foto a avut o fixare slabă a sateliților
* Deviație GPS în timpul captării

**Soluție**: Aceasta este de obicei o problemă legată de momentul captării; luați în considerare utilizarea GPS-ului PPK/RTK pentru aplicații de precizie
