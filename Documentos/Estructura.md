# Estructura de Landing Page - Portafolio Dinámico

## Visión General
Este flujo está diseñado para guiar al visitante de forma natural desde el primer impacto hasta la acción de contacto, con transiciones claras y llamadas a la acción en puntos estratégicos.

---

## 1. HEADER (Navegación Superior)
**Siempre visible o fijo**

- Logo/Nombre Personal a la izquierda
- Menú de navegación a la derecha:
  - [Inicio] [Sobre Mí] [Servicios] [Portafolio] [Testimonios] [Contacto]
> **Nota (DEC-006)**: Este orden reemplaza al propuesto en Brief.md L70 ("Inicio | Servicios | Portafolio | Sobre Mí | Contacto").
- Iconos de redes sociales (derecha extrema): **LinkedIn, GitHub** (Twitter se mueve al Footer)
- **Fluye a:** Hero Section (inicio) o cualquier sección interna por menú directo

---

## 2. HERO SECTION (Encima de la doblez)
**Primer impacto - Objetivo: Capturar atención en 3 segundos**

Estructura de flujo:
```
[Foto profesional o imagen de fondo optimizada]
[Título principal: Valor propuesta en 8 palabras máximo]
[Subtítulo: Quién soy + qué hago en 2 líneas]
[Botón CTA principal: "Ver mis servicios" O "Conocer mi portafolio"]
[Botón CTA secundario: "Contactar"]
```

**Flujo siguiente:** Si hace clic en CTA principal → Sección "Sobre Mí". Si hace clic en secundario → Formulario de contacto o LinkedIn.

---

## 3. SOBRE MÍ / PERFIL
**Objetivo: Construir confianza y credibilidad**

Estructura:
```
[Foto de perfil]
[Nombre y título profesional]
[Historia breve: 3-4 líneas sobre trayectoria]
[Misión/Visión profesional en 1 frase]
[Iconos rápidos: Años de experiencia, clientes, industrias]
```

**Elementos de flujo:**
- "Leer más" botón → Sección Servicios
- Flecha hacia abajo implícita → Siguiente sección (Portafolio)
- Texto seleccionable para copiar/interesar

---

## 4. SERVICIOS / QUE HAGO
**Objetivo: Clarificar oferta de valor**

Estructura de tarjetas (3-4 columnas o lista vertical):
```
[Icono representativo]
[Título del servicio]
[Descripción corta: 1-2 líneas de beneficio]
[Indicador de nivel: "Principiante / Avanzado / Experto"]
```

**Flujo intencional:**
- Cada tarjeta tiene enlace interno o hover effect
- Hacer clic en una tarjeta → Lightbox o sección detallada de ese servicio
- Si no hace clic → Flecha abajo continúa al Portafolio
- **Call-to-Action estratégica:** "Ver casos de estudio" al final de las tarjetas

---

## 5. PORTAFOLIO / TRABAJOS (Centro de la página)
**Objetivo: Demostrar capacidad y estilo**

Estructura de grid/masonry:
```
[Proyecto 1] [Proyecto 2] [Proyecto 3] [Proyecto 4]
```

Cada tarjeta de proyecto contiene:
- Imagen/captura del proyecto
- Título del proyecto
- Tecnologías/Herramientas usadas
- Breve descripción del problema resuelto
- **Botón "Ver proyecto completo"** (link interno o externo)
- **Botón "Estudio de caso"** (abre modal o página nueva)

**Flujo crítico - SIN VACÍOS:**
- Proyecto completo debe tener: "Ver más" → Expande descripción
- Al final de cada proyecto: "Proyecto siguiente" o "Proyecto anterior" (navegación entre proyectos)
- Si está en el último proyecto: Botón "Volver al inicio" o "Ver testimonios"
- Si está en el primer proyecto: Botón "Conocer mis servicios"

**Patrón de navegación:**
```
Proyecto 1 → Proyecto 2 → Proyecto 3 → Proyecto 4 → [Volver inicio] o [Testimonios]
```

---

## 6. TESTIMONIOS / PROBABILIDADES
**Objetivo: Validación social y reducción de riesgo percibido**

Estructura:
```
[Foto cliente]
[Nombre cliente]
[Cargo/Empresa]
"Testimonio de 2-3 líneas sobre resultados obtenidos"
[Icono: ⭐⭐⭐⭐⭐]
```

**Flujo de salida:**
- Al final del testimonial: "¿Tienes un proyecto en mente?" → Llamada a acción contacto
- Flecha "Anterior" → Retorna a Portafolio
- "Ver más testimonios" → Grid expandido

---

## 7. CTA DE CONTACTO / CONSULTACIÓN
**Objetivo: Convertir interés en conversación**

Estructura:
```
[Título: "¿Listo para comenzar tu proyecto?"]
[Texto explicativo: "Agenda una consulta gratuita de 15 minutos"]
[Formulario simplificado: Nombre, Email, Servicio de interés, Presupuesto, Mensaje, Botón Enviar]
[Enlaces directos: Calendly/Agenda | WhatsApp | LinkedIn Mensaje]
```

**Flujo de convergencia:**
- Al enviar formulario → Página de "Gracias" con próximo paso
- Botones de atajo: "Ver mis precios" / "Ver mi blog" / "Portafolio completo"
- **Nunca un camino sin salida:** Siempre debe haber al menos 2 opciones después de enviar formulario

---

## 8. FOOTER (Pie de página)
**Objetivo: Recapitulación y accesibilidad**

Estructura en columnas:

| Enlaces rápidos: Inicio | Servicios | Portafolio | Contacto |
|-----------|-----------|-----------|-----------|
| ¿Quién soy? | Testimonios | Blog | Redes sociales: LinkedIn, Twitter, GitHub |
| Preguntas frecuentes | Términos | Política de privacidad | Suscribir newsletter |

**Elementos finales:**
- Copyright año actual
- Enlaces de afiliación si aplica
- Créditos de diseño si corresponde

**Flujo de salida del sitio:**
- Redes sociales externas (se abren en nueva pestaña)
- Enlaces a plataformas profesionales (LinkedIn en nueva pestaña)
- Nunca enlaces internos que saquen al usuario fuera sin opción de regreso

---

## DIAGRAMA DE FLUJO RESUMIDO (Texto)

```
[Usuario entra a la página]
            |
            v
      [Hero Section]
            |
    ----------------
    |              |
    v              v
[Sobre Mí]    [Servicios]
    |              |
    v              v
[Portafolio]  [Testimonios]
    |              |
    v              v
[CTA Contacto] [Volver al Inicio]
      \            /
       \          /
        [Footer]
            |
            v
[Usuario sale o navega a redes]
```

---

## PRINCIPIOS DE DISEÑO PARA FLUJO SIN VACÍOS

1. **Regla de las 3 acciones:** En cada sección, el usuario debe tener como mínimo 2 caminos claros:
   - Continuar al siguiente tema relevante
   - Retornar a sección anterior
   - Contactar / Convertir

2. **Navegación "Volver al Inicio":** Siempre disponible en la parte inferior de secciones largas (Portafolio, Testimonios).

3. **Indicador de posición:** El usuario debe saber en qué parte de la página está (breadcrumbs visuales o progreso de scroll).

4. **Sin pop-ups inesperados:** Si se usan modales, deben tener botón "Cerrar" y "X" visible.

5. **Mobile-first navigation:** En móviles, menú debe convertirse en "hamburger" y el flujo debe ser vertical sin horizontales complejas.

6. **Search/Filter en Portafolio:** Si hay más de 6 proyectos, permitir filtrar por categoría (Web, Mobile, Branding, etc.) para que no abrumen.

7. **CTAs contextualmente apropiados:** No ofrecer "Contactar" en cada minúscula sección - usarlo en Hero, Servicios, Portafolio y Footer.

---

## ARQUITECTURA TÉCNICA CONFIRMADA

Arquitectura confirmada: **SPA con scroll suave** (ver DEC-005).

- Todo en una sola página con anclajes (scroll suave)
- Navegación que actualiza URL por sección (#hero, #servicios, #portafolio)
- Mejor para: Cargar rápido, experiencia fluida, fácil mantenimiento

---

## ERRORES COMUNES A EVITAR (Vacíos en el flujo)

❌ **Portafolio sin título:** Proyecto con solo imagen, sin contexto de qué es
❌ **CTAs ambiguos:** Botones "Click aquí" sin indicar a dónde llevan
❌ **Formularios largos al inicio:** Solicitar muchos datos antes de haber mostrado valor
❌ **Menús desplegables complejos:** En móvil que ocultan opciones importantes
❌ **Falta de información de contacto:** Usuario se va sin saber cómo contactarte
❌ **Flechas "Anterior" rotas:** Que regresan al inicio en lugar de sección anterior
❌ **Testimonios sin autoría:** Frases sin nombre o empresa que valide la opinión

---

*Esta estructura ha sido diseñada basándose en principios de UX/UI para portafolios profesionales, con flujo de usuario optimizado, ausencia de caminos sin salida.*
