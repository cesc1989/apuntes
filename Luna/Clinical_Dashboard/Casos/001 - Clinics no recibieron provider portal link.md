# 001 - Clinics no recibieron el link en la fecha esperada

Ver más cosas en la carpeta `caso-001`.

## Providers afectados

Ampliación de esto en [[clinical-dashboard-link-investigation-summary]]

Clinics

- Duke `b8608be4-22bd-41e3-9173-fefaf19bdc50`
- PAM Denver `eb285efb-5a49-4f45-87df-836c81c9e19d`
- PAM Houston `aba0b530-6f6c-4520-94dd-c4a4dd0afe71`
- PAM San Antonio `607ad309-9eef-4a65-9521-18de95986cf6`

> [!Note]
> Confirmar si está afectada por el no soporte en Contactable pero de alguna forma ha recibido enlaces semanales.

Memorial Healthcare System (South Florida) `91de8e85-e7ef-411c-a6aa-477d8159dc39`

Practice

- PAM `542a4199-aa41-4884-9d28-d29d32f23409`

## Datos

### Clinics

**Duke Health (DUK) Charlotte**:

- Links enabled: yes
- Verified recipients: yes, 3
- Delivery cadence: weekly
- Active care plans: 57
- Last sent: 2025-07-08 08:42 (2781.16 min ago) OK
	- Gap: ~2,781 minutes = ~46 hours = ~2 days

**PAM Health (PAM) Denver**:

- Links enabled: yes
- Verified recipients: yes, 7
- Delivery cadence: weekly
- Active care plans: 788
- Last sent: 2025-06-10 09:40 (43043.93 min ago) OK

**PAM Health (PAM) Houston**:

- Links enabled: yes
- Verified recipients: yes, 6
- Delivery cadence: weekly
- Active care plans: 301
- Last sent: 2025-06-10 09:40 (43043.5 min ago) OK

**PAM Health (PAM) San Antonio**:

- Links enabled: yes
- Verified recipients: yes, 6
- Delivery cadence: weekly
- Active care plans: 278
- Last sent: 2025-06-10 09:40 (43043.36 min ago) OK

De las tres de PAM:
- Gap: ~43,044 minutes = ~717 hours = ~30 days

### Practices

**PAM Health (PAM) San Antonio**:

- Links enabled: yes
- Verified recipients: yes, 4
- Delivery cadence: weekly
- Active care plans: 3
- Last sent: 2025-07-07 11:04 (4079.6 min ago) OK
	- Gap: ~4,080 minutes = ~68 hours = ~3 days


## Revisión de Logs y Sentry

### Sentry

Excepciones a revisar si aparecen en Sentry:

- `PortalEmailDuplicateAttempt`
- `PortalEmailNoneVerified`
- `PortalEmailUpstreamError`

Al buscar `PortalEmailNoneVerified` encuentro un registro con 2.7K eventos que ya lo tenía asignado. Eso no ayuda mucho porque necesito es saber si pasó para los 4 IDs afectados de Clinics.

Busqué la excepción `PortalEmailUpstreamError` y no salió nada en Sentry.

### Sentries por IDs

Para Duke `b8608be4-22bd-41e3-9173-fefaf19bdc50` encontré:
```
unknown provider for id - b8608be4-22bd-41e3-9173-fefaf19bdc50
```

Que ocurrió el Jul 21, 1:31 PM.

[Sentry](https://sentry.omega.getluna.com/organizations/sentry/issues/30146/events/46b6b797e5eb45a8837b7c0bee6d1028/?project=3).

### Logs

Logs para buscar en Grafana:

- `"Portal email send failed to #{name} at #{email}"`
- `"Failed to make Portal links for: #{name}"`

Busqué para todo el día de 21 de Julio 2025:
```sql
{app="backend-sidekiq-worker"} |= `Portal email send failed to`
```

Y devolvió cero resultados.

Busqué para todo el día de 7, 14, 21 de Julio 2025:
```sql
{app="backend-sidekiq-worker"} |= `Failed to make Portal links for`
```

Y devolvió cero resultados.

También busqué para:
```sql
{app="edge"} |= `Failed to make Portal links for`
```

En todo el mes de Julio y sin resultados.

## ¿Por qué pasa?

A través de los varios resultados de los prod-ops pude ver que si bien había links enviados estos no eran en la cadencia regular porque es muy probable que se enviaran mediante medios manuales.

¿Cuáles son estos medios manuales?

- El botón "Send Physician Portal Email" en el perfil del Clinic
- La generación del link al verificar un email
- Pedir un nuevo link cuando se vencía uno previo

Estas acciones hacen que se actualice el campo `link_last_sent_at` de la tabla `portal_configs`. Al pasar eso daba la ilusión de que se estaba enviando en la cadencia pero no era así. Veamos para MHS.

⚙️ PORTAL CONFIGURATION
✅ Has PortalConfig: YES
📅 Email cadence: weekly
🎯 Active case threshold: 3

📨 Last sent: 2025-07-24 04:25:18 -0700


📊 CLINICAL DASHBOARD
```
 ClinicalDashboard::Dashboard Load (2.8ms)  SELECT "clinical_dashboards".* FROM "clinical_dashboards" WHERE "clinical_dashboards"."provider_id" = $1 LIMIT $2  [["provider_id", "91de8e85-e7ef-411c-a6aa-477d8159dc39"], ["LIMIT", 1]]
```

✅ Has Dashboard: YES (`46a6f876-aca4-4a59-8a92-f1ae7c661b79`)

```
ClinicalDashboard::Link Load (5.2ms)  SELECT "clinical_dashboard_links".* FROM "clinical_dashboard_links" WHERE "clinical_dashboard_links"."dashboard_id" = $1 ORDER BY "clinical_dashboard_links"."created_at" DESC LIMIT $2  [["dashboard_id", "46a6f876-aca4-4a59-8a92-f1ae7c661b79"], ["LIMIT", 1]]
```

🔗 Most recent link: 2023-06-14 15:07:34 -0700

```
ClinicalDashboard::Link Count (1.7ms)  SELECT COUNT(*) FROM "clinical_dashboard_links" WHERE "clinical_dashboard_links"."dashboard_id" = $1  [["dashboard_id", "46a6f876-aca4-4a59-8a92-f1ae7c661b79"]]
```

📈 Total links: 2
⏰ Days since last link: 770.6
🚨 ISSUE: Last link is 770.6 days ago (should be weekly)

---

Queda en evidencia que el campo `link_last_sent_at` fue actualizado pero en realidad no se generó un nuevo `Link`.