# BRIEF TÉCNICO - BACKEND MD
**Documento de especificación del backend para landing page de marca personal**  
*Miguel Garzón - Product Manager Freelance*  
*Versión: 1.0 - Agosto 2026*  

---

## 1. VISIÓN GENERAL

**Propósito del sistema:**  
Landing page corporativa que capta leads calificados mediante formulario de contacto. El backend procesa las solicitudes de contacto, aplica validaciones de negocio y envía notificaciones por correo electrónico al titular del proyecto. No hay base de datos propia; el almacenamiento se delega en Google Sheets u otro servicio externo en iteraciones futuras. En la fase actual, todos los leads se envían por correo directo a la cuenta de Miguel Garzón.

**Filosofía del proyecto:**  
"Cada trabajo que cae en mis manos se vuelve el más óptimo en todo sentido". El backend debe ser liviano, de bajo mantenimiento y funcional sin complejidad innecesaria. Arquitectura SPA (Single Page Application) en el frontend, backend como servicio auxiliar que complementa la experiencia sin opacarla.

**Alcance de esta documentación:**  
Arquitectura, contratos de API, modelo de datos, validaciones, integraciones y configuración. Queda excluido el código de implementación concreto, pseudocódigo o snippets de programación. Este documento sirve como guía técnica para otro desarrollador que pueda implementar el backend sin necesidad de interpretar la arquitectura.

**Restricciones clave confirmadas:**
- Sin base de datos propia en Fase 1
- Envío de email directo via SMTP Google
- Campo "presupuesto" obligatorio en formulario
- Área de auth planeada para Fase 2
- Mantenimiento por cuenta de Miguel Garzón ("YO Mismo")
- Presupuesto y tiempo: MODERADO

---

## 2. ESTRUCTURA Y ORGANIZACIÓN DEL BACKEND

**Patrón arquitectónico:**  
Aplicación Single Page Application (SPA) con backend ligero que funciona como servicio de apoyo. El frontend (landing page) se comunica con el backend únicamente a través de los endpoints definidos en este documento. No hay capas complejas, routers anidados ni estructuras de directorios profundas.

**Organización de carpetas y módulos (sugerencia de estructura, no código):**
```
/backend/
├── api/           # Módulo de rutas y endpoints públicos
├── services/      # Módulo de servicios auxiliares (envío de email)
├── validators/    # Módulo de validación de entradas
├── config/        # Módulo de configuración y variables de entorno
├── middleware/    # Módulo de middleware del servidor (rate limiting, error handling)
└── README.md      # Documentación interna para configuración y despliegue
```

**Principios de organización:**
- Cada módulo cumple una responsabilidad única y bien definida (principio de responsabilidad única).
- Sin acoplamiento fuerte entre módulos; cada uno puede ser reemplazado o modificado independientemente.
- Configuración externa por medio de variables de entorno (archivo .env).
- Sin dependencia de frameworks complejos; estructura liviana compatible con Node.js/Express o servicios serverless equivalentes.

**Nivel de complejidad:**  
Bajo. El backend no es el foco de la aplicación; su propósito es dar soporte al frontend y a las integraciones externas.

---

## 3. MÓDULOS Y RESPONSABILIDADES

**Módulo API (Rutas y Endpoints):**  
Responsable de recibir peticiones HTTP del frontend (landing page) y procesarlas según su tipo. Este módulo expone una mínima cantidad de endpoints, todos de acceso público en Fase 1. No hay sesiones de usuario ni áreas protegidas en esta fase. Sus responsabilidades son:
- Recibir y validar solicitudes entrantes.
- Aplicar reglas de negocio básicas.
- Despachar a los módulos correspondientes (validación, envío de email).
- Responder con estructura JSON estandarizada.

**Módulo Services (Servicios auxiliares):**  
Responsable de la lógica que interacciona con servicios externos. En la fase actual, el único servicio externo es el envío de correo electrónico via SMTP. Este módulo:
- Configura la conexión con el proveedor de email (Google SMTP).
- Formatea el contenido del correo a partir de los datos del formulario.
- Ejecuta el envío y maneja respuestas éxito/fallo.
- No contiene lógica de negocio compleja; su único objetivo es transportar información de un punto a otro con el formato adecuado.

**Módulo Validators (Validaciones):**  
Responsable de verificar que los datos entrantes cumplan con el formato y reglas esperadas antes de ser procesados por otros módulos. Sus validaciones son:
- Formato de email (regex básica).
- Longitud de campos de texto.
- Elección obligatoria de campos de selección (ej. servicio_interes, presupuesto).
- Ausencia de contenido vacío en campos críticos.
- No implementa validaciones complejas de base de datos ni búsquedas en tiempo real.

**Módulo Config (Configuración):**  
Responsable de leer y proveer las variables necesarias para que el backend funcione en distintos entornos (desarrollo, producción, staging). Almacena:
- Credenciales de proveedores externos (Google SMTP).
- Configuración de rate limiting.
- URLs de redirección o endpoints externos.
- Banderas de entorno (NODE_ENV: production, development, staging).
Este módulo no contiene código lógico; es pura configuración externalizable.

**Módulo Middleware (Capas intermedias):**  
Capa que se sitúa entre la recepción de la petición y el procesamiento final. Sus responsabilidades son:
- Rate limiting: máximo de solicitudes por período de tiempo por dirección IP para prevenir spam o abusos.
- Manejo estandarizado de errores: convertir errores inesperados en respuestas JSON consistentes.
- CORS (Cross-Origin Resource Sharing) si futuras expansiones lo requieran.
- Registro básico de peticiones (logging) para auditoría y debugging.

**Archivo README.md:**  
Documento interno que instruye sobre:
- Cómo desplegar el backend.
- Qué variables de entorno son obligatorias.
- Cómo configurar credenciales de servicios externos.
- Comandos útiles para testing o debugging.
- Guía para migraciones o actualizaciones futuras.

---

## 4. MODELO DE DATOS (ENTIDADES, MODELOS Y RELACIONES)

**Entidad principal: Lead (Lead de contacto)**  
Representa una solicitud de contacto enviada desde el formulario de la landing page. Esta entidad es *standalone* (no tiene relaciones con otras entidades en Fase 1). Sus atributos son:

| Atributo | Tipo | Descripción | Obligación |
|----------|------|-------------|------------|
| `id` | Identificador único (UUID o similar) | Identificador único para cada lead generado. | Obligatorio. Generado automáticamente al crear el registro. |
| `nombre` | Cadena de texto | Nombre completo de la persona que contacta. | Obligatorio. Mínimo 2 caracteres, máximo 100. |
| `email` | Cadena de texto (formato email) | Correo electrónico del remitente. | Obligatorio. Debe pasar validación de formato regex. |
| `servicio_interes` | Enum (categoría cerrada) | El servicio del que el usuario quiere más información. | Obligatorio. Valores permitidos: "Definición MVP", "Optimización", "Acompañamiento", "Validación". |
| `mensaje` | Texto opcional | Mensaje libre escrito por el usuario. | Opcional. Si viene, máximo 500 palabras o 1000 caracteres. |
| `presupuesto` | Cadena de texto (OBLIGATORIO por decisión confirmada) | Indica si el usuario tiene presupuesto para el proyecto. | **Obligatorio** (decisión confirmada por usuario). Valores permitidos: "tiene presupuesto", "no tiene presupuesto", "consultar precio". > **Nota (DEC-004)**: Presupuesto es obligatorio por decisión confirmada del stakeholder, que actualiza Brief.md L119/L168. |
| `fecha_creacion` | Marca de tiempo ISO 8601 | Fecha y hora exacta en que se creó el lead. | Obligatorio. Generado automáticamente en el momento del registro. |
| `fuente` | Cadena de texto | Cadena fija o identificador de dónde provino el lead. | Opcional. Por defecto: "formulario_web". |

**Relaciones:**  
Ninguna en Fase 1. La entidad Lead es independiente. Posible relación futura con registro en Google Sheets o CRM, pero fuera del alcance actual.

**Modelo de persistencia (decisión técnica):**  
No hay modelo de base de datos propia en Fase 1. La persistencia se logra mediante el envío de correo electrónico: cada lead genera un email que llega a la bandeja de Miguel. Esto significa que "el modelo de datos" existe principalmente para validar y formatear la información antes de enviarla; no hay tablas, consultas SQL ni ORM. En iteraciones futuras (Fase 2), se podrá considerar integración con Google Sheets o un CRM ligero, pero la Fase 1 está diseñada para ser operativa sin base de datos propia.

---

## 5. API Y CONTRATOS DE COMUNICACIÓN (ENDPOINTS Y OPERACIONES A NIVEL FUNCIONAL)

**Protocolo:**  
HTTP/1.1 sobre HTTPS. Todas las respuestas vienen en formato JSON. El frontend (landing page) consumirá estos endpoints a través de llamadas fetch o similar, sin necesidad de construir URLs complejas ni rutas anidadas.

### Endpoint 1: POST /api/contacto

*Objetivo:* Recibir y procesar un nuevo lead desde el formulario de la landing page.

- **Método:** POST
- **Ruta:** /api/contacto
- **Cuerpo de solicitud (JSON):** Objeto con los campos del formulario. Estructura esperada:

```json
{
  "nombre": "Juan Pérez",
  "email": "juan@ejemplo.com",
  "servicio_interes": "Definición MVP",
  "mensaje": "Hola, me interesa saber más sobre cómo definir un MVP para mi startup",
  "presupuesto": "tiene presupuesto"
}
```

- **Campos obligatorios en el cuerpo:** nombre, email, servicio_interes, presupuesto.
- **Campos opcionales:** mensaje.
- **Headers esperados:** `Content-Type: application/json` (el frontend debe enviar JSON).

**Comportamiento esperado:**
1. Recepción de los datos JSON.
2. Aplicación de validaciones (ver sección de Validaciones).
3. Si las validaciones fallan: respuesta JSON con `success: false` y lista de errores; código de estado HTTP 400.
4. Si las validaciones pasan: procesamiento del lead, envío de email y respuesta exitosa.
5. Respuesta exitosa: JSON con `success: true`, `id` (identificador generado) y mensaje de confirmación; código de estado HTTP 200.

**Respuestas esperadas:**

Éxito (200):
```json
{
  "success": true,
  "id": "uuid-generado-o-identificador-según-implementacion",
  "mensaje": "Formulario recibido. Te contactaré pronto."
}
```

Error de validación (400):
```json
{
  "success": false,
  "errores": [
    "El campo nombre es obligatorio",
    "El email no tiene formato válido",
    "El campo presupuesto es obligatorio y debe ser 'tiene presupuesto', 'no tiene presupuesto' o 'consultar precio'"
  ]
}
```

Error de servidor (500):
```json
{
  "success": false,
  "error": "Error interno, intente nuevamente o contacte directamente"
}
```

### Endpoint 2: GET /health

*Objetivo:* Health check simple para monitoreo y verificación de que el servicio está activo.

- **Método:** GET
- **Ruta:** /health
- **Cuerpo de solicitud:** Ninguno (vacío).

**Respuesta esperada:**
```json
{
  "status": "ok",
  "timestamp": "marca-de-tiempo-ISO-8601"
}
```

- **Código de estado:** 200 si el servicio responde.
- **Propósito:** Permitir a herramientas de monitoreo (o el propio Miguel) verificar que el backend está en línea y funcionando sin necesidad de enviar un formulario. Es opcional y puede implementarse en cualquier momento, pero su presencia facilita el mantenimiento.

**Contratos adicionales (consideraciones generales):**
- Todas las respuestas JSON deben tener la propiedad `success` como primer campo identificable.
- Los códigos de estado HTTP deben respetarse: 200 para éxito, 400 para errores de cliente (validaciones), **429 para rate limiting**, 500 para errores de servidor.
- El cuerpo de las respuestas siempre será JSON; no hay retornos HTML o texto plano esperados en los endpoints definidos.
- El charset de todas las respuestas es UTF-8.

**Tabla de códigos de estado HTTP:**

| Código | Nombre | Payload de ejemplo | Condición de disparo |
|--------|--------|-------------------|---------------------|
| 200 | OK | `{"success": true, "id": "...", "mensaje": "..."}` | Lead válido, email enviado |
| 400 | Bad Request | `{"success": false, "errores": ["..."]}` | Validación fallida (campo faltante, formato inválido) |
| **429** | **Too Many Requests** | `{"success": false, "error": "rate_limit", "retry_after": <segundos>}` | **Se excede límite de 5 envíos/hora/IP (ver L231, L530)** |
| 500 | Internal Server Error | `{"success": false, "error": "..."}` | Fallo interno (SMTP, config, error no anticipado) |

---

## 6. REGLAS DE NEGOCIO

Reglas definidas y aplicables en el backend:

1. **Formulario completo obligatorio:** Para que un lead sea considerado válido y procese, deben cumplirse todas las validaciones de campos obligatorios (nombre, email, servicio_interes, presupuesto). Si falta algún campo obligatorio, la solicitud es rechazada con errores específicos.
2. **Valor único de presupuesto:** El campo presupuesto es obligatorio y debe ser exactamente uno de los tres valores permitidos: "tiene presupuesto", "no tiene presupuesto", "consultar precio". No se aceptan valores vacíos, null, espacios en blanco apenas, ni variaciones como "SI", "No", "si", "no" (solo los tres valores literales definidos). Esto asegura consistencia en la recepción de leads y facilita el seguimiento por parte de Miguel.
3. **Formato de email:** El campo email debe pasar una validación de formato regex básica (presencia de "@", dominio, punto TLD). No se aceptan strings que no tengan aspecto de email, aunque tengan contenido de texto.
4. **Longitud razonable de nombre:** El campo nombre debe tener como mínimo 2 caracteres y como máximo 100. Nombres demasiado cortos o demasiado largos son rechazados.
5. **Un lead por envío:** Cada envío exitoso del formulario genera un único lead. No hay lógica de "duplicados" estricta en Fase 1 (el mismo usuario puede enviar múltiples consultas), pero cada envío exitoso crea un nuevo registro con fecha y hora únicas.
6. **Respuesta al usuario:** Independientemente del éxito o fallo del procesamiento backend, el frontend debe mostrar siempre una respuesta al usuario. En caso de éxito, se muestra mensaje de confirmación. En caso de validación fallida, se muestran los errores para que el usuario corrija y reenvíe. En caso de error de servidor, se muestra mensaje genérico para no frustrar al usuario con detalles técnicos.
7. **Rate limiting (límite de velocidad):** Para proteger el backend y evitar spam, se aplica un límite de 5 envíos de formulario por hora por dirección IP. Si un mismo IP intenta enviar más de 5 formularios en una ventana de 1 hora, las solicitudes subsiguientes responden con error 429 (Too Many Requests) y un mensaje genérico de "Demasiados intentos, intente más tarde". Esto protege la cuenta de email de Miguel de recibir cientos de spam y respeta la filosofía "MODERADO" del proyecto.
8. **Hora de envío registrada:** Cada lead incluye `fecha_creacion` con marca de tiempo exacta. Esto permite a Miguel ordenar y priorizar leads según cuándo fueron recibidos, respaldando su filosofía de trabajo optimizado y "en todo sentido".

---

## 7. VALIDACIONES

Validaciones aplicadas en el backend (lado servidor, aunque el frontend también las debería tener):

**Validaciones de campo obligatorio:**
- `nombre`: Presencia de valor, longitud mínima 2, máxima 100. Si falta o está vacío, error.
- `email`: Presencia de valor y paso de validación regex `^\S+@\S+\.\S+$` (o similar). Si falta formato, error.
- `servicio_interes`: Presencia y que sea uno de los 4 valores permitidos. Si viene vacío o con valor no previsto, error.
- `presupuesto`: OBLIGATORIO (decisión confirmada por usuario). Presencia y que sea uno de los 3 valores permitidos: "tiene presupuesto", "no tiene presupuesto", "consultar precio". Si falta o no es un valor válido, error de validación inmediata.

**Validaciones de campo opcional:**
- `mensaje`: Si viene presente, validar longitud máxima (500 palabras o 1000 caracteres aprox.). Si viene vacío o nulo, no es error; simplemente no hay contenido de mensaje. Esto permite que el formulario funcione aunque el usuario no escriba nada en el campo de mensaje.

**Validaciones cruzadas o de contexto:**
- No hay validaciones cruzadas complejas en Fase 1 (ej. "si el usuario elige X, entonces Y es obligatorio" no aplica, ya que el formulario tiene campos independientes).
- El backend no realiza enriquecimiento de datos (no busca en bases externas, no valida que el email exista realmente, no verifica titulares de dominio). Solo valida formato y reglas de negocio definidas.

**Flujo de validación en el endpoint POST /api/contacto:**
1. Recepción de cuerpo JSON.
2. Verificación de presencia de nombre. Si no pasa, error 400 y retorno.
3. Verificación de formato de email. Si no pasa, error 400 y retorno.
4. Verificación de presencia y valor de servicio_interes. Si no pasa, error 400 y retorno.
5. Verificación crítica: Presencia y valor de presupuesto. Si no pasa, error 400 y retorno. Esta es la validación diferenciadora decidida en el análisis.
6. Verificación opcional de longitud de mensaje (solo si viene presente).
7. Si todas pasan, continuación al paso de procesamiento (envío de email).

Todas las validaciones devuelven al frontend una respuesta JSON estructurada con `success: false` y un array `errores` listando cada falla encontrada. Nunca se devuelve al usuario el raw error técnico; siempre se traduce a mensajes comprensibles para el usuario.

---

## 8. AUTENTICACIÓN Y AUTORIZACIÓN

**Estado actual (Fase 1):**  
Sin autenticación de usuarios. El backend no distingue entre quién está haciendo peticiones. Cualquier cliente puede llamar a los endpoints públicos definidos. Esto es intencional: la landing page es pública y su único objetivo es captar leads, no controlar acceso de usuarios registrados.

**Mecanismos presentes (aunque inactivos para auth):**
- No hay sistema de login, logout.
- No hay tokens (JWT, sesiones, cookies de auth).
- No hay verificación de identidad del emisor de la solicitud.
- No hay áreas de usuario privadas en la landing page.

**Planificación para Fase 2 (confirmada en decisiones del usuario):**  
El usuario marcó "Sí" ante la pregunta si se planea área de usuario privada en futuras fases. Esto implica:

- **Diseño preparatorio:** El backend está arquitectado de manera que la autenticación puede agregarse en Fase 2 sin necesidad de reestructurar endpoints principales. Los endpoints actuales (POST /api/contacto, GET /health) seguirán funcionando públicamente, pero estarán preparados para ser protegidos.
- **Posibles mecanismos a implementar en Fase 2** (solo mencionados como futuros, no documentados como parte activa del presente archivo para no sobreespecificar):
  - Autenticación basada en JWT (JSON Web Tokens): emitiendo token tras login de usuario registrado.
  - Sesiones de usuario con almacenamiento en cookie firmado.
  - Integración con proveedores de identidad (Google Login, LinkedIn) dados los perfiles de agentes del proyecto.
  - Middleware de protección que verifica token en cada petición a endpoints privados.
- **Endpoints que eventualmente se protegerían** (solo como especificación futura, no implementados ahora):
  - Posiblemente un nuevo endpoint POST /api/login y POST /api/refresh-token.
  - Endpoints para gestión de perfil de usuario (aunque el enfoque principal del proyecto sigue siendo la captura de leads, no la gestión de perfiles de usuario).
- **Decisión técnica:** Se documentará en el README.md y en comentarios de código futuro que la autenticación es una Fase 2. No se implementa código ni lógica ahora, pero se deja la arquitectura "abierta" para recibir el sistema de auth cuando Miguel lo requiera.

**Autorización:**  
Dado que en Fase 1 no hay autenticación, no hay autorización granular. Cualquier persona puede enviar formularios (sujeto a rate limiting y validaciones). En Fase 2, la autorización dependerá del sistema de auth elegido y definirá quién puede acceder a qué endpoints o ver qué información.

---

## 9. SEGURIDAD

**Medidas de seguridad aplicadas (Fase 1):**

1. **HTTPS obligatorio:** Todo tráfico entre frontend y backend debe ser mediante HTTPS. El certificado SSL/TLS es responsabilidad del proveedor de hosting (Webflow, Vercel, Netlify, etc.) y debe estar configurado antes del despliegue. Nunca se aceptan conexiones HTTP puras al endpoint del backend.
2. **Rate limiting (limitación de tasa):** Como se describió en Reglas de Negocio, máximo 5 envíos de formulario por hora por dirección IP. Esto mitiga ataques de fuerza bruta o envío masivo de spam que podría saturar la cuenta de email de Miguel o agotar límites de plan del proveedor de email. El límite se implementa en el módulo Middleware y se configura mediante variable de entorno `RATE_LIMIT_MAX` (valor 5) y `RATE_LIMIT_WINDOW_MS` (valor 3600000 = 1 hora en milisegundos).
3. **Sanitización de entradas (obligatoria):** El backend **debe** sanitizar/escapar `<`, `>`, `&` en todas las entradas de texto libre (mensaje, nombre), de forma obligatoria e independiente de la sanitización del frontend. Esto previene injection de contenido que podría romper el formato del email o causar comportamientos inesperados en el cliente que reciba el correo.
4. **Validación del lado servidor (no confiar en frontend):** Aunque el frontend (landing page) pueda tener validaciones visuales, el backend siempre aplica sus propias validaciones. Nunca se asume que los datos entrantes son seguros o correctos solo porque el frontend los validó. Esta es una regla de oro en el diseño del backend.
5. **No se almacenan datos sensibles:** El formulario nunca solicita ni maneja contraseñas, números de tarjeta, SSN u otro tipo de datos altamente sensibles. Los datos capturados son nombre, email, elección de servicio y mensaje opcional. Esto reduce la responsabilidad de seguridad del backend considerablemente.
6. **Exposición mínima de información en errores:** En caso de error 500 o errores inesperados, la respuesta al usuario nunca detalla la causa técnica (pilas de llamadas, nombres de variables, rutas de archivo). Siempre se devuelve un mensaje genérico: "Error interno, intente nuevamente o contacte directamente". Los detalles técnicos quedan registrados en los logs internos del servidor pero no se filtran al usuario.

**Medidas de seguridad delegadas a proveedores externos:**
- **Encriptación en tránsito:** Corresponde al proveedor de hosting y al protocolo SMTP (Google) encriptar el tránsito con TLS. El backend solicita conexión segura; le corresponde a la infraestructura garantizarla.
- **Almacenamiento de credenciales:** Las credenciales de Google SMTP (email y contraseña de aplicación u OAuth) quedan almacenadas en variables de entorno .env que están en el .gitignore y no se comparten públicamente. La seguridad de esas credenciales es responsabilidad del dueño del proyecto (Miguel Garzón) al momento de configurar su entorno de despliegue.

**Amenazas mitigadas por diseño:**
- Spam masivo: Mitigado por rate limiting.
- Injection en email: Mitigado por sanitización y validación estricta.
- Rotura de formato de email: Mitigado por validación de campos y construcción cuidadosa del cuerpo del email.
- Exposición de credenciales: Mitigado por .env y buenas prácticas de despliegue.

**Amenazas no mitigadas (por diseño/filosofía MODERADO):**
- Compromiso de cuenta email de Miguel si él mismo maneja credenciales de forma descuidada fuera del entorno controlado (ej. escribiendo passwords en documentos compartidos). El backend está diseñado para ser seguro "por diseño", pero la seguridad operativa depende también de prácticas de Miguel.
- No hay auditoría detallada de quién hizo qué y cuándo, más allá del `fecha_creacion` en el lead. En Fase 1 no hay sistema de usuarios ni login, por lo que no hay necesidad de auditoría de usuarios individuales.

---

## 10. PERSISTENCIA Y ACCESO A DATOS

**Decisión crítica (confirmada por usuario):**  
Persistencia por envío de email directo. No hay base de datos propia, no hay ORM, no hay esquema SQL/NoSQL en Fase 1. Los leads se "persisten" en el sentido de que generan un email que llega a la bandeja de entrada de Miguel Garzón. Esto cumple la función de almacenamiento y registro.

**Cómo funciona la persistencia:**
1. Validación exitosa del lead en el endpoint POST /api/contacto.
2. Construcción del cuerpo del email con todos los atributos del lead formateados en texto plano y HTML (ver descripción en sección de API).
3. Envío del email vía SMTP a la cuenta de destino (configurada por Miguel).
4. Resultado: El lead queda "persistido" en la medida en que el email queda en la bandeja de entrada y historial de Miguel. Puede buscar, filtrar, archivar ese email como cualquier otro mensaje en su cliente de email (Gmail, Outlook, etc.).

**No hay consulta de leads almacenados:**  
A diferencia de una base de datos donde se pueden hacer queries `SELECT * FROM leads ORDER BY fecha_creacion DESC`, en esta arquitectura no hay manera interna de listar o filtrar leads desde el backend. Miguel debe acceder a su bandeja de email para ver los leads. Esto refuerza la filosofía de "ligereza" y "optimización" del proyecto: no hay overhead de base de datos, búsquedas, backups de BD, etc.

**Persistencia en Fase 2 (futura):**  
El usuario contempló la posibilidad de área privada y auth, lo cual podría implicar en el futuro:
- Integración con Google Sheets API para almacenar leads en una hoja de cálculo compartida.
- Integración con CRM ligero (HubSpot free) para gestionar leads con más campos y etapas.
- Estas serían decisiones para Fase 2 y no forman parte de la especificación actual. Si se llega a implementar, el modelo de datos Lead se extendería con relaciones a esas entidades externas, pero actualmente no existen.

**Campos que se persisten implícitamente:**  
Además de los atributos definidos en el modelo de datos, cada lead que llega por email lleva implícitamente:
- La dirección de correo del remitente (campo email).
- La fecha y hora de recepción en el servidor (diferente a `fecha_creacion` que es cuando se generó el registro lógico, la fecha de recepción en email puede variar levemente por tiempos de red y procesamiento).
- Posibles encabezados de spam, autenticación DKIM/SPF (gestión por proveedor de email).

---

## 11. MANEJO DE ERRORES

**Filosofía de manejo de errores:**  
El backend debe fallar de manera predecible y comunicar el error al frontend de manera clara, sin exponer información técnica sensible al usuario final. El diseño busca la resiliencia: el sistema debe recuperarse gracefully de situaciones inesperadas.

**Códigos de estado HTTP y cuándo usarlos:**
- **200 (OK):** Solicitud procesada exitosamente. Se usa cuando el lead es recibido, validado y el email es enviado sin incidencias. El cuerpo JSON incluye `success: true`.
- **400 (Bad Request):** Solicitud no procesada por error del cliente. Se usa cuando las validaciones fallan (campo faltante, formato inválido, valor de presupuesto incorrecto). El cuerpo JSON incluye `success: false` y `errores` listando las fallas. Este es el código de respuesta más frecuente esperado en operaciones normales (errores del usuario corrigiendo el formulario).
- **429 (Too Many Requests):** Límite de velocidad excedido. Se usa cuando el mismo IP intenta enviar más de 5 formularios en 1 hora. El cuerpo JSON puede incluir `success: false` y un mensaje como "Demasiados intentos, intente más tarde". Este código respeta la filosofía de protección contra abusos sin ser tan restrictivo como otros sistemas más agresivos.
- **500 (Internal Server Error):** Error inesperado en el servidor. Se usa cuando algo falla internamente que no fue previsto por las validaciones ni por el rate limiting. Ejemplos: caída del servicio SMTP, error de configuración de variables de entorno, error de sintaxis no anticipado. El cuerpo JSON incluye `success: false` y un mensaje genérico para el usuario. Nunca se debe incluir el stack trace o detalles técnicos en el cuerpo de la respuesta 500 vista por el usuario.

**Formato de respuesta de error (todos los casos):**

Para errores de validación:
```json
{
  "success": false,
  "errores": [
    "Mensaje legible describiendo qué salió mal"
  ]
}
```

Para errores de servidor más generales:
```json
{
  "success": false,
  "error": "Descripción genérica del error, no técnica para el usuario"
}
```

**Registro de errores (logging):**  
El backend debe registrar errores en sus logs internos (consola, archivo, servicio de monitoring). Lo que se debe registrar:
- Tipo de error (validación, rate limiting, SMTP error, error desconocido).
- Hora/fecha del error (marca de tiempo).
- Datos relevantes para debugging (siempre respetando privacidad y no exponiendo datos sensibles del usuario; por ejemplo, se puede loguear que "falló validación de email" sin loguear el email completo si contiene datos sensibles).
- Petición entrante (IP, hora, endpoint solicitado) para correlacionar con accesos.

**Qué NO se debe hacer en manejo de errores:**
- Mostrar pilas de llamadas (stack traces) al usuario final.
- Exponer rutas de archivo del servidor.
- Filtrar datos del usuario en mensajes de error (ej. no incluir el email completo en mensaje de "error de validación", solo decir "formato de email inválido").
- Ignorar errores silenciosamente (siempre debe haber una respuesta al frontend, aunque sea de error).

**Recuperación después de error:**  
El frontend debe estar preparado para manejar las tres categorías de respuestas (éxito 200, error de cliente 400/429, error de servidor 500) y actuar en consecuencia: en caso de error de validación, limpiar campos y mostrar mensajes; en caso de rate limiting, informar al usuario y bloquear temporalmente el envío; en caso de error de servidor, informar al usuario y ofrecer alternativa (ej. contacto directo por email/LinkedIn que aparecen en el footer de la landing page).

---

## 12. INTEGRACIONES

**Integraciones confirmadas y activas en Fase 1:**

### SMTP Google Mail (Email directo)

- **Propósito:** Enviar cada lead recibido al correo electrónico de Miguel Garzón.
- **Tecnología:** Nodemailer (o equivalente) con configuración de SMTP de Google (smtp.gmail.com, puerto 465 o 587).
- **Credenciales necesarias** (para Miguel configurar en .env):
  - `GOOGLE_EMAIL`: dirección de correo de Miguel.
  - `GOOGLE_APP_PASSWORD`: contraseña de aplicación de Google (generada desde la cuenta de Google, necesaria si se usa autenticación de dos factores) u token OAuth 2.0.
- **Funcionamiento:** Al validar exitosamente un lead, el servicio de email toma los datos del lead y construye un mensaje con:
  - Remitente: "Formulario Landing" <GOOGLE_EMAIL>.
  - Destinatario: GOOGLE_EMAIL (el propio Miguel recibe los leads a su propia bandeja).
  - Asunto: Nuevo lead: [servicio_interes].
  - Cuerpo texto y HTML con todos los campos del lead formateados.
- **Límites:** Dependiente del plan de Google. Cuenta Gmail free permite envío de hasta 100 emails/día (aprox.). Si Miguel requiere mayor capacidad, tendría que considerar servicio transaccional (SendGrid, Mailgun) en Fase 2, pero Fase 1 está diseñada dentro de estos límites.

**Validación de formato y campos (interno):**  
No es una "integración externa" en sentido estricto, pero el backend valida y estructura los datos antes de integrarlos en el email. Esto asegura que la integración con SMTP funcione sin errores de formato.

**Integraciones futuras (Fase 2, no parte de especificación actual):**
- Google Sheets API: Para almacenar leads en hoja de cálculo en lugar de/además de email.
- HubSpot CRM API: Para gestionar leads con más campos y etapas de embudo.
- Calendly API: Para permitir agendar consultas directamente desde el formulario.
- SendGrid/Mailgun: Si se superan los límites de envío gratuito de Google.

**Notas de integración:**
- Todas las integraciones externas se configuran mediante variables de entorno en .env. Ninguna credencial o clave API queda hardcodeada en el código.
- Las integraciones están diseñadas para ser "plug and play": Miguel puede activar o desactivar una integración modificando su .env y reiniciando el servicio, sin necesidad de tocar código.
- Cada integración tiene su propio bloque de código/módulo separado, de modo que agregar una nueva no afecte a las existentes (principio de responsabilidad simple y bajo acoplamiento).

---

## 13. CONFIGURACIÓN Y VARIABLES NECESARIAS

**Archivo .env (ejemplo y guía de variables obligatorias/opcionales):**

Todas las variables a continuación deben definirse en un archivo llamado .env en la raíz del proyecto backend. Este archivo nunca debe subirse a repositorios públicos ni compartirse. Es de uso local/servidor exclusivo.

**Variables OBLIGATORIAS (deben definirse para que el backend funcione):**

| Variable | Descripción | Valor esperado | Ejemplo |
|----------|-------------|-----------------|---------|
| `GOOGLE_EMAIL` | Correo electrónico de la cuenta Google que enviará los leads. | Dirección de email válida. | miguel@miguelgarzon.com |
| `GOOGLE_APP_PASSWORD` | Contraseña de aplicación de Google (o token OAuth). | String generado por Google. | abcd1234efgh5678ijkl9012mnop3456 |
| `RATE_LIMIT_MAX` | Número máximo de envíos por ventana de tiempo por IP. | Entero positivo. | 5 |
| `RATE_LIMIT_WINDOW_MS` | Ventana de tiempo para el rate limiting, en milisegundos. | Entero positivo. | 3600000 (1 hora) |

**Variables OPCIONALES (el backend funcionará sin ellas, pero facilitan configuración):**

| Variable | Descripción | Valor esperado | Ejemplo |
|----------|-------------|-----------------|---------|
| `NODE_ENV` | Entorno de ejecución. Afecta comportamiento de algunos módulos. | development, production, staging. | production |
| `SITE_NAME` | Nombre del sitio/web, para incluir en emails o logs. | Cualquier string. | Miguel Garzón - Product Manager |
| `PORT` | Puerto en que escuchará el servidor backend. | Puerto disponible. | 3000 (o dejar que el hosting asigne) |
| `EMAIL_DESTINO` | Opcional: Si se quiere sobreescribir el destinatario default. | Dirección email. | miguel@miguelgarzon.com |
| `LOG_LEVEL` | Nivel de detalle de los logs. | info, warn, error, debug. | info |

**Notas sobre el archivo .env:**
- El archivo .env debe incluirse en .gitignore para que no se versione.
- Miguel deberá crear su propio .env con sus credenciales reales antes de despliegue.
- Se recomienda no commitear nunca el archivo .env real en ningún repositorio.
- Un archivo .env.example (sí versionable) puede incluirse en el repositorio para que otros desarrolladores sepan qué variables son necesarias, sin exponer valores reales.

**Ejemplo de archivo .env.example (versionable):**

```
# Configuración obligatoria - Miguel debe reemplazar con sus valores reales
GOOGLE_EMAIL="tu-email@dominio.com"
GOOGLE_APP_PASSWORD="tu-contraseña-de-aplicacion-aqui"

# Configuración de rate limiting (filosofía MODERADO)
RATE_LIMIT_MAX=5
RATE_LIMIT_WINDOW_MS=3600000

# Entorno
NODE_ENV=production

# Opcionales
SITE_NAME="Miguel Garzón - Product Manager"
PORT=3000
```

**Carga de variables:**  
El backend debe leer estas variables al iniciarse. Si alguna variable obligatoria falta, el backend debe fallar un startup check y negarse a levantarse, evitando así que un servidor se despliegue sin las credenciales necesarias para enviar emails. Se debe mostrar un mensaje de error claro en consola al respecto.

---

## 14. FLUJOS DE INFORMACIÓN

### Flujo 1: Envío exitoso de formulario (ruta feliz)

1. Usuario llena y envía el formulario en la landing page (frontend).
2. El frontend envía un POST a `POST /api/contacto` con cuerpo JSON con los datos del formulario.
3. El backend recibe la solicitud.
4. Módulo Validators ejecuta validaciones sobre nombre, email, servicio_interes, presupuesto.
5. Si alguna validación falla:
   - Se construye respuesta JSON `success: false` con array de errores.
   - Se devuelve al frontend código 400 (o 429 si es rate limiting).
   - El frontend muestra los mensajes de error al usuario; el usuario corrige y reenvía.
   - Flujo termina aquí. No hay persistencia ni envío de email.
6. Si todas las validaciones pasan:
   - Se procede al siguiente paso.
   - Módulo Services construye el cuerpo del email con todos los campos del lead formateados (texto plano y HTML).
   - Módulo Services ejecuta el envío SMTP a través de las credenciales de Google configuradas.
   - Si el envío de email falla (caída de servicio, credenciales incorrectas, límites excedidos):
     - Se captura el error.
     - Se registra en logs internos.
     - Se devuelve al frontend respuesta `success: false` con error genérico: "Error al procesar formulario. Intente nuevamente o contacte directamente".
     - El lead no se considera "enviado" aunque el usuario haya completado el formulario. Se le pide que intente más tarde o contacte por otros medios.
   - Si el envío de email tiene éxito:
     - Se devuelve al frontend respuesta `success: true` con id generado y mensaje de confirmación.
     - El usuario ve un mensaje de éxito en la landing page: "Formulario recibido. Te contactaré pronto."
     - El email llega a la bandeja de entrada de Miguel Garzón.
     - Flujo finalizado. El lead queda registrado en la bandeja de Miguel.

### Flujo 2: Health check (GET /health)

1. Sistema o usuario accede a GET /health.
2. Backend responde sin necesidad de validaciones ni parámetros.
3. Respuesta JSON `{"status": "ok", "timestamp": "..."}` con código 200.
4. Flujo terminado. Sirve para verificar que el servicio está en línea.

### Flujo 3: Rate limiting excedido

1. Un mismo IP realiza envío de formulario más de 5 veces en 1 hora.
2. Módulo Middleware detecta el patrón.
3. Se devuelve respuesta código 429 con `success: false` y mensaje "Demasiados intentos, intente más tarde".
4. El frontend muestra mensaje al usuario y bloquea el botón de envío temporalmente.
5. Después de la ventana de 1 hora, el contador reinicia y el usuario puede intentar nuevamente.

---

## 15. REQUERIMIENTOS FUNCIONALES Y NO FUNCIONALES

**Requerimientos Funcionales (qué debe hacer el backend):**

- **F-01:** Recibir solicitudes POST en endpoint /api/contacto con cuerpo JSON conteniendo los campos nombre, email, servicio_interes, presupuesto, y opcionalmente mensaje.
- **F-02:** Aplicar validaciones obligatorias a cada campo y devolver errores estructurados si alguna falla.
- **F-03:** Si todas las validaciones pasan, construir y enviar un email via SMTP a la cuenta configurada con los datos del lead.
- **F-04:** Responder al frontend con JSON estandarizado indicando éxito o fracaso, incluyendo `success: boolean`, y opcionalmente `id` o lista de errores.
- **F-05:** Aplicar rate limiting de 5 envíos por hora por IP para prevenir spam.
- **F-06:** Implementar endpoint GET /health que responda 200 OK con `status: "ok"` y marca de tiempo, sin requerir parámetros.
- **F-06 (bis):** Mantener todas las comunicaciones bajo protocolo HTTPS.
- **F-07:** No almacenar datos sensibles ni bases de datos propias en Fase 1; la persistencia es el envío de email.

**Requerimientos No Funcionales (cómo debe comportarse el backend):**

**Rendimiento (NF-01):**
- Tiempo de respuesta promedio bajo 500ms para el endpoint /api/contacto (considerando tiempo de envío de email, que es la operación más costosa).
- El tiempo de respuesta puede variar según la velocidad del SMTP, pero el resto del procesamiento (validaciones, construcción de respuesta) debe ser casi instantáneo (<50ms).

**Escalabilidad (NF-02):**
- Arquitectura diseñada para escalar horizontalmente si fuere necesario, aunque en Fase 1 es una sola instancia (serverless o pequeño VPS).
- El principal límite de escalabilidad es el proveedor de SMTP (Google free: 100 emails/día). Para escalar mayor, se requiere cambiar a servicio transaccional (SendGrid) en Fase 2, pero la arquitectura del backend está preparada para recibir ese cambio sin reestructurar endpoints.

**Disponibilidad (NF-03):**
- El backend debe estar disponible 99% del tiempo durante el horario de operación de la landing page.
- Si el servicio SMTP falla, el backend debe manejar el error gracefully (no caerse, devolver error al usuario, pero el sitio web en sí continúa funcionando).
- El endpoint /health debe responder siempre que el servidor esté en línea.

**Seguridad (NF-04):**
- Cumplir todas las medidas descritas en la sección de Seguridad (HTTPS, rate limiting, sanitización, sin datos sensibles).
- Las credenciales de Google deben manejarse siguiendo buenas prácticas (nunca hardcodeadas, siempre vía .env).
- El backend debe ser resistente a intentos de abuso (rate limiting, validaciones estrictas).

**Mantenibilidad (NF-05):**
- El código (cuando se implemente) debe estar organizado en los módulos descritos (api, services, validators, config, middleware).
- El archivo README.md debe ser suficiente para que otro desarrollador (o Miguel mismo) entienda cómo configurar y mantener el backend sin necesidad de reverse-engineering.
- Las variables de entorno deben ser la única forma de configurar comportamientos; no deben "hardcodearse" valores en el código fuente.

**Rendimiento de costo (NF-06):**
- Los costos operativos deben ser mínimos (plan free de Google SMTP, sin servidores caros, sin bases de datos).
- Si los costos comienzan a crecer desproporcionadamente, debe ser fácil identificar qué parte del backend contribuye y tomar decisiones de optimización (ej. cambiar proveedor de email, ajustar rate limiting).

---

## 16. ESCALABILIDAD, MANTENIBILIDAD Y CONSIDERACIONES TÉCNICAS

**Escalabilidad:**

- **Limitaciones actuales:** El principal cuello de botella en Fase 1 es el proveedor de email (Google free plan: 100 emails/día). Si el volumen de leads excede ese límite, el backend comenzará a devolver errores de "envío fallado" una vez agotada la cuota diaria. Esto es intencional y esperado; el negocio de Miguel Garzón Product Manager freelance no requiere cientos de leads por día, y la filosofía "MODERADO" contempla este límite.
- **Ruta de escalado (Fase 2):** Si en el futuro se requiere mayor capacidad:
  - Migración a servicio de email transaccional (SendGrid, Mailgun, Amazon SES) que ofrezcan límites mucho más altos o planes ilimitados según pago.
  - Posible integración con Google Sheets o CRM para no depender exclusivamente del volumen de emails.
  - El backend está arquitectado con módulos separados (services/, config/) de modo que cambiar el proveedor de email es un cambio aislado: solo se modifican las variables GOOGLE_EMAIL y GOOGLE_APP_PASSWORD (o se sustituyen por variables equivalentes del nuevo proveedor) y se ajustan, si es necesario, los templates de email. Los endpoints y validaciones permanecen inalterados.
- **Escalabilidad de lecturas:** Dado que no hay base de datos y los leads se consultan por email, la "escritura" (envío de leads) es el único cuello de botella. La "lectura" (Miguel revisando su email) es tan escalable como su capacidad para revisar su bandeja, que es ilimitada en términos técnicos.

**Mantenibilidad:**

- **Estructura de módulos:** Cada módulo del backend (api, services, validators, config, middleware) cumple una responsabilidad única y puede ser entendido independientemente. Esto significa que un desarrollador puede abrir el archivo services/email.js y entender exactamente qué hace sin necesidad de leer todo el código del backend.
- **Configuración externa:** Casi toda la configuración del backend (credenciales, límites, nombres de sitio) vive en el archivo .env. Esto significa que modificar comportamientos no requiere tocar código, solo las variables de entorno. Esto reduce la probabilidad de introducir bugs al cambiar configuraciones.
- **Documentación incluida:** El propio backend.md (y el README.md interno) provee la documentación necesaria. No se esperan documentos externos extensos para entender el funcionamiento básico del backend.
- **Tests:** Se requieren tests unitarios e de integración mínimos para garantizar la calidad del backend:
  - Tests unitarios de validadores (Joi/Zod): cada regla de validación (nombre 2-100, email regex, servicio_interes enum 4, presupuesto 3 valores, mensaje ≤1000 car)
  - Test de integración del endpoint POST /api/contacto: cubrir 200 (ruta feliz), 400 (validación fallida), 429 (rate limiting), 500 (fallo SMTP)
  - Test del servicio de envío de email: mock de Nodemailer, verificar construcción de cuerpo texto/HTML y headers
  - Test del endpoint GET /health: respuesta 200 con status "ok" y timestamp ISO 8601
  - Cobertura mínima objetivo: 80% en módulos validators/ y services/

**Consideraciones Técnicas Adicionales:**

- **Entorno de despliegue:** El backend puede desplegarse en cualquier entorno que soporte Node.js (si se usa Express/Nest) o serverless (Vercel Functions, Netlify Functions, Cloud Functions). Dado que el frontend es una landing page SPA, el backend no necesita estar en el mismo hosting necesariamente; puede ser un servicio separado que el frontend consuma por URL. La decisión de alojamiento dependerá de Miguel y sus preferencias, pero el backend debe ser agnóstico al hosting (usar variables de entorno, no rutas de archivo hardcodeadas).
- **Versiones y dependencias:** El backend utilizará dependencias mínimas. Lo esencial: un framework web (Express es el más ligero y conocido, compatible con la filosofía), un cliente SMTP (Nodemailer), y quizás un validador de esquema (Joi o Zod, aunque dada la simplicidad las validaciones podrían ser nativas vanilla). Las dependencias deben mantenerse actualizadas en seguridad, pero no se esperan actualizaciones frecuentes dada la estabilidad de las funcionalidades.
- **Monitoreo básico:** Dado el enfoque ligero, el monitoreo no es complejo. Lo básico es:
  - Revisar logs de consola después de cada despliegue.
  - Verificar que el endpoint /health responde 200.
  - Revisar periódicamente la bandeja de entrada de Miguel para asegurarse de que los leads llegan (o configurar algún alertamiento simple si se espera un volumen crítico).
  - Monitorear el uso del rate limiting para asegurarse de que no está bloqueando leads genuinos (si sucede, ajustar RATE_LIMIT_MAX).

**Futuras expansiones no incluidas en alcance actual:**
- Sistema de gestión de relaciones con clientes (CRM) completo.
- Automatizaciones de email de seguimiento.
- Paneles de análisis de leads.
- Integración con redes sociales automáticas.

Todas estas expansiones están fuera del alcance de backend.md versión 1.0 y serían consideraciones para Fase 2 o proyectos posteriores, manteniendo el principio de "no sobreingeniería" y "optimización en todo sentido".

---

**Cierre del documento:**  
Este backend.md constituye la especificación técnica completa y autorizada del backend para la landing page de Miguel Garzón Product Manager freelance, versión 1.0. Está diseñado para ser suficiente como guía técnica de implementación, coherente con la arquitectura global del proyecto y las decisiones confirmadas durante el análisis. Cumple la función de definir qué debe construirse, cómo debe comportarse y cuáles son sus reglas técnicas, sin contener código alguno, sirviendo como puente entre el diseño arquitectónico y la implementación concreta por parte del equipo de desarrollo o de Miguel propio.
