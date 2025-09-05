# Lecciones Sprint 27 - AVZ
Sobre todos los campos en el modelo `Document` que pertenecen a un tipo exclusivo de documento:

| **Tipo de Documento**                                                               | **Campos**                                                                                                |
| ----------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| Tarjeta de Propiedad                                                                | - owner_name<br>- vehicle_plate<br>- vehicle_brand<br>- vehicle_line<br>- vehicle_model<br>- vehicle_city |
| Revisión quincenal y Revisión Anual                                                 | - revision_company_name                                                                                   |
| Seguro todo riesgo                                                                  | - sales_representative_name                                                                               |
| Revisión tecno mecánica, Contra, Extra, Investigación forense, Tarjeta de Operación | - car_company(relación a la tabla)                                                                        |
| Revisión Tecno Mecánica                                                             | - diagnostic_center_name                                                                                  |
| Tarjeta de Operación                                                                | - worker_name                                                                                             |
|                                                                                     |                                                                                                           |

## Configuraciones de atajos de teclado de Sublime Text 3

Bueno, resulta que como empecé a usar teclado nuevo ya no tenía las teclas INICIO y FIN para avanzar más rápido al comienzo(`bol`) o al final de la línea(`eol`).

En [esta respuesta en Stack Overflow](https://stackoverflow.com/a/14406139/1407371) encontré la forma de configurar el atajo de teclado para tal fin.

Además, [encontré esta página de la documentación de Sublime](http://docs.sublimetext.info/en/latest/reference/commands.html) sobre los comandos que existen. Actualmente reemplacé el comando `**slurp_find_string**` que se ejecuta con la combinación `ctrl+e` por el atajo de ir al final de la línea.

