# Brief para Claude — Implementar portafolio en GitHub

Copia este documento (o el repo completo) y pide a Claude que **implemente, commitee y deje listo GitHub Pages** en el repositorio indicado.

---

## Repositorio

- **URL:** `https://github.com/bigkokoz/Taller-Interfaces-UAI`
- **Rama:** `main`
- **Dueño del contenido:** Jose (diseñador, taller Interfaces UAI)
- **Idioma UI:** español (`lang="es"`)

---

## Objetivo del producto

Sitio estático de **portafolio público** donde:

1. **Jose** agrega proyectos **sin editar HTML a mano** (solo JSON + imágenes en carpetas, vía GitHub web o Desktop).
2. **Cualquier visitante** ve una home ordenada y puede abrir el detalle de cada proyecto con imagen, texto y contacto.

No construir login ni panel complejo en el MVP. La “subida” = **commit de archivos al repo** con una estructura fija.

---

## Estado actual (problemas a resolver)

| Problema | Detalle |
|----------|---------|
| Datos duplicados | Tarjetas en HTML hardcodeadas + array `projects` en JS — textos no coinciden |
| Imagen rota | `proyecto1-interfaz-impresora.png` referenciada pero **no existe** en el repo |
| Placeholders | Texto “portafolio de ejemplo” y “PLACEHOLDER” en thumbnails |
| Archivos muertos | `sketch.js`, `style.css`, `p5.min.js` local no usados por `index.html` |
| UX | Solo el link “Ver proyecto” abre el modal; la card parece clickeable por hover |
| Deep links | Hash `#proyecto-N` funciona; mejor evolucionar a slug legible si Pages lo permite |

---

## Enfoque MVP (obligatorio)

**Static site + una fuente de verdad JSON + carpeta de assets.**

- Sin backend, sin auth, sin CMS externo en v1.
- **Fuente de verdad:** `data/projects.json`
- **Imágenes:** `assets/projects/<slug>/thumb.webp` (o `.png`) y `hero.webp`
- **Sitio:** un solo `index.html` (o `index.html` + `app.js` + `styles.css` separados si queda más limpio) que **lee el JSON y renderiza** grid + modal (o página de detalle).

Opcional v1.1: GitHub Action que valide JSON al push (no bloqueante si complica).

---

## Modelo de datos (`data/projects.json`)

Array ordenado por campo `order` (número ascendente). Solo proyectos con `"published": true` se muestran al público.

```json
{
  "site": {
    "title": "Mis Proyectos",
    "ownerName": "Jose",
    "tagline": "Diseño de interfaces · Taller UAI",
    "contactEmail": "",
    "contactLinkedIn": ""
  },
  "projects": [
    {
      "id": "interfaz-impresora",
      "order": 1,
      "published": true,
      "title": "Interfaz impresora",
      "subtitle": "Diagrama de interacción",
      "cardExcerpt": "Una línea para la tarjeta (máx. ~120 caracteres).",
      "description": "Párrafo largo para el detalle del modal.",
      "year": 2026,
      "tags": ["interfaces", "diagrama"],
      "role": "Diseño de interacción",
      "thumb": "assets/projects/interfaz-impresora/thumb.png",
      "hero": "assets/projects/interfaz-impresora/hero.png",
      "links": []
    }
  ]
}
```

### Reglas de negocio

- `id` = slug URL-safe (minúsculas, guiones). URL detalle: `#proyecto/<id>` o `?p=<id>` si hash complica OG.
- `cardExcerpt` y `description` **obligatorios** si `published: true`.
- `thumb` y `hero` obligatorios si hay imagen; si faltan archivos, mostrar fallback de color + iniciales del título (no texto “PLACEHOLDER”).
- Proyectos con `published: false` no aparecen en la grid (Jose puede dejar borradores en el JSON).

---

## Estructura de carpetas objetivo

```text
/
├── index.html          # shell mínima
├── css/
│   └── styles.css      # estilos (migrar desde inline actual)
├── js/
│   └── app.js          # fetch JSON, render grid, modal, routing
├── data/
│   └── projects.json
├── assets/
│   └── projects/
│       └── interfaz-impresora/
│           ├── thumb.png
│           └── hero.png
├── docs/               # no tocar salvo README
└── README.md           # instrucciones para Jose: cómo agregar un proyecto
```

Eliminar o dejar de referenciar: `sketch.js`, `p5.min.js` local, CDN p5 si ya no hace falta.

---

## Lógica de la aplicación (front)

### 1. Carga

1. Al cargar la página, `fetch('data/projects.json')`.
2. Si falla el fetch, mostrar mensaje amigable (“No se pudieron cargar los proyectos”) — no pantalla en blanco.
3. Filtrar `projects` donde `published === true`.
4. Ordenar por `order`.

### 2. Home

1. Renderizar header desde `site` (título, tagline, sin texto de “ejemplo/plantilla”).
2. Renderizar grid de cards desde JSON:
   - Imagen = `thumb`
   - Título = `title`
   - Texto = `cardExcerpt`
   - CTA = “Ver proyecto →”
3. **Toda la card es clickeable** (mismo handler que el CTA).
4. Hover: elevación suave (mantener estética dark actual).

### 3. Detalle (modal)

1. Al abrir proyecto `id`:
   - Mostrar overlay modal.
   - Hero = `hero` (o `thumb` si no hay hero).
   - Subtitle, title, description, tags, year, role.
   - Botones: cerrar (✕), “← Volver”, Escape, clic fuera.
2. Si hay `links[]` ( `{ "label": "...", "url": "https://..." }` ), listar enlaces externos.
3. Footer del modal: CTA contacto si `site.contactEmail` o `site.contactLinkedIn` están definidos.

### 4. Routing

1. Al abrir: `history.pushState` con hash `#proyecto/<id>`.
2. Al cerrar: limpiar hash.
3. `popstate` + carga inicial: si hash coincide, abrir ese proyecto.
4. Validar `id`; si no existe, volver a home sin error críptico.

### 5. Accesibilidad mínima

- Modal: `role="dialog"`, `aria-modal="true"`, foco al abrir en botón cerrar o título.
- Imágenes: `alt` = `title` o `subtitle`.

---

## Flujo de Jose (documentar en README)

Pasos que Jose debe poder seguir **sin código**:

1. En GitHub: **Add file** → subir imagen(es) a `assets/projects/<nuevo-id>/`.
2. Editar `data/projects.json`: copiar un bloque de proyecto, cambiar campos, poner `order` y `published: true`.
3. Commit a `main`.
4. Esperar GitHub Pages (1–2 min) y abrir la URL pública.

Incluir en README una **plantilla JSON** de proyecto vacío y checklist (id único, rutas de imagen, excerpt vs description).

---

## GitHub Pages

1. Añadir instrucciones en README: Settings → Pages → Source **Deploy from branch** → `main` → `/ (root)`.
2. Verificar que rutas relativas funcionen (`data/projects.json`, assets).
3. URL esperada: `https://bigkokoz.github.io/Taller-Interfaces-UAI/`

---

## Contenido semilla (migración)

Migrar el Proyecto 1 real desde el HTML actual:

- **title / subtitle / description** del modal actual (impresora Bambu Lab X1c).
- Crear entradas placeholder para Proyecto 2 y 3 con **`published: false`** hasta que Jose tenga contenido (no mostrar placeholders al visitante).

Si no existe el PNG del diagrama:

- Dejar ruta `assets/projects/interfaz-impresora/hero.png` documentada en README.
- Usar fallback visual hasta que Jose suba el archivo (no link roto sin alt).

---

## Criterios de aceptación (Definition of Done)

Claude debe considerar terminado cuando:

- [ ] Solo `data/projects.json` define proyectos visibles; no hay textos de proyecto duplicados en HTML.
- [ ] Visitante ve 1+ proyecto publicado con tarjeta y modal coherentes (mismo título/excerpt/description logic).
- [ ] Imagen hero/thumb carga o fallback elegante (nunca icono roto sin mensaje).
- [ ] Card entera abre detalle; hash `#proyecto/<id>` funciona al compartir y al recargar.
- [ ] Header sin copy de “portafolio de ejemplo”.
- [ ] README en español: cómo agregar proyecto + activar Pages.
- [ ] Repo limpio: sin p5/sketch muertos salvo que se justifiquen.
- [ ] Cambios commiteados en `main` con mensaje claro.

---

## Fuera de alcance (no hacer en esta entrega)

- Panel web con login
- Subida drag-and-drop en runtime
- Base de datos, Supabase, Firebase
- CMS Sanity/Contentful
- Tests E2E automatizados (opcional manual en README)

---

## Prompt sugerido para Claude

Pega esto junto con el repo:

```text
Lee docs/BRIEF-CLAUDE-implementacion-github.md y docs/journey-map-sistema-portafolio.md.
Implementa el MVP en https://github.com/bigkokoz/Taller-Interfaces-UAI (rama main):
JSON único, render dinámico, estilos ordenados, README para Jose, GitHub Pages.
Cumple todos los criterios de aceptación del brief. Commitea y pushea.
Si falta hero.png de interfaz-impresora, usa fallback y documenta en README cómo subirlo.
```

---

## Notas de diseño (mantener)

- Tema oscuro actual: fondo `#0f1115`, cards `#1a1d24`, acento `#6c8cff`.
- Tipografía: Segoe UI / system sans.
- Grid responsive `auto-fit minmax(280px, 1fr)`, max-width ~1100px.
- Modal max-width ~900px, scroll interno si el contenido es largo.

Jose puede ajustar copy en JSON después; la prioridad es **lógica correcta y sitio presentable para un tercero**.
