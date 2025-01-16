# 🐞 Surgery Date: Fechas Antiguas
Las fechas en de los surgery dates han sido un problema siempre y también reciente. En este documento pondré todo el contexto necesario para intentar descubrir qué pasa y luego pensar una solución.

# Contexto
## Datos de fechas antiguas

En este [documento](https://docs.google.com/spreadsheets/d/1rOwp17vvjMXWTjQE2lFxaGT6yIXcdppH5WosgaEyjTk/edit#gid=2141584444) se puede, de una muestra de 500 formularios, como hay unas fechas anteriores al año 1900.

    0198-10-01 0:00:00
    1019-04-01 0:00:00
    1120-11-01 0:00:00
    1317-01-01 0:00:00
    1520-01-01 0:00:00

Incluso está esta:

    0001-01-01 00:00:00 BC


## Campo de fecha manda el día actual si se deja vacía

Como se puede apreciar en esta captura:

![](https://paper-attachments.dropboxusercontent.com/s_13BF39157CAFBE008BF421C15BAFBB22BA04F9A1E304855DA6B3D0013457B63E_1709329840770_101.surgery.date.png)


Solo se ingresó el nombre “mi cirugia” y en la peticiónel campo “date” se envió con el valor “2024-03-01”.

# Causas

La posible causa es la máscara tratando de ajustar la fecha para poder enviarla al backend.

