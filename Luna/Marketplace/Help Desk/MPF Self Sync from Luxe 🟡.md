# MPF Self Sync from Luxe to Reduce help desk requests

Etiquetas: #luna_help_desk 

## Contexto

El sync al MPF falla en muchas ocasiones. Puede ser por la lambda, puede ser error de conexión entre servicios, puede ser que RQ acaba todos los reintentos, puede ser que falle la conexión a Google.

Hay muchos puntos donde puede fallar y cada vez toca arreglar manualmente.

## Propuesta de Solución: Botón Self Sync

Esto es una propuesta para permitirle a los miembros del equipo Credentialing disparar un sync manual desde Luxe.

Mediante un botón en el perfil del therapist se hace una petición a un endpoint en Marketplace que encola un worker por cada archivo que tenga el therapist en su carpeta en S3.

### Datos Clave

El ID del therapist que tienen en la tabla `tc_therapists` en Clinical es el mismo que está en la tabla `therapist_sign_up` en Marketplace.

Marketplace toma el ID de la ruta del archivo en S3. Lo extrae de la primera parte de la llave del archivo. Esto se ve en `app/marketplace/onboarding/models.py`:
```python
class TherapistSignUp(Base):
    __tablename__ = "therapist_sign_up"

    id = sa.Column(postgresql.UUID, primary_key=True)  # per therapist-sign-up service
```

Ese ID es el nombre de la carpeta en S3. Lo define así el modelo `Credentialing::Therapist` en Clinical:
```ruby
def s3_folder_name
	id
end
```

