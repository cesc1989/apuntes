# 07 Mejorando en la Estimación de Software
Estos son apuntes basados en la lectura del libro de Steve McConnel *Software Estimation: Demystifying the Black Art*.


- Estimar con valores únicos es propenso a errores, ejemplo: integrar Stripe toma 8 horas.
- Cuando estimamos con valores únicos estamos siendo optimistas y en términos del [cono de la incertidumbre](https://es.wikipedia.org/wiki/Cono_de_incertidumbre) al ser optimistas lo que creemos que es un 90% de certeza es apenas un escaso 30%.
- Un paso para mejorar la estimación es usar rangos: peor de los casos(*worst*) y mejor de los casos(*best*).
- Sin embargo, aún se queda corto el rango en esas dos opciones y hay que agregar otra opción: peor de los casos(*worst*), caso más probable(*most likely*), mejor de los casos(*best*).
- Al tener el rango con los puntos anteriores se podría estimar un estimado esperado más realista con la formula PERT:
    - expected_case = [best case + (4 * most likey case) + worst case] / 6
- La formula anterior se puede mejorar para quitar la subjetividad de la estimación del mejor y peor de los casos:
    - expected_case = [best case + (3 * most likey case) + (2 * worst case)] / 6
- Importante contar con una lista de chequeo que indique los puntos a tener en cuenta cada que se estima.
    - Ir actualizando la lista de chequeo en base a experiencias anteriores

