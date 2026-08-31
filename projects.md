# Interfață grafică: Proiecte

Chloros vă permite să creați proiecte care pot fi redeschise ulterior. Un proiect este un dosar obișnuit (aflat în dosarul dvs. de proiecte) care conține:

* `project.json` — setări de proiect, lista de fișiere și preferințe de afișare
* `cameras.json` — camerele și matricea conectate în timp ce proiectul era deschis, împreună cu setările acestora
* `sensors.json` — senzori de lumină DAQ conectați în timp ce proiectul era deschis, plus asocierile cameră↔senzor
* capturile dvs., înregistrările `.daq` și folderele cu rezultatele procesate

Nu există un format proprietar de fișier de proiect — folderul și fișierele sale JSON constituie proiectul, ceea ce face ca proiectele să fie ușor de copiat, arhivat și transferat de pe [CLI](CLI.md) sau [Python SDK](api-python-sdk.md).

## Proiect nou

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Selectați „Proiect nou” din meniul principal și introduceți un nume unic pentru proiectul dvs.

Dacă ați salvat șabloane de proiect, sub câmpul de nume apare un meniu derulant **Selectare șablon** — alegerea unuia dintre acestea va iniția noul proiect pornind de la setările acelui șablon. Șabloanele se salvează din [Setări proiect](project-settings/project-settings.md): introduceți un nume în câmpul „Salvați numele șablonului de proiect” și faceți clic pe pictograma de salvare.

## Deschideți proiect

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>„Deschidere proiect” afișează toate proiectele din folderul dvs. de proiecte, iar în partea de jos se află opțiunea <strong>„Deschidere folder de proiecte”</strong></p></figcaption></figure>Selectați „Deschidere proiect” pentru a vedea o listă a proiectelor existente în folderul de proiecte. Dacă nu există proiecte, meniul lateral secundar nu se va deschide. În fotografia de mai sus puteți vedea câteva proiecte create prin interfața grafică (t1, t2, t3). Proiectele DATE\_TIME au fost create de CLI folosind schema implicită de denumire a proiectelor. Dacă faceți clic pe numele oricărui proiect, acesta se va deschide.

Dacă faceți clic pe butonul „Deschide folderul proiectului”, se va deschide exploratorul de fișiere al computerului dvs. la calea proiectului. Puteți modifica calea proiectului în [Setările proiectului](project-settings/project-settings.md).

Dacă vreunul dintre fișierele imagine sursă ale proiectului a fost mutat sau șters de la ultima deschidere, Chloros afișează o fereastră de dialog care listează exact fișierele lipsă, în loc să deschidă o grilă goală.

## Duplicare proiect

Disponibilă odată ce un proiect este deschis. Selectați „Duplicare proiect” pentru a copia proiectul curent sub un nume nou — Chloros sugerează următorul nume disponibil (de exemplu, „Proiectul meu (2)”) — iar duplicatul se deschide imediat.

## Adăugare fișiere

După deschiderea unui proiect, selectați „Adăugare fișiere” din meniul principal pentru a adăuga fișiere imagine individuale la proiectul curent. Această funcție reflectă funcționalitatea de adăugare a browserului de fișiere, dar este accesibilă direct din meniul principal pentru mai multă comoditate.

## Adăugare de dosare

După deschiderea unui proiect, selectați „Adăugare de dosare” din meniul principal pentru a adăuga dosare cu imagini la proiectul curent. Puteți selecta mai multe dosare dintr-o singură mișcare. Fișierele duplicate sunt ignorate.

## Porniți / Opriți procesarea

După ce fișierele sunt adăugate la un proiect, opțiunea „Porniți procesarea” devine disponibilă în meniul principal. Aceasta corespunde acțiunii de a face clic pe butonul Redare/Start din antetul de sus. În timpul procesării, opțiunea din meniu se schimbă în „Opriți procesarea” pentru a vă permite să opriți fluxul de lucru.

## Conectare la cameră / Conectare la senzorul de lumină

În partea de jos a meniului principal există două comenzi rapide pentru hardware, disponibile indiferent dacă este deschis sau nu un proiect:

* **Conectare la cameră** — deschide [fila Camere](lattice/) pentru a conecta o cameră sau o matrice LATTICE.
* **Conectare la senzorul de lumină** — deschide [fila „Senzori de lumină”](daq/) pentru a conecta un senzor de lumină DAQ.

Conectarea echipamentului în timp ce un proiect este deschis salvează conexiunea în proiect (a se vedea mai jos). Fără un proiect, conexiunile sunt valabile doar pentru sesiunea curentă.

{% hint style="info" %}
Elementele de meniu **Adăugare fișiere**,**Adăugare folder**și**Pornire/Oprire procesare**sunt vizibile sau activate numai atunci când un proiect este deschis și au fost adăugate fișiere. Acestea oferă acces rapid la acțiuni disponibile și prin bara laterală**File Browser** și butoanele din antet.
{% endhint %}

## Proiectele rețin configurația hardware-ului dvs.

Noutate în versiunea 1.2.0: un proiect reține configurația hardware-ului pe care îl conectați cât timp este deschis. Camerele și matricea de camere (împreună cu setările specifice fiecărei camere, numele, culorile și dispunerea în grilă) sunt salvate în `cameras.json`, iar senzorii de lumină (împreună cu numele, culorile și asocierile cu camerele) în `sensors.json` — automat, pe măsură ce lucrați.

Când **redeschideți** un proiect, Chloros nu accesează imediat niciun dispozitiv hardware. Fiecare jumătate se reconectează la prima accesare a filei care o deține:

* Deschiderea filei **Camere** reconectează camerele și matrice-urile salvate și reaplică setările lor salvate.
* Deschiderea filei **Senzori de lumină** reconectează senzorii DAQ salvați.

Astfel, deschiderea unui proiect doar pentru a răsfoi sau a exporta imagini nu activează niciodată camerele în modul de transmisie în flux. Dacă un dispozitiv salvat nu poate fi găsit la deschiderea filei corespunzătoare, o fereastră de dialog vă indică ce dispozitive sunt indisponibile, astfel încât să le puteți reconecta sau elimina.

## Înregistrările DAQ și fișierele .daq dintr-un proiect

* Înregistrările `.daq` realizate în timp ce proiectul este deschis (din fila Senzori de lumină sau în timpul capturilor) sunt **adăugate automat la proiect**.
* Fișierele `.daq` importate, precum și toate înregistrările proiectului, sunt listate în secțiunea **Senzor de lumină DAQ** din [Setări proiect](project-settings/project-settings.md), fiecare cu profilul său de corecție a capului.
* În timpul procesării, fișierele `.daq` ale proiectului furnizează iluminarea descendentă pentru produsele de reflectanță — consultați [Formate de imagine de ieșire](output-image-formats.md).

## Executarea unui proiect salvat fără interfață grafică

Un proiect salvat poate fi rulat fără interfața grafică (GUI):

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` operează pe calea către folderul proiectului — consultați [Referința CLI](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` returnează un identificator de proiect; `connect_all()` activează toate camerele și senzorii salvați cu setările lor salvate — consultați [Referința SDK](reference/sdk-reference.md).
