# Interfață grafică: Proiecte

Chloros vă permite să creați proiecte care pot fi redeschise în viitor. Un proiect este un dosar obișnuit (din dosarul „Proiecte”) care conține:

* `project.json` — setările proiectului, lista de fișiere și preferințele de afișare
* `cameras.json` — camerele și matricea conectate în timp ce proiectul era deschis, împreună cu setările acestora
* `sensors.json` — senzori de lumină DAQ conectați în timp ce proiectul era deschis, plus asocierile cameră↔senzor
* capturile dvs., înregistrările `.daq` și folderele cu rezultatele procesate

Nu există un format proprietar de fișier de proiect — proiectul este reprezentat de folderul respectiv și de fișierele JSON din acesta, ceea ce facilitează, de asemenea, copierea, arhivarea și gestionarea proiectelor din [CLI](CLI.md) sau [Python SDK](api-python-sdk.md).

Dacă nu modificați această setare în [Setările proiectului](project-settings/project-settings.md), folderul proiectului este `Documents\MAPIR\Chloros Projects` din folderul „Documente” (`~/Documents/MAPIR/Chloros Projects` pe Linux).

{% hint style="warning" %}
**Windows: păstrați folderele de proiect scurte, cu excepția cazului în care opțiunea pentru căi lungi este activată.**Windows vine cu opțiunea pentru căi lungi dezactivată, iar în acest caz nicio cale de fișier nu poate depăși 260 de caractere. Produsele procesate se află la aproximativ 130 de caractere sub folderul de proiect, astfel încât, începând cu versiunea 1.3.0, Chloros refuză un folder de proiect prea adânc — atunci când îl creați sau îl deschideți, sau când începeți procesarea — în loc să ruleze și să nu scrie nimic. Mesajul indică cel mai lung folder de proiect care se încadrează. Utilizați un dosar de proiect mai scurt sau activați căile lungi Windows (opțiunea**Activați căile lungi Windows** din programul de instalare, bifată implicit) și reporniți Chloros. Consultați [Pornirea procesării](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).
{% endhint %}

## Proiect nou

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Selectați „Proiect nou” din meniul principal și introduceți un nume unic pentru proiectul dvs.

Dacă ați salvat șabloane de proiect, sub câmpul de nume apare un meniu derulant **Selectare șablon** — alegerea unuia dintre acestea va iniția noul proiect pornind de la setările acelui șablon. Șabloanele se salvează din [Setări proiect](project-settings/project-settings.md): introduceți un nume în câmpul „Salvați numele șablonului de proiect” și faceți clic pe pictograma de salvare.

## Deschideți proiectul

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>Secțiunea „Deschide proiect” afișează toate proiectele din folderul dvs. de proiecte, iar în partea de jos se află opțiunea <strong>Deschide folderul de proiecte</strong></p></figcaption></figure>

Selectați „Deschide proiect” pentru a vedea o listă a proiectelor existente în folderul de proiecte. Dacă nu există proiecte, meniul lateral secundar nu se va deschide. Puteți vedea câteva proiecte create prin interfața grafică (t1, t2, t3) listate în fotografia de mai sus. Proiectele DATE\_TIME au fost create prin CLI, utilizând schema implicită de denumire a proiectelor. Dacă faceți clic pe numele oricărui proiect, acesta se va deschide.

Dacă faceți clic pe butonul „Deschide folderul proiectului”, se va deschide exploratorul de fișiere al computerului dvs. la calea proiectului. Puteți modifica calea proiectului în [Setările proiectului](project-settings/project-settings.md).

Dacă vreunul dintre fișierele imagine sursă ale proiectului a fost mutat sau șters de la ultima deschidere, Chloros afișează o fereastră de dialog care enumeră exact fișierele lipsă, în loc să deschidă o grilă goală.

## Duplicare proiect

Opțiune disponibilă odată ce un proiect este deschis. Selectați „Duplicare proiect” pentru a copia proiectul curent sub un nume nou — Chloros sugerează următorul nume disponibil (de exemplu, „Proiectul meu (2)”) — iar duplicatul se deschide imediat.

## Adăugare fișiere

După deschiderea unui proiect, selectați „Adăugare fișiere” din meniul principal pentru a adăuga fișiere imagine individuale la proiectul curent. Această funcție reflectă funcționalitatea de adăugare a browserului de fișiere, dar este accesibilă direct din meniul principal pentru mai multă comoditate.

## Adăugare dosar

După deschiderea unui proiect, selectați „Adăugare dosar” din meniul principal pentru a adăuga dosare cu imagini la proiectul curent. Puteți selecta mai multe dosare dintr-o singură operațiune. Fișierele duplicate sunt ignorate.

## Pornire / Oprire procesare

După ce fișierele sunt adăugate la un proiect, opțiunea „Pornire procesare” devine disponibilă în meniul principal. Aceasta corespunde acțiunii de a face clic pe butonul Redare/Pornire din antetul superior. În timpul procesării, opțiunea din meniu se schimbă în „Oprire procesare” pentru a vă permite să opriți fluxul de procesare.

## Conectare la cameră / Conectare la senzorul de lumină

În partea de jos a meniului principal se află două comenzi rapide pentru hardware, disponibile indiferent dacă este deschis sau nu un proiect:

* **Conectare la cameră** — deschide [fila Camere](lattice/) pentru a conecta o cameră sau o matrice LATTICE.
* **Conectare la senzorul de lumină** — deschide [fila Senzori de lumină](daq/) pentru a conecta un senzor de lumină DAQ.

Conectarea echipamentului în timp ce un proiect este deschis salvează conexiunea în proiect (a se vedea mai jos). Fără un proiect, conexiunile sunt valabile doar pentru sesiunea curentă.

{% hint style="info" %}
Opțiunile de meniu **Adăugare fișiere**,**Adăugare folder**și**Pornire/Oprire procesare**sunt vizibile sau activate numai atunci când un proiect este deschis și au fost adăugate fișiere. Acestea oferă acces rapid la acțiuni disponibile și prin bara laterală**File Browser** și butoanele din antet.
{% endhint %}

## Proiectele rețin configurația hardware-ului

Noutate în versiunea 1.2.0: un proiect păstrează configurația hardware-ului conectat cât timp este deschis. Camerele și matricea (cu setările specifice fiecărei camere, numele, culorile și dispunerea în grilă) sunt salvate în `cameras.json`, iar senzorii de lumină (cu numele, culorile și asocierile cu camerele) în `sensors.json` — automat, pe măsură ce lucrați.

Când **redeschideți** un proiect, Chloros nu accesează imediat niciun dispozitiv hardware. Fiecare jumătate se reconectează la prima accesare a filei care o conține:

* Deschiderea filei **Camere** reconectează camerele și matricile salvate și reaplică setările salvate ale acestora.
* Deschiderea filei **Senzori de lumină**reconectează senzorii DAQ salvați și reaplică setările lor salvate, inclusiv capacul. Un senzor setat la**Niciunul (senzor gol)** în meniul său Cap rămâne gol. Un proiect salvat de o versiune anterioară a Chloros a înregistrat un senzor gol ca opțiune implicită, așa că acesta revine cu corecția cosinusului luminii solare; setați-l din nou la Niciunul și salvați proiectul.

În acest fel, deschiderea unui proiect doar pentru a răsfoi sau a exporta imagini nu va activa niciodată camerele în modul de transmisie în flux. Dacă un dispozitiv salvat nu poate fi găsit la deschiderea filei sale, o fereastră de dialog vă indică ce dispozitive sunt indisponibile, astfel încât să le puteți reconecta sau elimina.

## Înregistrările DAQ și fișierele .daq dintr-un proiect

* Înregistrările `.daq` realizate în timp ce proiectul este deschis (din fila „Light Sensors” sau în timpul capturilor) sunt **adăugate automat la proiect**.
* Fișierele `.daq` importate, precum și toate înregistrările proiectului, sunt listate în secțiunea **Senzor de lumină DAQ** din [Setări proiect](project-settings/project-settings.md), fiecare cu profilul său de corecție a capului.
* În timpul procesării, fișierele `.daq` ale proiectului furnizează iluminarea descendentă pentru produsele de reflectanță — consultați [Formate de imagine de ieșire](output-image-formats.md).

## Rularea unui proiect salvat fără interfață grafică

Un proiect salvat poate fi rulat fără interfața grafică:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` operează pe calea către folderul proiectului — consultați [Referința CLI](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` returnează un identificator de proiect; `connect_all()` activează toate camerele, matricea și senzorii de lumină salvați, cu setările lor salvate (senzorii de lumină prin intermediul unui backend Chloros în execuție) — consultați [Referința SDK](reference/sdk-reference.md).
