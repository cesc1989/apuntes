# Apuntes Ciclo 05 - Coshi Notes - Imágenes en Tiptap
Este ha sido el ciclo más intenso debido a que esta característica no fue tan sencilla de montar.

Finalmente lo logré gracias a un Gist con código para Tiptap 2.0, además de leer comentarios en los gists y repasar un poco de JavaScript. Hice el endpoint para cargar archivos y configuré un bucket en S3 con una política para archivos.

Aquí espero documentar todos esos aprendizajes.


# Turbo Stream: en controlador y/o en modelo

Importante que el turbo_stream, si se usa en controlador y modelo, hará que se duplique el contenido renderizado.

Ver [+Apuntes Ciclo 01 - Coshi Notes](https://paper.dropbox.com/doc/Apuntes-Ciclo-01-Coshi-Notes-lS9twafbkRVI39xNaHwat) 


# Política para que los archivos del Bucket sean públicos

Se configura en la pestaña permisos en el detalle del bucket. Esta es la política:

    {
      "Version":"2012-10-17",
      "Statement":[
        {
          "Sid": "PublicRead",
          "Effect": "Allow",
          "Principal": "*",
          "Action": "s3:GetObject",
          "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
        }
      ]
    }

También hay que configurar el bucket según explican [aquí](https://stackoverflow.com/questions/68757380/make-every-object-in-an-amazon-s3-bucket-publicly-accessible/68803020#68803020):

- Go to the bucket's **Permissions** tab
- Go to the **Block public access** options
- **Turn off** the two options that mention **Bucket Policies**:
![S3 Block Public Access](https://i.stack.imgur.com/aJP1i.png)

# Mostrar Imagen en Tiptap: mostrar y cargar

Tiptap tiene una [extensión](https://tiptap.dev/docs/editor/api/nodes/image) para mostrar imágenes pero solo sirve para eso, mostrar. La carga de imágenes queda para la implementación.

> This extension does only the rendering of images. It doesn’t upload images to your server, that’s a whole different story.

Enlaces:

- Tutorial donde [implementan](https://www.codemzy.com/blog/tiptap-drag-drop-image) la carga usando la función `editorProps` cuando se configura Tiptap.
    - Este se veía más complejo así que no lo seguí de primero.
- Gist [inicial](https://gist.github.com/slava-vishnyakov/16076dff1a77ddaca93c4bccd4ec4521) que inspira al del cual sale la solución
    - Este gist serviría pero para Tiptap 1.0
- Comentario el cual [implementa la solución](https://gist.github.com/slava-vishnyakov/16076dff1a77ddaca93c4bccd4ec4521?permalink_comment_id=3744392#gistcomment-3744392) propuesta para Tiptap 2.0
- Gist que [implementa la solución para React](https://gist.github.com/waptik/f44b0d3c803fade75456817b1b1df6b4)
    - De este saqué la solución quitando lo de React y Typescript
    - Se basa en el código del comentario
    - Tiene demo en React donde se implementa

Del gist con código para React importante este [comentario](https://gist.github.com/waptik/f44b0d3c803fade75456817b1b1df6b4?permalink_comment_id=4259284#gistcomment-4259284) que corrige como configurar la sección `nodeInputRule`:

    nodeInputRule({
      find: IMAGE_INPUT_REGEX,
      type: this.type,
      getAttributes: (match) => {
        const [, alt, src, title] = match;
        return {
          src,
          alt,
          title,
        };
      },
    }),



# Cargar Código JavaScript además de controladores de Stimulus con Importmaps

Importmaps rails funciona bien para librerías en NPM y los controladores de Stimulus. Sin embargo, cuando de paquetes locales se trata la cosa es más enredada.

Al final, para poder importar paquetes personalizados tocó tocar tres partes.

Primero, crea una carpeta que tendrá los paquetes y configura en el archivo de Importmaps:

    # config/importmap.rb
    
    pin_all_from 'app/javascript/src', under: 'src', to: 'src'

Segundo, importa cada paquete en el archivo `application.js`:

    // app/javascript/application.js
    
    import "src/custom_image"
    import "src/drop_image"

NOTA: sin hacer esto habrá problemas al compilar e intentar cargar los archivos JS en producción.

Tercero, importa el paquete que interesa sin ruta relativa

    // app/javascript/controllers/rich_editor_controller.js
    
    import { CustomImage } from 'src/custom_image'


## Archivo JS con MIME text/html

El error:

    Se bloqueó la carga de un módulo de “https://coshinotes.devaspros.com/assets/src/custom_image” debido a un tipo MIME no permitido (“text/html”)

Si no se configura correctamente la carga del javascript personalizado al importar el modulo, si se usa import relativo, no se encontrará el archivo y la respuesta del servidor será 404.

Al pedir un archivo JS que no se encuentra y el servidor responde con 404, la respuesta es un HTML y por eso explota el error del mensaje.

Para corregir hay que fijarse en la forma en que se configuran los imports.

En esta [respuesta en Stack Overflow](https://stackoverflow.com/questions/70548841/how-to-add-custom-js-file-to-new-rails-7-project/72855705#72855705), un tipo escribe todo un ensayo sobre cómo configurar correctamente los imports en Importmaps y cómo no morir en el intento.

**Importante**
No usar imports relativos.

> **Do not use relative imports**, such as `import "./plugin/app"`, it may work in development, but it will break in production.
> 
> See the output of `bin/importmap json` to know what you can import and verify importmap.rb config.

No precompilar.

> **Do not precompile** in development, it will serve precompiled assets from `public/assets` which do not update when you make changes.

Si pinnear un archivo no lo carga, hay que importarlo en `application.js`  como mencioné antes.

> Pinning your files doesn't make them load. They have to be imported in `application.js`:
    // app/javascript/application.js
    
    import "plugin"



# JavaScript: imports y promesas
## Promesas

Cuando una función devuelve una promesa, el `return` debe ser explícito. Sino habrá error de función `undefined`.

Así tuve que escribir la función para hacer la petición de la carga de archivo arrastrado al editor:

    uploadFile(img) {
      const themeId = document.getElementById('theme_title').dataset.themeId
    
      let formData = new FormData()
      formData.append('uploads[file]', img)
      formData.append('uploads[theme_id]', themeId)
    
      return fetch('/v1/uploads', {
        method: 'POST',
        body: formData
      })
    }

La clave es el `return` explícito del fetch el cual es una promesa por resolver.

Con eso dado, pude usar la función en `dropImagePlugin`:

    upload(image).then(res => res.json()).then((response) => {
      const node = schema.nodes.image.create({
        src: response['result'],
      });
    
      const transaction = view.state.tr.replaceSelectionWith(node);
    
      view.dispatch(transaction);
    });

NOTA: recuerda que para poder usar el resultado de la promesa, toca continuar la ejecución dentro del bloque del `then()`.

Por eso en Avanzza tengo los diferentes bloques de fetch que en la resolución invocan una función pasándole el resultado de la promesa.

    Avanzza.getCarBrandLines = (carBrandId, $carLinesSelect) => {
      API.get(`car_lines?car_brand_id=${carBrandId}`)
      .then((response) => {
        return response.json();
      })
      .then((json) => {
        set_options_for($carLinesSelect, json.data);
        $carLinesSelect
        .find(`option[value=${$carLinesSelect.data('carLineId')}]`)
        .prop('selected', true)
      });
    }


## Importando y Exportando

Cuando un módulo se exporta con `export default` este se puede importar directamente.

Ejemplo veamos como se exporta `@tiptap/extension-link`:

    import { Link } from './link.js'
    
    export * from './link.js'
    
    export default Link

Y así lo importo:

    import Link from '@tiptap/extension-link'

Cuando no se usa default, hay que hacer “destructuring”?

Ejemplo, exportando `dropImagePlugin`:

    export const dropImagePlugin = (upload) => {}

Luego se importa asi:

    import { dropImagePlugin } from 'src/drop_image'

