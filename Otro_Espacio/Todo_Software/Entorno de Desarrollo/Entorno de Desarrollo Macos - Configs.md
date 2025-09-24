# Entorno de Desarrollo Macos - Configuraciones

## Option como Meta key

Desde siempre era capaz de escribir en la terminal el caracter `~` usando la combinación `Opt` + `+/*` del teclado en Español. Sin embargo, de un momento a otro no me era posible. Al presionar la combinación si estaba en Zellij actuaba como un zoom. Si no ponía espacios en blanco.

Busqué en internet pero no logré nada conciso así que le pregunté a ChatGpt y me dijo que probablemente fuera una configuración del perfil de la aplicación Terminal.

Tuve que deschulear este setting:

**Terminal.app**: Preferencias → Perfil → Teclado → "Usar la tecla Opción como Meta" → Desactívalo.

![[macos.terminal.option.png]]

Al deschulearla, puedo hacer la combinación sin problemas.