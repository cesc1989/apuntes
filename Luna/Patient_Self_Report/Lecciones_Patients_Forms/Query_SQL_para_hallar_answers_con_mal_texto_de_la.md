# Query SQL para hallar answers con mal texto de la choice

Con esta query podíamos encontrar aquellas answers cuyo contenido no equivaldría a ningún texto de las opciones válidas de la pregunta.

```sql
SELECT a.id, a.content, a.question_id
FROM answers AS a
INNER JOIN questions AS q ON q.id = a.question_id
INNER JOIN form_types AS ft ON ft.id = q.form_type_id
WHERE ft.acronym = 'mdq'
AND q.content != 'Pain Scale'
AND a.content != ''
AND a.content NOT IN (
  SELECT DISTINCT(oc.label)
  FROM option_choices AS oc
  INNER JOIN questions AS que ON que.id = oc.question_id
  INNER JOIN form_types AS ftype ON ftype.id = que.form_type_id
  WHERE ftype.acronym = 'mdq'
);
```