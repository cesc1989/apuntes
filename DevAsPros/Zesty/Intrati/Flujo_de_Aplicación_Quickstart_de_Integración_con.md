# Flujo de Aplicación Quickstart de Integración con Twilio
Las funciones mencionadas están en el archivo `uickstart/src/index.js`


1. Cuando carga la página, se pide el access token de Twilio con `$getJSON`
    1. En dicho llamado, se hace *bind* a los botones de *joinroom* y *leaveroom*
    2. Hay que revisar el tema del **identity** cuando se genera el token
2. Función `roomJoined()`
    1. Setea una variable global que mantiene la sala(*room*) para hacer la desconexión
    2. Sino se está previsualizando la cámara/audio, se pide permisos al navegador y luego muestra la cámara en el navegador y activa el audio
3. Agregar los `remoteTracks` de los participantes que ingresen a la sala
    1. La función que se encarga es `participantConnected(participants, remoteMediaContainer)`
4. Cuando entra un nuevo participante en la sala, se agregan sus `remoteTracks` a la llamada
    1. La función que se encarga es `participantConnected()`
5. Cuando un participante deja la llamada, se desconectan sus `remoteTracks`
    1. La función encargada es `detachParticipantTracks(participant)`
6. Desconecta los *tracks* de todos cuando el anfitrion(*localParticipant*) deja la sala
    1. Método: `track.stop()`
    2. Función `detachParticipantTrack()`

