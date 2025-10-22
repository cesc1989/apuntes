# Nuevo Config del MPF

Por los múltiples problemas que Marketplace encontraba con la configuración de varias Service Accounts llegando al límite de archivos, Ryan decidió cambiar la forma en que el MPF funciona. Aquí trataré de ir describiendo todo lo nuevo.

## Legacy

La env `GOOGLE_DRIVE_ROOT_ONBOARDING_FOLDER_ID` es legacy y ya no se usa en la nueva configuración.

Se usa activamente en `app/marketplace/onboarding/jobs.py`:
```python
# Legacy folder - kept for backward compatibility during migration
ROOT_CREDENTIAL_FOLDER = settings.GOOGLE.GOOGLE_DRIVE_ROOT_ONBOARDING_FOLDER_ID
```

Pero al buscar la constante `ROOT_CREDENTIAL_FOLDER` en el archivo no hay resultados. O sea que ya no está en uso.

## Arquitectura de 4 carpetas

### Explicación de Claudio sobre las 4 carpetas

1. **S3 Clone Drive (S3_CLONE_DRIVE_ID)**

Purpose: Exact mirror of what's in S3 (AWS storage)

- Stores original files as they were uploaded
- Acts as a backup/reference of the raw uploads
- Organized by therapist UUID: {uuid}/filename.pdf
- Think of it as: "The archive of originals"

2. **Processed Drive (PROCESSED_DRIVE_ID)**

Purpose: System-generated processed files

- Stores converted PDFs and processed documents
- Contains temporary processing/ folder for Google Doc conversions
- Final PDFs live here: {uuid}/final.pdf
- Think of it as: "The system's workspace for file processing"

3. **Human Drive (HUMAN_DRIVE_ID)**

Purpose: Human-friendly access for staff/admins

- Organized by human-readable names: First Last [{uuid}]/
- Contains shortcuts to files in other drives (not duplicates)
- This is what Luna staff see when browsing for therapist documents
- Think of it as: "The user-friendly interface"

4. **Legacy Drive (LEGACY_DRIVE_ID)**

Purpose: Old folders without therapist IDs

- Migration-only drive for folders that predate the UUID system
- Folders that can't be matched to a therapist ID go here
- You probably don't need this for local dev
- Think of it as: "The migration cleanup bucket"

### Crear nuevas carpetas y Compartir con email del Service Account

Hay que crear 4 carpetas a mismo nivel y todas deben ser compartidas con el mismo email de la Service Account.

El email es:
```
nalufilesmagic@nalu2023.iam.gserviceaccount.com
```

Las carpetas pueden llamarse como sea. Al final lo que nos interesa son los IDs. Claudio sugiere estos nombres:
```
MPF-S3-Clone-Test          → MPF_S3_CLONE_DRIVE_TEST_ID
MPF-Processed-Test         → MPF_PROCESSED_DRIVE_TEST_ID
MPF-Human-Test             → MPF_HUMAN_DRIVE_TEST_ID
MPF-Legacy-Test (optional) → MPF_LEGACY_DRIVE_TEST_ID
```

### Flujo

Here's what happens when a therapist uploads a resume:

1. File uploaded to S3
```
└─> luna-alpha-workloads-therapist-signup/4c7ff406.../resume.jpg
```

2. RQ Worker processes the upload
```
└─> Creates folder structure in all drives
```

3. S3 Clone Drive gets the original
```
└─> 4c7ff406-8267.../resume.jpg (original)
```

4. Processed Drive gets converted files
```
└─> 4c7ff406-8267.../
	 ├─> processing/resume-temp.gdoc (temporary)
	 └─> resume.pdf (converted)
```

5. Human Drive gets shortcuts
```
└─> Dwight Schrute Test [4c7ff406-8267...]/
	 ├─> ✨ MPF Backlink ✨ → Points to S3 Clone folder
	 └─> resume.pdf → Shortcut to Processed Drive PDF
```