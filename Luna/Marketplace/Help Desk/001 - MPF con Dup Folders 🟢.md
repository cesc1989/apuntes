# 001 - Therapist Folder en MPF Duplicado

Etiquetas: #luna_help_desk 

Caso EDG-2766

Para un therapist se crearon varias veces el mismo folder en el MPF. A pesar de que el código de Marketplace tiene varias defensas y usa cache para evitar esto, pasó.

Intenté replicar con varios scripts con Claude y no se pudo. La sugerencia al final fue aumentar el TTL del cache a 30 días.

## Duplicados

Se dieron en varios días. El therapist fue completando el form en el transcurso de varias semanas y así quedó reflejado en el MPF ya que cada carpeta se fue creando en fechas diferentes.

Así se ve las fechas de los archivos cargados en la carpeta en S3:
```
2025-10-04 22:35:38         34 Payout.txt
2025-10-04 22:39:52         10 PersonalInformation_2025-09-20.txt
2025-10-04 22:43:33    2155019 attestation_2025-09-20.pdf
2025-09-20 04:31:41    2493879 back_of_government_photo_id.jpg
2025-09-20 04:31:41     211081 basic_life_support_certificate.pdf
2025-10-04 22:43:32    2179812 d'martin_nnonyem_2025-09-20_packet.pdf
2025-10-04 22:43:34      22700 employment_history_2025-09-20.pdf
2025-10-04 22:43:34    2176403 friendly_packet_2025-09-20.pdf
2025-09-20 04:31:41    2493879 government_photo_id.jpg
2025-10-04 22:43:31    1853866 handwritten_signature.jpg
2025-10-04 22:43:13     703785 malpractice_liability_insurance_coverage.jpeg
2025-10-04 22:43:29      30552 medicare_requirement.pdf
2025-10-03 13:47:58     720620 photo.jpeg
2025-09-20 04:31:41      47165 resume.pdf
2025-10-04 22:23:48      34478 tuberculosis_screening_2025-09-20.pdf
2025-10-04 22:35:38    1266497 w9_format.jpeg
```

## Solución #1

Claudio sugirió aumentar el TTL del cache de 1 hora a 30 días.
```python
cache.set(
	is_folder_created_key,
	{
			"s3_clone": s3_clone_folder_id,
			"processed": processed_folder_id,
			"processing": processing_folder_id,
			"human": human_folder_id,
	},
	ex=30 * pendulum.SECONDS_PER_DAY,  # Cache for 30 days to handle multi-week onboarding (EDG-2766)
)
```

No estoy convencido pero creo que es un cambio que no debería traer mayores complicaciones.