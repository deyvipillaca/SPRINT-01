# Enfoque arquitectónico

## Sistema de Alertas Tempranas ante Desastres Naturales y Lluvias en la Región de Ayacucho

| Elemento | Descripción aplicada al Sistema de Alertas Tempranas |
|---|---|
| **Patrón / enfoque arquitectónico** | Clean Architecture (Arquitectura Limpia), aplicada dentro de cada módulo del monolito modular. |
| **Objetivo** | Separar responsabilidades y controlar las dependencias hacia el dominio, de modo que las reglas de las alertas no dependan de la interfaz web ni de tecnologías externas. |
| **¿Qué problema resuelve?** | Evita el acoplamiento entre la interfaz web (panel administrativo y mapa de alertas), las reglas de negocio de las alertas (tipos de desastre, niveles de riesgo, estados, validación) y las tecnologías externas, como la base de datos, la fuente de datos meteorológicos (p. ej. SENAMHI), el servicio de notificaciones (SMS / push / correo) y el servicio de mapas. |
| **Capas definidas** | Presentación, Aplicación, Dominio e Infraestructura. |
| **Beneficios** | <ul><li>Facilita el mantenimiento y las pruebas unitarias de las reglas de alertas sin depender de servicios externos.</li><li>Permite cambiar implementaciones técnicas (proveedor de SMS, fuente meteorológica, motor de base de datos) sin modificar innecesariamente las reglas del negocio.</li><li>Mejora la organización y separación de responsabilidades del código.</li><li>Favorece la disponibilidad: si falla un servicio externo, el núcleo del sistema sigue funcionando.</li></ul> |

## Capas definidas

| Capa | Responsabilidad en el sistema | Componentes |
|---|---|---|
| **Presentación** | Recibe las peticiones del cliente web, autentica, valida la entrada y responde en JSON. Muestra alertas, mapa de riesgo y recomendaciones. | Rutas y controladores (`*.routes.js`, `*.controller.js`), panel administrativo, mapa de alertas. |
| **Aplicación** | Coordina los casos de uso: registrar alerta, validar, publicar, actualizar estado, enviar notificaciones, consultar lluvias e historial. | Servicios de aplicación (`*.service.js`), casos de uso, puertos (interfaces) hacia la infraestructura. |
| **Dominio** | Contiene las reglas de negocio y las entidades, sin depender de ninguna tecnología. | Alerta, Zona de riesgo, Evento natural, Usuario, Notificación, Registro meteorológico; reglas de nivel de riesgo y estados de la alerta. |
| **Infraestructura** | Implementa los detalles técnicos y la comunicación con sistemas externos. | Repositorios (Sequelize + PostgreSQL/PostGIS), cliente de la fuente meteorológica, proveedor de notificaciones, servicio de mapas/geolocalización. |

## Regla de dependencias

```
Presentación ──► Aplicación ──► Dominio ◄── Infraestructura
```

- Las dependencias apuntan siempre **hacia el dominio**.
- El dominio no conoce a Express, Sequelize, PostgreSQL ni a ningún servicio externo.
- La infraestructura implementa las interfaces (puertos) que define la capa de aplicación (por ejemplo, `NotificadorPort`, `MeteorologiaPort`, `AlertaRepositoryPort`).

## Relación con el estilo arquitectónico

El **estilo** (monolito modular en 3 capas físicas) y el **enfoque** (Clean Architecture en 4 capas lógicas) se complementan:

| Capa física del monolito (estilo) | Capas de Clean Architecture (enfoque) |
|---|---|
| 1. Presentación (`routes`, `controller`) | Presentación |
| 2. Lógica de negocio (`service`) | Aplicación + Dominio |
| 3. Datos (`repository`, Sequelize) | Infraestructura |

## Ejemplo: flujo "Publicar una alerta"

1. **Presentación:** `alertas.controller` recibe la petición del administrador y valida los datos de entrada.
2. **Aplicación:** el caso de uso `PublicarAlerta` verifica permisos y coordina el proceso.
3. **Dominio:** la entidad `Alerta` valida sus reglas (tipo, nivel de riesgo, zona, estado válido) y pasa a estado *publicada*.
4. **Infraestructura:** `alertas.repository` guarda la alerta en PostgreSQL y el proveedor de notificaciones envía los avisos por SMS, push o correo.
