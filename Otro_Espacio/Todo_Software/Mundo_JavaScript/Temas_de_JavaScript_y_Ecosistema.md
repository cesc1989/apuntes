# Temas de JavaScript y Ecosistema

## ReactJS

- Las diferencias entre *CSS in JS, Inline Styles con Radium, CSS Modules y Styled Components* basándome en el capítulo del libro *React Design Patterns and Best Practices.*
- Cómo escribir condicionales en JSX http://devnacho.com/2016/02/15/different-ways-to-add-if-else-statements-in-JSX/

**Arrancar App React sin usar Create React App**
Lo necesario para andar un proyecto en React sin `create-react-app` y lo como esta herramienta ahorra tiempo. Revisa también marcadores en Raindrop al respecto de este tema.

Para poner a correr un proyecto en real con los juguetes mínimos, se necesita, aparte de los paquetes de react(`react` y `react-dom`), varios paquetes de `babel`, `webpack` y `webpack dev server`.

Dependencias:

- react
- react-dom

Dependencias de desarrollo:

- babel-cli
- babel-loader
- babel-preset-es2015
- babel-preset-react-app
- webpack
- webpack-cli
- webpack-dev-server

Comandos del `package.json`:
```json
"scripts": {
	"start": "webpack-dev-server --hot",
	"compile": "webpack"
}
```

y la configuración del `webpack.config.js`:
```js
const path = require('path');

module.exports = {
	mode: 'development',
	context: path.join(__dirname, 'src'),
	entry: [
		'./index.js',
	],
	output: {
		path: path.join(__dirname, 'www'),
		filename: 'bundle.js',
	},
	module: {
		rules: [
			{
				test: /\.js$/,
				exclude: /node_modules/,
				use: [
					'babel-loader',
				],
			},
		],
	},
	devServer: {
		contentBase: path.join(__dirname, 'www'),
		port: 9000
	}
};
```


## jQuery

- [jQuery Context - Diferencias con selector descendente](https://stackoverflow.com/questions/16422478/what-is-context-in-jquery-selector)
- [jQuery Event Delegation](https://learn.jquery.com/events/event-delegation/)


## JavaScript

- jQuery a Vanilla JS. Ver → [+JavaScript: Enlaces y Temas Importantes](https://paper.dropbox.com/doc/JavaScript-Enlaces-y-Temas-Importantes-94RMURr5MLPA2cFzZmDXs) 

**Caso de argumento de función dentro de** `**map()**`
Ocurre que una función al ser utilizada dentro de un *map* no necesita que se le indique el argumento que tenga ya que el map lo está mandando "automaticamente"

Documentación de map: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map

```js
renderOrder(key) {}

// más adelante

{ orderids.map(this.renderOrder) }
```

En el código anterior vemos cómo al llamar `renderOrder` dentro de `map` no se necesita pasar el argumento que la función requiere.

# Lecciones Sprints - Wikit

## Turbolinks y jQuery

Para el caso del `bootstrap-select` Turbolinks entró en acción a molestar. Como en este caso al `selectpicker` aplicaba a un elemento en un modal de bootstrap, no servía con darle la clase para que le aplicará los estilos.

A pesar de haber probado con esta [forma sugerida en Upcase](https://thoughtbot.com/upcase/videos/turbolinks)(vídeo viejo):
```js
$(document).on('ready page:load', function() {
	$('#category_visibility').selectpicker()
});
```

Al final, pude hacerlo funcionar con
```js
$(function () {
	$(document).on('show.bs.modal', function () {
		$('#category_visibility').selectpicker()
	})
})
```

siguiendo un poco las recomendaciones en [RSJZ](http://ricostacruz.com/rsjs).

## Sobre llenar listas de selección con `**options_for_select**`

Para las [opciones de una lista de selección hay un módulo completo](https://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html) con varios métodos para usar.

Para llenar la lista de selección de los usuarios disponibles para agregar a una categoría usé inicialmente un `<select></select>` normal, sin embargo, como necesitaba colocar un atributo *data* en cada opción, debía llenar la lista de una forma diferente.

Usando el helper *select_tag* y el helper de opciones `**options_for_select**``(container, selected = nil)` lo pude hacer.

Teniendo en cuenta que este helper recibe un arreglo de arreglos con los valores a conformar cada `<option></option>` donde el primer elemento del subarray es el texto, el segundo el valor y puede haber un tercer elemento que conformaría otros atributos HTML del *option*

```ruby
module CategoriesHelper
	def category_members_options(collection)
		collection.map do |member|
			[member.email, member.id, data: { photo: avatar_url(member, 25) }]
		end
	end
end

# Vista

<%= select_tag(:user_ids, options_for_select(category_members_options(@available_members)), { multiple: true, class: 'form-control' }) %>
```


## Select2, eventos y jQuery `detach()`

El paquete *select2* cuenta con eventos de DOM personalizados. Se pueden [ver en la documentación oficial](https://select2.org/programmatic-control/events).

Destacan: `select2:select` para cuando se hace una selección y `select2:open` cuando se abre la lista de selección.

Se pueden [ocultar elementos de una lista que implementa](https://stackoverflow.com/questions/36682901/how-to-remove-selected-option-from-the-option-list-in-select2-multiselect-and-sh) `select2` usando estilos y jQuery `detach()`

**jQuery** `**detach()**`
Quita un elemento del DOM y [mantiene su información relacionada a](https://api.jquery.com/detach/) [*jQuery*](https://api.jquery.com/detach/) para permitirle volver a ser reinsertado tal cual y como estaba.

