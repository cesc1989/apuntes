# Modelo de Datos para Enlacito

Este sería el posible modelo para la tabla que guardará los enlaces:

```sql
CREATE TABLE urls (
  id SERIAL PRIMARY KEY,
  short_code VARCHAR(10) UNIQUE NOT NULL,
  long_url TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  clicks INTEGER DEFAULT 0
);
```

# Arquitectura

## Código corto para los enlaces

En una conversación con ChatGPT dijo que los códigos podrían generarse con:

- Base62 Encoding: Convierte un número en caracteres alfanuméricos (a-z, A-Z, 0-9).
- UUID + Corta longitud: Uso de identificadores únicos globales reduciendo su tamaño.

Le pregunté entre usar Base62 o SecureRandom y dijo que Base62 sería más eficiente. SecureRandom podría ser para URLs más seguras.

