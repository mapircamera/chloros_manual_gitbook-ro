# Marcaje pe hartă

Fila Hartă afișează imaginile pe o hartă 2D interactivă, pe baza coordonatelor GPS ale acestora. Aceasta oferă o imagine de ansamblu geografică a sesiunii de captură și vă ajută să vizualizați acoperirea spațială. De asemenea, este utilă atunci când importați pentru prima dată imaginile, pentru a elimina rapid imaginile pe care nu trebuie să le procesați.

## Accesarea filei Hartă

1. Deschideți sau creați un proiect în Chloros.
2. Importați imagini care conțin metadate GPS.
3. Faceți clic pe fila **Hartă** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> din bara laterală din stânga
4. Harta va afișa marcaje la locația GPS a fiecărei imagini

{% hint style=&quot;info&quot; %}
**GPS necesar**: Numai imaginile cu coordonate GPS încorporate în metadatele EXIF vor apărea pe hartă. Asigurați-vă că aparatul foto are GPS-ul activat în timpul capturii.
{% endhint %}

***

## Reglarea imaginilor din fila Hartă

Fila **Hartă**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> are aceleași funcții de adăugare  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  și ștergere  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  ca și fila [**Browser fișiere**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . De asemenea, afișează aceeași listă de fișiere de proiect, dar cu anteturi de coloane diferite:

### Nume fișier

* Numele fișierului original din aparatul foto
* Păstrează convenția de denumire a aparatului foto (de exemplu, IMG\_0001.RAW)

### Latitudine

* Latitudinea imaginii

### Longitudine

* Longitudinea imaginii

### Altitudine

* Altitudinea imaginii

{% hint style=&quot;info&quot; %}
Dacă faceți clic pe anteturile coloanelor tabelului, datele din rânduri sunt sortate
{% endhint %}

***

## Marcaje imagine

Fiecare imagine cu date GPS este reprezentată de un marcaj pe hartă:

### Afișare marcaj

* Marcajele indică coordonatele GPS exacte ale locului în care a fost capturată fiecare imagine.
* Marcajele grupate pot fi grupate împreună atunci când se micșorează imaginea.
* Măriți imaginea pentru a vedea locațiile individuale ale imaginilor.

{% hint style=&quot;success&quot; %}
SUPER-ZOOM: Când atingeți nivelul maxim de zoom de la furnizorul de plăci de hartă, placa este apoi mărită la zoom suplimentar, permițându-vă să vedeți marcajele care sunt apropiate între ele.
{% endhint %}

### Previzualizare la trecerea mouse-ului

* **Treceți mouse-ul** peste orice marcaj pentru a vedea o previzualizare miniaturală a imaginii respective
* Acest lucru permite identificarea vizuală rapidă fără a părăsi vizualizarea hărții
* Util pentru localizarea imaginilor specifice într-o sesiune de captură mare

***

## Furnizori de plăci de hartă

{% hint style=&quot;success&quot; %}
**Selecție automată**: Chloros alege automat serviciul de plăci care oferă cel mai bun nivel de zoom pentru locația dvs. actuală pe hartă. Puteți comuta manual între furnizori, dacă doriți.
{% endhint %}

Fila Hartă acceptă doi furnizori de plăci pentru imaginile de fundal ale hărții:

### Google Maps

* Imagini standard din satelit și hărți de la Google
* Cea mai bună opțiune pentru acoperire generală la nivel mondial

### ESRI

* Imagini din satelit și aeriene de la ESRI ArcGIS
* Oferă adesea imagini cu rezoluție mai mare în anumite regiuni

***

## Tipuri de plăci de hartă

Puteți alege tipul stratului de hartă (de la stânga la dreapta):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Teren

Afișează profiluri de altitudine și imagini cu detalii (drumuri etc.)

### Hartă

Afișează imagini standard (lățime de bandă mai mică) cu detalii (drumuri etc.)

### Satelit

Afișează imagini detaliate (lățime de bandă mai mare) din satelit

### Hibrid

Afișează imagini din satelit cu detalii suplimentare (drumuri etc.)

***

## Navigare pe hartă

### Comenzi de zoom

* **Mărire/micșorare**: utilizați rotița mouse-ului sau butoanele de zoom
* **Ecran complet**: afișează harta pe ecran complet

### Comenzi de panoramare

* **Panoramare**: faceți clic și trageți pentru a vă deplasa pe hartă***

## Cazuri de utilizare

### Vizualizarea traiectoriei de zbor

* Vizualizați zona de acoperire a sesiunilor de captare cu drona
* Identificați lacunele în acoperirea imaginii
* Verificați execuția traiectoriei de zbor

### Revizuirea studiului la sol

* Vedeți distribuția spațială a capturilor la sol
* Localizați imaginile țintă de calibrare în raport cu zona de studiu
* Planificați locații suplimentare de captare

### Controlul calității

* Identificați rapid imaginile capturate în locații neașteptate
* Verificați acuratețea GPS-ului în întregul set de date
* Comparați locațiile imaginilor cu notele de teren

***

## Depanare

### Nu apar marcaje

**Cauze posibile:**

* Imaginile nu conțin metadate GPS
* GPS-ul a fost dezactivat pe cameră în timpul capturii
* Datele EXIF au fost șterse de un software extern

**Soluție**: Verificați dacă GPS-ul este activat pe camera dvs. și reimportați fișierele originale

### Marcaje în locații greșite

**Cauze posibile:**

* GPS-ul camerei a avut o fixare slabă a satelitului
* Deriva GPS în timpul capturii

**Soluție**: Aceasta este de obicei o problemă legată de timpul de captură; luați în considerare utilizarea GPS-ului PPK/RTK pentru aplicații de precizie
