# Cómo Funciona la Creación de Sala en el Proyecto en Svelte
El documento [+Flujo de Aplicación Quickstart de Integración con Twilio](https://paper.dropbox.com/doc/Flujo-de-Aplicacion-Quickstart-de-Integracion-con-Twilio-PRH5c5W9mSQKOj5G2sDMe) explica cómo, en términos generales, se crea una sala usando la API de Vídeo de Twilio. Sin embargo, este documento pretende explicar cómo se hizo en Svelte.

## Access Token

Lo primero que hay que asegurar es tener el `accessToken` que se pide usando la función creada en Twilio. Para usar la función se necesita pasar un *identity* el cual se genera con la función `randomName()`.


    import { randomName } from "../utils/randomname";
    
    const identity = randomName();
    const url = "https://corn-macaw-8615.twil.io/intrati-token?identity=";
## Crear o Conectarse a Sala

Se necesita el `accessToken` resultado del paso anterior y el `roomName` para poder proseguir. Ya en este paso vamos a usar la librería Twilio que instalamos desde NPM.

    import Video from "twilio-video";
    
    function createOrConnectToRoom() {
      const connectOptions = {
        name: roomName
        // logLevel: "debug"
      };
      if (!roomName) {
        console.debug("No se puede crear sala sin nombre");
        return;
      }
      Video.connect(accessToken, connectOptions).then(roomJoined, error => {
        console.debug(`Unable to connect to Room: ${error.message}`);
      });
    }
# Función `roomJoined(room)`

En esta función pasan un montón de cosas:

Se asigna a `window.room` y a `activeRoom` el valor de `room`

    window.room = activeRoom = room;

Acá viene un poco de manipulación(de manera imperativa) del DOM:

    const previewContainer = document.getElementById("local-media");
    # Busca el elemento: <div id="local-media"></div>
    
    const microphoneToggler = document.getElementById("toggle-mic");
    # Busca el elemento: <button id="toggle-mic">Microfono</button>
    
    const cameraToggler = document.getElementById("toggle-cam");
    # Busca el elemento: <button id="toggle-cam">Camera</button>

A continuación se asignan elementos de vídeo HTML al contenedor de media local.  Se usa la función `attachTrack` y `getTracks` importada de la librería local `intrati-twilio`.

    import {
        attachTrack,
        attachTracks,
        getTracks,
        detachTrack,
        participantConnected,
        detachParticipantTracks
    } from "./intrati-twilio";
    
    if (!previewContainer.querySelector("video")) {
      attachTracks(getTracks(room.localParticipant), previewContainer);
      microphoneToggler.innerText = "Desactivar Microfon";
      cameraToggler.innerHTML = "Desactivar Cam";
    }

Luego, se configura el contenedor de vídeo HTML para todos los participantes que pueda haber

    // en el inicio ya se importaron las funciones en uso
    import { participantConnected } from "./intrati-twilio";
    
    // Attach the Tracks of the Room's Participants
    const remoteMediaContainer = document.getElementById("remote-media");
    // Busca el elemento: <div id="remote-media" />
    
    room.participants.forEach(function(participant) {
      console.log(`Already in Room: ${participant.identity}`);
      participantConnected(participant, remoteMediaContainer);
    });

Ahora, por cada nuevo participante que llega a la sala, se registra en esta

    // When a Participant joins the Room, log the event
    room.on("participantConnected", function(participant) {
      console.log(`Joining: ${participant.identity}`);
      participantConnected(participant, remoteMediaContainer);
    });

Y cuando un participante se va, remueve el elemento HTML vídeo del mismo

    // When a Participant leaves the Room, detach its Tracks
    room.on("participantDisconnected", function(participant) {
      console.log(`RemoteParticipant ${participant.identity} left the room`);
      detachParticipantTracks(participant);
    });

Finalmente, cuando el anfitrión de la sala se va, elimina cada elemento de vídeo HTML. En este paso **se acaba la sala de vídeo**.

    // Once the LocalParticipant leaves the room, detach the Tracks
    // of all Participants, including that of the LocalParticipant
    room.on("disconnected", function() {
      if (previewTracks) {
        previewTracks.forEach(function(track) {
          track.stop();
        });
        previewTracks = null;
      }
      detachParticipantTracks(room.localParticipant);
      room.participants.forEach(detachParticipantTracks);
      activeRoom = null;
    });

