# Editores de Texto Rico - Coshi Notes

# Editores de Texto
- Trix [https://trix-editor.org/](https://trix-editor.org/)
- Tiptap [https://tiptap.dev/docs/editor/introduction](https://tiptap.dev/docs/editor/introduction)
- Rhino editor [https://rhino-editor.vercel.app/tutorials/usage-with-rails/](https://rhino-editor.vercel.app/tutorials/usage-with-rails/)
- Richer Text https://www.richer-text.com/
- Prose Mirror [https://prosemirror.net/](https://prosemirror.net/)


# Trix y Action Text

Docs → https://guides.rubyonrails.org/action_text_overview.html

Notas:

- La instalación es sencilla.
- Action Text incluye Trix.
- Trix es bastante básico:
    - No soporta markdown
    - Hay que estilizar el editor y el contenido dependiendo del estilo de la página
- Trix se deja modificar pero siempre dentro de lo básico
    - Se pueden agregar otros botones para texto rico, por ejemplo, pero dar soporte a markdown es más complicado


# Tiptap

Docs → [https://tiptap.dev/docs/editor/introduction](https://tiptap.dev/docs/editor/introduction)

Notas:

- Es básicamente un backend para editores de texto enriquecido
- Es bastante configurable
- Escrito encima de Prose Mirror

Recursos

- Instalar y configurar Tiptap en Rails mediante Stimulus → https://maxencemalbois.medium.com/migrating-from-trix-to-tiptap-in-a-rails-7-app-with-turbo-and-stimulus-js-97f253d13d0
- 



# Rhino Editor

Docs → [https://rhino-editor.vercel.app/tutorials/usage-with-rails/](https://rhino-editor.vercel.app/tutorials/usage-with-rails/)


> The goal of Rhino Editor is to provide a drop-in replacement for Trix that allows you to continue using ActionText, but be able to provide a more rich text editing experience.
> 
> [Why Rhino Editor](https://rhino-editor.vercel.app/references/why-rhino-editor/)

Notas:

- **Aún no está listo para producción.**
- Creado sobre Tiptap
- Se integra con Action Text


# Richer Text

Docs → https://www.richer-text.com/

Notas:

- Aún no está listo para producción.
- Depende de React y react-dom
    - No es necesario usarlos pero deben estar instalados en el proyecto
# Prose Mirror

Docs → [https://prosemirror.net/](https://prosemirror.net/)

