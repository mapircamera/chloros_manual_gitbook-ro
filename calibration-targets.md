---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Ținte de calibrare

MAPIR oferă diverse ținte de calibrare pentru o gamă largă de aplicații. Modelul compact T4-R50 prezentat mai jos conține 4 panouri a căror reflectanță a luminii a fost măsurată în intervalul 250 - 2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Țintele de referință difuze T4 au următoarele curbe de reflexie, [descărcați datele aici](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR Reflectanță T4 :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR Reflectanță T4 :: 400-1000 nm</p></figcaption></figure>Țintele de referință difuze T4P au următoarele curbe de reflectanță, [descărcare date aici](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR Reflectanță T4P :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR Reflectanță T4P :: 400-1000 nm</p></figcaption></figure>Analizând graficul de reflectanță, puteți observa că valorile reprezintă lungimea de undă (axa x) în raport cu procentul de reflectanță (axa y). Când capturăm o imagine a țintei de calibrare, creăm o relație între valoarea pixelului și procentul de reflectanță, în cadrul spectrului la care fiecare dintre benzile senzorului camerei este sensibilă.

Aceasta înseamnă că, pentru fiecare imagine capturată cu camerele noastre, puteți utiliza o fotografie a țintelor noastre de reflectanță, cum ar fi [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) sau [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), pentru a calibra imaginile în funcție de reflectanță. Odată calibrate, fiecare pixel din imagine este egal cu procentul de reflectanță.

Dacă exportați imaginile calibrate în Chloros ca fișiere JPG obișnuite sau TIFF, procentul de reflectanță se calculează împărțind valoarea pixelului la adâncimea de biți a formatului de imagine. Deci, pentru JPG se împarte la 255, iar pentru TIFF se împarte la 65.535. De asemenea, puteți alege formatul de ieșire PERCENT în Chloros, iar atunci fiecare pixel va avea o valoare procentuală cuprinsă între 0,0 și 1,0 (reflectanță de la 0% la 100%). Rețineți însă că unele aplicații de imagine nu acceptă imaginile procentuale (cu virgulă mobilă) și că acestea ocupă mult spațiu de stocare.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
