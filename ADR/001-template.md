# ADR-001: Refactorización del módulo de autenticación para eliminar vulnerabilidades críticas de seguridad y mejorar mantenibilidad

## Contexto

El sistema login-caos implementa funcionalidades básicas de autenticación y registro mediante endpoints HTTP que reciben credenciales como parámetros de consulta. Durante la auditoría del código se identificaron múltiples problemas críticos de seguridad y diseño, incluyendo el uso de consultas SQL concatenadas que permiten SQL Injection, exposición de credenciales en la URL, almacenamiento de contraseñas con hashing débil (MD5) y mezcla de responsabilidades dentro de los controladores.

Adicionalmente, se evidenció que la configuración de infraestructura contiene credenciales expuestas y que el manejo de errores no sigue buenas prácticas de Clean Code. Estos problemas representan un riesgo alto en producción, ya que podrían permitir accesos no autorizados, filtración de información sensible y comprometer la integridad de la base de datos.

Desde el punto de vista arquitectónico, la concentración de lógica de negocio, validación y acceso a datos en una misma clase dificulta el mantenimiento, la escalabilidad y la testabilidad del sistema. Por lo tanto, se requiere una decisión estructural que mejore la seguridad, reduzca el acoplamiento y alinee el sistema con principios SOLID y buenas prácticas de Clean Code.

---

## Decisión

Se decide realizar una refactorización del módulo de autenticación con las siguientes acciones concretas:

1. Reemplazar las consultas SQL concatenadas por consultas parametrizadas (PreparedStatement o JdbcTemplate), eliminando la posibilidad de SQL Injection.

2. Sustituir el uso de hashing MD5 por un algoritmo seguro como BCrypt, garantizando almacenamiento adecuado de contraseñas conforme a estándares actuales de seguridad.

3. Cambiar el contrato de autenticación para recibir credenciales mediante un DTO en el cuerpo de la solicitud (JSON) en lugar de parámetros en la URL, evitando la exposición de datos sensibles en logs e historial del navegador.

4. Aplicar arquitectura en capas (Controller → Service → Repository) para separar responsabilidades, cumplir el principio de responsabilidad única (SRP) y mejorar la testabilidad.

5. Implementar manejo global de excepciones con respuestas HTTP consistentes (400, 401, 500) y eliminar el uso de printStackTrace() en producción.

Estas decisiones permiten fortalecer la seguridad del sistema y mejorar su mantenibilidad sin necesidad de una reescritura total.

---

## Consecuencias

### Consecuencias positivas

- Eliminación del riesgo de SQL Injection en los endpoints de autenticación.
- Mejora significativa en la seguridad del almacenamiento de contraseñas.
- Reducción de exposición de credenciales en URLs y registros.
- Código más limpio, modular y fácil de mantener.
- Mayor facilidad para implementar pruebas unitarias y futuras mejoras.

### Consecuencias o riesgos

- Incremento en el tiempo de desarrollo debido al refactor.
- Posibles regresiones si no se realizan pruebas adecuadas.
- Necesidad de migrar contraseñas existentes si estaban almacenadas con MD5.
- Requiere ajustes en clientes que consuman el endpoint antiguo basado en query params.

---

## Alternativas consideradas

1. Corregir únicamente la vulnerabilidad de SQL Injection sin modificar la arquitectura.
   Esta alternativa fue descartada porque, aunque reduciría un riesgo crítico, no solucionaría la exposición de credenciales ni el problema estructural de acoplamiento excesivo en los controladores.

2. Reescribir completamente el módulo de autenticación desde cero.
   Esta opción fue descartada debido al tiempo limitado del ejercicio y al mayor riesgo de introducir nuevos errores durante una reimplementación total.

3. Mantener el sistema actual y documentar los riesgos.
   Se descartó por no ser aceptable en un entorno productivo donde la seguridad es prioritaria.