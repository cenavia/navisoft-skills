# Web Layout Specialist — Guía de uso y prompts base

Skill para generar el esqueleto visual completo de una página web o aplicación a partir de un PRD y style-guidelines.

---

## Activación

La skill se activa automáticamente cuando el agente detecta intención de maquetación. También puedes invocarla directamente con cualquiera de los prompts de este documento.

---

## Prompts base

### 1. Inicio desde cero (PRD + style-guidelines adjuntos)

```
Actúa como Web Layout Specialist.

Aquí están los documentos del proyecto:

**PRD:**
[pega el contenido del PRD aquí]

**Style-guidelines (design tokens):**
[pega el contenido de las style-guidelines aquí]

Genera el esqueleto visual completo del proyecto respetando estrictamente ambos documentos.
```

---

### 2. Inicio desde cero (documentos como archivos)

```
Actúa como Web Layout Specialist.

Los documentos del proyecto están en:
- PRD: @[ruta/al/prd.md]
- Style-guidelines: @[ruta/al/style-guidelines.md]

Genera el esqueleto visual completo del proyecto respetando estrictamente ambos documentos.
```

---

### 3. Maquetar una sola vista/página

```
Actúa como Web Layout Specialist.

Documentos:
- PRD: @[ruta/al/prd.md]
- Style-guidelines: @[ruta/al/style-guidelines.md]

Genera únicamente el esqueleto de la vista "[nombre de la vista del PRD]",
incluyendo todos sus estados (vacío, carga, error, con datos).
```

---

### 4. Solo el archivo de tokens CSS

```
Actúa como Web Layout Specialist.

Style-guidelines: @[ruta/al/style-guidelines.md]

Genera únicamente el archivo styles/tokens.css con todas las variables CSS
extraídas literalmente de las style-guidelines. No inventes ningún valor.
```

---

### 5. Generar el README.md del proyecto

```
Actúa como Web Layout Specialist.

Documentos:
- PRD: @[ruta/al/prd.md]
- Style-guidelines: @[ruta/al/style-guidelines.md]

Los archivos HTML ya existen en [ruta/del/proyecto].
Genera el README.md del proyecto documentando: vistas, design tokens aplicados,
breakpoints activos, reglas de tokens e instrucciones de extensión.
```

---

### 6. Verificar una maquetación existente

```
Actúa como Web Layout Specialist.

Documentos fuente:
- PRD: @[ruta/al/prd.md]
- Style-guidelines: @[ruta/al/style-guidelines.md]

Archivos a verificar: @[ruta/del/proyecto]

Pasa los 10 checklists de la skill sobre los archivos existentes
y reporta qué pasa, qué falta y qué hay que corregir.
```

---

### 7. Añadir una nueva vista a un proyecto ya maquetado

```
Actúa como Web Layout Specialist.

El proyecto ya existe en @[ruta/del/proyecto].
Documentos fuente (no cambiar lo que ya existe):
- PRD: @[ruta/al/prd.md]
- Style-guidelines: @[ruta/al/style-guidelines.md]

Añade el esqueleto de la nueva vista "[nombre]" definida en la sección [X] del PRD,
usando los tokens ya establecidos en styles/tokens.css.
```

---

### 8. Convertir diseño estático a esqueleto responsive

```
Actúa como Web Layout Specialist.

Tengo este HTML estático (solo desktop):
@[ruta/al/archivo.html]

Style-guidelines con breakpoints: @[ruta/al/style-guidelines.md]

Conviértelo a responsive Mobile First usando únicamente
los breakpoints definidos en las style-guidelines.
```

---

### 9. Auditoría de accesibilidad sobre maquetación existente

```
Actúa como Web Layout Specialist.

Revisa los siguientes archivos y aplica el Checklist 7 (Accesibilidad WCAG AA)
de la skill:

@[ruta/al/archivo.html]

Reporta cada punto que no cumple e indica exactamente cómo corregirlo,
sin cambiar la estructura visual ni los design tokens.
```

---

### 10. Auditoría de performance sobre maquetación existente

```
Actúa como Web Layout Specialist.

Revisa los siguientes archivos y aplica el Checklist 6 (Performance)
de la skill:

@[ruta/al/archivo.html]

Reporta cada punto que no cumple e indica cómo corregirlo
sin agregar dependencias externas no especificadas en el PRD.
```

---

## Estructura de salida esperada

Cuando se ejecuta el flujo completo, el agente genera:

```
[nombre-del-proyecto]/
├── styles/
│   └── tokens.css          ← design tokens del style-guidelines
├── index.html              ← vista principal del PRD
├── [vista-2].html          ← una por cada vista del PRD
├── [vista-N].html
└── README.md               ← documentación generada
```

---

## Notas de uso

- Si el agente genera algo que no está en el PRD o en las style-guidelines, indícale:
  > "Eso no está en los documentos fuente. Elimínalo y ajústate estrictamente al PRD."

- Si necesitas que use un framework específico (React, Vue, etc.), inclúyelo en el prompt:
  > "El PRD indica que el proyecto usa [framework]. Adapta la estructura de carpetas a sus convenciones."

- Los prompts 6, 9 y 10 (auditorías) funcionan de forma independiente, sin necesidad de generar archivos nuevos.
