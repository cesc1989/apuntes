# Copiando Archivos a/desde S3 mediante CLI

Para el proyecto Therapist Signup, los archivos PDF de los *agreements* están en un bucket en AWS S3. Para simplificar todo, accedo mediante CLI para revisar, descargar o subir los archivos.

Primero, podemos listar archivos así:
```bash
$ aws s3 ls # para ver los buckets
# 2021-03-17 10:48:42 luna-alpha-workloads-therapist-signup

$ aws s3 ls [NOMBRE_DEL_BUCKET] # para ver contenido del bucket
$ aws s3 ls luna-alpha-workloads-therapist-signup

# para ver contenido de carpeta
$ aws s3 ls luna-alpha-workloads-therapist-signup/quintero_kansas_fran_kansas_2021-05-14/
#                           PRE signup/
# 2021-05-14 16:48:27          0 
# 2021-05-14 17:07:38         54 Payout.txt
```

Para verificar, en staging, la cantidad de archivos para los Agreements
```bash
# Alpha
aws s3 ls luna-alpha-workloads-therapist-signup-agreement-pdfs --profile alpha | grep "_agreement.pdf" | wc -l

# Omega
aws s3 ls luna-omega-workloads-therapist-signup-agreement-pdfs --profile omega | grep "_agreement.pdf" | wc -l
```

Se suben así:
```bash
aws s3 cp ~/Documents/luna-dev-files/therapists-signup/agreements-pdf/maryland_agreement.pdf s3://luna-alpha-workloads-therapist-signup-agreement-pdfs/ --profile alpha
```

Cuando hay que modificar varios al tiempo, lo mejor es subirlos al tiempo también.

**En Alpha**
```
aws s3 cp ~/Documents/luna-dev-files/therapists-signup/agreements-pdf/ s3://luna-alpha-workloads-therapist-signup-agreement-pdfs/ --recursive
```

**En Omega**
```
aws s3 cp ~/Documents/luna-dev-files/therapists-signup/agreements-pdf/ s3://luna-omega-workloads-therapist-signup-agreement-pdfs/ --recursive
```

Documentación del [comando cp](https://docs.aws.amazon.com/cli/latest/reference/s3/cp.html).

Si necesito borrar archivos en grupo, sería así:
```
aws s3 rm s3://luna-alpha-workloads-therapist-signup-agreement-pdfs/ --exclude "*_agreement.pdf" --recursive
```

> Documentación del [comando rm](https://docs.aws.amazon.com/cli/latest/reference/s3/rm.html). Sobre los filtros [exclude e include](https://docs.aws.amazon.com/cli/latest/reference/s3/index.html#use-of-exclude-and-include-filters).


## Bonus Track: renombrar varios archivos mediante bash

Visto en [Stack Overflow](https://stackoverflow.com/a/24592379/1407371).

Quería cambiarles el nombre a varios archivo que solo eran [nombre].pdf a `nombre_agreement.pdf`.

Lo logré así:
```bash
for file in ~/Documents/luna-dev-files/therapists-signup/agreements-pdf/*.pdf; do
  # echo $file
  mv "$file" "${file%.pdf}_agreement.pdf"
done
```


