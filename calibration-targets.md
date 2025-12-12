---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Ținte de calibrare

MAPIR oferă diverse ținte de calibrare pentru a acoperi o gamă largă de aplicații. Modelul compact T4-R50 prezentat mai jos conține 4 panouri care au fost măsurate pentru reflexia luminii între 250 și 2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Țintele de referință difuze T4 au următoarele curbe de reflexie, [descărcare date aici](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Reflectanță :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Reflectanță :: 400-1000 nm</p></figcaption></figure>Privind graficul de reflectanță, puteți observa că valorile sunt lungimea de undă (axa x) versus procentul de reflectanță (axa y). Când capturăm o imagine a țintei de calibrare, creăm o relație între valoarea pixelului și procentul de reflectanță, în spectrul la care sunt sensibile fiecare dintre benzile senzorului camerei.

Aceasta înseamnă că, pentru fiecare imagine capturată cu camerele noastre, puteți utiliza o fotografie a țintelor noastre de reflectanță, cum ar fi [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) sau [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), pentru a calibra imaginile în funcție de reflectanță. Odată calibrat, fiecare pixel din imagine este egal cu procentul de reflectanță.

Dacă exportați imaginile calibrate în Chloros ca JPG tipic sau TIFF, atunci procentul de reflectanță este calculat prin împărțirea valorii pixelului la adâncimea de biți a formatului imaginii. Deci, pentru JPG împărțiți la 255, iar pentru TIFF împărțiți la 65.535. De asemenea, puteți alege formatul de ieșire PERCENT în Chloros, iar apoi fiecare pixel va avea o valoare procentuală cuprinsă între 0,0 și 1,0 (reflectanță de 0% până la 100%). Rețineți că unele aplicații de imagine nu acceptă imaginile procentuale (cu virgulă mobilă) și că acestea ocupă mult spațiu de stocare.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
