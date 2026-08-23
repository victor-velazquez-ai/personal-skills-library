---
name: website-design
description: Punto de entrada único para cualquier trabajo de diseño visual web (landings, componentes, redesigns, audits, dashboards, sitios). Cuestionario largo adaptativo (~20 preguntas) que mapea las respuestas al combo correcto de skills (taste base + Emil polish transversal + operadores correctivos + builder de output). Reemplaza los routers manuales (taste-router, vercel-taste-router) y los from-copy intakes (vercel-from-copy, ghl-from-copy) — todas borradas o absorbidas el 2026-05-27. Triggers — `/website-design`, "quiero diseñar una web", "monta una landing", "hazme un componente", "redesign", "no sé por dónde empezar con esta web".
version: 1.2.0
user-invocable: true
argument-hint: "[brief opcional o tipo de proyecto]"
---

# /website-design — Orquestador maestro de diseño web

Este skill es el ÚNICO entry point que el usuario debe necesitar para arrancar cualquier trabajo de diseño visual. NO contiene código de diseño propio — orquesta las skills modulares que ya existen (impeccable, emil-design-eng, ui-ux-pro-max, las tastes, los operadores y los builders) y se asegura de invocar la combinación correcta.

**Principio rector**: cada skill invocada se ejecuta a pleno potencial (lectura íntegra de su SKILL.md). `/design` solo orquesta, no resume ni dilute.

---

## FASE 0 — Detección de input

Antes de preguntar nada, inspeccionar lo que el usuario ya ha aportado en el mensaje:

| Detectado | Implica |
|---|---|
| PDF adjunto | Análisis visual previo (ruta similar a `vercel-landing-builder` FASE 0) |
| URL de landing | Análisis con Playwright/Firecrawl antes del cuestionario |
| Copy en texto largo | Input "from-copy", saltar Bloque 2.3 |
| Referencia visual (imagen, screenshot) | Capturar paleta, layout antes del cuestionario estético |
| Solo un brief textual corto o nada | Cuestionario completo desde cero |

Si hay PDF, URL o imagen: hacer análisis breve antes del Bloque 1 y referirse a él en las preguntas posteriores. Si solo hay brief textual o nada: pasar directo al Bloque 1.

---

## FASE 1 — Cuestionario largo adaptativo

Hacer las preguntas en orden por bloques. Saltar preguntas cuya respuesta ya esté clara del input o del CLAUDE.md del cliente. **Usar AskUserQuestion** para que las opciones sean clicables.

> Regla: nunca preguntar dos cosas en una sola pregunta. Una pregunta = una decisión.

### BLOQUE 1 — Tipo y entrada

**P1. ¿Qué quieres construir?**
- Landing Vercel (Next.js o Astro, deploy real)
- Landing GHL (bloques Custom Code dentro de GHL)
- Componente o página suelta (artifact HTML/React)
- Redesign de algo existente
- Audit / crítica de algo existente
- Solo brief / pensar el diseño antes de implementar

**P1b. ¿Dónde va a VIVIR esta página?** — solo si P1 = Landing Vercel. **Nunca la asumas: pregúntala siempre.**
- **Dominio propio en Vercel, sin iframe** (`go.cliente.com`, `evento.cliente.com`…) — DEFAULT desde ago-2026
- **Embebida dentro de una página de GHL** (`<iframe>` apuntando a `*.vercel.app`) — patrón legacy

> 🚨 **Esta pregunta manda sobre medio contrato técnico de la skill.** No es una preferencia de despliegue: cambia qué está permitido en el CSS, qué animaciones puedes usar, quién dispara el píxel, cómo llega la atribución y si los endpoints pueden validar su origen. Ver **"Los dos contratos"** justo debajo del cuestionario y aplicarlo ENTERO. Mezclar los dos contratos es la fuente número uno de bugs silenciosos en producción.
>
> **Por qué el default cambió (auditoría 6-ago-2026, medido en producción):** `vercel.app` está en la Public Suffix List, así que un iframe ahí **jamás** podrá compartir la cookie `_fbp` con el dominio del cliente — no es un bug, es imposible por diseño. Además el iframe cuesta **+334/+368 ms de mediana** hasta contenido visible (p90 +569/+738 ms, y más de 1 s con DNS frío, que es el caso de todo clic de anuncio), duplica las sesiones de Clarity si el tag está arriba y abajo, e impide validar `Origin` en los endpoints propios. Un subdominio del cliente resuelve las cuatro cosas con **un CNAME aditivo**: no toca nameservers, no sustituye ningún registro y se revierte borrando una línea.
>
> **Cuándo sigue estando justificado el iframe:** cuando la página tiene que convivir con cabecera/pie de GHL, cuando el cliente no da acceso a su DNS, o cuando es un parche temporal sobre un funnel con anuncios vivos que no se quiere tocar.

**P2. ¿Input que traes?** (saltar si ya detectado en FASE 0)
- PDF de referencia
- URL de referencia
- Copy en texto
- Imagen / screenshot
- Nada — partimos del brief

### BLOQUE 2 — Cliente y contexto

**P3. ¿Para qué cliente o marca?**
Listar las marcas conocidas del CLAUDE.md global:
- Propiedades Dharma
- Conchita Martínez (Tao / sexualidad)
- Canto Chamánico (Julio Martín Segura)
- Sergio Nature
- Brotherhood Funnels
- Otro / cliente externo / proyecto personal

> Si responde una marca conocida: leer `_context/Brand Voice.md`, `_context/ICP.md`, `_context/Products.md` de esa marca antes de seguir. Las respuestas a Bloque 3 se calibran con esa info.

**P4. ¿Qué tipo de pieza es?**
- Lanzamiento de oferta (alta intensidad, urgencia, conversión directa)
- Evergreen (debe envejecer bien)
- Página de evento o webinar
- Checkout / order form
- Página de replay
- Bridge / pre-sell entre ad y VSL
- Otro

**P5. ¿Hay Campaign Log previo del cliente que revisar?**
Buscar `_context/Campaign Log.md` y, si existe, leer aprendizajes recientes.

### BLOQUE 3 — Estética (núcleo del cuestionario)

NO preguntar "elige minimalist o brutalist". Preguntar EJES concretos. El mapeo a tastes se hace al final del bloque.

**P6. Volumen visual**
- Silencioso (mucho aire, tipografía limpia, color contenido)
- Equilibrado
- Ruidoso (contraste alto, color saturado, energía visual)

**P7. Densidad**
- Aireado / editorial (espacios amplios, una idea por sección)
- Equilibrado
- Denso / informativo (mucha info por viewport)

**P8. Tipografía**
- Editorial contrastada (serif + sans-serif, jerarquía marcada, tamaños grandes)
- Tecnológica uniforme (sans-serif, modular scale estricta)
- Humanista (serif o slab cálido, ritmo orgánico)
- Brutalist (mono, sin antialiasing, escala extrema)

**P9. Movimiento**
- Estático (cero animación, solo hover sutil)
- Sutil (fades on mount, hover micro)
- Presente (scroll reveals, stagger, parallax suave)
- Exuberante (GSAP ScrollTrigger, pin, scrub, shaders)

**P10. Temperatura cromática**
- Cálida (ámbar, tierras, dorados, naranjas)
- Neutra (blancos, grises, negros)
- Fría (azules, verdes oscuros, púrpuras)
- Saturada policroma (varios colores, contraste alto)

**P11. Forma / lenguaje visual**
- Orgánico / humano (curvas, fotografía de personas, blobs)
- Geométrico (rectángulos, grids estrictos, sin curvas)
- Mixto

**P12. Riesgo estético**
- Convencional (probado, no asusta al cliente conservador)
- Atrevido (rompe expectativas dentro del nicho)
- Extremo (impacto máximo, posible polarización)

**P13. ¿Referencias visuales concretas?**
Si el usuario tiene URLs o nombres (ej: "como linear.app", "como el sitio de Emil Kowalski", "como una revista impresa"), pedirlas. Si no, saltar.

### BLOQUE 4 — Producción técnica

Solo aplica si el tipo elegido en P1 es Landing Vercel o Landing GHL.

**P14. ¿Stack preferente?** (si Vercel)
- Next.js App Router + Tailwind + Framer Motion (default)
- Astro static + Tailwind v4 + GSAP (mejor para landings puras sin estado)
- Sin preferencia, decide tú

**P15. ¿Form GHL involucrado?**
- Sí, formulario propio integrado en la landing (Direct Fetch + reCAPTCHA Enterprise)
- Sí, modal de captura ANTES del checkout (patrón modal → GHL → redirect)
- Sí, iframe nativo de GHL embebido
- No

> Si sí: pedir formId, locationId, reCAPTCHA site key. Si no los tiene → enseñar el patrón de descubrimiento empírico con Playwright (en `vercel-landing-builder` "Cómo descubrir esos datos empíricamente").
>
> **Si el form captura TELÉFONO, aplicar la Regla crítica 16 (normalización) sin preguntar.** Es obligatoria siempre que el contacto posterior vaya por WhatsApp/SMS: sin ella entran leads con el prefijo de país duplicado que el CRM acepta y a los que nunca se podrá escribir.

**P16. ¿Tracking?**
- Facebook Pixel (pedir ID)
- GTM (pedir container ID)
- Ambos
- Ninguno

> Si hay tracking en checkout o evento, invocar `/ghl-event-configuration` después del deploy.
> ⚠️ **Si hay Pixel + un form propio embebido en iframe GHL** (P15 = form propio o modal pre-checkout): el form DEBE reenviar la atribución del padre al iframe (`fbclid`/`utm`/`_fbp`/`_fbc`/`parent_url`) y mandar el `eventData` completo, o la Conversions API falla con "No attribution data available" en TODOS los forms. No es opcional. El patrón está en `vercel-landing-builder` → _Patrones de producción → "Atribución cross-domain"_ (léelo entero al construir). Verificar siempre en GHL: contacto → "Latest attribution details" con Session Source = Social media.

**P17. ¿Modo de embed en GHL?** (SOLO si P1b = embebida en GHL. Si va a dominio propio, **saltar esta pregunta entera**: no hay embed que configurar)
- Scroll interno del iframe (recomendado si la landing es la página completa, soporta sticky/pin/scrub)
- Bridge bidireccional parent↔iframe (solo si hay header/footer GHL alrededor — NO soporta pinning)

> 🚨 **Si esta landing va a recibir TRÁFICO DE PAGO, el snippet de embed se copia LITERAL del bloque canónico de `vercel-landing-builder` (_Embed de scroll interno_ / _FASE 4_). Nunca escrito a mano ni "adaptado".** Toda variante manual vista en producción ha perdido alguna pieza silenciosa. La que más se pierde: el fallback `if(!fbc&&p.get('fbclid'))fbc='fb.1.'+Date.now()+'.'+p.get('fbclid')`, que construye el `_fbc` cuando la cookie todavía no existe — es decir, **en la primera visita de todo el tráfico frío**.
>
> **Verificación obligatoria tras publicar, con las cookies `_fbp`/`_fbc` BORRADAS** (sin borrarlas no estás probando el caso real): cargar `?fbclid=TEST123&utm_source=TEST` y confirmar que el `src` del iframe contiene `_fbc=fb.1.<ts>.TEST123`. **El test de solo UTMs no vale: lo pasa un embed roto**, porque los UTM viajan en `location.search` y se reenvían solos.
>
> Caso real (C7D Canto Chamánico, 20-jul-2026): embed manual sin ese fallback. Pasaba el test de UTMs. Perdía el click id de todo el tráfico frío, degradando el `{{contact.fbclid}}` que alimenta los workflows de Purchase → **coste en atribución de ventas**, detectado solo tras una tarde de diagnóstico. Detalle en `vercel-landing-builder` → Regla 9.

**P18. ¿Analítica de comportamiento en la landing? (Microsoft Clarity — heatmaps + grabaciones)**
- Sí, con Clarity (pedir el **Project ID**, ej. `wv84mpwkvo`)
- Sí, pero no sé el Project ID (ayudar a localizarlo: clarity.microsoft.com → Settings → Setup/Overview; o crear proyecto nuevo)
- No

> ⚠️ **El tag SIEMPRE va en el código de la landing Vercel**, nunca solo en el host de GHL: si solo está en el host, Clarity graba el cascarón vacío del iframe y se pierde TODO el comportamiento (scroll, CTA, modal, form). Ver Regla crítica 11.
>
> 🚨 **NUNCA el mismo Project ID arriba y abajo — corregido 6-ago-2026, la instrucción anterior era justo la equivocada.** Decía "usar el mismo que el dominio host para intentar unir sesiones". Las sesiones **no se unen**: se DUPLICAN. Verificado en producción en dos clientes (NE `xcr9590yj8`, C7D `x8r9mcg7gg`): cada visita generaba dos registros —el cascarón, que parece rebote instantáneo, y la página real—, inflando los pageviews y ensuciando el engagement. Y el scroll del padre marcaba **99,95%** cuando el real era **10,39%**: un error de 9,6× en la dirección que te hace decidir al revés.
>
> **Regla correcta, según dónde viva la página:**
> - **Dominio propio (P1b = Vercel):** un solo tag, en la página. No hay nada más. Punto.
> - **Embebida en GHL:** el tag va en la landing Vercel, y en GHL **se quita del nivel funnel** (Settings → Tracking Code → Head) y se pone **a nivel de página** solo en las páginas GHL nativas que NO tienen gemela en Vercel (checkout, OTO, portal). Si se quita de golpe del nivel funnel, esas páginas dejan de medirse.
> - **Un proyecto de Clarity por funnel, nunca compartido.** Mezclar funnels en un mismo proyecto es lo que rompió el dashboard de NE: el extractor pedía el total del proyecto —los 10 pasos más dos apps de Vercel— y lo usaba como "visitas a la landing". Salían 2,73 visitas por cada clic pagado. Si de todas formas comparten proyecto, el extractor DEBE pedir `&dimension1=URL` y quedarse solo con la URL de la landing.

### BLOQUE 5 — Restricciones

**P19. Timeline**
- Necesito empezar a ver algo HOY
- Tengo unos días
- No hay prisa, calidad sobre velocidad

**P20. Assets**
- Tengo todas las imágenes y copy listos
- Tengo copy pero las imágenes las generamos
- Falta copy también, lo escribimos juntos
- Solo idea, partimos de cero

> ⚠️ Si el usuario aporta imágenes o vídeos propios, aplicar la **Regla crítica 12 (aviso de peso)** al recibirlos: comprobar peso/formato y avisar + ofrecer comprimir (con su OK) antes de embeberlos o deployarlos.

**P21. ¿Algo prohibido específico?**
Pregunta libre. Ejemplo: el cliente odia gradientes, el avatar rechaza un tono "ventoso", la marca prohíbe rojos.

---

## LOS DOS CONTRATOS (decide P1b — leer ENTERO antes de escribir una línea)

Una landing Vercel se construye de dos maneras **incompatibles** según dónde vaya a vivir. La mitad de las
reglas de este documento solo existen por culpa del iframe: aplicarlas cuando la página va a dominio propio
es tirar calidad a la basura, y no aplicarlas cuando va embebida rompe la página en silencio.

**Elige una columna entera. Nunca mezcles.**

| | **A · Dominio propio** (`evento.cliente.com`) | **B · Embebida en GHL** (`<iframe>` a `*.vercel.app`) |
|---|---|---|
| Alturas de viewport (`100dvh`, `min-h-screen`, `h-screen`) | **Permitido** | **PROHIBIDO** — bucle infinito: el padre fija altura → dvh crece → scrollHeight crece → hasta 300.000 px y página invisible. Usar px fijos |
| `position: sticky`, pinning, scrub | **Permitido** | **PROHIBIDO** — el iframe no scrollea de verdad, la sección se va y deja hueco |
| ScrollTrigger, IntersectionObserver, `useInView` | **Permitido** | **PROHIBIDO** — el padre scrollea, el hijo no: todo se queda en `opacity:0`. Animar al montar con stagger |
| `HeightSync` (postMessage de altura) | **No hace falta** | **OBLIGATORIO** — sin él el iframe queda a altura fija |
| `Content-Security-Policy: frame-ancestors` | **Mejor NO ponerlo** (que nadie te embeba) | **OBLIGATORIO** `frame-ancestors *`. Nunca `X-Frame-Options: ALLOWALL` (no es valor estándar → `ERR_BLOCKED_BY_RESPONSE`) |
| Modales | `position: fixed` normal | `position: absolute` recalculado en cada `vl-scroll` — `fixed` se ancla al iframe, no al viewport |
| Píxel de Meta | **En la propia página.** Base + PageView + ViewContent | Lo dispara el padre vía Funnel Events. En el hijo `fbq` no existe |
| Cookies `_fbp` / `_fbc` | **Primera persona**, `domain=.cliente.com`, compartidas con todos los subdominios | **Imposible.** `vercel.app` está en la Public Suffix List: el navegador RECHAZA `domain=.vercel.app` |
| Atribución (`fbclid`, `utm_*`) | Llega sola en la URL | Passthrough TOTAL de `location.search` + `_fbp`/`_fbc` de cookie + `parent_url`. **Copiar el bloque canónico LITERAL** de `vercel-landing-builder` |
| La carrera del `_fbp` | **No existe** | **Existe siempre y es determinista.** El píxel se inyecta al `window.load`; el iframe arranca con el HTML. En visita fría el iframe pide ANTES de que la cookie exista. Medido: 105, 149, 166, 179, 188, 502, 556 ms. **El fallback `if(!fbc&&p.get('fbclid'))fbc='fb.1.'+Date.now()+'.'+p.get('fbclid')` no es opcional** |
| `Origin` en endpoints propios (`/api/*`) | **Viaja siempre** → validación de origen real | **No viaja en los GET** (el fetch es same-origin desde el iframe). Solo `Referer`, y algunos webviews lo quitan. En POST sí viaja |
| Clarity | Un tag, una vez | Un tag en el hijo. En GHL, a nivel de PÁGINA y solo donde no hay gemela. Nunca el mismo ID arriba y abajo |
| Velocidad | Referencia | **+334/+368 ms de mediana**, p90 +569/+738, >1 s con DNS frío |
| PageSpeed / Lighthouse | Miden bien | Dan `NO_FCP` "sin puntuación" sobre el wrapper. **Medir siempre contra la URL de Vercel** |
| `noindex` | No, la quieres indexable | **Sí**, en la URL `*.vercel.app`: si no, Google la indexa y entra gente saltándose el padre — sin píxel, sin `_fbp`, sin medir |

### Extras obligatorios de la columna A (dominio propio)

1. **DNS:** un **CNAME aditivo** en el panel actual del cliente hacia el valor que muestre la domain card del
   proyecto en Vercel. **Ya NO es el histórico `cname.vercel-dns.com`**: es per-proyecto (tipo
   `d1d4fc829fe7bc7c.vercel-dns-017.com`) y hay que copiarlo de ahí. No se tocan nameservers, no se sustituye
   ningún registro, se revierte borrando la línea. **El apex y los comodines sí son otra cosa** (el apex
   sustituye el registro A que hoy sirve la web; los comodines exigen nameservers en Vercel): por eso el
   default es un SUBDOMINIO.
2. **Dominios con eñe u otros IDN:** darlos de alta en **punycode** en el registrador y en Vercel
   (`evento.niñosempresarios.com` → `evento.xn--niosempresarios-zqb.com`).
3. **Mismo dominio raíz que el checkout**, para que `_fbp` se comparta. Si el checkout de GHL vive en
   `pay.cliente.com` y la landing en `evento.cliente.com`, la cookie viaja sola.
4. **Arrastrar la query a los enlaces salientes** hacia GHL (checkout, gracias): cuatro líneas que pegan
   `location.search` al final del link. Sustituye al puente del iframe y es mucho más robusto.
5. **Plan Pro de Vercel.** El plan Hobby es **solo uso no comercial**: *"All commercial usage of the platform
   requires either a Pro or Enterprise plan"*. Con landings de cliente y ads corriendo, la sanción no es
   lentitud: es proyecto deshabilitado.
6. **Validar `Origin` en todos los endpoints propios desde el primer día.** Es la ventaja que la columna B no
   puede darte.

### Regla de migración (de B a A)

Si la página YA existe embebida y hay anuncios vivos, **no se saca del wrapper**: se cambia el `src` del
iframe de `*.vercel.app` a `go.cliente.com` y punto. Eso mata la carrera, el passthrough y la cookie
imposible **sin tocar ni una URL de anuncio, ni un email ya enviado, ni un custom value, ni el checkout**.
Sacarla del wrapper del todo obliga a reimplementar píxel, Clarity y deduplicación, y a cambiar el destino
de cada anuncio activo: 8-16 h por cliente frente a 1-2 h. **Nunca desplegar el día de un evento en vivo.**

---

## FASE 2 — Mapeo de respuestas a skills

Con las respuestas en mano, decidir QUÉ skills se ejecutan y en qué orden. **No preguntar al usuario qué tastes invocar — decidirlo tú con esta tabla.**

### Tabla de mapeo a TASTE BASE

Mirar combinación de P6 (volumen), P7 (densidad), P8 (tipografía), P10 (temperatura), P11 (forma) y P12 (riesgo).

| Patrón de respuestas | Taste base |
|---|---|
| Silencioso + aireado + editorial/humanista + neutro/cálido + cualquier riesgo | `/minimalist-ui` |
| Equilibrado + equilibrado/denso + editorial/humanista + cálido/neutro + convencional/atrevido | `/high-end-visual-design` |
| Ruidoso + denso + brutalist + neutro saturado + atrevido/extremo | `/industrial-brutalist-ui` |
| Ruidoso + denso + tecnológica + cualquier temperatura + atrevido/extremo + movimiento Presente/Exuberante | `/gpt-taste` |
| Cualquier respuesta que no encaje limpiamente arriba | `/design-taste-frontend` (default robusto) |
| Cliente quiere DESIGN.md generado para Google Stitch | `/stitch-design-taste` (caso especial, raro) |

> Si dos tastes empatan: priorizar la que mejor case con el cliente y su `_context/Brand Voice.md`.

### EMIL ES TRANSVERSAL — siempre se aplica

Independientemente de la taste base elegida, **`/emil-design-eng` se invoca SIEMPRE como filtro de polish, criterio de animación y micro-detalle**. No compite con minimalist/brutalist/etc — opera sobre el resultado de ellos.

Punto de invocación:
- Después de aplicar la taste base, antes del HTML intermedio.
- Otra vez después de las skills correctivas, antes del polish final.

### Operadores correctivos (Familia A Impeccable)

Después del primer pase + critique + audit, invocar los operadores recomendados. Mapeo directo (igual que `vercel-landing-builder` FASE 1 Paso 6):

| Diagnóstico de /critique o /audit | Operador |
|---|---|
| Bland, genérico, demasiado seguro | `/bolder` |
| Agresivo, recargado | `/quieter` |
| Tipografía débil o genérica | `/typeset` |
| Paleta sin energía | `/colorize` |
| Grid monótono, composición débil | `/layout` |
| Demasiados elementos, falta foco | `/distill` |
| Microcopy confuso, CTA poco claro | `/clarify` |
| Problemas responsive | `/adapt` |
| Performance / bundle / loading | `/optimize` |
| Falta movimiento o sin criterio | `/animate` |
| Funcional pero olvidable | `/delight` |
| Cliente quiere impacto máximo | `/overdrive` |

Regla mantenida: nunca `/bolder` y `/quieter` en el mismo pass.

### Builder de output (Familia D)

Según P1 + P14 + P17:

| Tipo proyecto | Builder |
|---|---|
| Landing Vercel + input copy/brief/imagen | `vercel-landing-builder` FASE 0 Rama B (intake copy) → FASE 1 → ... |
| Landing Vercel + input PDF | `vercel-landing-builder` FASE 0 Rama A (análisis PDF) → FASE 1 → ... |
| Landing GHL + input copy/brief | `ghl-landing-builder` FASE 0 Rama B (intake copy) → FASE 1 → ... |
| Landing GHL + input PDF | `ghl-landing-builder` FASE 0 Rama A (análisis PDF) → FASE 1 → ... |
| Componente / artifact | `/impeccable craft` directamente |
| Redesign de existente | `/redesign-existing-projects` |
| Audit | `/critique` + `/audit` solos, sin builder |
| Solo brief | `/shape` solo |

---

## FASE 3 — Pipeline orquestado

Una vez decididas taste base + operadores + builder, ejecutar en este orden estricto:

1. **Contexto del cliente** — leer `_context/` si aplica (Brand Voice, ICP, Products, Campaign Log)
2. **`/shape`** — discovery breve para consolidar dirección (si no se ha hecho en el cuestionario, saltar)
3. **`/impeccable teach`** o lectura de `.impeccable.md` si existe
4. **`/ui-ux-pro-max`** — sistema de diseño completo (paleta OKLCH, par tipográfico, layout)
5. **Taste base elegida** — aplicar su SKILL.md íntegro
6. **`/emil-design-eng`** (pase 1) — criterio de animación, polish y micro-detalle aplicado sobre la taste base
7. **Builder ejecuta el HTML intermedio standalone** — `landing-preview.html`
8. **`/critique` + `/audit`** sobre el HTML intermedio
9. **Operadores correctivos** según diagnóstico
10. **`/emil-design-eng`** (pase 2) — pasada transversal final de pulido
11. **`/polish`** — alineación, spacing, consistencia
12. **CHECKPOINT 1** — mostrar `landing-preview.html` al usuario para aprobación
13. **Builder convierte a proyecto deployable** (Next.js, Astro, o bloques GHL)
14. **Deploy preview** + CHECKPOINT 2
15. **Deploy producción + entregables** (URL + GHL embed code + tracking + **tag de Clarity inyectado y verificado en la landing Vercel — Regla 11** + docs)

Si en cualquier punto el usuario pide cambios, volver al paso anterior y re-ejecutar solo lo afectado.

---

## Reglas críticas de output

1. **`/design` NUNCA ejecuta diseño por su cuenta.** Siempre delega en las skills modulares. Si te encuentras escribiendo HTML/CSS dentro de `/design`, has fallado.

2. **Lectura íntegra de cada SKILL.md invocado.** No resumir. No saltarse secciones. Cuando llames a `/vercel-landing-builder` para el builder, su SKILL.md de 69KB se carga entero.

3. **El cuestionario no es opcional.** Aunque el usuario diga "haz lo que quieras", hacer al menos el bloque mínimo: P1, **P1b**, P3, P6, P9, P14 (si aplica). Los demás pueden tener defaults inteligentes pero estos 6 son obligatorios.

3-bis. **P1b (dónde vive la página) no se infiere NUNCA, ni del cliente ni del historial.** Es la pregunta que decide el contrato técnico entero: alturas, animaciones, quién dispara el píxel, cómo llega la atribución, si los endpoints pueden validar su origen. Que un cliente tenga hoy 15 proyectos en iframe **no** significa que el siguiente lo sea — desde ago-2026 el default es el contrario. Preguntar siempre, y aplicar **una columna entera** de "Los dos contratos". Mezclarlas es la fuente número uno de bugs silenciosos: la página parece funcionar y pierde la atribución sin dar ni un error.

4. **Defaults inteligentes leyendo Obsidian.** Si P3 = marca conocida con `_context/Brand Voice.md`, muchas respuestas de Bloque 3 se pueden inferir. Decir al usuario: "Según el Brand Voice de [marca], asumo X, Y, Z. ¿Confirmas o cambias?" — no preguntar a ciegas lo ya documentado.

5. **Patrones de producción se preservan al 100%.** El form GHL Direct Fetch + reCAPTCHA + `eventData` completo, el **reenvío de atribución cross-domain** (`fbclid`/`utm`/`_fbp`/`_fbc`/`parent_url` del padre → iframe, obligatorio para que la CAPI atribuya — y **el snippet reenvía SOLO esa allowlist**: cualquier dato que una página del funnel pase a otra por URL para prefill/matcheo de contacto —`email`/`nombre`/`telefono` de registro → confirmación— viaja DENTRO de `parent_url`, no suelto al iframe; la página destino DEBE leerlo de `parent_url` o el form se envía con campos vacíos y GHL no casa el contacto), el **`country` explícito (ISO-2) en el submit** derivado del prefijo del teléfono (sin él GHL asigna el país-default de la subcuenta —ej. México— a TODOS los contactos, ignorando el prefijo), el modal de captura pre-checkout, el patrón de hero "texto izquierda + foto derecha con gradiente direccional", el CSS de checkout One-Step Order Form (styling por Custom CSS de alta especificidad — **NO MutationObserver**, causó page-freeze; y la **descripción del Order Bump con `white-space: pre-line`** + Enter en blanco entre bloques, o sale como un párrafo apelmazado — ver `ghl-landing-builder` BLOQUE 8), el bridge bidireccional iframe, el `--vh` anti-loop, y el **modelo de altura por página** (auto-altura vs `100svh` fija — no mezclar el listener `vl-height`), todo eso lo provee `vercel-landing-builder` o `ghl-landing-builder` en su FASE de builder. `/design` no los reescribe.

6. **Tastes raras solo si el cuestionario las pide.** `stitch-design-taste` y `gpt-taste` no se ofrecen por defecto. Solo se invocan si las respuestas a Bloque 3 las requieren explícitamente.

7. **Emil siempre, en dos puntos.** Una vez después de la taste base (criterio inicial), otra vez al final tras correctivos (filtro de pulido transversal). No saltarse ninguna de las dos.

8. **Nunca preguntar dos veces lo mismo.** Si el cuestionario ya consiguió la info, no volver a pedirla cuando se invoca la skill builder.

9. **Cuestionario en bloques, no de golpe.** Lanzar Bloque 1 → recibir respuestas → lanzar Bloque 2 → etc. Mantener el control de progreso visible para el usuario.

10. **Nunca emojis en código ni en output final.** Misma regla que el resto del ecosistema.

11. **Microsoft Clarity (y cualquier analítica de comportamiento) va EN EL CÓDIGO DE LA LANDING VERCEL, nunca solo en el dominio host.** Una landing Vercel embebida en GHL/dominio propio se sirve dentro de un `<iframe>` cross-origin. El Clarity del dominio host (ej. `sergionature.com`, GHL) solo ve el documento contenedor — que está VACÍO (`document.body.scrollHeight === 0`, solo el iframe a pantalla completa) — y NO captura nada de dentro: ni scroll real, ni clicks de CTA, ni el modal de captura, ni el form. Resultado: scroll depth ~99%, engagement y "0 fricción" son métricas FALSAS del cascarón, no de la landing. **[Error garrafal real: ECDV jun-2026 — toda la lectura de comportamiento de la sales page (`el-camino-de-vuelta`) fue basura hasta detectar el iframe con Playwright. No repetir.]** Al construir/desplegar una landing Vercel:
    - (a) PEDIR el Clarity Project ID en P18; usar el **mismo** ID que el dominio host para intentar coser las sesiones cross-domain.
    - (b) Inyectar el snippet oficial de Clarity en el `<head>` de la landing Vercel (Next.js: `next/script` `afterInteractive` en `app/layout.tsx`; Astro: `<head>` del layout).
    - (c) Si el funnel tiene varias landings en Vercel (ventas, registro, confirmación, gracias…), taggear TODAS — cada una es un punto ciego independiente.
    - (d) Activar en Clarity el tracking de iframes / cross-origin para el playback embebido.
    - (e) Verificar tras deploy: abrir una grabación y confirmar que registra scroll/clicks reales (no `scrollHeight 0`). El conteo de pageviews del host SÍ es válido para volumen/etapas; lo que se recupera con esto es TODO el comportamiento.
    - Lo inyecta el builder (`vercel-landing-builder`); `/design` garantiza que se pida el ID y se inserte. Snippet de instalación (sustituir `PROJECT_ID` por el ID real):

```html
<script type="text/javascript">
  (function(c,l,a,r,i,t,y){c[a]=c[a]||function(){(c[a].q=c[a].q||[]).push(arguments)};
  t=l.createElement(r);t.async=1;t.src="https://www.clarity.ms/tag/"+i;
  y=l.getElementsByTagName(r)[0];y.parentNode.insertBefore(t,y);})(window,document,"clarity","script","PROJECT_ID");
</script>
```

12. **Aviso de peso de assets — avisar y comprimir ANTES de embeber/deployar.** Cuando el usuario aporte media propia para la web (en P20, o en cualquier momento que pase un archivo), comprobar **peso y formato ANTES de usarlo**. Si supera el umbral, PARAR y avisar con el mensaje **"este archivo pesa demasiado"** + el peso concreto + ahorro estimado, proponer comprimir, y **esperar su visto bueno explícito** (NO comprimir sin OK). Umbrales orientativos: **vídeo > 2-3 MB**, **imagen > 300-400 KB**, o **cualquier PNG fotográfico** (debería ir en WebP/AVIF). Compresión: vídeo con ffmpeg (downscale a ~1.5× del tamaño real de display + CRF 27-28 + audio AAC 64-96k, mono si es voz); imagen a WebP/AVIF redimensionada al tamaño de uso. **Y nunca `autoplay`+`loop` en vídeo servido desde Vercel `/public`** (cada visita lo descarga en bucle): `preload="none"` + carga al click, o alojar el vídeo fuera de Vercel (Bunny / Cloudflare / GHL CDN `filesafe.space`, que soporta Range/206). **[Error garrafal real: ECDV jun-2026 — la sales page (`el-camino-de-vuelta`) servía 7 vídeos + 13 audios (~60 MB) desde Vercel `/public` con autoplay+loop; reventó el free tier (100 GB Fast Data Transfer) en mitad del lanzamiento. No repetir.]**

13. **Todo embed de landing Vercel en GHL DEBE ser full-bleed — nunca un `<iframe width:100%>` a secas.** GHL envuelve el bloque Custom Code en una Row con `max-width` (~1170px) + padding, así que un iframe `width:100%` ocupa solo el ancho de esa Row y queda centrado con márgenes a los lados (no llega a los bordes del viewport). GHL a menudo NO expone un ajuste de "Row full width" fiable, así que el breakout del contenedor es **obligatorio**. El embed que `/design` entrega DEBE usar uno de los dos patrones del builder (`vercel-landing-builder` → _Debug: iframe no ocupa el ancho completo_ y _Modelos de embed_):
    - **(a) Scroll interno — recomendado para página dedicada full-screen SIN bridge de altura** (no toca el código de la landing): `<style>#frame{position:fixed;inset:0;width:100%;height:100%;border:0;display:block}</style>` + `<iframe id="frame" data-base="…" allow="payment">` + el script de atribución (Regla 14) que le pone el `src`. `position:fixed` ignora por completo el contenedor de GHL y cubre el viewport entero → full-bleed por construcción; el iframe scrollea su propio contenido. Los anchors internos (`#planes`) y los botones `target="_top"` (redirect a Stripe) siguen funcionando.
    - **(b) Auto-altura — para página de contenido natural donde scrollea el host** (requiere HeightSync postMessage en la landing): wrapper `margin-left:calc(-50vw + 50%);margin-right:calc(-50vw + 50%);width:100vw;overflow:hidden;line-height:0` + iframe `width:1px;min-width:100%;scrolling="no"`. El `overflow:hidden` es obligatorio (`100vw` incluye la scrollbar → scroll horizontal sin él); el `1px/min-width` (NO `width:100%`) evita el bug de iOS Safari por el que el iframe se expande al ancho del contenido.
    `/design` GARANTIZA que el embed entregado sea full-bleed **aunque el embed se escriba a mano fuera del pipeline completo del builder**. [Error real: La Tribu jul-2026 — embed hand-written con `width:100%` naíf salió centrado con márgenes grises; el patrón full-bleed ya vivía en `vercel-landing-builder` pero se bypasseó al escribir el embed a mano. No repetir: coger SIEMPRE el embed del builder.]

14. **Todo iframe embebido en GHL lleva atribución passthrough — el `src` fijo está PROHIBIDO.** El anuncio aterriza en la página GHL (el padre) con `?utm_…&fbclid…`, pero el script de captura corre DENTRO del iframe, que es cross-origin y no puede leer la URL del padre: con `src` fijo, la landing ve `location.search` vacío y **todos los leads entran sin atribución, silenciosamente** (el form funciona, nada parece roto). El embed que `/design` entrega — para CUALQUIER página GHL que embeba una landing (registro, confirmación, gracias, VSL, checkout…), tenga form o no — DEBE ser el **bloque atómico del builder** (`vercel-landing-builder` → _Output del GHL Embed Code_ / _Embed de scroll interno_): iframe **sin `src`, con `data-base`** + script que construye el `src` con **passthrough TOTAL de `location.search`** (nunca whitelist — se queda vieja al añadir params tipo `adset_name`) + `_fbp`/`_fbc` de cookies + catch que carga `data-base` si algo falla. **Verificación obligatoria al publicar**: abrir la URL pública con `?utm_source=TEST123` y confirmar que el `src` del iframe lo contiene. [Error garrafal real: Niños Empresarios 10-jul-2026 — la página de registro llevaba un embed viejo con `src` fijo; la campaña `NEW1_EVGREEN` arrancó y **40 leads entraron sin NINGUNA atribución en 2 horas** (utm/fbclid/ad_id/adset_name todos vacíos). El patrón correcto ya vivía en la página /confirmacion del mismo funnel. No repetir: embed SIEMPRE del builder + verificación con `?utm_source=TEST123`.]

15. **Performance de serie — presupuesto <800 KB antes de interacción.** Toda landing que `/design` entregue nace con las prácticas de la **Regla 14 de `vercel-landing-builder` ("PERFORMANCE por defecto")**, no como optimización posterior: (a) third-parties pesados que solo se usan tras interacción (reCAPTCHA ~375 KB, Stripe, players, chats) cargan **LAZY** al primer toque del form, nunca en el `<head>` (con `preconnect` al origen); (b) **`vercel.json` con headers de caché SIEMPRE** — el default de Vercel para `/public` es `max-age=0` y re-descarga todo en cada visita (frames/fonts=immutable 1 año, imágenes=7d+swr); (c) logo/iconos en WebP al 2× del display + favicon pequeño propio + banderas `w40/*.png` de flagcdn (nunca sus `.svg`); (d) fuentes self-hosted en WOFF2; (e) secuencias de frames por oleadas (eager + idle con `fetchPriority=low`); (f) verificación al cerrar: peso inicial <800 KB, ningún third-party pesado sin usar en el arranque, `curl -I` a los assets confirmando el `Cache-Control`; (g) Google Fonts async (`media="print" onload` + noscript). **Medición: PSI/Lighthouse SIEMPRE contra la URL del CONTENIDO (Vercel), nunca contra el wrapper GHL** — el wrapper es un iframe cross-origin y da NO_FCP/"sin puntuación" aunque la página funcione perfectamente (Pingdom/GTmetrix sí valen sobre el wrapper porque miden red, no paint). [Caso real: NE 10-jul-2026 — rebote alto con campaña activa; el HAR mostró reCAPTCHA en el head = 56% del peso de la página, `/public` sin caché y 2 MB de frames descargando de golpe. La carga inicial pasó de ~2 MB a ~650 KB aplicando esto; Lighthouse móvil 75-79 con TBT <100 ms y CLS 0.]

16. **Todo campo de teléfono NORMALIZA antes de enviar — un número mal capturado es un lead mudo.** En funnels donde el contacto va por WhatsApp/SMS, el teléfono es el ancla real (grupo de WhatsApp, recordatorios, acceso). Un número mal escrito no rompe nada visible: el form envía 200, el lead entra en el CRM y **el fallo no aparece hasta días después**, cuando el envío rebota y ya no hay a quién preguntar. Todo form con selector de país (`+52 ▾` + input) que `/design` entregue DEBE normalizar en el submit, **auto-corrigiendo lo recuperable y bloqueando solo lo irrecuperable** (rechazar de más cuesta leads; aceptar de más cuesta leads mudos):
    - **(a) Prefijo de país duplicado — el fallo dominante.** El selector aporta `+52` y el padre escribe su número *con* el 52 → `+52 52 xxxxxxxxxx`. Quitar el prefijo repetido **solo si sobra longitud** (si el nacional ya cuadra, es un número real: no tocar).
    - **(b) México, el "1" histórico.** Hasta 2019 los móviles se marcaban `+52 1 <10 dígitos>`; ya no es oficial pero WhatsApp mantiene millones registrados así y **ambas formas son el MISMO teléfono**. Si quedan 11 dígitos y empiezan por `1`, quitarlo.
    - **(c) Validar la longitud NACIONAL por país, nunca la total.** MX/US/CO/AR = 10; ES/PE/CL/EC = 9. Para países sin longitud conocida, solo un rango de cordura (6-14) — nunca bloquear de más.
    - **(d) Errar del lado de aceptar en lo ambiguo.** Un número de longitud correcta que "parece raro" se acepta: bloquear por sospecha cuesta más leads de los que salva.
    - **(e) Mensaje accionable + foco al campo**: "Revisa tu número de WhatsApp: parece que le faltan o le sobran dígitos" (no un genérico "revisa los datos"), y `focus()` al input del teléfono.
    - **(f) Normalizar en los DOS sitios** si hay prefetch/lookup cacheado por teléfono: si la clave del prefetch se construye sin normalizar y la del submit sí, no coinciden y se paga la latencia entera igualmente.
    - **Verificación al cerrar**: probar el caso del prefijo duplicado y comprobar en el payload real (interceptando `fetch`) que sale con la longitud correcta. [Caso real: Niños Empresarios 25-jul-2026 — auditoría de 1.041 registros: **24 números con longitud imposible y el 100% de ellos falló** al enviarles WhatsApp; los de 14 dígitos empezaban TODOS por `5252` (prefijo duplicado). Eran leads que pagaron CPL, entraron al CRM y jamás pudieron ser contactados. Ojo con la regla naíf "siempre 12 dígitos": habría bloqueado los `+1` de EE.UU., que son 11 y válidos.]

---

17. **JAMÁS se fabrica un identificador de tracking. Si no está, se manda vacío.** El patrón prohibido, literal, porque ya se ha copiado a mano en TRES proyectos distintos:

    ```js
    // ❌ PROHIBIDO. Ni con Math.random, ni con un contador, ni con un hash del email.
    let fbp = getCookie('_fbp');
    if (!fbp) { fbp = 'fb.1.' + Date.now() + '.' + Math.floor(Math.random() * 1e10);
                document.cookie = '_fbp=' + fbp + ';max-age=7776000'; }
    ```

    Nació como parche del patrón iframe: la landing no podía leer la cookie del padre, así que "algo había que mandar". Y en columna B esa rama **salta en TODA primera visita desde un anuncio**, no en un caso raro. Daño medido en producción (Niños Empresarios, 6-ago-2026): **el 95,5% de los contactos de GHL tienen en `contact.fbp` un identificador que Meta nunca emitió**, y se encontró una cookie fabricada viva de 41 días. Hoy es inofensivo solo porque ese campo no se manda a la CAPI; el día que alguien lo mapee "para mejorar el tracking", empieza a inyectar basura a escala y **contamina el dataset entero**.

    Y el coste de NO fabricarlo es cero: el `_fbp` aporta **0 puntos porcentuales** de emparejamiento (PoPETs 2024, n=1.833, tráfico real de Meta Ads). Lo que se pierde inventándolo no es alcance: es la credibilidad del dato.

    **La única síntesis permitida es el `_fbc` a partir del `fbclid` de la URL**, porque el `fbclid` es un dato REAL que Meta te acaba de dar y el formato `fb.1.<ts>.<fbclid>` está documentado por Meta para generarlo fuera del navegador. Esa sí se queda.

    **Regla de revisión:** antes de cerrar cualquier landing, `grep -rn "Math.random" src/` y comprobar que ninguna aparición está cerca de `_fbp`, `_fbc`, `event_id`, `session_id` o `external_id`.

18. **Un guard no puede prohibir un valor que además es el correcto.** Al escribir defensas contra "pegar los IDs del funnel viejo", prohibir SOLO lo que de verdad es exclusivo de ese funnel. Caso real (semana-emprendimiento-juvenil, 6-ago-2026):

    ```js
    // ❌ Bloquea la configuración BUENA: el locationId es el mismo en los dos funnels
    if (FORM_ID === V1.formId || LOCATION_ID === V1.locationId) throw new Error(...)
    ```

    El `locationId` es la **sub-cuenta**, y dos funnels del mismo cliente viven en la misma. Lo único que jamás puede repetirse es el `formId`, que es lo que decide en qué workflows entra el registro. Con el guard mal puesto, rellenar la configuración correcta lanzaba error y el formulario quedaba muerto. **Antes de escribir un guard, preguntarse: ¿este valor es exclusivo del contexto prohibido, o es compartido?** Si es compartido, no va en la lista negra.

19. **No concluyas "esta página no tiene tracking" haciendo grep sobre el HTML.** En cualquier stack con islas (React, Vue, Svelte) o con inyección en tiempo de ejecución, el código de tracking **no está en el HTML servido**: vive en un bundle aparte o lo inyecta otro script. Casos reales del mismo día (6-ago-2026): (a) se dio por bueno que `semana-emprendimiento-juvenil` no enviaba nada a GHL — el envío estaba en `src/lib/ghl.ts`, dentro de un bundle de React; (b) se dio por muerto el gateway de Stape de NE porque `capig` no aparecía en ningún bundle — lo inyectaba `fbevents.js` por configuración del píxel en Meta, y respondía 200 en 8 de los 10 pasos del funnel.

    **Para afirmar que un emisor NO existe hace falta captura de red**, no un grep: cargar la página y mirar las peticiones reales. Un grep sobre el HTML solo sirve para afirmar que algo **SÍ** está, nunca para afirmar que falta.

---

## Anti-patrón: cuándo NO usar /design

- El usuario quiere ejecutar UNA skill suelta concreta y la nombra explícitamente (ej: "corre /audit sobre esto", "haz /polish a esta página"). Respetar la invocación directa.
- El usuario solo quiere análisis de competencia (`/landing-page-analysis`) o investigación.
- Trabajo no relacionado con diseño visual: copy puro, ads, automatización, marketing strategy.

---

## Skills que /design coordina (mapa completo)

**Familia A — Operadores Impeccable (verbos modulares):**
`/impeccable` (orquestador interno) · `/shape` · `/critique` · `/audit` · `/bolder` · `/quieter` · `/colorize` · `/typeset` · `/layout` · `/distill` · `/clarify` · `/adapt` · `/animate` · `/delight` · `/overdrive` · `/optimize` · `/polish` · `/redesign-existing-projects`

**Familia B — Tastes (estéticas, una se elige por proyecto):**
`/minimalist-ui` · `/high-end-visual-design` · `/industrial-brutalist-ui` · `/gpt-taste` · `/design-taste-frontend` · `/stitch-design-taste`

**Capa transversal (siempre activa):**
`/emil-design-eng`

**Sistema base:**
`/ui-ux-pro-max`

**Builders de output:**
`/vercel-landing-builder` · `/ghl-landing-builder`
(fusionados desde Fase 3 — antes había también `vercel-from-copy` y `ghl-from-copy` separados)

**Auxiliares:**
`/landing-page-analysis` · `/ghl-event-configuration`

---

## Notas de mantenimiento

- **v1.2.0 (6-ago-2026) — P1b y "Los dos contratos".** La skill asumía que toda landing Vercel iba embebida
  en un iframe de GHL. Ya no: se pregunta explícitamente y el default es dominio propio. Las reglas 11, 13,
  14 y la mitad del Bloque 4 son de la **columna B** y no se aplican en la A — aplicarlas ahí destruye
  calidad sin ganar nada (prohíben `sticky`, ScrollTrigger, `100dvh` y `position:fixed` por un problema que
  solo existe dentro de un iframe). Origen: auditoría de flota del 6-ago-2026 sobre 9 clientes; medidas y
  fuentes en `obsidian/decisiones/2026-08-06 Hosting de landings - Vercel directo vs iframe en GHL.md` y
  `obsidian/operaciones/2026-08-06 Auditoria de flota - tracking, pagos y seguridad.md`.
  **Al añadir una regla nueva, di SIEMPRE a qué columna pertenece.** Una regla sin columna se aplicará a las
  dos y romperá una.
- Cuando se añada una nueva taste, añadir entrada en la tabla de mapeo del Bloque 3.
- Cuando se añada un nuevo operador, añadir entrada en la tabla de correctivos.
- Cuando se añada un nuevo builder (ej: nuevo target de deploy), añadir entrada en la tabla de builders.
- El cuestionario NO debe crecer sin freno. Si una pregunta nueva se puede inferir de las existentes, no añadirla.
- Toda landing Vercel embebida en iframe es un punto ciego para el Clarity del dominio host (Regla 11): el tag de Clarity debe ir SIEMPRE en el código del Vercel. Si se añade un builder/target nuevo que sirva contenido en iframe, replicar esta exigencia.
- Todo campo de teléfono normaliza antes de enviar (Regla 16): el fallo no es visible al capturar — el form da 200 y el lead entra — sino días después, al rebotar el WhatsApp. Si se añade un builder/target nuevo con captura de teléfono, replicar esta exigencia. Implementación de referencia: `ninos-empresarios-clase/src/pages/index.astro` → `normalizarTel()`.
- Todo embed de landing en GHL DEBE ser full-bleed (Regla 13): nunca `<iframe width:100%>` a secas — GHL lo constriñe a la Row (~1170px). Usar el patrón `position:fixed;inset:0` (scroll interno) o el breakout `-50vw` (auto-altura). Aunque el embed se escriba a mano, cogerlo del builder. Si se añade un target nuevo servido en iframe, replicar esta exigencia.
- Todo iframe embebido en GHL lleva atribución passthrough (Regla 14): NUNCA `src` fijo — siempre `data-base` + script con passthrough total de `location.search` + `_fbp`/`_fbc`, y verificación al publicar con `?utm_source=TEST123`. Un `src` fijo pierde TODA la atribución en silencio (caso NE 10-jul-2026: 40 leads ciegos). Si se añade un target nuevo servido en iframe, replicar esta exigencia.
- Toda landing nace con performance de serie (Regla 15): third-parties pesados LAZY (reCAPTCHA nunca en el head), `vercel.json` con caché (el default de Vercel es `max-age=0`), imágenes UI en WebP al 2× del display, fuentes WOFF2, secuencias de frames por oleadas, y presupuesto <800 KB antes de interacción verificado al cerrar. Detalle operativo en `vercel-landing-builder` Regla 14. Si se añade un builder/target nuevo, replicar esta exigencia.
