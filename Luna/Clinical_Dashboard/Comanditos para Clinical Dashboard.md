# Comanditos para Clinical Dashboard

Etiquetas: #comanditos 

Para crear cosas.

# Nuevo AdminUser

Desde una consola de Rails:
```ruby
password = SecureRandom.alphanumeric(12)
User.create(
  first_name: "Brandon",
  last_name: "Hite",
  email: "bhite@getluna.com",
  password: password
)
```

# Nuevo ClinicalDashboard::User en Edge

```ruby
ClinicalDashboard::User.create(
  email: "admin1@admin.com",
  password: "123456789",
  first_name: "Hola",
  last_name: "Mundo"
)
```

# Enlaces

Enlaces de Dashboards para probar casos de enlaces vencidos. Cuando el link está vencido debe aparecer la UI que muestra un botón con el texto:

> Access links are for use by the recipient only and expire after 24 hours. Request a new access link to sign in to the Luna Dashboard
>
> Request a new access link

- Enlace en Local vencido - [Abrir](http://localhost:8080/dashboard/eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJwaHlzaWNpYW4iLCJleHAiOjE3NTE0OTY2MTUsImlhdCI6MTc1MTQ5NjExNS4yMzAyNTIsImlzcyI6InBoeXNpY2lhbi1wb3J0YWwtbGlua3MtZ2VuZXJhdG9yIiwibmJmIjoxNzUxNDk2MTE1LCJzdWIiOiI3ZTZmZTcyOC1hMmY4LTRlNzUtOTkzNS1jZjIzODQ5OTkzODUiLCJwcm92aWRlcl9uYW1lIjoiQWFyb24gU2FseWFwb25nc2UiLCJwcm92aWRlcl9raW5kIjoicGh5c2ljaWFuIiwicHJvdmlkZXJfaWQiOiI3ZTZmZTcyOC1hMmY4LTRlNzUtOTkzNS1jZjIzODQ5OTkzODUiLCJwb3J0YWxfcmVjaXBpZW50X2VtYWlsIjoiZnJhbmNpc2NvLnF1aW50ZXJvKzI1QGlkZWF3YXJlLmNvIiwicHJvdmlkZXJfY29kZSI6bnVsbCwiZGFzaGJvYXJkX3ZlcnNpb24iOiJ2MyIsImRhc2hib2FyZF9pZCI6ImQzNTFlMzI4LTg5ODYtNGMwZC05Y2UzLWM0MGNhMzAxMDE5YyJ9.qkYBgr06XyYPx9pkTNfcBXov6CkjEr_4_Ik0eZAhxnY)
- Enlace en Omega vencido - [Abrir](https://outcomes.getluna.com/dashboard/eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJjbGluaWMiLCJleHAiOjE3NTE3MjM3ODQsImlhdCI6MTc1MTYzNzM4NC44OTY5NzI3LCJpc3MiOiJwaHlzaWNpYW4tcG9ydGFsLWxpbmtzLWdlbmVyYXRvciIsIm5iZiI6MTc1MTYzNzM4NCwic3ViIjoiOTFkZThlODUtZTdlZi00MTFjLWE2YWEtNDc3ZDgxNTlkYzM5IiwicHJvdmlkZXJfbmFtZSI6Ik1lbW9yaWFsIEhlYWx0aGNhcmUgU3lzdGVtIChTb3V0aCBGbG9yaWRhKSIsInByb3ZpZGVyX2tpbmQiOiJjbGluaWMiLCJwcm92aWRlcl9pZCI6IjkxZGU4ZTg1LWU3ZWYtNDExYy1hNmFhLTQ3N2Q4MTU5ZGMzOSIsInBvcnRhbF9yZWNpcGllbnRfZW1haWwiOiJmcXVpbnRlcm9AZ2V0bHVuYS5jb20iLCJwcm92aWRlcl9jb2RlIjoiTUhTIiwiZGFzaGJvYXJkX3ZlcnNpb24iOiJ2MyIsImRhc2hib2FyZF9pZCI6ImRjYWE2N2FkLWMyNjQtNGQ5OS05ZjBiLTVmYzdkZTRhNmU5ZCJ9.g-wR7Mxs2od2ninpW20sDzTojSC6pUGOAtn3oGV-lPA)

# Otros Comanditos

El de Patient Self Report -> [[Comanditos para Patient Self Report]]

El de Therapist Signup -> [[Comanditos para Therapist Signup]]