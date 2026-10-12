# Decisiones arquitectónicas

## Sistema de Alertas Tempranas ante Desastres Naturales y Lluvias en la Región de Ayacucho

### Resumen

| ID | Decisión | Drivers relacionados |
|---|---|---|
| DEC01 | Monolito modular en lugar de microservicios | DA01, DA05, DA08 |
| DEC02 | Arquitectura en tres capas dentro de cada módulo | DA07, DA08 |
| DEC03 | Node.js 20 LTS + Express como plataforma backend | DA02, DA08 |
| DEC04 | API REST con JSON sobre HTTPS | DA04, DA06 |
| DEC05 | PostgreSQL (con PostGIS) como única base de datos | DA02, DA07 |
| DEC06 | Sequelize como ORM y patrón Repository | DA08 |
| DEC07 | Autenticación JWT y control de acceso por roles | DA03 |
| DEC08 | Comunicación entre módulos solo mediante services | DA08 |
| DEC09 | Integraciones externas encapsuladas (notificaciones y meteorología) | DA04, DA01 |
| DEC10 | Notificaciones por múltiples canales | DA01, DA06 |
| DEC11 | Cliente web ligero y tolerante a baja conectividad | DA06, DA01 |
| DEC12 | Validación de alertas antes de su publicación | DA07 |

---

### DEC01. Monolito modular en lugar de microservicios
- **Contexto:** el sistema debe operar durante emergencias, con un equipo pequeño y recursos limitados.
- **Decisión:** construir una sola aplicación desplegable, dividida en módulos de negocio (usuarios, alertas, zonas-riesgo, notificaciones, monitoreo).
- **Justificación:** simplifica el despliegue, la operación y el monitoreo; evita la complejidad y la latencia de red de los microservicios.
- **Consecuencias:** (+) menor costo y complejidad; (−) todo se escala junto. Los límites de módulo permiten extraer servicios en el futuro.

### DEC02. Arquitectura en tres capas
- **Decisión:** organizar cada módulo en presentación (routes/controller), lógica de negocio (service) y datos (repository).
- **Justificación:** separa responsabilidades y facilita pruebas y cambios.
- **Consecuencias:** cada capa solo invoca a la inmediatamente inferior; evita lógica de negocio en controladores y SQL en servicios.

### DEC03. Node.js 20 LTS + Express
- **Decisión:** usar Node.js 20 LTS con Express como framework web.
- **Justificación:** ecosistema amplio, buen manejo de E/S concurrente (muchas consultas simultáneas de alertas), versión con soporte a largo plazo.
- **Consecuencias:** un solo lenguaje (JavaScript) en cliente y servidor.

### DEC04. API REST con JSON sobre HTTPS
- **Decisión:** exponer la API en `/api/v1/` con JSON sobre HTTPS.
- **Justificación:** estándar, simple de consumir por el cliente web y por futuras apps móviles; el versionado protege a los clientes de cambios.
- **Consecuencias:** el cifrado en tránsito protege datos de usuarios y funciones administrativas.

### DEC05. PostgreSQL con PostGIS
- **Decisión:** una única base de datos PostgreSQL con extensión PostGIS.
- **Justificación:** el sistema requiere ubicación geográfica de zonas de riesgo; PostGIS permite consultas espaciales (¿qué usuarios o zonas están dentro del área de una alerta?). Ofrece transacciones ACID para mantener información consistente.
- **Consecuencias:** una sola fuente de verdad; requiere respaldos y réplica para disponibilidad.

### DEC06. Sequelize (ORM) y patrón Repository
- **Decisión:** el acceso a datos pasa por repositories que usan Sequelize, con modelos y pool de conexiones compartidos (`src/shared/db`).
- **Justificación:** aísla la lógica de negocio del motor de base de datos y centraliza las consultas.
- **Consecuencias:** facilita cambios de esquema y pruebas con repositorios simulados.

### DEC07. Autenticación JWT y control de roles
- **Decisión:** autenticar con tokens JWT y autorizar por rol (Ciudadano, Administrador, Autoridad local, Personal de emergencia) mediante middleware.
- **Justificación:** las funciones administrativas requieren autenticación y las alertas solo deben ser registradas por usuarios autorizados (restricciones del sistema).
- **Consecuencias:** la consulta de alertas vigentes puede ser pública y sin autenticar, para no frenar el acceso de la población en emergencias.

### DEC08. Comunicación entre módulos solo mediante services
- **Decisión:** un módulo no accede al repository ni a las tablas de otro módulo; usa su service.
- **Justificación:** mantiene bajo acoplamiento y permite extraer un módulo a futuro.
- **Consecuencias:** requiere disciplina de diseño y revisiones de código.

### DEC09. Integraciones externas encapsuladas
- **Decisión:** el consumo de la fuente meteorológica (monitoreo) y del servicio de notificaciones se encapsula en sus respectivos services, vía HTTPS/REST.
- **Justificación:** el sistema depende de servicios externos que pueden fallar; aislarlos permite aplicar reintentos, tiempos de espera y reemplazar proveedores.
- **Consecuencias:** si la fuente meteorológica falla, el sistema sigue funcionando con registro manual de alertas por el administrador.

### DEC10. Notificaciones por múltiples canales
- **Decisión:** soportar varios canales (SMS, push, correo, WhatsApp) mediante el servicio de notificaciones.
- **Justificación:** en zonas de Ayacucho puede haber conectividad limitada; el SMS llega donde no hay datos móviles estables.
- **Consecuencias:** mayor alcance; depende de los servicios de comunicación disponibles.

### DEC11. Cliente web ligero y tolerante a baja conectividad
- **Decisión:** cliente web (HTML/CSS/JS) con carga liviana, mapa de alertas y caché de las últimas alertas consultadas.
- **Justificación:** usabilidad y disponibilidad para la población con conexiones inestables.
- **Consecuencias:** se puede evolucionar a PWA para uso sin conexión.

### DEC12. Validación de alertas antes de su publicación
- **Decisión:** las alertas siguen un flujo de estados (registrada → validada → publicada → actualizada → cerrada), y solo se difunden tras validación.
- **Justificación:** la información debe validarse antes de difundirse para evitar falsas alarmas.
- **Consecuencias:** añade un paso de validación, pero mejora la confiabilidad y la confianza de la población.
