---
title: volca sample2 polyphonic chromatic player - Un plugin Max4Live para usar un
  KORG volca sample2 como reproductor cromático de samples
date: 2022-05-22 01:13:00 +02:00
categories:
- MIDI
tags:
- MIDI
- Software
---

![Sin título.png](https://feranern.sirv.com/Images/Sin%20t%C3%ADtulo.png)

Este pequeño proyecto partió de mi objetivo de aprender programación MIDI y el lenguaje de programación Max, uniendo mis dos pasiones, el software y la música.

Estaba leyendo la implementación MIDI de mi nuevo volca sample2 para ver qué se puede hacer y, para mi sorpresa, pude transformar este dispositivo en un sintetizador basado en muestras con una transformación muy simple.

CC#49 se usa para la velocidad cromática (es decir, establecer la nota exacta de la muestra), así que pensé que podría traducir los mensajes MIDI Note On a valores CC#49 y un mensaje de nota para activar el sonido.

Abrí un monitor MIDI y verifiqué cómo se correlacionaban las notas y CC#49. Resulta que CC#49 Do central tiene el valor 64 mientras que la especificación MIDI usa un valor 60 como Do central. Matemáticas simples: suma 4 al valor de la nota

Bueno, eso fue fácil, solo me tomó unas pocas horas con poco conocimiento de Max. Pero pensé... Bien, este dispositivo tiene polifonía de 8 notas, si asigno una nota a un canal, ¡puedo tener polifonía de 8 voces! Pero aquí la complejidad se disparó, principalmente debido a las limitaciones de Ableton Live/Max4Live.

Permítanme explicar el flujo: Se presiona la tecla x en el teclado MIDI → Se envía el mensaje NoteON → Obtener el canal MIDI para usar, tratando de evitar los canales en uso → Transformar el valor de la nota en el mensaje CC#49 → Enviar CC#49 y NoteOn a Volca

Pero solo puedo enviar a un canal MIDI por instancia de complemento ... Entonces, ¿cómo determino dónde enviar? Al tener una instancia por pista y configurar manualmente a qué canal MIDI tiene que salir cada instancia.

¿Y cómo me comunico entre diferentes instancias? Es complicado, no puedes conectarte con instancias específicas, solo puedes transmitir mensajes. ¿Cómo sabes qué canal usar si no sabes cuál es el último que se usó?

Tuve que recurrir a un algoritmo simple pero imperfecto, "Cuenta de notas". Usando el componente "bórax", puede obtener la cantidad de notas activas, así que...

1 nota activa → Canal 1, ...8 notas activas → Canal 8

Problemas:

Con melodías, la siguiente nota colapsa con la anterior... La polifonía se destruye, funciona como un sintetizador monofónico
Lo mismo sucede con los acordes y es fácil perder notas mientras se toca.
Cosas buenas:

Funciona y sirve como prueba de concepto
Puede ser bastante musical y proporcionar limitaciones interesantes para tocar de manera diferente.
Como última característica, agregué un selector global para muestras. Usando la transmisión de mensajes (componentes de envío y recepción), envío la misma muestra seleccionada a los 8 canales. Creo que era absolutamente necesario que este complemento fuera utilizable. Cambiar muestras una por una para todas las pistas es tedioso.

¡Aquí también hay una lógica interesante! El volca sample2 tiene una capacidad máxima de 200 muestras, pero los mensajes de cambio de programa/CC solo van de 0 a 127 (7 bits), lo que significa que se utilizan 2 mensajes para crear el mensaje.

Primero, un MSB (bit más significativo, CC#3) que representa si la muestra está entre 0-99 o 100-199 y es 0 o 1

Luego, el LSB (bit menos significativo, CC#35) va de 0 a 99

Entonces, la matemática es la siguiente para obtener el número de muestra de los mensajes: MSB*100 + LSB

Y esta es la matemática para obtener cada parte:

MSB = Número de muestra / 100

LSB = Número de muestra % 100 (operación de módulo)

Fue una gran experiencia de aprendizaje, siéntete libre de modificarlo y jugar con el código. Dado que Max es un lenguaje de programación visual, también puedo mostrar en una captura de pantalla cómo funciona:

![volcasamplemax.png](https://feranern.sirv.com/Images/volcasamplemax.png)


Ampliando y mejorando este proyecto, creé una nueva versión basada en WebMIDI. Es MUCHO mejor, ya verás :P