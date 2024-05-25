# Apuntes version 1
Forzar encodeo en UTF-8 [http://help.leanpub.com/author-help/i-see-strange-characters-in-my-book-how-do-i-switch-on-utf-8-character-encoding](http://help.leanpub.com/author-help/i-see-strange-characters-in-my-book-how-do-i-switch-on-utf-8-character-encoding)

# Generar libro con pandoc
    $ pandoc ./title.txt $(find ./manuscript/ -iname "*\.md" | sort -n) -o book.epub

Tener en cuenta organizar los archivos como muestran acá Cómo hacerlo según la guía introductoria de Pandoc [https://pandoc.org/epub.html#a-real-book](https://pandoc.org/epub.html#a-real-book)


# ¿Dónde publicar?
- Scribd: exportando un PDF
    - Publicado https://es.scribd.com/book/437356792/Backend-Handbook-for-Ruby-on-Rails-Apps
    - Lo hice mediante Publish Drive
- Leanpub
- Publish Drive
    - Publica libros en tiendas: Play Store, Apple Books, Kobo, Scribd
- [Open Libra](https://openlibra.com/es/page/suggest)
    - Aquí son muy selectivos o no revisan nada.


