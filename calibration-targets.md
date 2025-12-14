---
description: Lab-measured panels used to calibrate captured data in post-processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---
# Kalibracijske tarče

MAPIR ponuja različne kalibracijske tarče za različne namene uporabe. Kompaktni T4-R50, prikazan spodaj, vsebuje 4 plošče, ki so bile izmerjene za svetlobno odbojnost od 250 do 2500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

Difuzni referenčni cilji T4 imajo naslednje krivulje odbojnosti, [podatki za prenos tukaj](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Odbojnost :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Odbojnost :: 400–1000 nm</p></figcaption></figure>

Na grafu odbojnosti lahko vidite, da so vrednosti valovna dolžina (os x) v primerjavi z odstotkom odbojnosti (os y). Ko zajamemo sliko kalibracijskega cilja, ustvarimo razmerje med vrednostjo pikslov in odstotkom odbojnosti v spektru, na katerega so občutljivi vsi senzorji kamere.

To pomeni, da lahko za vsako sliko, ki jo zajamete z našimi kamerami, uporabite fotografijo naših ciljev odbojnosti, kot sta [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) ali [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), da kalibrirate slike za odbojnost. Po kalibraciji je vsak piksel v sliki enak odstotku odbojnosti.

Če kalibrirane slike izvozite v Chloros kot tipični JPG ali TIFF, se odstotek odbojnosti izračuna tako, da se vrednost piksla deli z bitno globino formata slike. Za JPG se deli z 255, za TIFF pa z 65.535. Lahko izberete tudi izhodni format PERCENT v Chloros, pri čemer bo vsak piksel v razponu od 0,0 do 1,0 (0 % do 100 % odbojnosti). Upoštevajte, da nekatere aplikacije za slike ne sprejemajo slik v odstotkih (plavajoča vejica) in da so te slike velike po velikosti shranjevanja.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
