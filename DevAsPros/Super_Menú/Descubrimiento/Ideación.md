# Ideación Super Menú - Yummy

![[supermenu.yummy.png]]

Un menú digital para restaurantes.

La idea es hacer un MVP limitado pero que se pueda probar en negocios reales. Para ellos vamos a enfocarlo en dos cosas puntuales:

- Menú Digital
- Backoffice

## Menú Digital

La cara al usuario. Se accede por una URL y se configura todo desde el Back-office. La finalidad es mostrar información sobre todo el menú de un restaurante en una interfaz limpia, bonita y que cargue rápido.

> [!Important]
> Esto sería algo diferencial que deberíamos procurar.
> Las interfaces de Ola Click y Cluvi (las que he visto) son bastante toscas, algo malucas y se podría decir que lentas.
> Hay que procurar que Yummy provea una UI linda.

Al ser la parte de cara al usuario será hecha en React a través de Inertia.

### ¿Por qué React?

Por varias razones:
1. Es la herramienta frontend más potente para hacer sistemas de componentes reusables.
2. El ecosistema de plugins/componentes es mucho más amplio para React.
3. Stimulus se podría quedar corto para este tipo de interfaces.
	1. La sintaxis de Stimulus no es tan sencilla comparada a la de React.

React será solo la cara visible del menú. No necesitamos montar NextJS ni un repositorio aparte. Mediante Inertia podremos tener React conectado directamente con Rails.

## Back Office

El back office es la zona donde los administradores podrán controlar todo con respecto a su menú:

- Gestionar el menú: nombre de plato, descripción, fotos y precio.
- Gestionar categorías del menú.
- Gestionar usuarios para ayudar al administrador.
- Gestionar sedes del restaurante.
- Acceder a estadística del menú.

Este será hecho totalmente en Rails. La UI aprovechará Turbo y Stimulus para la interactividad más avanzada. Aquí no es necesario React y no se usará para esta parte.

## Tecnología

### Backend

- [Puntapie](https://github.com/devaspros/puntapie/) como plantilla para crear una aplicación Ruby on Rails
- Hotwire + Boostrap para la UI
- SQLite como base de datos
- Linode como VPS
- Sentry para captura de excepciones
- AWS S3 o Mega S4 para almacenaje de imágenes
- [Inertia Rails](https://inertia-rails.dev/) para renderizar componentes React en el Menú Digital

## Frontend

- React como framework para la UI del Menú Digital
- [Tailwind](https://tailwindcss.com/docs/installation/framework-guides/ruby-on-rails) para los estilos
- Vite para el compilado de React y Tailwind