<p align="center">
  <img src="https://user-images.githubusercontent.com/47336480/143769986-0c468db4-cbfe-478a-a291-f5c99a285f66.png">
</p>

# RaspiFM

## Creación de una emisora FM controlada por el móvil mediante bluetooth

###  Objetivos a cumplir

- [x] Instalar raspbian y configurarlo.
- [x] Ser capaz de emparejar el móvil a la raspberry mediante bluetooth.
- [x] Ser capaz de conectar el móvil a la raspberry mediante bluetooth.
- [x] Ser capaz de trasmitir en FM con la rapsberry.
- [x] Ser capaz de trasmitir audio por bluetooth y trasmitirlo por FM con la raspberry.
- [ ] (Overclock) Mejorar rendimiento de la raspi.
- [ ] (Opcional) Realizar el método de manera automática.

### Materiales

- [x] Una memoria micro SD >=4GB (Podria valer de menos capacidad).
- [x] Un ordenador.
- [x] Una Raspberry modelo 3B, similar o superior (En nuestro caso utilizaremos la raspberry pi zero 2 W).
- [x] Una antena o cable (Requiere soldar si no se tiene de pines).
- [x] Una bateria o cargador (En nuestro caso utilzaremos una bateria que ofrece 5V - 2.4 A).

### Enlaces de interés

- https://www.instructables.com/Raspberry-Pi-Wireless-Bluetooth-Audio-FM-Radio-Tra/
- https://github.com/ndrewh/Raspberry-pi-fm-radio-bluetooth
- https://atareao.es/ubuntu/gestionar-el-bluetooth-desde-la-terminal-en-linux/
- https://abnevme.wordpress.com/2015/05/28/bluetooth-audio-player-using-a-raspberrypi/
- https://gist.github.com/oleq/24e09112b07464acbda1
- http://www.crazy-audio.com/2014/09/pulseaudio-on-the-raspbery-pi/
- https://unix.stackexchange.com/questions/258074/error-when-trying-to-connect-to-bluetooth-speaker-org-bluez-error-failed
- https://raspberrypi.stackexchange.com/questions/92153/list-of-bluetoothctl-paired-devices-is-empty-after-a-reboot
- https://scribles.net/streaming-bluetooth-audio-from-phone-to-raspberry-pi-using-alsa/#Step03
- https://variwiki.com/index.php?title=BlueZ5_and_A2DP

---

### 1. Instalación de Raspbian

##### 1.1 Descarga de la herramienta Pi Imager

> Para la instalacion del sistema operativo (raspbian lite) utilizaremos la herramienta Pi Imager. Para su instalación en linux basado en debian ejecutamos el siguiente comando.

~~~bash
wget https://downloads.raspberrypi.org/imager/imager_latest_amd64.deb
sudo apt install ./imager_latest_amd64.deb
~~~

> Se descargara el ejecutable .deb desde la pagina oficial e instalara la herramienta (*Alternativamente se puede realizar la descarga a través de un navegador*).
>
> 
>
> Seguidamente ejecutamos el programa, abriendose su interfaz.



![pi-imager](https://user-images.githubusercontent.com/47336480/143769448-88335eca-17c7-4f95-8e78-56ae9c89fc40.png)
##### 1.2 Instalación del SO a la raspberry

> El siguiente paso a realizar es la instalación del propio sistema operativo en la raspberry. Para ello insertamos la tarjeta micro SD al ordenador y seleccionamos el sistema operativo que se quiere utilizar (en nuestro caso sera **Raspberry Pi OS Lite (32-Bit)**).



![pi-imager-paso-2](https://user-images.githubusercontent.com/47336480/143769459-e3edddaa-4e4e-4e3a-a099-4b63c73cafe2.png)


> Antes de comenzar la escritura del SO, necesitamos configurar algunos parámetros para ser capaces de acceder a la raspberry sin la necesidad de utilizar una conexión HDMI a un monitor.
>
> 
>
> *Si no es tu caso, puede obviar el siguiente paso* y clicar en write para empezar la instalación.

###### 1.2.1 (Opcional) Configuración avanzada en pi imager

> Para poder acceder a la configuración avanzada, debemos pulsar simultáneamente el conjunto de teclas <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>x</kbd>. Esto nos abrirá una ventana emergente como la siguiente.

![pi-imager-adv](https://user-images.githubusercontent.com/47336480/143769476-f9bbaff3-0ed2-4fd0-b908-2fab02d3a142.png)
> Aqui debemos configurar los siguiente parametros:
>
> - **Disable overscan**: Dejamos sin marcar
>
> - **Set hostname**: Introducimos el nombre del host. En nuestro caso utilizaremos *piportalnet*
>
> - **Enable SSH**: Marcamos la casilla
>  - Use password authentication: Marcamos la casilla
>    - Set password for 'pi' user: Introducimos la contraseña para el SSH
>   - Allow public-key authentication only: Nosotros no la marcamos. Si se quisiese utilizar una pubkey se marcaría.
>
> - **Configure wifi**: Marcamos la casilla.
>   - SSID: Introducimos el nombre del wifi. En nuestro caso devolo-97c
>  - Password: Introducimos la contraseña del wifi.
>   - Wifi country: Buscamos el país que corresponda. En nuestro caso ES (España)
>
> - **Set local settings**: Marcamos la casilla
>  - Time zone: Buscamos la zona horaria en la que estamos. En nuestro caso Europe/Madrid.
>   - Keyboard layout: Introducimos la distribución de teclado que se tenga. En nuestro caso lo dejamos por defecto
>
> 
>
>  El resto de apartados los dejamos como están.

> Tras finalizar guardamos y clicamos en write para empezar la instalación.

### 2. Conexión a raspberry y primeros pasos

##### 2.1 Conexión a la rasbperry pi

> Una vez finalice el proceso, expulse el medio extraíble del ordenador e introdúzcalo en la raspberry. Conectele asimismo una batería o cargador a la raspi y espere 30s ~ 1min.

> Tras haberse arrancado, pasamos a conectarnos por SSH. Para ello, es necesario estar conectado a la misma red que la raspberry y conocer su IP. Para ello, podemos hacer uso de herramientas como nmap, escaneando la red en busca de un dispositivo que tenga habilitado el puerto 22 (SSH) unicamente o alternativamente hacer uso de aplicaciones móviles como Fing.

*Usando nmap*

~~~bash
nmap 192.168.1.0/24
[...]
Nmap scan report for 192.168.1.12
Host is up (0.015s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
[...]
~~~

> Una vez conocemos su Ip, realizamos la conexión SSH. Necesitaremos la contraseña definida en *1.2.1*

~~~bash
ssh pi@192.168.1.12
pi@192.168.1.12's password: 
Linux piportalnet 5.10.63-v7+ #1459 SMP Wed Oct 6 16:41:10 BST 2021 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Nov 11 21:26:56 2021 from 192.168.1.35
pi@piportalnet:~ $ 
~~~

##### 2.2 Actualización raspberry y primeros pasos

###### 2.2.1 Actualización

> Tras conectarnos por ssh, pasamos a actualizar el SO. Para ello ejecutamos los siguientes comandos.

~~~bash
sudo apt update
sudo apt upgrade -y
~~~

###### 2.2.2 (Opcional) Establecemos contraseña root

> Tras actualizar pasamos a establecer la contraseña del usuario root.

~~~bash
sudo su
passwd
New password: 
Retype new password: 
passwd: password updated successfully
New password: 
Retype new password: 
passwd: password updated successfully
~~~

###### 2.2.3 (Opcional) Aumentamos memoria SWAP

> Verificando las características del hardware podemos ver que el tamaño de la memoria swap es insuficiente.

~~~bash
htop
~~~

![htop](https://user-images.githubusercontent.com/47336480/143769487-877988d8-d562-48cd-8029-3f742ebf47df.png)
> Para aumentar el tamaño de la memoria swap pasamos a realizar los siguientes pasos:

~~~bash
sudo dphys-swapfile swapoff
sudo nano /etc/dphys-swapfile # Cambiamos el parametro CONF_SWAPSIZE al tamaño que queramos. En nuestro caso CONF_SWAPSIZE=2048
sudo dphys-swapfile setup
sudo dphys-swapfile swapon
~~~

> Reiniciamos posteriormente la rasperry. Comprobamos el nuevo tamaño de la memoria swap.

![htop-2](https://user-images.githubusercontent.com/47336480/143769499-c21dc314-a125-496d-8c5c-7301e7da6d8e.png)
###### 2.2.4 (Opcional) Establecemos WIFI prioritarias

> En el paso 1.2.1 realizamos la configuración para que conectarse a una wifi en concreto. No obstante, en este paso pasamos a definir dentro de un pool a nuestra elección, a que wifi debe conectarse la raspberry en función de una prioridad definida.

> Para ello hay que modificar el siguiente archivo.

~~~bash
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf

network={
        ssid="<nombre del wifi a conectarse>"
        psk="<contraseña del wifi a conectarse>"
        priority=1 # Prioridad de la red
}
~~~

> Por cada wifi que queramos realizar alguna acción debe definirse en el fichero wpa_supplicant.conf el esquema que se muestra. En ella se establece el nombre del wifi, su contraseña y la prioridad que tiene. La prioridad define a que wifi debe la raspberry conectarse antes en caso de que haya 2 o mas wifi definidas en el fichero. **Mayor número, mayor prioridad**.

> Tras realizar los cambios se guarda y se reinicia.

### 3. Instalación y configuración de los recursos necesarios

###### 3.1. Instalación de los paquetes necesarios

> A continuación pasamos a realizar la instalación de los paquetes, recursos necesarios para llevar a cabo el proyecto. Para ello, en primer lugar, pasamos a instalar los siguientes paquetes.

~~~bash
sudo apt update
sudo apt install alsa-utils git bluez bluez-tools pulseaudio pulseaudio-module-bluetooth pulseaudio-utils sox libsox-fmt-all python-gobject python-gobject-2
~~~

> Asimismo, para realizar ciertas pruebas, bajamos estos paquetes adicionales

~~~bash
sudo apt install youtube-dl ffmpeg
~~~

##### 3.2. Configuración de los ficheros necesarios

**/etc/bluetooth/audio.conf**

> Añadimos estas lineas. Creamos el fichero si no lo esta

~~~bash
sudo nano /etc/bluetooth/audio.conf

[General]:
Enable=Source,Sink,Media,Socket
~~~

**/etc/pulse/daemon.conf**

> Añadimos estas lineas al final del documento

~~~bash
sudo nano /etc/pulse/daemon.conf
[...]
resample-method = trivial
exit-idle-time = -1
~~~

**/etc/bluetooth/main.conf**

> Modificamos estas lineas para personalizar el nombre del dispositivo y la clase

~~~bash
sudo nano /etc/bluetooth/main.conf
[...]
Name = piportal
Class = 0x20041C
~~~

**/etc/pulse/default.pa**

> Comentamos la siguiente línea para que no se pierda la conexión bt al pausar el audio.

~~~bash
sudo nano /etc/pulse/default.pa
[...]
#load-module module-suspend-on-idle
[...]
~~~

### 4. Emparejamiento y conexión Bluetooth

> En esta apartado pasamos a conectar la raspberry pi a un teléfono móvil mediante bluetooth. Para ello pasamos a emparejarlos. 

> Antes de nada, pasamos a verificar que el demonio bluetooth esta activo.

~~~bash
sudo systemctl status bluetooth.service 
● bluetooth.service - Bluetooth service
     Loaded: loaded (/lib/systemd/system/bluetooth.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2021-11-13 13:57:01 CET; 23h ago
       Docs: man:bluetoothd(8)
   Main PID: 486 (bluetoothd)
     Status: "Running"
      Tasks: 1 (limit: 409)
        CPU: 207ms
     CGroup: /system.slice/bluetooth.service
             └─486 /usr/libexec/bluetooth/bluetoothd

Nov 13 13:57:01 piportalnet bluetoothd[486]: Bluetooth daemon 5.55
Nov 13 13:57:01 piportalnet systemd[1]: Started Bluetooth service.
Nov 13 13:57:01 piportalnet bluetoothd[486]: Starting SDP server
Nov 13 13:57:01 piportalnet bluetoothd[486]: Bluetooth management interface 1.18 initialized
Nov 13 13:57:01 piportalnet bluetoothd[486]: profiles/sap/server.c:sap_server_register() Sap driver initialization failed.
Nov 13 13:57:01 piportalnet bluetoothd[486]: sap-server: Operation not permitted (1)
Nov 13 13:57:01 piportalnet bluetoothd[486]: Failed to set privacy: Rejected (0x0b)
Nov 14 13:03:01 piportalnet bluetoothd[486]: Endpoint registered: sender=:1.27 path=/MediaEndpoint/A2DPSink/sbc
Nov 14 13:03:01 piportalnet bluetoothd[486]: Endpoint registered: sender=:1.27 path=/MediaEndpoint/A2DPSource/sbc
~~~

> Una vez comprobado, pasamos ahora si a emparejar el móvil con la raspberry.

~~~bash
sudo bluetoothctl 
Agent registered
[CHG] Controller **:**:**:**:**:** Pairable: yes
[bluetooth]# agent on
Agent is already registered
[bluetooth]# default-agent 
Default agent request successful
[bluetooth]# show
Controller **:**:**:**:**:** (public)
	Name: piportalnet
	Alias: piportalnet
	Class: 0x002c0000
	Powered: yes
	Discoverable: no
	DiscoverableTimeout: 0x000000b4
	Pairable: yes
[...]
[bluetooth]# discoverable on
Changing discoverable on succeeded
[CHG] Controller **:**:**:**:**:** Discoverable: yes
~~~

> Desde el móvil buscamos el nombre del dispositivo bt de la raspberry (en nuestro caso piportal) y nos conectamos. En la consola de la raspberry aparecerá la confirmación de la passkey que aparece en el móvil. Se acepta tanto en el móvil como en la consola de la raspberry y se establece el emparejamiento.
>
> 
>
> **NOTA**:*Cabe aclarar que como hemos cambiado la clase del bt de la raspberry, este aparecerá como unos periféricos de audio (0x20041C)*

~~~bash
[NEW] Device **:**:**:**:**:** sride # Mi telefono movil
Request confirmation
[agent] Confirm passkey ****** (yes/no): yes
~~~

> Tras emparejar el móvil, pasamos a marcar la opción de dispositivo de confianza en la consola de la raspberry.

~~~bash
[sride]# paired-devices 
Device **:**:**:**:**:** sride
[sride]# trust **:**:**:**:**:**
[CHG] Device **:**:**:**:**:** Trusted: yes
Changing **:**:**:**:**:** trust succeeded
~~~

> **NOTA:** Si se ha podido realizar el emparejamiento pero no la conexión entre el móvil y al raspberry, pasamos a verificar si el demonio de pulseaudio está activo. 

~~~bash
systemctl --user status pulseaudio.service 
● pulseaudio.service - Sound Service
     Loaded: loaded (/usr/lib/systemd/user/pulseaudio.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2021-11-14 13:03:01 CET; 29min ago
TriggeredBy: ● pulseaudio.socket
   Main PID: 615 (pulseaudio)
      Tasks: 2 (limit: 409)
        CPU: 42.780s
     CGroup: /user.slice/user-1000.slice/user@1000.service/app.slice/pulseaudio.service
             └─615 /usr/bin/pulseaudio --daemonize=no --log-target=journal

Nov 14 13:19:38 piportalnet pulseaudio[615]: Configured latency of 200.00 ms is smaller than minimum latency, using minimum instead
Nov 14 13:19:38 piportalnet pulseaudio[615]: Cannot set requested source latency of 150.75 ms, adjusting to 250.00 ms
Nov 14 13:19:39 piportalnet pulseaudio[615]: Configured latency of 200.00 ms is smaller than minimum latency, using minimum instead
Nov 14 13:19:39 piportalnet pulseaudio[615]: Cannot set requested source latency of 66.67 ms, adjusting to 68.54 ms
Nov 14 13:19:40 piportalnet pulseaudio[615]: Too many underruns, increasing latency to 205.00 ms
Nov 14 13:20:00 piportalnet pulseaudio[615]: Too many underruns, increasing latency to 210.00 ms
Nov 14 13:20:50 piportalnet pulseaudio[615]: Too many underruns, increasing latency to 215.00 ms
Nov 14 13:21:50 piportalnet pulseaudio[615]: Too many underruns, increasing latency to 220.00 ms
Nov 14 13:31:39 piportalnet pulseaudio[615]: Cannot set requested source latency of 66.67 ms, adjusting to 68.54 ms
Nov 14 13:31:39 piportalnet pulseaudio[615]: Too many underruns, increasing latency to 205.00 ms
~~~

> En caso contrario seria necesario activarlo.

~~~bash
systemctl --user start pulseaudio.service 
systemctl --user enable pulseaudio.service 
~~~

### 5. Transmisor FM 

##### 5.1.Descarga e instalación del código

> Pasamos en este apartado a descargar el código para poder convertir la raspberry en un transmisor FM. El código que se usa es el realizado por 
>
> [F5OEO](https://github.com/F5OEO) en su proyecto [rpitx](https://github.com/F5OEO/rpitx).

~~~bash
git clone https://github.com/F5OEO/rpitx.git
Cloning into 'rpitx'...
remote: Enumerating objects: 1591, done.
remote: Counting objects: 100% (83/83), done.
remote: Compressing objects: 100% (57/57), done.
remote: Total 1591 (delta 25), reused 46 (delta 16), pack-reused 1508
Receiving objects: 100% (1591/1591), 10.63 MiB | 2.54 MiB/s, done.
Resolving deltas: 100% (876/876), done.
~~~

~~~bash
cd rpitx
./install.sh
~~~

> Tras realizar la instalación, se requiere reinicio.

~~~bash
sudo reboot
~~~

##### 5.2 Configuración del dispositivo bt en pulseaudio

> Un paso adicional, es indicar al dispositivo bluetooth móvil detectado como tarjeta de audio en pulseaudio, el profile activo que tiene que asumir. En primero lugar necesitamos antes conectar el teléfono móvil a la raspberry mediante bt. 
>
> Una vez realizado, ejecutamos.

~~~bash
pactl list cards
Card #0
	Name: bluez_card.**_**_**_**_**_**
	Driver: module-bluez5-device.c
	Owner Module: 20
	Properties:
		device.description = "sride"
		device.string = "**:**:**:**:**:**"
		device.api = "bluez"
		device.class = "sound"
		device.bus = "bluetooth"
		device.form_factor = "phone"
		bluez.path = "/org/bluez/hci0/dev_**_**_**_**_**_**"
		bluez.class = "0x5a020c"
		bluez.alias = "sride"
		device.icon_name = "audio-card-bluetooth"
	Profiles:
		a2dp_source: High Fidelity Capture (A2DP Source) (sinks: 0, sources: 1, priority: 20, available: yes)
		headset_audio_gateway: Headset Audio Gateway (HSP/HFP) (sinks: 1, sources: 1, priority: 10, available: yes)
		off: Off (sinks: 0, sources: 0, priority: 0, available: yes)
	Active Profile: off
	Ports:
		phone-output: Phone (type: Phone, priority: 0, latency offset: 0 usec, availability unknown)
			Part of profile(s): headset_audio_gateway
		phone-input: Phone (type: Phone, priority: 0, latency offset: 0 usec, availability unknown)
			Part of profile(s): a2dp_source, headset_audio_gateway
~~~

~~~bash
pactl set-card-profile 0 a2dp_source 
pactl list cards
Card #0
	Name: bluez_card.**_**_**_**_**_**
	Driver: module-bluez5-device.c
	Owner Module: 20
	Properties:
		device.description = "sride"
		device.string = "**:**:**:**:**:**"
		device.api = "bluez"
		device.class = "sound"
		device.bus = "bluetooth"
		device.form_factor = "phone"
		bluez.path = "/org/bluez/hci0/dev_**_**_**_**_**_**"
		bluez.class = "0x5a020c"
		bluez.alias = "sride"
		device.icon_name = "audio-card-bluetooth"
	Profiles:
		a2dp_source: High Fidelity Capture (A2DP Source) (sinks: 0, sources: 1, priority: 20, available: yes)
		headset_audio_gateway: Headset Audio Gateway (HSP/HFP) (sinks: 1, sources: 1, priority: 10, available: yes)
		off: Off (sinks: 0, sources: 0, priority: 0, available: yes)
	Active Profile: a2dp_source
	Ports:
		phone-output: Phone (type: Phone, priority: 0, latency offset: 0 usec, availability unknown)
			Part of profile(s): headset_audio_gateway
		phone-input: Phone (type: Phone, priority: 0, latency offset: 0 usec, availability unknown)
			Part of profile(s): a2dp_source, headset_audio_gateway
~~~
### 6. Instalación de la antena
> Tras haber realizado la configuración de la raspberry, es recomedable que añadamos una antena para obtener un mayor alcance en la trasmisión. En un principio, los propios pines de las raspberry (si lo tiene) ya actuarían como una antena, aunque con un rango limitado. Por ello, para aumentar el alcance, bastaría con añadir un cable de unos 10 ~ 20 cm. En nuestro caso, pasaremos a utilizar una antena desplegable de una radio antigua.

> Para la instalación únicamente necesitamos conectar/soldar la antena en el gpio 4.

![gpio_pin_4](https://user-images.githubusercontent.com/47336480/143771819-5d1ab77c-83b7-4f65-8f45-3a63e74b5aba.png)

> El resultado en nuestro caso es el siguiente:

![antena_resultado](https://user-images.githubusercontent.com/47336480/143772074-9dea9a68-0652-40d8-8cf4-dea8ae0af925.jpeg)
### 7. Prueba de transmisión

> Una vez realizado todos los pasos anteriores, pasamos a realizar las distintas pruebas para comprobar que se cumplan los requisitos indicados al principio de este tutorial.

##### 7.1. Prueba de trasmisión FM de una canción descargada desde la raspberry.

> En esta prueba descargaremos una canción de youtube mediante la herramienta youtube-dl, convertiremos el fichero de audio en formato .wav (que es el que acepta rpitx) y trasmitiremos la canción obtenida por FM. 

##### 7.1.1. Descarga de la canción mediante youtube-dl

>Buscamos la canción que queramos descargar en youtube y copia su URL.
>
>En nuestro caso sera la siguiente cancion: [Queen – Bohemian Rhapsody (Official Video Remastered)](https://www.youtube.com/watch?v=fJ9rUzIMcZQ)

> Utilizando la herramienta youtube-dl, listamos en primer lugar las diferentes configuraciones en las que se puede descargar la canción.

~~~bash
youtube-dl -F https://www.youtube.com/watch?v=fJ9rUzIMcZQ
[youtube] fJ9rUzIMcZQ: Downloading webpage
[youtube] fJ9rUzIMcZQ: Downloading player 8d287e4d
[info] Available formats for fJ9rUzIMcZQ:
format code  extension  resolution note
249          webm       audio only tiny   51k , webm_dash container, opus @ 51k (48000Hz), 2.20MiB
250          webm       audio only tiny   67k , webm_dash container, opus @ 67k (48000Hz), 2.91MiB
140          m4a        audio only tiny  129k , m4a_dash container, mp4a.40.2@129k (44100Hz), 5.55MiB
251          webm       audio only tiny  132k , webm_dash container, opus @132k (48000Hz), 5.68MiB
394          mp4        256x144    144p   69k , mp4_dash container, av01.0.00M.08@  69k, 25fps, video only, 2.96MiB
278          webm       256x144    144p   87k , webm_dash container, vp9@  87k, 25fps, video only, 3.75MiB
160          mp4        256x144    144p  110k , mp4_dash container, avc1.4d400c@ 110k, 25fps, video only, 4.75MiB
395          mp4        426x240    240p  111k , mp4_dash container, av01.0.00M.08@ 111k, 25fps, video only, 4.79MiB
242          webm       426x240    240p  157k , webm_dash container, vp9@ 157k, 25fps, video only, 6.76MiB
133          mp4        426x240    240p  245k , mp4_dash container, avc1.4d4015@ 245k, 25fps, video only, 10.51MiB
396          mp4        640x360    360p  214k , mp4_dash container, av01.0.01M.08@ 214k, 25fps, video only, 9.20MiB
243          webm       640x360    360p  286k , webm_dash container, vp9@ 286k, 25fps, video only, 12.27MiB
134          mp4        640x360    360p  436k , mp4_dash container, avc1.4d401e@ 436k, 25fps, video only, 18.72MiB
397          mp4        854x480    480p  363k , mp4_dash container, av01.0.04M.08@ 363k, 25fps, video only, 15.57MiB
244          webm       854x480    480p  491k , webm_dash container, vp9@ 491k, 25fps, video only, 21.06MiB
135          mp4        854x480    480p  835k , mp4_dash container, avc1.4d401e@ 835k, 25fps, video only, 35.79MiB
398          mp4        1280x720   720p  688k , mp4_dash container, av01.0.05M.08@ 688k, 25fps, video only, 29.51MiB
247          webm       1280x720   720p  950k , webm_dash container, vp9@ 950k, 25fps, video only, 40.75MiB
136          mp4        1280x720   720p 1588k , mp4_dash container, avc1.4d401f@1588k, 25fps, video only, 68.07MiB
399          mp4        1920x1080  1080p 1241k , mp4_dash container, av01.0.08M.08@1241k, 25fps, video only, 53.19MiB
248          webm       1920x1080  1080p 1820k , webm_dash container, vp9@1820k, 25fps, video only, 77.99MiB
137          mp4        1920x1080  1080p 3147k , mp4_dash container, avc1.640028@3147k, 25fps, video only, 134.88MiB
18           mp4        640x360    360p  500k , avc1.42001E, 25fps, mp4a.40.2 (44100Hz), 21.44MiB (best)
~~~

> En nuestro caso solo queremos el audio, por lo que seleccionamos el 3 perfil, con código 140.
>
> Para bajar la canción con esa configuración ejecutamos.

~~~bash
youtube-dl -f 140 https://www.youtube.com/watch?v=fJ9rUzIMcZQ 
[youtube] fJ9rUzIMcZQ: Downloading webpage
[download] Destination: Queen – Bohemian Rhapsody (Official Video Remastered)-fJ9rUzIMcZQ.m4a
[download] 100% of 5.55MiB in 01:11
[ffmpeg] Correcting container in "Queen – Bohemian Rhapsody (Official Video Remastered)-fJ9rUzIMcZQ.m4a"
pi@piportalnet:~ $ ls
'Queen – Bohemian Rhapsody (Official Video Remastered)-fJ9rUzIMcZQ.m4a'   rpitx
~~~

##### 7.1.2. Conversión del formato del fichero de audio

> Tras descargar la canción necesitamos cambiar la extensión del mismo a .wav. Para ello hacemos uso de la herramienta ffmpeg.

~~~bash
ffmpeg -hide_banner -i Queen\ –\ Bohemian\ Rhapsody\ \(Official\ Video\ Remastered\)-fJ9rUzIMcZQ.m4a queen.wav
Input #0, mov,mp4,m4a,3gp,3g2,mj2, from 'Queen – Bohemian Rhapsody (Official Video Remastered)-fJ9rUzIMcZQ.m4a':
  Metadata:
    major_brand     : isom
    minor_version   : 512
    compatible_brands: isomiso2mp41
    encoder         : Lavf58.45.100
  Duration: 00:05:59.49, start: 0.000000, bitrate: 129 kb/s
    Stream #0:0(eng): Audio: aac (LC) (mp4a / 0x6134706D), 44100 Hz, stereo, fltp, 127 kb/s (default)
    Metadata:
      handler_name    : ISO Media file produced by Google Inc.
Stream mapping:
  Stream #0:0 -> #0:0 (aac (native) -> pcm_s16le (native))
Press [q] to stop, [?] for help
Output #0, wav, to 'queen.wav':
  Metadata:
    major_brand     : isom
    minor_version   : 512
    compatible_brands: isomiso2mp41
    ISFT            : Lavf58.45.100
    Stream #0:0(eng): Audio: pcm_s16le ([1][0][0][0] / 0x0001), 44100 Hz, stereo, s16, 1411 kb/s (default)
    Metadata:
      handler_name    : ISO Media file produced by Google Inc.
      encoder         : Lavc58.91.100 pcm_s16le
size=   61928kB time=00:05:59.49 bitrate=1411.2kbits/s speed=84.9x    
video:0kB audio:61928kB subtitle:0kB other streams:0kB global headers:0kB muxing overhead: 0.000123%
pi@piportalnet:~ $ ls
'Queen – Bohemian Rhapsody (Official Video Remastered)-fJ9rUzIMcZQ.m4a'   queen.wav   rpitx
~~~

##### 7.1.3. Transmisión de la canción

> Solo queda ejecutar rpitx para que trasmite la canción decargada. En el directorio donde tengamos la canción y el código, ejecutamos.

~~~bash
sudo /home/pi/rpitx/pifmrds -ps Srid_FM -rt 'Audio trasmitido por Raspberry Pi Zero 2 W' -freq 88 -audio queen.wav
~~~

> En el comando anterior se inicializa el modulo fmrds donde con frecuencia 88.0 Mhz (-freq 88) va a trasmitir la canción que bajamos (-audio queen.wav). Los otro parámetros no son necesarios, es para poder ver en receptor el título (-rt) y el nombre de la cadena (-ps)

> Tras pulsar <kbd>Enter</kbd>, deberemos sintonizar una radio FM en la frecuencia indicada para escuchar la canción. Si todo ha salido bien, debería escucharse sin mayor problema.
>
> 
>
> **CONSEJO:** Si no se escucha en condiciones, pruebe a trasmitir en una frecuencia sin emisión aparente.
>
> 
>
> **NOTA:** Emitir en la banda de frecuencias FM es ilegal en la mayoría de países. Por favor, realicen los pasos de este tutorial baja vuestra responsabilidad.

##### 7.2. Prueba de transmisión FM de una canción trasmitido por bt desde el móvil a la raspberry.

> En esta segunda prueba implementaremos el uso del móvil para trasmitir audio a través de bluetooth, donde los datos trasmitidos son redirigidos a rpitx para que pueda realizar la transmisión FM.

> En primer lugar conectamos el móvil con la raspberry mediante bt. 

> Seguidamente pasamos a ejecutar el siguiente comando. Se hará uso de sox y pipelines.

~~~bash
pacat -r -d bluez_source.**_**_**_**_**_**.a2dp_source | sox -t raw -r 44100 -e signed-integer -b 16 -c 2 - -t wav - |
sudo /home/pi/rpitx/pifmrds -ps Srid_FM -rt 'Audio trasmitido por Raspberry Pi Zero 2 W' -freq 88 -audio -
~~~

---

> En el comando anterior se usa el reproductor pacat en modo grabación (-r) cogiendo como fuente el bluetooth del móvil (-d) configurado en el **apartado 5.2**. 
>
> Seguidamente si concatena la salida de este con sox, herramienta que nos permite convertir los datos en crudo reproducidos por pacat en datos con formato wav. Se especifica el ratio de muestro (-r), el numero de bits por muestra (-b), el numero de canales (-c) y el tipo de formato final (-t). El - final indica que se va a redirigir el resultado. 
>
> Por ultimo, los datos generados por sox son consumidos por rpitx, donde el - final indica que se va a consumir de una redirección.

> El resultado final será que a través del móvil pongamos a reproducir un audio y este se trasmite por FM en la frecuencia indicada.

## Líneas futuras
> En líneas futuras, se pretende que la trasmisión FM se incie nada mas se conecte el móvil a la raspberry de manera automática. De la misma forma, cuando se detecte que se desconecta le móvil de la raspberry, se finalice la trasmisión FM .
