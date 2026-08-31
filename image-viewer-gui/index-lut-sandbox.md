# Index/LUT Sandbox

Index/LUT Sandbox este spațiul de lucru interactiv din bara laterală a vizualizatorului de imagini Chloros. Alegeți o formulă, asociați canalele camerei dvs. la aceasta, colorați-o cu un gradient și reglați intervalul de valori — iar imaginea se actualizează în timp real pe măsură ce efectuați aceste operațiuni. Începând cu versiunea 1.2.0, puteți, de asemenea, **să salvați ceea ce ați creat**, fie pentru o singură imagine, fie pentru întregul proiect, fără a fi necesară reprocesarea.

## La ce servește Sandbox-ul

| Index/LUT Sandbox (interactiv)        | Procesarea proiectului (în lot)       |
| -------------------------------------- | -------------------------------- |
| Câte o imagine pe rând, feedback instantaneu  | Întregul set de date într-o singură execuție     |
| Experimental și iterativ             | Setări preconfigurate          |
| Renderizează în timp real; salvează doar la cerere  | Scrie întotdeauna fișierele finale      |
| Perfect pentru găsirea setărilor potrivite | Cel mai bun rezultat odată ce setările sunt definitive |

{% hint style="success" %}
**Fluxul de lucru obișnuit**: reglați în Sandbox până când vizualizarea arată așa cum doriți, apoi fie exportați direct din Sandbox, fie copiați aceleași setări de index și LUT în [Setări proiect](../project-settings/project-settings.md), astfel încât următoarea rundă de procesare să le încorporeze în fiecare imagine.
{% endhint %}

***

## Deschiderea Sandbox-ului

1. Faceți clic pe o imagine din grilă — aceasta se deschide pe ecran complet în fila **Vizualizator de imagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Faceți clic pe pictograma **Vizualizator de imagini** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> pentru a afișa bara laterală din stânga, dacă aceasta nu este deja deschisă
3. Alegeți un strat multibandă din meniul derulant al straturilor din colțul din dreapta sus — **RAW (Reflectanță)** este alegerea obișnuită, deoarece valorile indicelui calculate pe baza reflectanței calibrate sunt comparabile între imagini

Bara laterală afișează, de sus în jos:

* numele imaginii și modelul camerei foto
* butonul **Export/Salvare imagini**— apare odată ce opțiunea**Index**sau**LUT** este bifată
* casetele de selectare **Index**și**LUT**
* panoul de configurare a indexului
* panoul **Valori cursor** cu afișajul, histograma și controlul GSD

{% hint style="warning" %}
**Nu este disponibil pentru camerele monocrome.** Pe o imagine LATTICE M3M cu o singură bandă, ambele casete de selectare sunt dezactivate, cu textul de informație _„Nu este disponibil pentru senzori monocromi (M3M)”_ — un index multibandă nu este definit pe o singură bandă. Pentru a calcula indici din camerele M3M, combinați două sau mai multe într-un stack multibandă aliniat și utilizați motorul de indexare LATTICE.
{% endhint %}

***

## Aplicarea unui indice

1. Bifați caseta **Index** din partea de sus a barei laterale
2. Alegeți filtrul camerei dvs. din meniul derulant din stânga (`RGN`, `OCN`, `NGB`, `RGB`, `RE`, `NIR`)
3. Alegeți o formulă de index din meniul derulant din dreapta — 27 de formule încorporate, plus orice formule personalizate pe care le-ați salvat
4. Formula se afișează sub formă de expresie matematică mai jos, cu un cerc gol la fiecare slot de bandă. **Trageți un cerc colorat de canal pe un slot** pentru a-l lega
5. Odată ce fiecare slot utilizat de formulă este legat, imaginea se actualizează și afișează valorile indexului
6. Deplasați cursorul peste imagine pentru a citi valorile; panoul **Valori cursor** adaugă un rând de index cu valoarea de sub cursor

Faceți dublu clic pe un slot legat pentru a-l șterge. O formulă incompletă reprezintă o stare normală în timpul glisării, nu o eroare — imaginea pur și simplu nu se actualizează până când formula nu este completă.

Cercurile canalelor sunt codificate pe culori: roșu = Red, verde = Green, albastru = Blue, portocaliu = Orange, cian = Cyan, violet = NIR, magenta = RE. Aceleași culori sunt utilizate pentru punctele canalelor și curbele histogramei din panoul „Valori cursor”.

### Exemplu NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For a Survey3W RGN camera:
  NIR = 850 nm band
  Red = 661 nm band

Result range:          -1.0 to +1.0
Typical vegetation:     0.4 to 0.9
Stressed vegetation:    0.2 to 0.4
Bare soil:              0.0 to 0.2
Water:                 -0.1 to 0.1
```

Pentru referința completă a formulelor — toate cele trei liste de presetări și care nume funcționează unde — consultați [Formule de indice multispectral](../project-settings/multispectral-index-formulas.md).

### Cu opțiunea „Index” bifată, dar fără LUT

Imaginea este redată în **scale de gri**, întinsă între cele două valori de prag. Acest lucru este intenționat: imaginea indexată conține date scalare, iar scala de gri reprezintă redarea fidelă a acestora. Adăugați o LUT atunci când doriți culori.***

## Lucrul cu LUT-uri (tabele de consultare)

O **tabelă de consultare** asociază valorile indexului cu culori: introduceți NDVI 0,65, obțineți un anumit verde. Aceasta nu modifică datele — schimbă modul în care le interpretați.

### Adăugarea unei tabele LUT

1. Faceți clic pe butonul **„+ Adăugare LUT”** d<img src="../.gitbook/assets/image (1) (1) (1).png" alt="" data-size="line">, situat sub formulă
2. Alegeți un gradient de culori
3. Setați valorile minime și maxime de decupare
4. Alegeți un mod de decupare
5. Bifați caseta **LUT** din bara laterală pentru a o reda

Caseta de selectare **LUT** rămâne dezactivată până când o LUT a fost configurată efectiv pe index.

### Alegerea unui gradient de culoare

Treceți cu mouse-ul peste **bara de gradient**pentru a deschide lista de presetări — Chloros include**șapte** presetări de gradient:

| # | Gradient                            | Formă                                                               |
| - | ----------------------------------- | ------------------------------------------------------------------- |
| 1 | Red → Galben → Green (**implicit**)  | Divergent — corespunde percepției obișnuite asupra vegetației: verde = sănătos |
| 2 | Violet → Galben → Green             | Divergent, cu o porțiune inferioară distinctă                                  |
| 3 | Maro → Alb → Blue                | Divergentă în jurul unui punct mediu deschis                                   |
| 4 | Negru → Violet → Roz → Galben pal | Secvențial, de la întunecat la deschis                                           |
| 5 | Red → Galben → Blue                 | Divergent în jurul unui punct mediu deschis                                   |
| 6 | Violet → Blue → Green → Galben      | Secvențial, de la întunecat la deschis                                           |
| 7 | Orange → Alb → Violet             | Divergent în jurul unui punct central deschis                                   |

Un gradient **divergent**plasează o culoare neutră în mijlocul ferestrei, ceea ce se interpretează bine atunci când punctul mediu are o semnificație (un prag, o dată de referință). Un gradient**secvențial** se desfășoară monoton de la întunecat la deschis, ceea ce se interpretează bine pentru o cantitate care are doar „mai mult” și „mai puțin”.

Fiecare presetare are șapte puncte de culoare. Faceți clic pe o presetare și imaginea se actualizează imediat (când caseta LUT este bifată).

### Editarea punctelor de culoare

Sub bara de gradient se află un rând de mostre de culoare, câte una pentru fiecare punct:

* **Modificați o culoare**: faceți clic pe o mostră pentru a deschide selectorul de culori (roata de culori, glisoarele RGB/HSV sau un cod hexazecimal, cum ar fi `#FF0000`)
* **Adăugarea unui punct**: faceți clic pe butonul**+** de la capătul rândului — se adaugă un punct alb
* **Eliminarea unui punct**:**faceți dublu clic** pe mostră
* **Păstrați un gradient editat**: faceți clic pe pictograma de salvare de lângă bara de gradient pentru a adăuga gradientul editat la lista de presetări, astfel încât să îl puteți selecta din nou

Gradientul pe care l-ați configurat pentru un index este stocat împreună cu acel index în setările proiectului, astfel încât acesta se păstrează chiar și după închiderea și redeschiderea proiectului.

**Un număr mai mic de puncte**produce zone distincte care se interpretează ca o clasificare;**un număr mai mare de puncte** produce tranziții line, aproape fotografice. Trei până la cinci puncte sunt potrivite pentru diapozitive de prezentare și hărți de clasificare; șase până la zece sunt potrivite pentru analize generale; cincisprezece sau mai multe sunt potrivite pentru inspecții detaliate și figuri de publicație.

### Setarea intervalului de valori

Controlul pragului este un **glisor cu două mânere**care variază de la −1 la +1, cu o casetă de text editabilă la fiecare capăt pentru valori exacte și un buton**AUTO**.

* Trageți de oricare dintre mânere sau introduceți un număr în caseta corespunzătoare și apăsați Enter
* **AUTO**setează intervalul la**percentilele 2 și 98** ale valorilor de indice valide ale imaginii — un bun punct de plecare care ignoră valorile aberante. Chloros rotunjește rezultatul în mod adaptiv, la 4 zecimale pentru un interval foarte îngust, la 3 pentru unul îngust și la 2 în celelalte cazuri
* Orice ajustare manuală are prioritate față de AUTO până când apăsați din nou butonul AUTO

Exemplu de ferestre NDVI:

| Obiectiv                                    | Min  | Max |
| --------------------------------------- | ---- | --- |
| Afișează tot                         | −1,0 | 1,0 |
| Numai vegetație, exclude solul și apa | 0,2  | 0,9 |
| Numai vegetație sănătoasă                 | 0,5  | 0,9 |
| Evidențiază stresul                        | 0,2  | 0,5 |

Restrângerea ferestrei crește contrastul în interiorul zonei de interes și împinge tot restul în afara intervalului — unde **Modul de decupare** decide ce se întâmplă cu acesta.***

## Moduri de decupare

Când valoarea indexului unui pixel se află în afara ferestrei min/max, Modul de decupare decide cum este redat.

| Etichetă meniu derulant                  | Valoare stocată      | Pixelii aflați în afara intervalului sunt desenați ca                                                                                                |
| ------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Minim și maxim** (implicit) | `clip`            | Cea mai apropiată culoare de la capătul gradientului — valorile sub minim iau prima culoare, iar cele peste maxim iau ultima |
| **Fundal transparent**      | `transparent`     | Complet transparent (alfa real)                                                                                                  |
| **Fundal indexat**| `indexColor`      | Scală de gri, extinsă pe**întregul** interval de index al imaginii, astfel încât structura aflată în afara intervalului să rămână vizibilă în gri                |
| **Fundal original**         | `backgroundColor` | Imaginea de bază însăși, astfel încât suprapunerea de culoare se află deasupra scenei reale                                                |

| Mod                       | Cel mai potrivit pentru                               | Aspect                                      |
| -------------------------- | -------------------------------------- | ----------------------------------------- |
| **Minim și maxim**      | Afișare completă a datelor, analiză științifică | Fiecare pixel colorat                      |
| **Fundal transparent** | Suprapuneri GIS, izolarea unei benzi de valori   | Culoare în interiorul ferestrei, nimic în exterior |
| **Fundal indexat**       | Evidențiere păstrând contextul datelor    | Culoare în interior, gri în exterior               |
| **Fundal original**    | Rapoarte și prezentări              | Culoare în interior, fotografie în exterior         |

{% hint style="info" %}
**Pixelii fără date sunt întotdeauna transparenți, în orice mod.** Un pixel al cărui indice nu este finit (o împărțire la 0/0) sau este exact −1,0 sau +1,0 (indicatori de saturație, din cauza faptului că o bandă indică zero, în timp ce cealaltă nu) este tratat ca lipsit de date, mai degrabă decât ca o valoare extremă. Astfel, zonele suprasaturate și umbrele prea întunecate sunt excluse din scala de culori, în loc să fie redate ca cele mai extreme valori din cadru. Aceeași regulă definește care pixeli alimentează pragurile AUTO și histograma de indici, astfel încât toate trei să fie în concordanță.
{% endhint %}

Transparența este păstrată atunci când exportul este salvat în format PNG. Aceasta nu poate fi reprezentată în format JPG.

***

## Citirea valorilor în timp ce reglați

Panoul **Valori cursor** de sub panoul de configurare este instrumentul de măsurare pentru Sandbox:

* Deplasați cursorul peste imagine și citiți valorile sursei pentru fiecare canal, plus valoarea indexului în rândul său
* Activați butonul **INDEX** de deasupra histogramei pentru a vedea distribuția valorilor indexului în cadru, cu cele două praguri de decupare reprezentate ca linii punctate portocalii și valoarea cursorului ca o linie albă — aceasta este cea mai rapidă modalitate de a alege o fereastră care conține efectiv datele dvs.
* Activați **CURSOR** pentru a vedea linii de marcare la valorile aflate sub indicator
* Măriți imaginea peste 60× (mai puțin dacă este setată o dimensiune a blocului GSD) pentru a evidenția pixelii afișați individual cu o valoare flotantă

O procedură practică:

1. Notați valorile deasupra vegetației sănătoase, a vegetației stresate, a solului gol și a apei
2. Observați unde se află acele grupuri pe histograma indexului
3. Setați valorile minime și maxime pentru a delimita grupul care vă interesează
4. Alegeți un mod de decupare — _Original Background_ menține vizibilă scena din jurul acestuia

***

## Exportarea din Sandbox

Tot ce este prezentat mai sus reprezintă o previzualizare în timp real până când salvați imaginea. Butonul **Export/Salvare imagini** din partea de sus a barei laterale deschide un panou care se glisează peste bara laterală (în loc să acopere imaginea, astfel încât să puteți vedea în continuare elementele pe care le analizați).

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>### Opțiuni

| Opțiune                          | Efect                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Aplică la imaginea curentă**      | Salvează exact imaginea afișată, cu aceste setări                                                                                                |
| **Aplică la toate imaginile proiectului** | Reexecută configurația identică pe fiecare imagine din proiect. Imaginile fără benzile necesare acestui index sunt omise, nefiind tratate ca eșecuri |
| **Bara de gradient index/LUT**      | Salvează, de asemenea, o imagine separată cu legenda pentru fiecare export, cu intervalul de valori etichetat                                                                     |
| **Histograma index**             | Salvează, de asemenea, o imagine separată cu histograma pentru fiecare export, care arată valorile minime/maxime ale datelor și pragurile de decupare                                               |

Dacă **dimensiunea blocului GSD** din fila imaginii este mai mare decât 1, panoul vă avertizează înainte de a confirma: exportul salvează ceea ce vedeți, inclusiv media pe blocuri. Setați mai întâi controlul GSD la 1 dacă doriți rezoluție maximă.

### Unde sunt salvate fișierele

Fiecare clic pe **Export**alocă un**folder nou, care nu este reutilizat niciodată**:

```
<project folder>/Sandbox_Exports/<IndexName>_<Index|LUT>_<NNN>/
```

Exemple: `Sandbox_Exports/NDVI_LUT_001/`, apoi `Sandbox_Exports/NDVI_LUT_002/` pentru următoarea execuție. Numerotarea se generează prin scanarea conținutului existent deja pe disc, astfel încât aceasta rămâne intactă chiar și după reporniri sau după ștergerea manuală a dosarelor. Nimic nu este suprascris vreodată — scopul principal al Sandbox-ului este compararea unei încercări cu cea anterioară.

În interiorul folderului, pentru fiecare imagine:

| Fișier                                                   | Conținut                                                   |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| `<source name>_<IndexName>_<Index\|LUT>.png`           | Imaginea randată, pixel cu pixel, așa cum a fost afișată de vizualizator |
| `<source name>_<IndexName>_<Index\|LUT>_legend.png`    | Fișierul auxiliar cu bara de gradient, dacă este solicitat                     |
| `<source name>_<IndexName>_<Index\|LUT>_histogram.png` | Fișierul auxiliar cu histograma de indici, dacă este solicitat                  |

Cele două fișiere auxiliare sunt întotdeauna scrise la **rezoluție maximă**, chiar și atunci când imaginea principală este mediată pe blocuri: dimensiunea unui bloc este egală cu rezoluția ecranului, iar ambele fișiere auxiliare conțin valorile reale ale indexului pentru fiecare pixel. De asemenea, ele afișează mai multe informații decât versiunile de pe ecran — ambele indică fereastra de extindere _și_ valorile minime și maxime reale ale datelor, astfel încât o legendă salvată rămâne lizibilă chiar și după câteva luni, fără a fi necesară deschiderea proiectului.

### Progres și rezultate

Exportul întregului proiect durează câteva minute, astfel încât procesul raportează progresul printr-un canal de progres în timp real, în loc să blocheze sistemul:

* O bară de progres afișează `current / total` și fișierul care se scrie
* La finalizare, panoul raportează câte imagini au fost exportate, câte au fost omise și calea către folderul de ieșire
* Imaginile omise sunt listate împreună cu motivul (se afișează până la cinci, apoi o linie „+N mai multe”). Motivul obișnuit este un strat care nu are canalele de care are nevoie acest index
* Dacă **nicio** imagine din proiect nu poate utiliza indexul, procesul raportează o eroare, în loc să vă lase un folder gol

Se poate rula doar o singură operațiune de export în mediul de testare (sandbox) la un moment dat. Pornirea unei a doua operațiuni în timp ce una este în curs de desfășurare este refuzată cu un mesaj clar, în loc să se permită ca două operațiuni să intre în conflict pentru același fișier de proiect.

### Grila preia execuția

Fiecare execuție finalizată apare ca un buton separat în bara de instrumente [grila de imagini](image-grid.md), etichetat `<IndexName> <Index|LUT> <NNN>`. Astfel se compară execuțiile: exportați de două ori cu gradienturi sau praguri diferite, apoi comutați între cele două butoane din grilă.

***

## Formule de index personalizate (Chloros+)

{% hint style="info" %}
**Unde se creează**: în bara laterală Sandbox sau în**Setările proiectului** înainte de procesare. Ambele se salvează în aceeași listă la nivel de proiect.
{% endhint %}

1. Deschideți calculatorul de formule personalizate din meniul derulant al formulei de index (este necesară autentificarea cu un abonament Chloros+ eligibil)
2. Scrieți formula folosind **simbolurile de bandă și slot** `x`, `y`, `z`, `a`, `b`, `c` — nu denumiri de benzi
3. Operatori disponibili: `+`, `-`, `*`, `/`, `^` și `()` pentru grupare
4. Funcții disponibile: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
5. Denumiți-o și salvați-o — aceasta apare în partea de jos a meniului derulant al formulelor și îi puteți aloca sloturile trăgând cercurile canalelor, exact ca în cazul unei presetări încorporate

```

Modified NDVI with an offset:   (y-x)/(y+x+0.5)
Simple ratio:                   y/x
Three-band difference:          (y-x)/(y+x-z)
Squared ratio:                  (y/x)^2
```

{% hint style="warning" %}
**Formulele personalizate sunt disponibile doar în interfața grafică.** Opțiunea CLI/SDK `--indices` extinde cele 22 de nume de presetări încorporate și omite în mod implicit orice altceva, inclusiv formulele dvs. personalizate. Pentru a procesa în serie o formulă personalizată, configurați-o în Setările proiectului și rulați procesarea sau utilizați opțiunea de export „Aplică la toate imaginile proiectului” din Sandbox.
{% endhint %}

***

## Depanare

### „Acest strat nu are canalele de care are nevoie acest index”

Formula citește o poziție de canal pe care stratul curent nu o are — de exemplu, un index cu trei sloturi pe un fișier cu unul sau două canale. Treceți la un strat multibandă (reflectanță sau debayered) sau alegeți un index care se potrivește cu filtrul camerei dvs.

### „Nu s-a putut accesa backend-ul de procesare a imaginii”

Backend-ul nu răspunde. Verificați fila „Jurnal”; dacă backend-ul se restartează, Sandbox-ul se recuperează automat odată ce acesta revine.

### Imaginea nu s-a modificat când am tras un cerc

Formula nu este încă completă. O formulă incompletă este tratată ca o stare normală în timpul tragerii — nu se renderizează nimic și nu se raportează nicio eroare. Completați fiecare câmp utilizat de formulă.

### Întreaga imagine este de o singură culoare

Probabil că fereastra de decupare se află mult în afara datelor. Apăsați **AUTO**pentru a o alinia la percentilele 2 și 98, sau activați histograma**INDEX** pentru a vedea unde se află de fapt datele.

### Culorile exportate nu corespund cu ceea ce am văzut

Ar trebui să corespundă — calea de export este o oglindire intenționată a previzualizării în timp real, inclusiv alfa-ul din modul de decupare, iar medierea pe blocuri se aplică _după_ colorare, exact așa cum o face vizualizatorul. Dacă există diferențe, verificați dacă dimensiunea blocului GSD nu s-a modificat între vizualizare și export.

***

## Pași următori

* [**Straturi de imagine**](image-layers.md) — pe ce strat se aplică un indice și ce înseamnă valorile acestuia
* [**Deschiderea unei imagini pe ecran complet**](opening-an-image-full-screen.md) — citirea cursorului, histograma și controlul GSD în detaliu
* [**Formule de indici multispectrali**](../project-settings/multispectral-index-formulas.md) — fiecare presetare, pe fiecare suprafață
* [**Setări de proiect**](../project-settings/project-settings.md) — salvarea setărilor pe care le-ați găsit într-o sesiune de procesare
