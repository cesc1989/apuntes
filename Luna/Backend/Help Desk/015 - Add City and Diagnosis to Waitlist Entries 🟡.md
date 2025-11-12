# 015 - Add City and Diagnosis to Waitlist Entries

Etiquetas: #luna_help_desk 

Caso EDG-2811

El caso se sencillo. A la lista de waitlist entries agregarles las columnas City y Diagnosis. City se saca del location del patient. Y Diagnosis del appointment (si lo tiene) o del care plan.

# Solución

Para city fue sencillo:
```ruby
%td= entry.patient.city || "-"
```

Para Diagnosis la cosa cambia.

## Mostrar diagnosis

Primero intenté con esto:
```ruby
%td= entry.appointment&.episode&.diagnosis&.name || "-"
```

pero en muchas ocasiones mostraba el guión. Eso pasaba porque no tenía un appointment asociado.

Luego pasé a esto:
```ruby
%td= entry.patient.recent_care_plan&.diagnosis&.name || "-"
```

pero Alexis dijo que no es tan así por lo estados del waitlist entry:
> return the appointment's diagnosis or, if they really need the diagnosis that's going to be accepted, the recent care_plan's one. I would also return nil if the waitlist was rejected.
>
> looking at the possible statuses, I think it should be:
>
> - active -> recent care plan -> diagnosis
> - claimed -> appointment -> care plan -> diagnosis
> - else -> return nothing

Así que tengo que buscar una solución de ese estilo si Eli está de acuerdo.