# ADR 001 - Notificación del Estado de los Despliegues

Los despliegues de las aplicaciones Cashflow, Coshi Notes, Enlacito y Supermenu están configurados con Github Actions. Generalmente los despliegues van bien pero si fallan no tengo más forma de enterarme que yendo al repo y revisar la página de actions.

No es de mí interés abrir un espacio en Slack, Teams o similares. Ni tampoco configurar un espacio en el Discord personal para esto.

Busco una alternativa más directa. Que no necesite de un servicio que eventualmente tendría que pagar o al cual necesito registrarme.

Hay dos alternativas claras:

- Usar un servicio de notificaciones Push al móvil
- Crear una página web de estado de las aplicaciones montada en Github Pages

Me gusta mucho más la idea de una notificación push porque suena más sencillo de hacer que la segunda opción.

Exploraré esa en este documento.

## Notificación Push con NTFY

Puedo usar [NTFY](https://ntfy.sh/). Es un servicio para enviar notificaciones al móvil o escritorio mediante scripts.

La clave aquí es configurar el action para que al finalizar el despliegue mande un ping a NTFY, desde la aplicación móvil estaré suscrito a un canal y ahí recibiré la notificación.

Así sería la configuración según DeepSeek:
```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      # ... (tus pasos de build y deploy) ...

      - name: Notify success via NTFY
        if: success()
        run: |
          curl -H "Title: 🚀 Despliegue Exitoso - ${{ github.repository }}" \
               -H "Tags: rocket, white_check_mark" \
               -d "El commit en la rama '${{ github.ref }}' se desplegó correctamente. Ver workflow: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}" \
               https://ntfy.sh/${{ secrets.NTFY_TOPIC }}

      - name: Notify failure via NTFY
        if: failure()
        run: |
          curl -H "Title: ❌ Despliegue Fallido - ${{ github.repository }}" \
               -H "Tags: warning, skull" \
               -d "El despliegue falló para el commit en '${{ github.ref }}'. Ver logs: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}" \
               -H "Priority: high" \
               https://ntfy.sh/${{ secrets.NTFY_TOPIC }}
```

### Precio de NTFY

Cuesta 6 usd al mes si se paga mensual. 5 si es anual.

## Configuración de NTFY

1. Elige un nombre único para el tema.
2. Configura el nombre del tema como secreto en el repo llamado `NTFY_TOPIC`.
3. Configura el action con el código anterior.
4. En la app cliente suscríbete el tema.