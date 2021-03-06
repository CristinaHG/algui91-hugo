---
author: alex
categories:
- android
- dev
mainclass: android
date: '2016-01-01'
lastmod: 2017-10-08T18:59:29+01:00
url: /programa-que-envia-mensajes-desde/
tags:
- curso android pdf
title: "Programa que envía mensajes desde Android a PC"
---

Hace poco tiempo empecé a seguir los [videotutoriales de Android][1], en parte porque estoy interesado en aprender a programar para android, y por otra parte porque necesito hacer un proyecto de fin de curso en el que me es imprescindible que se establezca una comunicación PC-Dispositivo Android.

Así que después de mucho buscar, encotré un programita en java que me iba a ser útil, concretamente este pequeño [chat][2] que encontré en CasiDiablo.

<!--more--><!--ad-->

A este código le hice algunas modificaciones en la parte del Cliente, y, finalmente conseguí que el dispositivo android enviara un mensaje y que el pc lo recibiera. Aún tengo que hacer algunas modificaciones porque tiene un pequeño problema, y es que, el cliente(El dispositivo android) se conecta y desconecta del server cada vez que envia un mensaje.

Como prueba dejo dos capturas de pantalla, una realizando la conexión desde el emulador, y otra probandolo con mi móvil:

<figure>
    <img sizes="(min-width: 1600px) 1600px, 100vw" on="tap:lightbox1" role="button" tabindex="0" layout="responsive"  height="640" width="1600" src="https://2.bp.blogspot.com/-NhzqkbbVSlI/TZSLKW_mJeI/AAAAAAAAAXs/fLJMMsGSYbI/s1600/Screenshot.png"></img>
</figure>

<figure>
    <img sizes="(min-width: 600px) 600px, 100vw" on="tap:lightbox1" role="button" tabindex="0" layout="responsive"  height="450" width="600" src="https://2.bp.blogspot.com/-IP60xZKxqEo/TZSMSDUnHcI/AAAAAAAAAX0/eXLpj7fD5PY/s320/31032011045.jpg"></img>
</figure>

 [1]: https://elbauldelprogramador.com/video-tutorial-programacion-android/
 [2]: http://casidiablo.net/java-socket-chat-basico/
