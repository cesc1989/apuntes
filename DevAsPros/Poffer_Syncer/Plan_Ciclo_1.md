# Plan Ciclo 1 ✅ 

## ¿Qué sería lo primero a trabajar?

La gema para conectar a Pocket. Esta gema me permitirá establecer una conexión con Pocket para traer artículos de mi cuenta que estén marcados con una etiqueta en particular, extraer la información de los mismos en forma de un objeto (no un hash) y luego poder archivarlos cuando termine de trabajar con los mismos.

Cuando digo en forma de objeto es que quisiera que fuera algo como:

    articles = Pocket::Articles.get
    ap articles.first
    
    <#<Pocket::Article#12198798>
      title = "The article title",
      content = "The article content",
      tags = "the, article, tags",
      url = "the article url"
    >

en lugar de algo como:

    articles = Pocket::Articles.get
    ap articles.first
    
    {
      title: "The article title",
      content: "The article content",
      tags: "the, article, tags",
      url: "the article url
    }

Con la forma de objeto puedo acceder a los métodos de una forma más bonita, más orientada a objetos y no tanto navegando en hashes. Además que de esa forma el control si el hash está vacío o no lo hago en la clase que manipula a Pocket en vez de la clase donde uso la gema.


## ¿Qué debe hacer?
- Traer artículos de pocket ✅ 
- Luego traer solo artículos con una etiqueta ✅ 
- Extraer su información ✅ 
    - Retornar en un objeto en vez de un hash ✅ 
- Archivarlos en Pocket ✅ 

