# Crea atajos de teclado con Karabiner, Hammerspoon en MacOS

Todo nace de ver [este tuit](https://twitter.com/fxn/status/1798623385339167128) de Xavier Noira y la respuesta de Andreas. Así que me llamó la atención hacer esa configuración en mi equipo para abrir fácilmente aplicaciones como Brave o Sublime Text.

**Enlaces**:
- [Better shortcuts with Karabiner Elements and Hammerspoon](https://dev.to/ccedacero/better-shortcuts-with-karabiner-elements-and-hammerspoon-1plf)
- [Level Up Shortcuts And The Hyper Key - Part 1](https://mattorb.com/level-up-shortcuts-and-the-hyper-key/)
- [Set up a Hyper Key with Hammerspoon on macOS](https://kalis.me/setup-hyper-key-hammerspoon-macos/)
- [A Hyper Key with Karabiner Elements, full instructions](https://brettterpstra.com/2017/06/15/a-hyper-key-with-karabiner-elements-full-instructions/)
- [Mapping a Hyper key on MacOS with Karabiner Elements (the easy way)](https://benholmen.com/blog/hyper-key-with-karabiner-elements-raycast/)
	- Este es de 2024.
	- Con estas instrucciones pude configurar Hyper key en Karabiner.

**Notas**

Si Karabiner pide reiniciar Mac, hay que hacerlo.

# Karabiner Elements y Hammerspoon

Para descargar Karabiner se va a su página oficial -> https://karabiner-elements.pqrs.org/

Hammerspoon en su página correspondiente -> https://www.hammerspoon.org/

# Ejemplos de configuraciones

Este es el ejemplo de Xavier:
```lua
hs.hotkey.bind({}, "f2", function()
  hs.application.launchOrFocus("iTerm")
end)
```

Y este es el de Andreas:
```lua
hyper:bind({}, "V", function()
  smartLaunchOrFocus({"Visual Studio Code"})
end)
```

# Configuraciones

Al final me terminó sirviendo lo de [este artículo](https://kalis.me/setup-hyper-key-hammerspoon-macos/) para hacer el mapeo de la Hyper key en Hammerspoon luego de tener todo configurado en Karabiner.

En Karabiner tuve que instalar esta [configuración](https://ke-complex-modifications.pqrs.org/) de internet:

> Caps Lock → Hyper Key (⌃⌥⇧⌘) (Caps Lock if alone)
> 
> Se importa y se activan. Se puede probar que CAPS LOCK esté en modo Hyper abriendo Karabiner Event-Viewer y presionar la tecla y ver que salgan las cuatro teclas que componen la combinación Hyper.

Luego tuve que mapear CAPS LOCK a F18:
![[rename_caps_to_f18.png]]

Finalmente, actualicé la configuración de Hammerspoon:
```lua
-- ~/.hammerspoon/init.lua

-- A global variable for the Hyper Mode
hyper = hs.hotkey.modal.new({}, 'F17')

-- Enter Hyper Mode when F18 (Hyper/Capslock) is pressed
function enterHyperMode()
  hyper.triggered = false
  hyper:enter()
end

-- Leave Hyper Mode when F18 (Hyper/Capslock) is pressed,
-- send ESCAPE if no other keys are pressed.
function exitHyperMode()
  hyper:exit()
  if not hyper.triggered then
    hs.eventtap.keyStroke({}, 'ESCAPE')
  end
end

-- Bind the Hyper key
f18 = hs.hotkey.bind({}, 'F18', enterHyperMode, exitHyperMode)

hyper:bind({}, "B", function()
  hs.application.launchOrFocus("Brave Browser")
end)

hyper:bind({}, "F", function()
  hs.application.launchOrFocus("Firefox")
end)

hyper:bind({}, "O", function()
  hs.application.launchOrFocus("Obsidian")
end)

hyper:bind({}, "S", function()
  hs.application.launchOrFocus("Sublime Text")
end)

hyper:bind({}, "T", function()
  hs.application.launchOrFocus("Terminal")
end)

hyper:bind({}, "D", function()
  hs.application.launchOrFocus("DBeaver")
end)

hyper:bind({}, "P", function()
  hs.application.launchOrFocus("Postman")
end)

hyper:bind({}, "Z", function()
  hs.application.launchOrFocus("Zed")
end)

hyper:bind({}, "E", function()
  hs.application.launchOrFocus("Finder")
end)

hyper:bind({}, "H", function()
  hs.application.launchOrFocus("Dash")
end)

hyper:bind({}, "N", function()
  hs.application.launchOrFocus("Bruno")
end)
```

Y recargué la configuración de Hammerspoon.