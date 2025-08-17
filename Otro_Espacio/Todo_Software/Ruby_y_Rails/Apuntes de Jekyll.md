# Apuntes de Jekyll

# Cacheando página Jekyll

Resulta que quería poner cache en la página https://www.franciscoquintero.com/herramientas. Lo quise hacer porque en el celular me di cuenta que cargó todo y demoró aún siendo una página que visito con frecuencia.

Me puse a averiguar y encontré que Jekyll ofrece una [API de caché](https://jekyllrb.com/tutorials/cache-api/) pero es para temas internos de compilación. No de nivel del navegador o CDN.

## Hospedaje en GitHub Pages

Tengo el sitio web montado en GitHub pages. Una forma gratuita y sencilla de hospedar sitios hechos en Jekyll.

Buscando más info encontré una [discusión](https://github.com/orgs/community/discussions/11884) donde comentan que github pages configura una cabecera `max-age 600` la cual solo da caché por 10 minutos. Lo cuál no está mal pero no es posible configurar ese valor.

## Conclusión: no cachear o irse a otro lado

La conclusión es que no se puede cachear en GitHub pages. Si se quiere tocar configurar Cloudflare como CDN o llevarse la página para Vercel o Netlify.