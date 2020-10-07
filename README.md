**Voorbereiding:**   
Zorg dat je je Nano of NX flashed met de laatste versie van jetpack.   
Deze kan je op volgende link vinden:   
https://developer.nvidia.com/embedded/jetpack   
Persoonlijk gebruiken we Balena Etcher om deze succesvol weg te schrijven naar een SD kaart:   
https://www.balena.io/etcher/   
   
Om het makkelijk te maken werken we met Docker containers, geen paniek, je krijgt alle info mee om het tot een goed einde (of begin?) te brengen.   

**Benodigdheden:**   
1x MicroSD kaart van 16GB of meer: (https://neuralis.ai/?product_cat=microsd)   
1x Nvidia Jetson Nano (https://neuralis.ai/?product=nvidia-jetson-nano-developer-kit-b01)   
OF   
1x Nvidia Jetson Xavier NX (https://neuralis.ai/?product=nvidia-jetson-xavier-nx-developer-kit)   


**Eerste stappen:**   
Maak na het opstarten en configureren van je Nano of NX een makkelijk vindbare map aan.   
Om het makkelijk te houden doen we dit op het bureaublad.   

Je hebt volgende pakketten nodig in deze handleiding, voer dit commando uit in de terminal:
```
sudo apt install -y nano git
```

Je kan dit doen via de bestandsverkenner of via de terminal met commando:

```
mkdir $HOME/Desktop/mapnaam
```

Ga naar de nieuw aangemaakte map en clone deze repository met het commando:   
```
git clone https://github.com/Neuralis-AI/Objectclassificatie-Jeton-Nano-NX.git
```

Open de nieuwe map "Objectclassificatie-Jeton-Nano-NX" en verfolgens de map "configs"

In deze map zitten alle configuraties die je nodig hebt om objectclassificatie te doen met als bron een IP-camera (RTSP.txt), een aangesloten camera op de CSI connector (CSI.txt) of een mp4bestand (MP4.txt)

**Docker/Deepstream:**   

Om echt van start te gaan gaan we een Dockercontainer downloaden.   
Een Docker container is een soort "virtuele harde schijf" downloaden waar alle programma's en tools in voorgeinstalleerd staan.   

Voer volgende commando uit in de terminal:

```
xhost +
sudo docker run -it --rm --net=host --runtime nvidia  -e DISPLAY=$DISPLAY -w /opt/nvidia/deepstream/deepstream-5.0.1 -v /tmp/.X11-unix/:/tmp/.X11-unix -v $HOME/Desktop/mapnaam/Objectclassificatie-Jeton-Nano-NX/configs:/configs nvcr.io/nvidia/deepstream-l4t:5.0.1-20.09-samples
```

Let vooral op het stukje "-v $HOME/Desktop/mapnaam/Objectclassificatie-Jeton-Nano-NX/configs:/configs"   
Hiermee geven we de Docker container toegang tot deze map in ons eigen bestandssysteem.   
Indien je de mapnaam hebt aangepast moet je "mapnaam" even aanpassen naar je eigen uitgekozen naam.   
   
In de "configs" map hebben we 3 configuraties gemaakt, beiden afgeregeld op het neurale netwerk genaamd "YoloV3-Tiny".   
Dit netwerk is getrained om 80 verschillende objecten correct te kunnen detecteren en classificeren.   
Dit model is ook het beste qua snelheid/performance in combinatie met een goede accuracy en eenvoudig in gebruik zijn.   

Om het makkelijk te houden gebruiken we de meegeleverde video om een eerste snelle objectclassificatie uit te voeren.   

**Configuratie:**   
Allereerst moeten we enkele standaardhandelingen uitvoeren iedere keer dat we de container opstarten.   
Er zitten verschillende voorbeelden en netwerken in deze Docker image, maar we willen zelf bepalen welke functionaliteit beschikbaar is.   
Voer volgende commando's in volgorde uit:   

```
./install.sh
apt install nano ffmpeg wget build-essential -y
cd /opt/nvidia/deepstream/deepstream-5.0/sources/objectDetectorr_Yolo
./prebuild.sh
cd nvdsinfer_custom_impl_Yolo
CUDA_VER=10.2 make
cd ..
```

De installatie en configuratie is hiermee voltooid, de volgende stappen gaan over het uitvoeren van de configuraties.   

**Uitvoering:**   
*MP4*   
De makkelijkste eerste stap is om objectdetectie op de inbegrepen mp4 uit te voeren.   
Dit doe je door simpelweg volgende commando uit te voeren:   
```
deepstream-app -c /configs/MP4.txt
```
Als alles goed is zie je nu live de objectclassificatie op het scherm gebeuren.   

*CSI/onboard camera*   
De onboard camera is als 2e een goed beginpunt om te testen.   
We gaan er van uit dat de camera is aangesloten op de eerste poort.   
Voer volgende commando uit, en je zal op het livebeeld de objectclassificatie zien plaatsvinden:   
```
deepstream-app -c /configs/CSI.txt
```

*RTSP/IP Camera*   
RTSP heeft meer configuratie nodig om te werken.   
We moeten namelijk weten op welk ipadres en met welke url je camera te benaderen is.   
De makkelijkste manier om dit te doen is om het modelnummer van je camera in te geven op:   
https://www.ispyconnect.com/sources.aspx   
Een andere manier is om software als ONVIF Device Manager te gebruiken:   
https://sourceforge.net/projects/onvifdm/   

Om de configuratie aan te passen moeten we de configuratiefile RTSP.txt aanpassen in de configs map op ons gewoon systeem.   

Je ziet rond regel 46 volgende lijn:   
```
uri=rtsp://gebruikersnaam:wachtwoord:554/media/video1
```
Pas de regel na "uri=" aan naar de juiste connectiegegevens voor jou camera.   

Na het aanpassen sla je op, en voeren we in de docker container volgende commando uit om classificatie op de livestream uit te voeren:
```
deepstream-app -c /configs/RTSP.txt
```
   
Hiermee heb je het begin gemaakt in de fantastische wereld van Computer Vision / Deepstream / Machine Learning / A.I.   

Voor vragen en antwoorden kan je terecht bij sander@neuralis.ai, of kan je een issue aanmaken op deze repository.   
   
