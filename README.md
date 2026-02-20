# Lista de Verificación para Aplicaciones Web Profesionales

Guía de referencia estructurada que cubre los requisitos funcionales, no funcionales y de seguridad para aplicaciones web de nivel productivo. Este documento está pensado como estándar base para equipos de ingeniería que construyen o auditan sistemas web.

---

## Tabla de Contenidos

1. [Arquitectura Base](#1-arquitectura-base)
2. [Seguridad](#2-seguridad)
3. [Gestión de Datos](#3-gestión-de-datos)
4. [Observabilidad](#4-observabilidad)
5. [Calidad y Pruebas](#5-calidad-y-pruebas)
6. [Rendimiento](#6-rendimiento)
7. [Escalabilidad](#7-escalabilidad)
8. [Transporte de Datos](#8-transporte-de-datos)
9. [Configuración y Secretos](#9-configuración-y-secretos)
10. [Gobierno y Mantenibilidad](#10-gobierno-y-mantenibilidad)

---

## 1. Arquitectura Base

### Frontend

- Estrategia de renderizado definida y justificada (SSR, CSR o híbrido según el caso de uso)
- Estrategia de gestión de estado global y local establecida
- Manejo centralizado de errores de UI con retroalimentación al usuario
- Validaciones del lado del cliente implementadas únicamente para UX, nunca como frontera de seguridad
- Gestión de sesiones mediante tokens o cookies `httpOnly`
- Estrategia de caché HTTP definida (headers de caché, service workers si aplica)
- Cumplimiento de accesibilidad mínimo WCAG 2.1 AA
- Protección contra XSS mediante codificación de salida y Content Security Policy
- Pipeline de optimización de build: code splitting, lazy loading, tree shaking, compresión de assets

### Backend

- Arquitectura modular aplicada (Clean Architecture o Arquitectura Hexagonal recomendadas)
- Separación clara por capas: controlador, servicio, repositorio
- DTOs definidos para todos los contratos de entrada y salida
- Validación fuerte de entradas en cada punto de entrada
- Manejo centralizado de excepciones con formato de respuesta de error consistente
- Logging estructurado configurado
- Configuración basada en entornos (sin valores hardcodeados)
- Estrategia de versionado de API definida y aplicada
- Documentación automática de la API generada (OpenAPI / Swagger)

---

## 2. Seguridad

Todos los puntos de esta sección son obligatorios para despliegues en producción.

### Autenticación

- JWT o sesiones seguras del lado del servidor implementadas
- Rotación de refresh tokens con access tokens de vida corta
- Protección contra fuerza bruta (rate limiting, bloqueo de cuenta, CAPTCHA si es necesario)
- Autenticación multifactor (MFA) requerida en sistemas sensibles

### Autorización

- RBAC (Control de Acceso Basado en Roles) o ABAC (Control de Acceso Basado en Atributos) implementado
- Principio de mínimo privilegio aplicado en cada capa

### Protección Web

- CORS configurado con orígenes explícitamente permitidos (wildcard `*` prohibido en producción)
- Protección CSRF habilitada cuando se usan sesiones basadas en cookies
- Sanitización de entradas aplicada contra XSS
- Rate limiting aplicado a todos los endpoints públicos
- SQL Injection prevenido mediante ORM o consultas parametrizadas (sin interpolación directa de strings)
- Headers de seguridad configurados:
  - `Content-Security-Policy`
  - `X-Frame-Options`
  - `X-Content-Type-Options`
  - `Strict-Transport-Security`

### Transporte

- HTTPS obligatorio en todos los entornos
- HSTS habilitado con `max-age` mínimo de 1 año

---

## 3. Gestión de Datos

- Toda validación realizada del lado del servidor, independientemente de las validaciones del cliente
- Datos cifrados en tránsito (TLS 1.2 mínimo, TLS 1.3 recomendado)
- Datos sensibles cifrados en reposo
- Contraseñas hasheadas con `bcrypt` o `argon2` con factores de costo apropiados
- Log de auditoría para mutaciones de datos sensibles (quién cambió qué y cuándo)
- Estrategia de soft delete definida donde se requiera recuperación de datos
- Migraciones de base de datos versionadas y reproducibles

---

## 4. Observabilidad

- Logs estructurados en formato JSON
- Niveles de log correctamente aplicados: `debug`, `info`, `warning`, `error`, `critical`
- Correlation IDs propagados a través de requests y servicios
- Métricas recolectadas (Prometheus o equivalente)
- Endpoints de health check expuestos: liveness y readiness
- Tracing distribuido implementado en sistemas complejos o multi-servicio (OpenTelemetry recomendado)
- Alertas configuradas sobre umbrales críticos

---

## 5. Calidad y Pruebas

- Pruebas unitarias cubriendo la lógica de negocio central
- Pruebas de integración cubriendo interacciones entre servicios y base de datos
- Pruebas end-to-end cubriendo los flujos críticos del usuario
- Linting y análisis estático aplicados en CI
- Pipeline de CI/CD configurado con ejecución automática de pruebas en cada pull request
- Proceso de revisión de código definido y aplicado
- Umbral mínimo de cobertura de código definido y monitoreado

---

## 6. Rendimiento

- Estrategia de caché implementada en las capas correspondientes (Redis, HTTP cache headers, CDN)
- Todos los endpoints de listado paginados (sin consultas sin límite)
- Índices de base de datos definidos para columnas frecuentemente consultadas y claves foráneas
- Connection pooling configurado para el acceso a la base de datos
- Compresión de respuestas habilitada (gzip o brotli)
- CDN configurado para assets estáticos de cara al público
- Lazy loading aplicado a rutas y componentes pesados en el frontend

---

## 7. Escalabilidad

- Backend stateless (sin estado de sesión local atado a una instancia específica)
- Responsabilidades separadas para permitir escalado independiente
- Tareas de larga duración o alto consumo de recursos delegadas a colas (RabbitMQ, SQS, BullMQ o equivalente)
- Escalado horizontal soportado y probado
- Servicios contenerizados con Docker
- Orquestación de contenedores definida (Kubernetes o equivalente para sistemas de alta disponibilidad)

---

## 8. Transporte de Datos

| Tipo de Dato | Transporte Recomendado |
|---|---|
| Datos no sensibles | JSON sobre HTTPS |
| Datos sensibles | HTTPS obligatorio, preferir cookies `httpOnly` + `Secure` |
| Tokens de autenticación | Cookies con `httpOnly`, `Secure`, `SameSite=Strict` — evitar `localStorage` |
| Acceso a archivos | URLs pre-firmadas con expiración |
| Webhooks | Verificación de firma HMAC en el receptor |

---

## 9. Configuración y Secretos

- Todos los valores de configuración obtenidos desde variables de entorno
- Secretos gestionados mediante un gestor dedicado (HashiCorp Vault, AWS Secrets Manager o equivalente)
- Ningún secreto comprometido en control de versiones bajo ninguna circunstancia
- Archivos `.env` excluidos del control de versiones mediante `.gitignore`
- Feature flags utilizados para desacoplar despliegues de releases
- Entornos separados mantenidos: desarrollo, staging, producción
- Paridad entre staging y producción lo más cercana posible

---

## 10. Gobierno y Mantenibilidad

- Convenciones de código documentadas y aplicadas mediante reglas de linting
- Documentación técnica mantenida actualizada junto con los cambios de código
- Architecture Decision Records (ADR) redactados para decisiones técnicas significativas
- Versionado semántico aplicado a todos los releases
- Compatibilidad hacia atrás mantenida entre versiones menores
- Política de deprecación definida para cambios de API
- Proceso de auditoría de dependencias establecido (escaneo automatizado de vulnerabilidades recomendado)
- `CHANGELOG.md` mantenido para cada release

---

## Matriz de Cumplimiento Resumida

| Area | Minimo para Produccion | Recomendado |
|---|---|---|
| Autenticacion | JWT o sesiones + HTTPS | + MFA + rotacion de tokens |
| Autorizacion | RBAC | ABAC para control granular |
| Validacion | Server-side en todas las entradas | + Capa de validacion por esquema (Zod, Joi) |
| Logging | Logs JSON estructurados | + Correlation IDs + agregacion centralizada |
| Pruebas | Unitarias + integracion | + E2E + umbral de cobertura |
| Secretos | Variables de entorno | + Gestor de secretos |
| Rendimiento | Paginacion + indices | + Redis + CDN |
| Observabilidad | Health checks + logs de error | + Metricas + tracing distribuido |

---

## Referencias

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP Application Security Verification Standard (ASVS)](https://owasp.org/www-project-application-security-verification-standard/)
- [The Twelve-Factor App](https://12factor.net/)
- [NIST Digital Identity Guidelines](https://pages.nist.gov/800-63-3/)
- [Web Content Accessibility Guidelines (WCAG) 2.1](https://www.w3.org/TR/WCAG21/)

---

> Este documento debe revisarse y actualizarse conforme el sistema evoluciona. Cada sección representa una base minima, no un techo.
