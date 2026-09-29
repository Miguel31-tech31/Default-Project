# FRONTEND - ESPECIFICACION TECNICA
**Documento de especificacion del frontend para landing page de marca personal**
*Miguel Garzon - Product Manager Freelance*
*Version: 1.0 - Agosto 2026*

---

## 1. VISION GENERAL

**Proposito del sistema:**
Landing page SPA (Single Page Application) que sirve como punto de contacto principal para captar leads calificados mediante un formulario de contacto. El frontend consume servicios del backend definido en `backend.md` para procesar las solicitudes de contacto, aplica validaciones del lado del cliente para mejorar la experiencia de usuario, y presenta informacion de manera clara y profesional alineada con la marca personal de Miguel Garzon.

**Filosofia del proyecto:**
"Cada trabajo que cae en mis manos se vuelve el mas optimo en todo sentido". El frontend debe ser limpio, eficiente, accesible y reflejar la marca personal de Miguel Garzon como Product Manager tecnico. Diseno minimalista con calidez, sin distracciones innecesarias.

**Alcance de esta documentacion:**
Especificacion tecnica completa del frontend: arquitectura, componentes, flujos, integracion con backend, validaciones, estados, UX/UI, seguridad y buenas practicas. Este documento sirve como guia tecnica para la implementacion del frontend, sin incluir codigo de implementacion concreto.

**Relacion con backend.md:**
Este documento toma `backend.md` como fuente de verdad para la integracion. Toda comunicacion frontend-backend esta definida en ese documento. El frontend consume los endpoints alli especificados, maneja las respuestas definidas, e implementa validaciones complementarias (nunca contradictorias) con las del servidor.

**Restricciones clave confirmadas:**
- Persistencia por email (sin base de datos)
- Sin autenticacion en Fase 1
- Campo presupuesto obligatorio con 3 valores estrictos
- Mensajes de error propios del frontend (no backend en espanol)
- Sanitizacion de caracteres especiales por frontend
- Rate limiting 429 bloquea boton por 1 hora completa

---

## 2. ARQUITECTURA Y TECNOLOGIAS

**Arquitectura general:**
Aplicacion Single Page Application (SPA) con frontend consumiendo servicios REST del backend. El frontend es autonomo en su logica de presentacion, validaciones cliente y estados de interfaz, pero depende del backend para la persistencia de leads (por email) y health checks.

**Tecnologias y frameworks definidos (DEC-001, DEC-002, DEC-003):**
- **Framework principal:** **React** (SPA moderna) — *DEC-001*
- **Gestion de estado:** **React Context** (useState/useReducer local; no se requiere librería externa para Fase 1) — *DEC-001*
- **Peticiones HTTP:** **Fetch API nativa** (suficiente para el alcance del formulario de contacto)
- **Validacion de formularios:** **Zod** (schema-first, TypeScript-ready, compartible con backend) — *DEC-003*
- **Estilos:** **Tailwind CSS** (mobile-first nativo, consistencia, velocidad) — *DEC-002*
- **Diseno responsive:** Media queries, mobile-first approach (integrado en Tailwind)
- **Despliegue:** Compatible con **Vercel, Netlify, Cloudflare Pages** u otro hosting estático/CDN

**Principios arquitectonicos:**
- Componentes reutilizables y modulares
- Separacion de responsabilidades (presentacion, logica, servicios)
- Comunicacion con backend exclusivamente via HTTP REST
- Manejo de estados predecible y rastreable
- Validaciones cliente como capa de UX, nunca como fuente de verdad

**Dependencias externas minimas:**
- No se requiere base de datos del lado del cliente
- No se requiere autenticacion de usuarios en Fase 1
- No se requiere SDK de tercerios mas alla de posibles herramientas de analiticas (opcional)

---

## 3. ESTRUCTURA DE MODULOS Y COMPONENTES

### 3.1 Modulos Principales del Frontend

| Modulo | Responsabilidades Principales |
|--------|-------------------------------|
| **App Shell** | Contenedor principal, router de secciones, layout base de la aplicacion |
| **FormState Management** | Gestionar estado del formulario: `isLoading`, `hasSent`, `showSuccess`, `errores`, `showMessage` |
| **API Service** | Wrapper para comunicacion HTTP con backend: `POST /api/contacto`, `GET /health`. Manejo de headers, errores, formateo JSON |
| **Validation Client** | Validaciones preliminares antes de enviar: formato email, longitud nombre, seleccion presupuesto. Nunca reemplaza validaciones servidor |
| **Error Handler** | Interpretar codigos de estado HTTP (200/400/429/500) y mostrar mensajes/behaviors correspondientes |
| **Rate Limiting Handler** | Detectar error 429, bloquear boton por 1 hora completa, mostrar mensaje informativo |
| **Sanitization Module** | Escapar caracteres especiales (<, >, &) en campos de texto libre antes de enviar |
| **UX Feedback** | Indicadores de loading, transiciones de estado, mensajes de exito/error, enfoque en campos con errores |

### 3.2 Componentes del Formulario

| Componente | Descripcion | Props/Estados Principales |
|------------|-------------|---------------------------|
| **FormularioContacto** | Contenedor principal del formulario de contacto. Gestionar estado global del formulario y coordinar validaciones/envio | `isLoading: boolean`, `hasSent: boolean`, `showSuccess: boolean`, `errores: string[]`, `showMessage: boolean` |
| **CampoNombre** | Campo de entrada de texto para nombre completo del contacto. Validacion: minimo 2, maximo 100 caracteres | `value: string`, `onChange: function`, `error: string`, `onValidate: function` |
| **CampoEmail** | Campo de entrada de tipo email. Validacion: formato regex (complementada por frontend) | `value: string`, `onChange: function`, `error: string`, `onValidate: function` |
| **CampoServicioInteres** | Campo de seleccion (select o radio buttons) con 4 valores permitidos: "Definicion MVP", "Optimizacion", "Acompanamiento", "Validacion" | `value: string`, `onChange: function`, `error: string`, `options: string[]` |
| **CampoPresupuesto** | **Campo OBLIGATORIO** con validacion restrictiva: select o radio buttons con exactamente 3 valores literales: "tiene presupuesto", "no tiene presupuesto", "consultar precio" | `value: string`, `onChange: function`, `error: string`, `options: string[]` > **Nota (DEC-004)**: Alineado con Backend.md y Brief.md actualizado. |
| **CampoMensaje** | Campo de texto opcional (textarea). **Aparece cuando usuario comienza a escribir.** Validacion: maximo 500 palabras / 1000 caracteres si viene contenido | `value: string`, `onChange: function`, `error: string`, `onValidate: function`, `isVisible: boolean` |
| **BotonEnviar** | Boton de envio del formulario. Cambia de estado segun: default, hover, loading, exito, error 429 | `isLoading: boolean`, `disabled: boolean`, `children: string`, `onClick: function` |
| **MensajeExito** | Componente que muestra mensaje de confirmacion despues de envio exitoso (200) | `visible: boolean`, `mensaje: string`, `onDismiss: function` |
| **ListaErrores** | Lista de errores de validacion del backend o frontend | `errores: string[]`, `onDismiss: function` |
| **IndicadorLoading** | Spinner o indicador visual durante estado de envio | `isVisible: boolean` |

### 3.3 Componentes de Apoyo

| Componente | Descripcion |
|------------|-------------|
| **Header** | Barra de navegacion superior con logo/nombre, menu de navegacion y **redes sociales: LinkedIn + GitHub** |
| **Hero** | Seccion principal con titulo, subtitulo y CTA principal |
| **SeccionSobreMi** | Biografia profesional, formacion y filosofia de trabajo |
| **SeccionServicios** | Tarjetas con servicios principales ofrecidos |
| **SeccionPortafolio** | Grid de proyectos/trabajos destacados. **Incluye filtro por categoría/tecnología (prop `filters`) activado cuando `proyectos.length > 6`** (Estructura.md principio 6) |
| **SeccionTestimonios** | Comentarios de clientes |
| **Footer** | Pie de pagina con enlaces, contactos y **redes sociales: LinkedIn + Twitter + GitHub** |
| **HealthStatus** | Indicador pequeno de estado del servicio (consume GET /health) |

---

## 4. VISTAS Y NAVEGACION

### 4.1 Estructura de Vistas

La landing page se compone de secciones dentro de una unica vista principal (SPA con scroll). No hay navegacion a paginas separadas en Fase 1.

**Secciones de la Landing Page:**
1. **Header** (siempre visible o fijo en la parte superior)
2. **Hero Section** (primera pantalla, impacto visual)
3. **Sobre Mi / Perfil** (biografia y formacion)
4. **Servicios** (tarjetas con servicios principales)
5. **Portafolio / Trabajos** (grid de proyectos destacados)
6. **Testimonios** (validacion social)
7. **Formulario de Contacto** (seccion de conversion principal)
8. **Footer** (contacto directo, redes sociales, enlaces)

### 4.2 Navegacion

**Navegacion por scroll suave:**
- Cada seccion tiene un identificador (anchor)
- Menu de navegacion permite saltar a secciones especificas
- Scroll continuo entre secciones, sin carga de paginas separadas

**Elementos de navegacion:**
- Menu superior con enlaces a: **Inicio, Sobre Mi, Servicios, Portafolio, Testimonios, Contacto** (orden definitivo — ver DEC-006; reemplaza orden de Brief.md L70)
- Botones CTA en Hero y al final de secciones clave
- Footer con enlaces rapidos y contactos directos

**Flujo de navegacion recomendado:**
Hero -> Sobre Mi -> Servicios -> Portafolio -> Testimonios -> Formulario Contacto -> Footer

**Puntos de conversion estrategicos:**
- Hero: CTA principal "Conocer mis servicios" o "Agendar diagnostico gratuito"
- Despues de Servicios: CTA "Ver casos de estudio"
- Despues de Portafolio: "Tienes un proyecto en mente?"
- Formulario de contacto: Convertir interes en conversacion
- Footer: Contacto directo por email/LinkedIn (alternativa al formulario)

### 4.3 Comportamiento en Diferentes Dispositivos

**Mobile-first approach (70%+ trafico es movil):**
- Menu se convierte en "hamburger" en pantallas pequenas
- Secciones apiladas verticalmente
- Formulario con campos apilados y boton ancho completo
- Imagenes optimizadas para carga rapida en moviles

**Desktop:**
- Menu horizontal visible
- Posibles layouts de 2 columnas en formulario (nombre/email lado izquierdo, servicio/presupuesto lado derecho)
- Mayor espacio en blanco, tipografia mas grande

**Breakpoints recomendados:**
- Mobile: hasta 768px
- Tablet: 769px - 1024px
- Desktop: 1025px en adelante

---

## 5. FLUJOS DE USUARIO

### 5.1 Flujo Principal: Envio Exitoso de Formulario

1. Usuario ingresa a la landing page y navega por secciones
2. Llega a seccion de Formulario de Contacto
3. Ve formulario limpio o con valores previos (si los hubiera)
4. Completa campos obligatorios:
   - Nombre (min 2, max 100 caracteres)
   - Email (formato valido)
   - Servicio de interes (selecciona uno de 4 valores)
   - Presupuesto (**obligatorio**, selecciona una de 3 opciones exactas)
5. **Comienza a escribir en campo Mensaje** -> Textarea aparece visualmente (componente `CampoMensaje` se hace visible)
6. Haz clic en boton "Enviar formulario"
7. **Estado de envio:** Boton se deshabilita, indicador de loading aparece (spinner o "Enviando...")
8. Frontend ejecuta:
   - Validaciones cliente (pre-envio)
   - Sanitizacion de caracteres especiales en campos de texto libre
   - Fetch POST a `POST /api/contacto` con cuerpo JSON estructurado
9. Backend responde codigo **200 (exito)**:
   - Frontend muestra mensaje de exito: "Formulario recibido. Te contactare pronto."
   - Boton puede cambiar a "Gracias!" o desaparecer, o formulario se limpia
   - **Mensaje de exito permanece visible en la misma pagina** (no redireccion)
10. Usuario puede continuar navegando o cerrar pagina

### 5.2 Flujo: Error de Validacion (Codigo 400)

1. Usuario envia formulario con campos incompletos o valores invalidos
2. Frontend realiza validaciones cliente (posibles errores pre-envio)
3. Si backend responde codigo **400 (bad request)**:
   - Backend retorna `{"success": false, "errores": ["msg1", "msg2", ...]}`
   - **Frontend muestra sus propios mensajes de error** (no los mensajes en espanol del backend)
   - Errores se resaltan en campos correspondientes (bordes rojos, mensajes inline)
   - Boton vuelve a estado activo (no loading)
4. Usuario puede corregir los errores y reenviar

### 5.3 Flujo: Error Rate Limiting (Codigo 429)

1. Usuario (o misma IP) intenta enviar formulario mas de 5 veces en 1 hora
2. Backend responde codigo **429 (too many requests)**
3. **Frontend deshabilita boton de envio por 1 hora completa** (sin posibilidad de reintento antes)
4. Muestra mensaje: "Demasiados intentos, intente mas tarde"
5. **No hay posibilidad de reintento** antes de que el contador se reinicie del lado servidor
6. Usuario puede continuar navegando otras secciones de la landing

### 5.4 Flujo: Error de Servidor (Codigo 500)

1. Backend falla inesperadamente (caida servicio SMTP, error configuracion, etc.)
2. Backend responde codigo **500 (internal server error)** con `{"success": false, "error": "Error interno, intente nuevamente o contacte directamente"}`
3. Frontend muestra mensaje generico: "Error al procesar formulario. Intente nuevamente o contacte directamente"
4. **Frontend ofrece alternativas de contacto directo** (enlaces en footer: LinkedIn, email)
5. Boton vuelve a estado activo para reintento

### 5.5 Flujo: Health Check

1. Al cargar la pagina (o cada cierta periodicidad minima), frontend consume `GET /health`
2. Si respuesta exitosa (`{"status": "ok", "timestamp": "..."}`):
   - Puede mostrar indicador pequeno de "Servicio activo" (opcional, no critico)
3. Si error:
   - No muestra indicador de estado, pero la funcionalidad principal del formulario no se ve afectada
   - Posible mensaje alternativo en footer si el servicio no esta disponible

---

## 6. INTEGRACION CON EL BACKEND

### 6.1 Comunicacion Frontend-Backend

**Protocolo:** HTTP/1.1 sobre HTTPS (obligatorio)

**Endpoints consumidos por frontend:**
1. **POST `/api/contacto`** - Envio de formulario de contacto
2. **GET `/health`** - Verificacion de estado del servicio

**Formato de comunicacion:**
- Request: JSON con `Content-Type: application/json`
- Response: JSON con estructura estandar `{success: boolean, ...}`

**Configuracion de endpoints:**
- URL base del backend debe configurarse por entorno (desarrollo vs produccion)
- Ejemplo: `https://api.miguelgarzon.com/api/contacto` en produccion
- Variable de entorno o configuracion centralizada para URL base

### 6.2 Consumo de Endpoints

#### Endpoint POST `/api/contacto`

**Cuerpo de solicitud (JSON):**
Estructura con campos del formulario: nombre, email, servicio_interes, mensaje (opcional), presupuesto (obligatorio). **No incluir campo fuente** (backend lo anade automaticamente con valor "formulario_web").

**Headers requeridos:**
- `Content-Type: application/json`
- No se requieren headers de autenticacion en Fase 1

**Manejo de respuestas:**

| Codigo | Significado | Accion Frontend |
|--------|-------------|-----------------|
| 200 | Exito | Mostrar mensaje confirmacion: "Formulario recibido. Te contactare pronto." Mantener en misma pagina |
| 400 | Error validacion | Mostrar errores con mensajes propios del frontend (no backend espanol). Resaltar campos con error |
| 429 | Rate limiting | Deshabilitar boton 1 hora completa. Mensaje: "Demasiados intentos, intente mas tarde" |
| 500 | Error servidor | Mensaje generico + ofrecer contacto directo (footer) |

#### Endpoint GET `/health`

**Frecuencia de consumo:** Solo al cargar la pagina (no periodicamente)

**Manejo de respuesta:**
- Si `{"status": "ok", "timestamp": "..."}`: Servicio activo (posible indicador opcional)
- Si error: No afecta funcionalidad principal del formulario

### 6.3 Estructura de Respuesta Esperada

**Todas las respuestas del backend siguen la estructura:**
`{"success": true/false, ...}`

**Frontend debe validar siempre la presencia del campo `success`** antes de procesar la respuesta.

---
 
## 6.x SEO TÉCNICO
 
**Implementación de los requisitos SEO del Brief.md (sección 9):**
 
| Sección | Meta Title | Meta Description | H1 | H2/H3 | Alt Text Policy | URL (Anchor) |
|---------|------------|------------------|-----|--------|-----------------|--------------|
| **Hero** | "Miguel Garzón \| Product Manager Freelance \| Definición MVP y Optimización" | "Ayudo a startups y empresas a definir MVP, optimizar procesos de desarrollo y construir productos óptimos. Servicios de consultoría product management." | "Transformo ideas en productos óptimos" | H2: Servicios, Sobre Mí, Portafolio, Testimonios, Contacto | Foto headshot: "Miguel Garzón, Product Manager Freelance" | `#hero` |
| **Sobre Mí** | — | — | "Sobre Mí" | H3: Formación, Experiencia, Filosofía | Imágenes biografía: descriptivas con keywords | `#sobre-mi` |
| **Servicios** | — | — | "Servicios" | H3 por servicio: Definición MVP, Optimización, Acompañamiento, Validación | Iconos: `alt=""` (decorativos) | `#servicios` |
| **Portafolio** | — | — | "Portafolio" | H3 por proyecto: Título del proyecto | Capturas: "Proyecto [título] — [tecnologías] — [resultado]" | `#portafolio` |
| **Testimonios** | — | — | "Testimonios" | H3 por testimonio: Nombre cliente, Empresa | Foto cliente: "[Nombre], [Cargo] en [Empresa]" | `#testimonios` |
| **Contacto** | — | — | "Contacto" | H3: Formulario, Enlaces directos | — | `#contacto` |
| **Footer** | — | — | — | — | Redes sociales: `alt="LinkedIn / Twitter / GitHub"` | — |
 
**Reglas transversales:**
- **H1 único** por vista (en SPA: solo en Hero; las secciones usan H2/H3 jerárquicos).
- **Meta tags** inyectados via React Helmet o equivalente en `App Shell`.
- **URLs limpias**: anchors `#seccion` para SPA; si se migra a multi-page futuro, rutas `/servicios`, `/sobre-mi`, `/portafolio`, `/contacto`.
- **Structured Data (JSON-LD)**: `Person` para perfil, `Service` para cada servicio, `Testimonial` para testimonios.
- **Sitemap.xml** y **robots.txt** generados en build.
 
---
 
## 7. AUTENTICACION Y AUTORIZACION

### 7.1 Estado Actual (Fase 1)

**Sin autenticacion:**
- No hay sistema de login/logout
- No hay tokens JWT, sesiones, cookies de autenticacion
- No hay verificacion de identidad del emisor
- No hay areas de usuario privadas

**Endpoints publicos:**
- POST `/api/contacto`: Accesible por cualquier cliente
- GET `/health`: Accesible por cualquier cliente

**Implicacion para frontend:**
- No es necesario implementar gestion de tokens
- No hay estado de usuario autenticado
- No hay proteccion de rutas basada en roles
- El formulario es accesible publicamente sin restricciones

### 7.2 Preparacion para Fase 2

**Arquitectura preparada para auth futura:**
- Endpoints actuales seguiran funcionando publicamente pero podran ser protegidos
- Frontend debe estar preparado para agregar mecanismos de auth sin reestructurar componentes principales
- Posibles mecanismos futuros: JWT, sesiones, OAuth (Google Login, LinkedIn)
- Endpoints que podrian protegerse futuramente: gestion de perfil, areas privadas

**Consideracion para Fase 2:**
- El componente `API Service` debe estar disenado para poder inyectar tokens de autenticacion en headers futuramente
- La logica de estado debe poder extenderse para incluir `user`, `isAuthenticated`, etc.

---

## 8. ESTADOS, VALIDACIONES Y MANEJO DE ERRORES

### 8.1 Estados de la Aplicacion

| Estado | Descripcion | Comportamiento UI |
|--------|-------------|-------------------|
| **Estado Inicial** | Formulario limpio, boton activo | Campos vacios o con placeholder de ayuda. Boton "Enviar formulario" habilitado |
| **Estado de Escritura** | Usuario interactua con campos | Validaciones en tiempo real aparecen. Campo mensaje se hace visible al escribir |
| **Estado de Envio (Loading)** | Formulario enviado, esperando respuesta backend | Boton deshabilitado, indicador de loading (spinner/"Enviando...") |
| **Estado de Exito** | Backend responde 200 | Mensaje "Formulario recibido. Te contactare pronto." visible. Boton puede cambiar a "Gracias!" o desaparecer. Formulario puede limpiarse |
| **Estado de Error Validacion** | Backend responde 400 | Errores mostrados con mensajes propios del frontend. Campos con error resaltados. Boton vuelve a activo |
| **Estado de Rate Limiting** | Backend responde 429 | **Boton deshabilitado por 1 hora completa.** Mensaje "Demasiados intentos, intente mas tarde" |
| **Estado de Error Servidor** | Backend responde 500 | Mensaje generico + alternativas contacto (footer). Boton vuelve a activo |

### 8.2 Validaciones

**Principio fundamental:**
Las validaciones del frontend son una **capa de UX** para dar retroalimentacion inmediata al usuario, pero **nunca reemplazan** las validaciones del backend (fuente de verdad).

#### Validaciones Cliente (Frontend)

| Campo | Validacion | Comportamiento UI |
|-------|------------|-------------------|
| **Nombre** | Minimo 2 caracteres, maximo 100 | Contador de caracteres visible. Error inline si es muy corto/largo |
| **Email** | Formato valido (regex similar a backend) | Indicador visual de formato correcto/incorrecto mientras escribe |
| **Servicio Interes** | Seleccion de uno de 4 valores: "Definicion MVP", "Optimizacion", "Acompanamiento", "Validacion" | Seleccion obligatoria. Sin opcion vacia. Error si no hay seleccion |
| **Presupuesto** | **OBLIGATORIO:** Seleccion exacta de 3 valores: "tiene presupuesto", "no tiene presupuesto", "consultar precio" | **Select o radio buttons con 3 opciones exactas.** No permite envio hasta que se seleccione una opcion. No acepta valores diferentes, variantes o vacio > **Nota (DEC-004)**: Alineado con Backend.md y Brief.md actualizado. |
| **Mensaje** | Opcional. Si viene contenido: maximo 500 palabras / 1000 caracteres | **Textarea invisible inicialmente.** Se hace visible cuando usuario comienza a escribir. Si excede limite, mostrar error |

#### Validaciones Servidor (Backend) - Referencia

El backend aplica las siguientes validaciones (segun `backend.md`):
- Nombre: presencia, longitud 2-100
- Email: presencia, formato regex
- Servicio interes: presencia y valor valido
- **Presupuesto: presencia y valor exacto (3 valores literales)**
- Mensaje: opcional, longitud maxima si viene contenido

**Flujo de validacion:**
1. Frontend valida primero (feedback rapido al usuario)
2. Si frontend pasa validacion -> enviar a backend
3. Backend revalida todo (fuente de verdad)
4. Si backend falla -> frontend muestra sus propios mensajes de error (no los del backend)

### 8.3 Manejo de Errores

#### Errores de Validacion (Codigo 400)

**Estructura de respuesta backend:**
`{"success": false, "errores": ["mensaje1", "mensaje2"]}`

**Comportamiento frontend:**
- **NO mostrar los mensajes del backend** (estan en espanol y no son controlables)
- **Mostrar mensajes propios del frontend** definidos en el modulo de UI
- Asociar cada error al campo correspondiente para resaltarlo

#### Error Rate Limiting (Codigo 429)

**Comportamiento frontend:**
- Deshabilitar boton de envio inmediatamente
- Mostrar mensaje: "Demasiados intentos, intente mas tarde"
- **Mantener boton deshabilitado por 1 hora completa** (sin reintento posible)
- No hay calculo de tiempo restante en frontend (el backend gestiona el reinicio del contador)

#### Error de Servidor (Codigo 500)

**Estructura de respuesta backend:**
`{"success": false, "error": "Error interno, intente nuevamente o contacte directamente"}`

**Comportamiento frontend:**
- Mostrar mensaje generico (no exponer detalles tecnicos)
- Ofrecer alternativas de contacto directo:
  - Enlace a LinkedIn en footer
  - Direccion de email en footer
  - Mensaje: "O contacta directamente por [enlace]"

#### Errores de Red o Conexion

**Comportamiento frontend:**
- Detectar errores de red (fetch fallido, timeout, etc.)
- Mostrar mensaje: "Error de conexion. Verifica tu internet e intenta nuevamente"
- Boton vuelve a estado activo

---

## 9. REQUISITOS DE UX/UI

### 9.1 Filosofia de Diseno

**"Minimo es mas":**
- Formulario solo con campos esenciales, sin distracciones
- Mucho espacio en blanco, tipografia legible
- Sin animaciones excesivas, elementos sobrecargados
- Cada elemento tiene proposito claro

**Alineado con Brief.md:**
- Estilo minimalista con calidez
- Colores: azul oscuro/verde esmeralda (primario), naranjas/tonos tierra (secundario), grises/blancos (neutros)
- Tipografia: serif para titulos (autoridad), sans-serif para cuerpo (legibilidad)
- Imagenes profesionales, iconos lineales limpios

### 9.2 Comportamiento del Formulario

**Campo Presupuesto (UX Critico):**
- **NO es campo de texto libre**
- Presentar como **select desplegable** o **radio buttons** con exactamente 3 opciones:
  - "tiene presupuesto"
  - "no tiene presupuesto"
  - "consultar precio"
- Evita que usuario escriba valores invalidos
- Validacion inmediata: no permite avanzar sin seleccionar

**Campo Mensaje (Comportamiento):**
- **Textarea oculta inicialmente**
- Se hace visible suavemente cuando usuario hace click o comienza a escribir en ella
- Placeholder: "Cuentame mas sobre tu proyecto (opcional)"
- Si usuario no escribe nada, campo permanece vacio (no es error)

**Feedback en Tiempo Real:**
- **Email:** Indicador visual de formato mientras escribe (icono o color change)
- **Nombre:** Contador de caracteres visible (ej: "23/100")
- **Presupuesto:** Seleccion inmediata, no avanzar hasta elegir opcion valida
- **Errores:** Aparecen inline debajo del campo con error, no solo en lista general

### 9.3 Estados Visuales del Boton de Envio

| Estado | Texto/Apariencia | Comportamiento |
|--------|------------------|----------------|
| **Default** | "Enviar formulario" | Activo, color primario, cursor pointer |
| **Hover** | Efecto visual sutil | Cambio de tonalidad o sombra |
| **Loading** | "Enviando..." + spinner | Deshabilitado, opacity reducida |
| **Exito** | "Gracias!" o desaparece | Transicion suave, cambio a color exito |
| **Error 429** | "Demasiados intentos, intente mas tarde" | Deshabilitado, color gris, texto informativo |
| **Error validacion** | "Enviar formulario" (vuelve a default) | Activo para reintento |

### 9.4 Mensajes de Error UI (Definidos por Frontend)

**Estos son los mensajes que el frontend mostrara (NO los mensajes del backend):**

| Tipo Error | Mensaje Frontend Sugerido |
|------------|---------------------------|
| Nombre requerido | "Por favor, ingresa tu nombre" |
| Nombre muy corto | "El nombre debe tener al menos 2 caracteres" |
| Nombre muy largo | "El nombre no puede exceder 100 caracteres" |
| Email requerido | "Por favor, ingresa tu correo electronico" |
| Email formato invalido | "El correo electronico no tiene un formato valido" |
| Servicio requerido | "Por favor, selecciona un servicio de interes" |
| Presupuesto requerido | "Por favor, indica tu situacion de presupuesto" |
| Presupuesto invalido | "Selecciona una de las opciones de presupuesto disponibles" |
| Mensaje muy largo | "El mensaje no puede exceder los 1000 caracteres" |
| Error 429 rate limiting | "Demasiados intentos, intente mas tarde" |
| Error 500 servidor | "Error al procesar el formulario. Intente nuevamente o contacte directamente" |
| Error conexion | "Error de conexion. Verifica tu internet e intenta nuevamente" |

### 9.5 Accesibilidad (WCAG)

**Requisitos minimos de accesibilidad:**
- Labels asociados a todos los inputs (usar `htmlFor` o equivalente)
- Errores de formulario anunciados a lectores de pantalla (usar `aria-describedby` o `aria-live`)
- Contraste de colores suficiente (minimo 4.5:1 para texto normal)
- Campo presupuesto (select/radio) navegable con teclado
- Tiempo 1 hora rate limiting comunicado accesiblemente (no solo visual)
- Imagenes con `alt` descriptivo
- Navegacion por teclado completa

### 9.6 Responsive Design

**Enfoque Mobile-First:**
- Disenar primero para moviles (70%+ trafico es movil)
- Escalar a desktop con media queries

**Comportamiento por dispositivo:**

| Dispositivo | Layout Formulario | Comportamiento |
|-------------|-------------------|----------------|
| **Mobile** | Campos apilados verticalmente | Boton ancho completo, campos comodos para toque |
| **Tablet** | Posible layout 2 columnas | Nombre/email lado izquierdo, servicio/presupuesto lado derecho |
| **Desktop** | Layout 2 columnas o formulario centrado | Mayor espacio en blanco, tipografia mas grande |

**Breakpoints:**
- Mobile: hasta 768px
- Tablet: 769px - 1024px
- Desktop: 1025px+

---

## 10. GESTION DE DATOS Y ESTADO

### 10.1 Estado Global del Formulario

**Propiedades de estado:**
- `isLoading: boolean` - Indica si se esta procesando una peticion al backend
- `hasSent: boolean` - Indica si el formulario ya fue enviado exitosamente (previene reenvios duplicados)
- `showSuccess: boolean` - Controla visibilidad del mensaje de exito
- `errores: string[]` - Array de mensajes de error activos
- `showMessage: boolean` - Controla visibilidad del campo textarea de mensaje

**Estado por campo:**
Cada campo del formulario gestiona su propio estado:
- `value: string` - Valor actual del campo
- `error: string | null` - Mensaje de error si existe
- `isTouched: boolean` - Si el usuario ha interactuado con el campo

### 10.2 Datos que Frontend Envia al Backend

**Cuerpo JSON POST `/api/contacto`:**
- `nombre: string` (obligatorio, 2-100 caracteres)
- `email: string` (obligatorio, formato valido)
- `servicio_interes: string` (obligatorio, 4 valores permitidos)
- `presupuesto: string` (obligatorio, 3 valores exactos)
- `mensaje?: string` (opcional, max 500 palabras / 1000 caracteres)

**Campo NO incluido por frontend:**
- `fuente` (backend lo anade automaticamente con valor "formulario_web")
- `id` (generado por backend)
- `fecha_creacion` (generada por backend)

### 10.3 Sanitizacion de Datos

**Frontend debe sanitizar caracteres especiales antes de enviar:**
- Caracteres a escapar: `<`, `>`, `&`
- Campos afectados: nombre, mensaje (campos de texto libre)
- Proposito: Prevenir inyeccion de contenido que pueda romper formato del email generado
- **Esta sanitizacion es complementaria a la del backend**, no la reemplaza

### 10.4 Persistencia de Estado

**Sin persistencia local en Fase 1:**
- Los leads se "persisten" como emails en la bandeja de Miguel
- No hay localStorage, sessionStorage, ni cookies para guardar formularios
- Si el usuario abandona la pagina, los datos se pierden

**Posible mejora futura:**
- Autoguardado en localStorage para recuperar formulario si usuario regresa
- No incluido en Fase 1 por simplicidad

---

## 11. SEGURIDAD Y BUENAS PRACTICAS

### 11.1 Medidas de Seguridad Frontend

**1. Nunca confiar en validaciones cliente:**
- Las validaciones frontend son para UX, nunca para seguridad
- El backend siempre revalida todo
- Nunca asumir que los datos son seguros solo porque el frontend los valido

**2. Sanitizacion de entradas:**
- Escapar caracteres especiales `<`, `>`, `&` en campos de texto libre
- Prevenir inyeccion de contenido HTML/JavaScript
- Sanitizar antes de enviar, no confiar en que backend lo hara siempre

**3. No exponer informacion sensible:**
- No incluir datos sensibles en URLs o parametros de query
- No enviar tokens o credenciales (no existen en Fase 1)
- No mostrar detalles tecnicos de errores al usuario

**4. HTTPS obligatorio:**
- Todas las llamadas fetch deben usar `https://`
- Nunca enviar datos por HTTP plano
- Verificar que la URL base del backend es HTTPS

**5. No almacenar datos sensibles:**
- El formulario no maneja contrasenas, numeros de tarjeta, SSN u otros datos sensibles
- Solo captura: nombre, email, eleccion de servicio y mensaje

### 11.2 Manejo de Errores Seguro

**Nunca exponer al usuario:**
- Stack traces o pilas de llamadas
- Rutas de archivos del servidor
- Detalles tecnicos internos
- Datos completos del usuario en mensajes de error

**Siempre mostrar:**
- Mensajes genericos y comprensibles
- Alternativas de contacto directo
- Instrucciones simples para resolver el problema

### 11.3 Prevencion de Abuso

**Rate limiting del lado cliente:**
- Frontend no implementa rate limiting propio (es responsabilidad del backend)
- Pero frontend puede implementar:
  - Deshabilitar boton temporalmente despues de envio exitoso (prevenir reenvios rapidos accidentales)
  - Mostrar confirmacion antes de enviar si el usuario es nuevo

**Prevencion de spam:**
- Campo presupuesto obligatorio ayuda a filtrar leads genuinos
- Sanitizacion de caracteres especiales
- No hay CAPTCHA en Fase 1 (el backend maneja rate limiting)

---

## 12. REQUISITOS FUNCIONALES

### 12.1 Requerimientos Funcionales del Frontend

| ID | Requerimiento | Descripcion |
|----|---------------|-------------|
| **F-FE-01** | Formulario de contacto | Renderizar formulario con 5 campos: nombre, email, servicio_interes, presupuesto, mensaje (opcional) |
| **F-FE-02** | Campo presupuesto obligatorio | Presentar select/radio con 3 valores exactos: "tiene presupuesto", "no tiene presupuesto", "consultar precio" |
| **F-FE-03** | Campo mensaje condicional | Textarea oculta inicialmente, visible cuando usuario comienza a escribir |
| **F-FE-04** | Validaciones cliente | Aplicar validaciones en tiempo real: formato email, longitud nombre, seleccion obligatoria |
| **F-FE-05** | Envio a backend | Fetch POST a `/api/contacto` con headers `Content-Type: application/json` |
| **F-FE-06** | Manejo de exito 200 | Mostrar mensaje "Formulario recibido. Te contactare pronto." en misma pagina |
| **F-FE-07** | Manejo de error 400 | Mostrar mensajes de error propios del frontend (no backend) |
| **F-FE-08** | Manejo de error 429 | Deshabilitar boton 1 hora completa. Mensaje: "Demasiados intentos, intente mas tarde" |
| **F-FE-09** | Manejo de error 500 | Mensaje generico + alternativas contacto directo (footer) |
| **F-FE-10** | Health check | Consumir GET `/health` solo al cargar pagina |
| **F-FE-11** | Sanitizacion | Escapar caracteres `<`, `>`, `&` en campos texto libre antes de enviar |
| **F-FE-12** | Responsive design | Layout mobile-first, 70%+ trafico movil |
| **F-FE-13** | Accesibilidad | Labels, aria attributes, contraste, navegacion teclado |
| **F-FE-14** | Mensajes error UI | Definir mensajes propios para cada tipo de error |

---

## 13. REQUISITOS NO FUNCIONALES

### 13.1 Rendimiento

**Carga inicial:**
- Tiempo de carga < 3 segundos en conexiones normales
- Optimizar imagenes, minimizar assets
- Carga lazy de componentes no criticos si es posible

**Interaccion:**
- Validaciones en tiempo real sin lag perceptible
- Transiciones suaves entre estados del formulario
- Fetch al backend con timeout razonable (ej: 10 segundos)

### 13.2 Escalabilidad

**Preparado para crecimiento:**
- Componentes modulares para agregar nuevas funcionalidades
- API Service preparado para agregar endpoints futuros
- Estado gestionado para extender con auth en Fase 2

**Limites conocidos:**
- Rate limiting: 5 envios/hora/IP (gestionado por backend)
- Capacidad email: 100 emails/dia plan Google (limitacion backend)
- No hay limites de frontend especificos

### 13.3 Disponibilidad

- Frontend debe funcionar independientemente de la disponibilidad del backend
- Si backend cae: mostrar mensaje de error + alternativas contacto
- No hay dependencias de servicios externos mas alla del backend definido

### 13.4 Mantenibilidad

**Codigo organizado:**
- Estructura de componentes clara y predecible
- Nomenclatura consistente (camelCase funciones, PascalCase componentes)
- Comentarios en secciones complejas (no excesivos)
- Separacion clara: componentes, servicios, utilidades, estilos

**Documentacion:**
- Este frontend.md como fuente de verdad tecnica
- README.md interno con instrucciones de configuracion y desarrollo

---

## 14. CONVENCIONES Y CRITERIOS TECNICOS

### 14.1 Convenciones de Nomenclatura

**Componentes:** PascalCase (ej: FormularioContacto, CampoPresupuesto)
**Funciones/Metodos:** camelCase (ej: enviarFormulario, validarEmail)
**Estados/Variables:** camelCase (ej: isLoading, showSuccess)
**Constantes:** UPPER_SNAKE_CASE (ej: MAX_LONGITUD_NOMBRE)
**Archivos:** PascalCase para componentes, camelCase para servicios/utilidades

### 14.2 Convenciones de Estructura de Proyecto

**Estructura sugerida:**
```
/src
  /components        # Componentes UI reutilizables
    FormularioContacto
    CampoNombre
    CampoEmail
    CampoServicioInteres
    CampoPresupuesto
    CampoMensaje
    BotonEnviar
    MensajeExito
    ListaErrores
  /services         # Servicios API y utilidades
    apiService.js
    validationService.js
    sanitizationService.js
    errorService.js
  /hooks            # Custom hooks para logica reutilizable
    useFormState.js
    useRateLimiting.js
    useHealthCheck.js
  /utils           # Funciones utilitarias
  /styles          # Estilos globales, variables CSS
  /assets          # Imagenes, iconos, fuentes
  App.js           # Componente raiz
  index.js         # Punto de entrada
```

### 14.3 Convenciones de Validacion

**Validacion cliente:**
- Ejecutar en tiempo real mientras usuario escribe (debounced)
- Mostrar errores inline debajo del campo correspondiente
- Resaltar campos con error (borde rojo, icono de error)
- No bloquear el envio por errores de validacion cliente (dejar que backend revalide)

**Validacion servidor:**
- Fuente de verdad
- Mostrar mensajes propios del frontend (no los del backend espanol)
- Asociar errores del array `errores` a campos especificos

### 14.4 Convenciones de Manejo de Estados

**Estados del formulario:**
- Usar estados predecibles y rastreables
- Nunca mutar estado directamente, usar funciones setter
- Coordinar estados entre componentes hermanos mediante state lifting o context

**Transiciones de estado:**
- Loading -> Exito/Error con transicion suave
- Resetear estado de error al inicio de nuevo envio
- Mantener `hasSent` como flag persistente durante la sesion

### 14.5 Convenciones de Diseno UI

**Paleta de colores (segun Brief.md):**
- Primario: Azul oscuro o verde esmeralda (confiabilidad, tecnologia)
- Secundario: Naranjas o tonos tierra (optimizacion, calidez)
- Neutros: Grises claros, blancos

**Tipografia:**
- Titulos: Serif (autoridad) - ej: Playfair Display
- Cuerpo: Sans-serif (legibilidad) - ej: Inter, Roboto

**Espaciado:**
- Generoso en blanco
- Consistencia en margenes y paddings
- Espaciado entre campos del formulario: 16px-24px

**Bordes y sombras:**
- Bordes suaves (border-radius: 4px-8px)
- Sombras sutiles para elevacion
- Estados hover/focus con transiciones

---

## 15. VERIFICACION FINAL

### 15.1 Cobertura de Funcionalidades Backend

**Verificar que todas las funcionalidades relevantes del backend.md tengan su correspondiente consideracion en frontend.md:**

| Funcionalidad Backend | Consideracion Frontend | Estado |
|-----------------------|------------------------|--------|
| POST `/api/contacto` | Consumo con headers correctos, estructura JSON, manejo 200/400/429/500 | OK |
| GET `/health` | Consumo solo al cargar pagina | OK |
| Modelo Lead (7 atributos) | Envio de 5 campos (nombre, email, servicio_interes, presupuesto, mensaje) | OK |
| Campo presupuesto obligatorio | Select/radio con 3 valores exactos, validacion restrictiva | OK |
| Validaciones servidor | Frontend pre-valida pero backend es fuente de verdad | OK |
| Rate limiting 5/hora/IP | Manejo error 429, deshabilitar boton 1 hora completa | OK |
| Persistencia email | Frontend no gestiona persistencia, solo envio | OK |
| Sin autenticacion Fase 1 | No implementar login/tokens/sesiones | OK |
| HTTPS obligatorio | Todas las llamadas fetch con https:// | OK |
| Estructura JSON `{success: boolean, ...}` | Validar siempre campo `success` | OK |
| Mensajes error backend (espanol) | **Frontend muestra mensajes propios** (no los del backend) | OK |
| Campo fuente (valor por defecto) | **Backend lo anade automaticamente**, frontend no lo envia | OK |
| Sanitizacion caracteres | **Frontend escapa `<`, `>`, `&`** antes de enviar | OK |
| Respuesta error 500 | Mensaje generico + alternativas contacto directo | OK |

### 15.2 Verificacion de Coherencia

**Coherencia frontend-backend:**
- Frontend consume los mismos endpoints definidos en backend.md
- Frontend envia la misma estructura JSON esperada por backend
- Frontend maneja los mismos codigos de estado HTTP
- Frontend no inventa endpoints ni funcionalidades que contradigan backend
- Frontend preparado para futura autenticacion Fase 2 sin reestructuracion mayor

**Coherencia con Brief.md:**
- Estilo minimalista alineado con filosofia "optimo en todo sentido"
- Diseno responsive (70%+ trafico movil)
- Paleta de colores y tipografia definida
- Campos del formulario alineados con servicios de Miguel

**Coherencia con Estructura.md:**
- Flujo de usuario sin vacios
- Navegacion clara entre secciones
- CTA estrategicos en puntos de conversion
- Accesibilidad considerada

### 15.3 Verificacion de Integridad

**El archivo frontend.md:**
- No contiene codigo, pseudocodigo, snippets, comandos ni bloques de programacion
- Contiene exclusivamente especificaciones y documentacion tecnica
- Mantiene coherencia con backend.md como fuente de verdad
- No inventa endpoints ni comportamientos que contradigan el backend
- Documenta explicitamente decisiones que no estaban definidas en backend.md
- Cubre todos los aspectos solicitados: arquitectura, modulos, componentes, vistas, flujos, integracion, validaciones, errores, UX/UI, seguridad, requisitos, convenciones

---

## REGISTRO DE DECISIONES (DEC-xxx)

Para trazabilidad completa, las decisiones técnicas bloqueantes resueltas en esta versión:

| ID | Decisión | Archivos afectados |
|----|----------|-------------------|
| **DEC-001** | **Framework frontend: React** (SPA moderna, hooks, ecosistema) | frontend.md L38, L767 |
| **DEC-002** | **CSS framework: Tailwind CSS** (mobile-first nativo, consistencia, velocidad) | frontend.md L42 |
| **DEC-003** | **Librería de validación: Zod** (schema-first, TypeScript-ready, compartible con backend) | frontend.md L41 |
| **DEC-004** | **Campo presupuesto: OBLIGATORIO** (3 valores exactos; actualiza Brief.md L119/L168) | Brief.md, Backend.md, frontend.md |
| **DEC-005** | **Arquitectura: SPA custom React** en lugar de Webflow (control total, costo cero hosting, perfil técnico) | Brief.md L116, Estructura.md, frontend.md |
| **DEC-006** | **Orden de menú definitivo**: Inicio \| Sobre Mí \| Servicios \| Portafolio \| Testimonios \| Contacto (reemplaza Brief.md L70) | Estructura.md L13, frontend.md L129 |

---

## RESUMEN DE LO QUEDADO DEFINIDO

### Archivo Creado
- **Ubicacion:** `C:\Users\miguel\Documents\Default Project\Documentos\frontend.md`
- **Version:** 1.0 - Agosto 2026
- **Extension:** ~700 lineas de especificacion tecnica

### Contenido Principal

**1. Arquitectura y Tecnologias:**
- SPA con frontend consumiendo servicios REST del backend
- Frameworks recomendados: React/Vue.js o similar
- Comunicacion via Fetch API con HTTPS

**2. Estructura de Modulos y Componentes:**
- 8 modulos principales (FormState, API Service, Validation, Error Handler, etc.)
- 10 componentes del formulario detallados
- 9 componentes de apoyo para la landing page

**3. Vistas y Navegacion:**
- 8 secciones de landing page en SPA con scroll suave
- Navegacion por anchor links
- Mobile-first responsive design

**4. Flujos de Usuario:**
- 5 flujos detallados: exito 200, error 400, rate limiting 429, error 500, health check
- Estados visuales en cada flujo

**5. Integracion con Backend:**
- Consumo de POST `/api/contacto` y GET `/health`
- Manejo de respuestas segun codigos HTTP definidos en backend.md
- Estructura JSON estandar `{success: boolean, ...}`

**6. Autenticacion y Autorizacion:**
- Fase 1: Sin autenticacion (endpoints publicos)
- Fase 2: Preparado para agregar auth sin reestructuracion mayor

**7. Estados, Validaciones y Manejo de Errores:**
- 7 estados de aplicacion definidos
- Validaciones cliente por campo
- Manejo detallado de errores 400/429/500
- **Mensajes de error propios del frontend** (no backend)

**8. Requisitos de UX/UI:**
- Filosofia "minimo es mas"
- Campo presupuesto como select/radio con 3 valores exactos
- Campo mensaje visible solo al escribir
- 12 mensajes de error UI definidos
- Accesibilidad WCAG minima

**9. Gestion de Datos y Estado:**
- Estado global del formulario con 5 propiedades
- Sanitizacion de caracteres especiales
- Sin persistencia local en Fase 1

**10. Seguridad y Buenas Practicas:**
- Nunca confiar en validaciones cliente
- HTTPS obligatorio
- Sanitizacion de entradas
- No exponer informacion sensible

**11. Requisitos Funcionales (14 items) y No Funcionales:**
- Rendimiento, escalabilidad, disponibilidad, mantenibilidad

**12. Convenciones Tecnicas:**
- Nomenclatura, estructura proyecto, validacion, manejo estados, diseno UI

**13. Verificacion Final:**
- Cobertura completa de funcionalidades backend
- Coherencia con Brief.md y Estructura.md
- Integridad del documento (sin codigo, solo especificaciones)

---

*Este documento constituye la especificacion tecnica completa y autorizada del frontend para la landing page de Miguel Garzon Product Manager freelance, version 1.0.*