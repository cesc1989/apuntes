# Capybara: Apuntes variados

## ¿cómo encontrar elementos ocultos?

Fuente: https://blog.tomoyukikashiro.me/post/how-to-find-invisible-element-using-capybara

```ruby
page.find("#delete-button", visible: false)
```

## Click en checkbox mediante etiqueta

Cuando el diseño hace que el frontend oculte el checkbox para que se vea bonito podemos clicarlo mediante su label con Capybara así:

```ruby
check('Spanish', allow_label_click: true)
```

Donde "Spanish" es la etiqueta (label) del checkbox.

También se puede con radio buttons:

```ruby
choose('I am fully vaccinated and boosted.', allow_label_click: true)
```

## Encontrar todos los elementos de un mismo patrón y hacer algo con ellos

Por ejemplo, una lista de campos de texto con el mismo placeholder pero diferentes _name_. Podemos hacer esto para encontrarlos y llenarlos:

```ruby
full_name_inputs = all('input[placeholder="Email"]')
full_name_inputs.each_with_index do |input, index|
  input.fill_in(with: "capybara_#{index + 1}@gmail.com")
end
```

## ¿Cómo mostrar el HTML de un elemento?

Resulta que react-select oculta la lista de selección apenas esta pierde el foco. Entonces para poder entender el HTML que generaba necesitaba imprimirlo cuando capybara lo encontraba. Eso lo logré así:

```ruby
find('.select__menu')['innerHTML']
```

Y este fue el HTML de la lista de selección:

```html
<div class="select__menu-list css-11unzgr">
  <div class="select__option select__option--is-focused css-1n7v3ny-option" id="react-select-2-option-0" tabindex="-1">American India/Alaska Native American</div>
  <div class="select__option css-yt9ioa-option" id="react-select-2-option-1" tabindex="-1">Asian/Pacific Islander</div>
  <div class="select__option css-yt9ioa-option" id="react-select-2-option-2" tabindex="-1">Black African American</div>
  <div class="select__option css-yt9ioa-option" id="react-select-2-option-3" tabindex="-1">Caucasian</div>
  <div class="select__option css-yt9ioa-option" id="react-select-2-option-4" tabindex="-1">Hispanic/Latino</div>
  <div class="select__option css-yt9ioa-option" id="react-select-2-option-5" tabindex="-1">Prefer not to disclose</div>
  <div class="select__option css-yt9ioa-option" id="react-select-2-option-6" tabindex="-1">Other</div>
</div>
```