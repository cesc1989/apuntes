# Mejores Prácticas de Twilio Video
Ver artículo → https://www.twilio.com/docs/video/build-js-video-application-recommendations-and-best-practices.

# Soporte a Navegadores

Ver [tabla](https://www.twilio.com/docs/video/javascript#supported-browsers). Y usar la [bandera](https://media.twiliocdn.com/sdk/js/video/releases/2.4.0/docs/module-twilio-video.html) `[isSupported](https://media.twiliocdn.com/sdk/js/video/releases/2.4.0/docs/module-twilio-video.html)` para saber si el navegador que abre la aplicación tiene soporte.

![](https://paper-attachments.dropbox.com/s_2175ECA48423340D5E4C53A205BFEA58D71879F61136726BC3DF26DD04549BA1_1588879503550_image.png)


Uso de `isSupported`:

    const { isSupported } = require('twilio-video');
    if (isSupported) {
      // Set up your video app.
    } else {
      console.error('This browser is not supported by twilio-video.js.');
    }
# Selección de Configuración de Vídeo

Ver [+Selección de Configuración de Vídeo](https://paper.dropbox.com/doc/Seleccion-de-Configuracion-de-Video-2tJfz3zkDfngLzZV2AOjv) .

# Obteniendo Medios Locales (Audio y Vídeo)
## Dominio de la Aplicación

`twilio-video.js` necesita de `[getUserMedia](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia)` para obtener los medios locales. Para que estas APIs estén disponibles, asegúrate de que la aplicación se ejecute en localhost o en un dominio con HTTPS.


## Política de Autoplay

La política de autoplay no permite activarlo en elementos `<audio>` o `<video>` hasta que el usuario haya interactuado con tu aplicación (clicar un botón, por ejemplo). Especialmente si la nota de interacción de la aplicación no es muy alta.

Se recomienda usar twilio-video.js después de que el usuario haya interactuado con la aplicación. Chrome 66+, Firefox 66+ y Safari 11+ hacen cumplir esta política.


    const { createLocalTracks, connect } = require('twilio-video');
    
    // To work around the autoplay policy, use twilio-video.js only
    // after a button click.
    document.getElementById('join-room').onclick = async () => {
      const tracks = await createLocalTracks();
      const room = await connect('token', {
        name: 'my-cool-room',
        tracks
      });
    };


## Obteniendo la Cámara en Navegadores Móviles

En navegadores móviles, la cámara se reserva solo para un `LocalVideoTrack` a la vez. Si se intenta crear un segundo `LocalVideoTrack`, los frames de video no serán enviados al primero `LocalVideoTrack`.

Si vas a previsualizar la cámara, obtén medios usando `[createLocalTracks](https://media.twiliocdn.com/sdk/js/video/releases/2.3.0/docs/module-twilio-video.html#.createLocalTracks__anchor)`. Luego envíalos en la función `[connect](https://media.twiliocdn.com/sdk/js/video/releases/2.3.0/docs/module-twilio-video.html#.connect__anchor)`.

    const { createLocalTracks, connect } = require('twilio-video');
    const tracks = await createLocalTracks();
    
    // Display camera preview.
    const localVideoTrack = tracks.find(track => track.kind === 'video');
    divContainer.appendChild(localVideoTrack.attach());
    
    // Join the Room with the pre-acquired LocalTracks.
    const room = await connect('token', {
      name: 'my-cool-room',
      tracks
    });

Si vas a alternar entre cámara frontal y trasera, detén y despublica cualquier `LocalVideoTrack` existente, usa `[createLocalTracks](https://media.twiliocdn.com/sdk/js/video/releases/2.3.0/docs/module-twilio-video.html#.createLocalTracks__anchor)` para crear un nuevo `LocalVideoTrack` y publícalo en la sala.

    const { createLocalTracks, createLocalVideoTrack, connect } = require('twilio-video');
    
    const tracks = await createLocalTracks({
      audio: true,
      video: { facingMode: 'user' }
    });
    
    // Join the Room with the pre-acquired LocalTracks.
    const room = await connect('token', {
      name: 'my-cool-room',
      tracks
    });
    
    // Capture the back facing camera.
    const backFacingTrack = await createLocalVideoTrack({ facingMode: 'environment' });
    
    // Switch to the back facing camera.
    const frontFacingTrack = tracks.find(track => track.kind === 'video');
    frontFacingTrack.stop();
    room.localParticipant.unpublishTrack(frontFacingTrack);
    room.localParticipant.publishTrack(backFacingTrack);


## Probando el Micrófono y la Cámara

En navegadores móviles, `[getUserMedia](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia)` es exitoso aún cuando el micrófono o cámara están reservados por otra aplicación o pestaña. Esto puede resultar en participantes móviles no siendo vistos o escuchados por los demás participantes en la sala.

Para solucionar esto, se sugiere que se le pida a los usuarios que prueben su cámara y micrófono antes de unirse a una sala. Puedes usar `[createLocalAudioTrack](https://media.twiliocdn.com/sdk/js/video/releases/2.3.0/docs/module-twilio-video.html#.createLocalAudioTrack__anchor)` para adquirir el micrófono y luego usar [Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API) para calcular su nivel. Si el nivel es 0 aún cuando el usuario está hablando, probablemente el micrófono está reservado para otra aplicación o pestaña.

Entonces podrás recomendar al usuario cerrar las demás aplicaciones y reiniciar tu aplicación, o en el peor de los casos reiniciar el navegador.

    // testmic.js
    const { createLocalAudioTrack } = require('twilio-video');
    const pollAudioLevel = require('./pollaudiolevel');
    
    const audioTrack = await createLocalAudioTrack();
    
    // Display the audio level.
    pollAudioLevel(audioTrack, level => {
      /* Update audio level indicator. */
    });

Puedes usar `[createLocalVideoTrack](https://media.twiliocdn.com/sdk/js/video/releases/2.3.0/docs/module-twilio-video.html#.createLocalVideoTrack__anchor)` para adquirir la cámara y adjuntar su correspondiente elemento `<video>` al DOM. Si no hay *frames* de vídeo, la cámara seguramente estará ocupada por otra aplicación o pestaña.

Recomienda al usuario cerrar las demás aplicaciones y reiniciar tu aplicación, o en el peor de los casos reiniciar el navegador.

    //testcamera.js
    const { createLocalVideoTrack } = require('twilio-video');
    
    const videoTrack = await createLocalVideoTrack();
    
    // Display the video preview.
    const divContainer = document.getElementById('local-video');
    const videoElement = videoTrack.attach();
    divContainer.appendChild(videoElement);
# Aplicación Corriendo en el Segundo Plano en Navegador Móvil

Cuando una aplicación está corriendo en un navegador móvil y es enviada al segundo plano, no tendrá acceso a la retroalimentación de vídeo de la cámara hasta que esté en primer plano.

Teniendo en cuenta esto, recomendamos que detengas y despubliques el *LocalVideoTrack* de la cámara, y que una vez la aplicación retome el primer plano, se publique un nuevo *LocalVideoTrack*.

Para los demás participantes, puedes escuchar los eventos [unsubscribed](http://media.twiliocdn.com/sdk/js/video/releases/2.3.0/docs/RemoteVideoTrackPublication.html#event:unsubscribed) y [subscribed](http://media.twiliocdn.com/sdk/js/video/releases/2.3.0/docs/RemoteVideoTrackPublication.html#event:subscribed) en el *RemoteVideoTrackPublication* correspondiente.

Usando [Page Visibility API](https://developer.mozilla.org/en-US/docs/Web/API/Page_Visibility_API), puedes detectar cuando se va al segundo o primer plano.

Código para el usuario que esté en el navegador:

    // mobileuser.js
    const { connect, createLocalTracks, createLocalVideoTrack } = require('twilio-video');
    
    const tracks = await createLocalTracks();
    
    let videoTrack = tracks.find(track => track.kind === 'video');
    
    const room = await connect('token1', {
      name: 'my-cool-room',
      tracks
    });
    
    if (/* isMobile */) {
      document.addEventListener('visibilitychange', async () => {
        if (document.visibilityState === 'hidden') {
          // The app has been backgrounded. So, stop and unpublish your LocalVideoTrack.
          videoTrack.stop();
          room.localParticipant.unpublishTrack(videoTrack);
        } else {
          // The app has been foregrounded, So, create and publish a new LocalVideoTrack.
          videoTrack = await createLocalVideoTrack();
          await room.localParticipant.publishTrack(videoTrack);
        }
      });
    }

Código para los demás participantes de la sala:

    // remoteuser.js
    const { connect } = require('twilio-video');
    
    function setupRemoteVideoNotifications(publication) {
      if (publication.isSubscribed) {
        // Indicate to the user that the mobile user has added video.
      }
    
      publication.on('subscribed', track => {
        // Indicate to the user that the mobile user has added video.
      });
    
      publication.on('unsubscribed', track => {
        // Indicate to the user that the mobile user has removed video.
      });
    }
    
    function setupRemoteVideoNotificationsForParticipant(participant) {
      // Set up remote video notifications for the VideoTracks that are
      // already published.
      participant.videoTracks.forEach(setupRemoteVideoNotifications);
    
      // Set up remote video notifications for the VideoTracks that will be
      // published later.
      participant.on('trackPublished', setupRemoteVideoNotifications);
    }
    
    const room = await connect('token2', { name: 'my-cool-room' });
    
    // Set up remote video notifications for the VideoTracks of RemoteParticipants
    // already in the Room.
    room.participants.forEach(setupRemoteVideoNotificationsForParticipant);
    
    // Set up remote video notifications for the VideoTracks of RemoteParticipants
    // that will join the Room later.
    room.on('participantConnected', setupRemoteVideoNotificationsForParticipant);
# Manejando Descargue de la Página

Cuando el usuario cierra la pestaña o navegador o navega a otra página, recomendamos que lo desconectes de la sala para que los demás participantes sean notificados.


    const { createLocalTracks, connect } = require('twilio-video');
    
    const tracks = await createLocalTracks();
    
    const room = await connect('token', {
      name: 'my-cool-room',
      tracks
    });
    
    // Listen to the "beforeunload" event on window to leave the Room
    // when the tab/browser is being closed.
    window.addEventListener('beforeunload', () => room.disconnect());
    
    // iOS Safari does not emit the "beforeunload" event on window.
    // Use "pagehide" instead.
    window.addEventListener('pagehide', () => room.disconnect());
    


