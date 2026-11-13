# Apuntes Apex - Ciclo 013 - Page Speed

## Lecciones de mejores para tener en cuenta siempre

- Bajar las versiones fijas de las librerias css/fuentes a una carpeta local
    - Ejemplo: boostrap 3.4.8 en modo min en carpeta local
- Cargar scripts con defer

```html
<script defer src="https://cdn.counter.dev/script.js" data-id="43bc7c2a-99e1-4d8d-80a7-c16657f1907e" data-utcoffset="-5"></script>
```

- carga css externo con media print y luego pasa a all en el onload

```html
media="print" onload="this.media='all'"
```

- usa aria-label en enlaces sin texto

```html
aria-label="Dev As Pros en Instagram"
```