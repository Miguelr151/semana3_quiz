# Fase 1 — Ambiente levantado

Se levantó el ambiente con Docker:

docker compose up --build

Verificación:
curl http://localhost:8080/health

Resultado:
{"ok": true}

# Fase 2 — Tabla de hallazgos

| # | Descripción del problema | Archivo | Línea aprox. | Principio violado | Riesgo |
|---|--------------------------|--------|--------------|-------------------|--------|
| 1 | Credenciales enviadas por query params en /login (/register) | AuthController.java | ~ | Seguridad (mínima exposición) | Alto |
| 2 | Posible SQL Injection si se arma SQL con strings del usuario | AuthController.java / repository | ~ | Seguridad básica | Alto |
| 3 | Contraseñas inseguras (hash débil o manejo en texto plano) | db/init.sql / service | ~ | Seguridad básica | Alto |
| 4 | Respuesta del login retorna datos sensibles (ej: hash/password/email completo) | AuthController.java | ~ | Seguridad de datos | Alto |
| 5 | Validación débil de password (solo largo pequeño) | AuthController.java / service | ~ | Seguridad básica | Medio |
| 6 | Manejo de errores pobre (printStackTrace / mensajes genéricos / códigos incorrectos) | AuthController.java | ~ | Clean Code | Medio |
| + | Credenciales DB visibles en docker-compose | docker-compose.yml | ~ | Seguridad infraestructura | Alto |