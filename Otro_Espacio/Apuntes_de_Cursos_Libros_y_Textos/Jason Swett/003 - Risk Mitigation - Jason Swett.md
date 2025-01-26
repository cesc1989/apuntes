# Risk Mitigation por Jason Swett

*Enviado el 4 de Noviembre de 2024.*

En este artículo Jason comenta sobre un sistema grande en el que tenía que intervenir y enviar muchos cambios. No quería cagarla en grande así que sus cambios en los PRs fueron reducidos a literalmente una o dos líneas de código.

De esta forma se mantuvieron los cambios realmente pequeños y cuando hubo problemas fue muy fácil detectar donde se introdujo.

Esto va en contraparte a la creencia que tenemos los desarrolladores de hacer PRs con muchos cambios a la vez. Lo hacemos así porque la forma contraria es muy dispendiosa:

- Una nueva rama
- Cambio pequeño
- Commit
- PR
- Review
- Mezclar
- Repite

En cambio con varios cambios a la vez solo se hace el proceso una vez y "ahorramos" tiempo. Sin embargo, Jason nos dice que esto es una falacia:

> But this kind of perceived efficiency is a false economy. When inevitably something goes wrong with the changes (...), it wipes out all the savings achieved by batching up changes to avoid overhead.

Este tema me recuerda al proyecto AppBlend. En el primer Milestone quise mezclar muchas cosas del upgrade de Rails:

- cambios de código
- gemas actualizadas
- configuraciones

Eso hacía el PR mucho más peligroso porque había muchas cosas en un software sumamente importante en Luna. Esa rama y PR inicial terminé descartando y reiniciando con un enfoque más acotado. Sacando cambios más pequeños y asegurándome cada vez que no afectaran el sistema en producción.

Los siguientes Milestones de AppBlend también los abordé de esa forma. Encontrando todos los cambios que tenían que hacerse y luego aplicarlos de manera individual con un PR para cada una.

Fue más cansino pero llegué al mismo camino con muchísima más confianza que si hubiera hecho todo de sopetazo.