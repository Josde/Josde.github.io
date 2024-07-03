---
title: "Cosas que NO hacer con una Wii"
date: 2021-06-12T18:11:28+02:00
draft: false
toc: true
images:
summary: "Hay muchas cosas que se pueden hacer con una Wii. Yo, que soy estúpido, decidí explorar las que no deberías de hacer."
tags:
  - wii
  - homebrew
  - linux
---

Durante los últimos días, mi atención ha sido absorbida por un tema en específico, como si de un agujero negro se tratara. ¿El tema en cuestión? I**ntentar encontrar cosas que hacer con la Wii que tenía por ahí.  **
  
Abajo documento todo lo que he encontrado, sólo para que no tengáis que seguir el mismo camino que tomé yo.

## Requisitos

- Tarjeta SD de más de 1GB. Se recomienda más grande. **NO puede ser SDXC.**
- Teclado USB.
- Opcionalmente, un USB para funcionar como partición de Swap.

## Homebrew

Lo primero de todo: instalar Homebrew es básicamente lo único que recomiento de todo este artículo.  
Es facilísimo y cualquiera lo puede hacer, y además permite instalar un montón de aplicaciones útiles (para rippear tus juegos, cargarlos desde un USB, jugar online después de que Nintendo quitase sus servidores, o joder, incluso poder ver películas desde la Wii).  

Para esto, os recomiendo seguir **[wii.guide](https://wii.hacks.guide/)**. En su momento yo usé el método de Letterbomb, pero desde que hice este artículo hasta que lo traducí se han descubierto métodos aún más fáciles y seguros.  
  
Veredicto final: ¡Deberíais de estar haciendólo ya! Es súper fácil y además no tiene complicación.

## Linux  
**NOTA: Esta guía fue escrita para una [versión anterior](https://github.com/neagix/wii-linux-ngx) de wii-linux-ngx basada en Linux 3.x, que llevaba estancada desde 2017. Desde entonces, un grupo ha cogido el proyecto y está trabajando en pasarlo a Linux 4.x, que es la versión del siguiente enlace, por tanto puede que las cosas de las que hable sean redundantes, erróneas e incluso dañinas.**
  
Después de instalar Homebrew en la Wii, instalar Linux es tan fácil como descargar la última versión de **[wii-linux-ngx](https://github.com/Wii-Linux/wii-linux-ngx)** y flashearlo en una tarjeta SD con **[balenaEtcher](https://www.balena.io/etcher/)**. Después de esto, hay que cambiar el tamaño de la partición que se ha creado para que sea más grande, y opcionalmente formatear una partición de swap en el USB.  
Con todo este setup montado, **conectamos el USB y el teclado a la parte trasera de la Wii, e insertamos la SD**. La encendemos y lanzamos el Homebrew Channel, le damos al botón HOME (la casita) en el mando de la Wii, y seleccionamos "Launch BootMii". Esto lanzará un menú parecido al de GRUB. Tras pulsar el botón de RESET en la Wii durante apróximadamente un segundo, entraremos en Linux por primera vez, aunque la instalación no ha terminado aún.  
  
Lo primero que os recomiento es activar el SWAP, puesto que la Wii sólo tiene ~70MB de RAM (¿loquísimo, eh?). Ejecutamos `fdisk -l` e identificamos nuestra partición de SWAP. Modificamos el `/etc/fstab/` para reflejar dónde está, y finalmente ejecutamos `swapon -a`.  
  
Otra cosa que seguramente queráis es acceso a Internet, en cuyo caso debéis de ir a `/etc/network/interfaces.d` y crear un fichero llamado `wlan1`. Dentro de este fichero, tenéis que introducir el siguiente contenido:  

```
auto wlan1
iface wlan1 inet dhcp
        wpa-ssid YOUR_SSID
        wpa-psk YOUR_PASSWORD
```

y después ejecutar `ifup -a`    
  
Después de esto, editáis `/etc/ssh/sshd_config` para poner `PermitRootLogin true`, y así podremos acceder desde nuestro ordenador a través de SSH y SFTP, que hará el resto del trabajo mucho más cómodo.  

Lo último que necesitamos para tener una versión básica de Linux funcional, es añadir esto a nuestro `/etc/apt/sources.list`:
**``deb http://archive.debian.org/debian jessie main``**

Veredicto final: ¡Dadle un try!  No perdéis nada y tener Linux en la Wii es gracioso de pelotas, aunque sinceramente es de escasa utilidad.  

## Servidor de Minecraft  
  
Un servidor de Minecraft... Los requisitos mínimos piden 512MB de RAM, así que seguro que no funciona en la Wii... ¿O, acaso... podría ser...?  
Pues, depende de cual sea tu definición de "funcionar".  

Intenté optimizar todo al máximo; para ello, elegí una versión vieja de Minecraft, previa a la actualización de la generación del terreno, y usar [Paper](https://papermc.io/), un fork del servidor de Minecraft optimizado para tener el mayor rendimiento posible. La versión mínima que pude usar fue Paper 1.8.8.  
Modifiqué la configuración a las opciones mínimas que se me ocurrían; **distancia de renderizado de 4 chunks, nada de timeouts a los jugadores, ningún tipo de logging ni de información de debug, activar cualquier opcion de Paper y Spigot que pudiera aumentar el rendimiento...**   

¿El resultado? Una configuración Frankenstein, que probablemente rompa todas las mecánicas de Minecraft Vanilla y cualquier tipo de máquina de redstone. Sin embargo, creo que literalmente no puedo sacar más rendimiento del servidor. También pre-generé el mundo con el plugin de [WorldBorder](https://dev.bukkit.org/projects/worldborder) en mi ordenador, porque tengo MUCHO tiempo libre pero no tanto como para generar el mundo en la Wii.  
  
Finalmente, para instalar esto en la Wii:

1. Metemos la carpeta del servidor en la Wii a través de SFTP.
2. Añadimos esto a **sources.list** ``deb [check-valid-until=no] http://archive.debian.org/debian jessie-backports main`` para poder instalar Java8.
3. Instais Java8 con ``apt-get install -t jessie-backports openjdk-8-jdk-headless`` y rezais para que no os dé tantos problemas como a mi...
4. **¡Ya puedes ejecutar Paper!** (Recomiento usar nice para sacar todo el performance que podamos): ``nice -n -10 java -Xmx2G -jar paper-1.8.8-443.jar``

¿Los resultados de este experimento?

- **Una hora de tiempo de inicio del servidor** (Supongo que la mayor parte de esto es alocación de memoria para la JVM, puesto que estamos funcionando prácticamente 100% sobre SWAP)
- **Tres intentos para conectarme**
- **2-3 TPS con un sólo jugador (quieto y sin hacer nada). Por contexto, la velocidad normal es 20TPS**  ![TPS](https://jos.s-ul.eu/6tWPLaMm)

Si queréis ver esta monstruosidad:

[![Gameplay en el servidor de la Wii](https://img.youtube.com/vi/Fe5foaQHgvw/0.jpg)](https://www.youtube.com/watch?v=Fe5foaQHgvw)

Veredicto final: **NO HAGÁIS ESTO.** Es una gilipollez. Aunque... está divertido.

## Wiihole
  
Finalmente, se me ocurrió que quizás y sólo quizás, podría utilizar mi Wii como servidor DNS usando [Pihole](https://pi-hole.net/), y así poder bloquear anuncios en toda mi red sin tener que gastar dinero en una Raspberry Pi (sé que son baratas de cojones, pero cuando escribí este artículo era un estudiante que no llegaba a fin de mes, además de una puta rata). 

Hacer que Pihole funcione no es tan fácil como en la Raspberry, pero desde tampoco es muy complicado. 

1. A través del router, le ponemos una IP estática a la Wii.
2. Pre-instalamos las dependencias (por cosas de la versión arcaíca de Debian que usa la Wii, no se instalan con el script de Pihole) ``apt-get install curl dnsmasq dhcpcd5 git dnsutils libsqlite3-0 libidn2-0 libperl4-corelibs-perl lsof dns-root-data psmisc sqlite3 unzip idn2 netcat``
3. Descargamos **Pihole 3.3.1** de GitHub. Esta es la última versión que podemos utilizar. ``curl -https://codeload.github.com/pi-hole/pi-hole/tar.gz/refs/tags/v3.3.1 > Pihole.tar.gz``
4. Lo descomprimimos ``tar -xf Pihole.tar.gz``
5. Entramos al directorio en el cual lo hemos descomprimido y ejecutamos el siguiente comando  ``PIHOLE_SKIP_OS_CHECK=true bash pihole``
6. En el instalador, **quitamos la interfaz web y el logging**. Tenemos que rascar rendimiento de donde de pueda.
7. Ponemos la DNs de nuestro ordenador o nuestro router a la IP de la Wii.

Se aceptan apuestas sobre los resultados del experimento. Es coña, fue horrible, como el resto.    
![DNS Performance Graph](https://jos.s-ul.eu/rhs6zC0p)  
En resumen, a pesar de que es una forma gratis de bloquear anuncios a nivel de red, tiene 4-5x veces la latencia que tiene el DNS de vuestro operador o de Cloudflare. Y eso que ese está a cientos de kilometros de vosotros.

Final verdict: **NO HAGÁIS ESTO.** Es que encima, esta vez nisiquiera es divertido.

## Conclusiones y futuras mejoras  
  
Posiblemente se podría extraer más rendimiento usando ZSwap (SWAP comprimido), y ZRAM ("RAM comprimida"), puesto que al disponer de sólo 70MB de RAM estamos constantemente tirando de SWAP, y el acceso a I/O es muy costoso. Sin embargo, a día de hoy ni ZSwap ni ZRAM están incluidos den wii-linux-ngx.   

Para mejorar el rendimiento de Wiihole, probablemente se podría usar un adaptador de Ethernet con la Wii y compilando una versión más nueva de Pihole, que permita usar FTLDNS.  

A pesar de todo, la Wii es vieja, y cómo todas las consolas de Nintendo, no puede presumir de hardware. Por muchas optimizaciones que se me puedan ocurrir, no hay mucho de lo que sacar. Sin embargo, esta experiencia me ha hecho valorar mucho más **lo bien optimizados que estaban los juegos antaño.** Poder hacer que cosas como Mario Galaxy o el TLOZ: Twilight Princess funcionasen con sólo 70MB de RAM (recordemos, el resto de hardware tampoco brilla por su potencia), es básicamente magia, y me maravilla a día de hoy.  