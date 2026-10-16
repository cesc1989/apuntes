# Apuntes Apex Ciclo 010 - Super Menú

## Optimizar Logo

El logo lo hice en Canva con medidas de 2000x500 para que fuera rectangular. Pesaba 47kbs pero solo se mostraba en unos escasos 130px de ancho. Entonces era mucho peso para mostrar tan poco. Claudio dijo que se optimizara.

Lo hizo con este comando:
```bash
cwebp -q 90 -sharp_yuv -resize 480 120 app/assets/images/logo.supermenu.png -o app/assets/images/logo.supermenu.webp
```

- `-q 90`: calidad de compresión, de 0 a 100. Por defecto es 75. Lo subí a 90 porque en un logo con texto la calidad baja deja bordes borrosos o manchas alrededor de las letras.
- `-sharp_yuv`: WebP guarda el color con menos resolución que el brillo, y en los bordes entre colores muy distintos (el texto negro sobre el fondo naranja) eso puede verse borroso. Esta bandera hace una conversión más precisa que mantiene esos bordes definidos. Tarda un poco más, pero el archivo pesa casi lo mismo.
- `-resize 480 120`: redimensiona a 480 px de ancho y 120 px de alto antes de comprimir. Si pones 0 en uno de los dos valores, se calcula solo según la proporción (por ejemplo, -resize 480 0).
- `-o archivo.webp`: ruta del archivo de salida.

