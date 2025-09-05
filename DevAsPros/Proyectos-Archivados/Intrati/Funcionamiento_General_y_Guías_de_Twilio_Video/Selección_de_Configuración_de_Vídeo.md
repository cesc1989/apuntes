# Selección de Configuración de Vídeo
En [esta guía](https://www.twilio.com/docs/video/tutorials/developing-high-quality-video-applications) están las opciones que mejor se pueden ajustar para los valores del objeto `connectOptions`.

# Opciones Recomendadas en General
- Para **Room Type** el recomendado es *Group Rooms*.
- Para **Room Mode** el recomendado es *Collaboration*.


## Opciones Recomendadas para sala P2P

En Navegador de Escritorio ([docs](https://www.twilio.com/docs/video/tutorials/developing-high-quality-video-applications#desktop-browser-in-p2p-rooms)):

    Twilio.Video.connect('$TOKEN', {
       name: 'my-room-name',
       audio: true,
       maxAudioBitrate: 16000, //For music remove this line
       video: { height: 720, frameRate: 24, width: 1280 }
    });

En Navegador de Dispositivos Móviles ([docs](https://www.twilio.com/docs/video/tutorials/developing-high-quality-video-applications#mobile-browser-in-p2p-rooms)):

     Twilio.Video.connect('$TOKEN', {
       name: 'my-room-name',
       audio: true,
       maxAudioBitrate: 16000, //For music remove this line
       video: { height: 480, frameRate: 24, width: 640 }
    });


## Opciones Recomendadas para sala Group Room

La calidad de una sala tipo *Group Room* depende de como el ancho de banda sea manejado. Para optimizar la calidad, hay que asegurarse que los tracks de video son priorizados y que el ancho de banda es asignado según el caso de uso.

Se consigue lo anterior usando la [Track Priority API](https://www.twilio.com/docs/video/tutorials/using-track-priority-api) y la [Network Bandwidth Profile API](https://www.twilio.com/docs/video/tutorials/using-bandwidth-profile-api).

**Recomendaciones para Track Priority API**
Se usa para determinar la importancia de cada track. Asignan ancho de banda y deciden que tracks deberían ser apagados en caso de congestión en la red.

Prioridades en caso de Audio Tracks

- Los audio tracks siempre tendrán la más alta prioridad que los tracks de video.
- Cambiar la prioridad de un track de audio no tendrá ningún efecto en la aplicación

Prioridades en caso de Video Track

- En la mayoría de los casos, aplica esta regla: la prioridad del video track debe estar relacionada con el tamaño de renderizado. Los tracks con más prioridad serán renderizados en tamaños más grandes que aquellos con menor prioridad.
- Solo debería haber un video track con prioridad `high`.
- Tracks de video renderizados como thumbnails deben tener prioridad `low`.

**Recomendaciones para Network Bandwidth Profile API**
Los Perfiles de Ancho de Banda tienen tres modos: `collaboration, presentation, grid`. El siguiente diagrama sirve para determinar el mejor modo según el caso de uso:

![Decision diagram for Network Bandwidth Profile mode selection](https://twilio-cms-prod.s3.amazonaws.com/images/bw-profile-mode-select.original.png)



# Creando Aplicaciones Usando el Modo `grid`
> **NOTA: Para la primera versión de Intrati, este es el modo que nos compete.**

El modo grilla se usa por alguna de estas razones:


- La aplicación es 1-1
- La aplicación es para comunicación en grupo pero *la interfaz no enfatiza ningún track de vídeo* sobre los demás. Todos los video tracks se dibujan del mismo tamaño.
- No se puede usar Simulcast. No usar Simulcast en salas de 5 o más participantes disminuirá la calidad del vídeo, aún en modo grilla.
![Typical GUI layout used for grid mode. Videos are displayed in a matrix where all video tracks have equal relevance.](https://s3.amazonaws.com/com.twilio.prod.twilio-docs/images/ibwa_grid_layout.original.png)

## Opciones Recomendadas para Navegador Escritorio Sala Tipo Group Room para modo `grid`
    Twilio.Video.connect('$TOKEN', {
      name: 'my-room-name'
      audio: true,
      video: { height: 720, frameRate: 24, width: 1280 },
      bandwidthProfile: {
        video: {
          mode: 'grid',
          maxTracks: 10,
          renderDimensions: {
            high: {height:1080, width:1980},
            standard: {height:720, width:1280},
            low: {height:176, width:144}
          }
        }
      },
      maxAudioBitrate: 16000, //For music remove this line
      //For multiparty rooms (participants>=3) uncomment the line below
      //preferredVideoCodecs: [{ codec: 'VP8', simulcast: true }],
      networkQuality: {local:1, remote: 1}
    });
## Opciones Recomendadas para Navegador Móvil Sala Tipo Group Room para modo `grid`
    Twilio.Video.connect('$TOKEN', {
      name: 'my-room-name'
      audio: true,
      video: { height: 480, frameRate: 24, width: 640 },
      bandwidthProfile: {
        video: {
          mode: 'grid',
          maxSubscriptionBitrate: 2500000,
          maxTracks: 4
        }
      },
      maxAudioBitrate: 16000, //For music remove this line
      //For multiparty rooms (participants>=3) uncomment the line below
      //preferredVideoCodecs: [{ codec: 'VP8', simulcast: true }],
      networkQuality: {local:1, remote: 1}
    });
# Creando Aplicaciones en Modo `collaboration`

El modo colaboración se usa normalmente en:


- Interacciones multiparte (un gran número de participantes se comunican)
- La *interfaz enfatiza un video track* más que en los demás (interlocutor dominante)
- El resto de video tracks se muestran de tamaño thumbnails
- Mantener todos los tracks visibles es más importante que tener alta calidad en el track principal
![Applications using collaboration mode typically enhance the dominant speaker and represent the rest of participants in thumbnail size.](https://s3.amazonaws.com/com.twilio.prod.twilio-docs/images/ibwa_collaboration_layout.original.png)

## Opciones Recomendadas para Navegador Escritorio Sala Tipo Group Room para modo `collaboration`
    Twilio.Video.connect('$TOKEN', {
      name: 'my-room-name'
      audio: true,
      video: { height: 720, frameRate: 24, width: 1280 },
      bandwidthProfile: {
        video: {
          mode: 'collaboration',
          maxTracks: 10,
          dominantSpeakerPriority: 'standard',
          renderDimensions: {
            high: {height:1080, width:1980},
            standard: {height:720, width:1280},
            low: {height:176, width:144}
          }
        }
      },
      dominantSpeaker: true,
      maxAudioBitrate: 16000, //For music remove this line
      preferredVideoCodecs: [{ codec: 'VP8', simulcast: true }],
      networkQuality: {local:1, remote: 1}
    });
## Opciones Recomendadas para Navegador Móvil Sala Tipo Group Room para modo `collaboration`
    Twilio.Video.connect('$TOKEN', {
      name: 'my-room-name'
      audio: true,
      video: { height: 480, frameRate: 24, width: 640 },
      bandwidthProfile: {
        video: {
          mode: 'collaboration',
          maxSubscriptionBitrate: 2500000,
          maxTracks: 5,
          dominantSpeakerPriority: 'standard'
        }
      },
      dominantSpeaker: true,
      maxAudioBitrate: 16000, //For music remove this line
      preferredVideoCodecs: [{ codec: 'VP8', simulcast: true }],
      networkQuality: {local:1, remote: 1}
    });
# Creando Aplicaciones en Modo `presentation`

Generalmente, tienen estas características:


- Las interacciones son uno a muchos, o sea, broadcasting. Un participante presenta a una gran audiencia
- La interfaz enfatiza un solo video track. Compartir pantall
- El resto de los video tracks podrán no ser mostrados ya que no son relevantes
- La calidad del presentador es critica y más importante que mantener a los tracks de los asistentes encendidos.
![Applications using presentation mode typically have a screen-share track whose quality must be maximized by all means. They may additionally display the presenter’s webcam or other participants webcam but with lower priority.](https://s3.amazonaws.com/com.twilio.prod.twilio-docs/images/ibwa_presentation_layout.original.png)



## Opciones Recomendadas para Navegador Escritorio Sala Tipo Group Room para modo `presentation`
    Twilio.Video.connect('$TOKEN', {
      name: 'my-room-name'
      audio: true,
      video: { height: 720, frameRate: 24, width: 1280 },
      bandwidthProfile: {
        video: {
          mode: 'presentation',
          maxTracks: 10,
          dominantSpeakerPriority: 'standard',
          renderDimensions: {
            high: {height:1080, width:1980},
            standard: {height:720, width:1280},
            low: {height:176, width:144}
          }
        }
      },
      dominantSpeaker: true,
      maxAudioBitrate: 16000, //For music remove this line
      preferredVideoCodecs: [{ codec: 'VP8', simulcast: true }],
      networkQuality: {local:1, remote: 1}
    });
## Opciones Recomendadas para Navegador Móvil Sala Tipo Group Room para modo `presentation`
    Twilio.Video.connect('$TOKEN', {
      name: 'my-room-name'
      audio: true,
      video: { height: 480, frameRate: 24, width: 640 },
      bandwidthProfile: {
        video: {
          mode: 'presentation',
          maxSubscriptionBitrate: 2500000,
          maxTracks: 5,
          dominantSpeakerPriority: 'standard',
          renderDimensions: {
            high: {height:1080, width:1980},
            standard: {height:720, width:1280},
            low: {height:176, width:144}
          }
        }
      },
      dominantSpeaker: true,
      maxAudioBitrate: 16000, //For music remove this line
      preferredVideoCodecs: [{ codec: 'VP8', simulcast: true }],
      networkQuality: {local:1, remote: 1}
    });
    

