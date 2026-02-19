---
name: color-palette-designer
description: Diseña y documenta paletas de colores para aplicaciones web y móviles aplicando psicología del color, accesibilidad WCAG y la regla 60-30-10. Usar cuando el usuario solicite: (1) Crear o definir una paleta de colores para un producto digital, (2) Mapear colores a emociones y objetivos de marca a partir de un PRD o brief, (3) Generar tokens de diseño (primario, secundario, acento, neutrales, semánticos), (4) Validar contraste y accesibilidad de una paleta existente, (5) Documentar una paleta con su significado psicológico en tabla estructurada, (6) Adaptar una paleta para modo claro y oscuro.
---

# Color Palette Designer

Guía para construir paletas de colores coherentes con la identidad del producto, aplicando psicología del color y estándares de accesibilidad.

## Flujo de trabajo

### 1. Recopilar contexto

Solicitar al usuario (si no se ha proporcionado):
- **Nombre del producto** y tipo (web / móvil / ambos)
- **Valores de marca** o palabras clave emocionales (ej: confianza, energía, calma)
- **Audiencia objetivo** (edad, contexto cultural relevante)
- **PRD o brief de identidad** (opcional pero recomendado)
- **Colores existentes** de marca o restricciones cromáticas

### 2. Definir estrategia cromática

Con base en los valores de marca, seleccionar:

| Esquema | Cuándo usarlo |
|---|---|
| Monocromático | Marcas minimalistas, elegancia |
| Análogo | Experiencias suaves y armoniosas |
| Complementario | Alto contraste, llamadas a la acción fuertes |
| Triádico | Marcas vibrantes y creativas |

Para asociaciones emocionales detalladas por color, ver [`references/color-psychology.md`](references/color-psychology.md).

### 3. Construir la paleta (rol por rol)

Seguir este orden estricto:

1. **Primario** (~60 % interfaz) — refleja la emoción central de la marca; generar variantes `100–900`
2. **Secundario** (~30 %) — armoniza con el primario; diferencia secciones
3. **Acento** (~10 %) — CTA y estados especiales; debe contrastar con fondos
4. **Neutrales** — escala de grises para fondos, texto y bordes
5. **Semánticos** — éxito (`verde`), advertencia (`amarillo`), error (`rojo`), información (`azul`)

### 4. Validar accesibilidad

- Verificar ratio de contraste mínimo WCAG 2.1: **4.5:1** texto normal, **3:1** texto grande
- Herramienta recomendada: [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- Validar paleta tanto en modo claro como oscuro

### 5. Entregar la paleta documentada como HTML visual

La salida final **siempre debe ser un archivo HTML** con swatches visuales agrupados por categoría.
Usar `assets/palette-template.html` como base y adaptarlo a la paleta generada.

#### Estructura del documento de salida

El archivo HTML debe contener exactamente estas secciones en este orden:

```
1. Primary Colors    → fondo negro en el header de sección
2. Accent Colors     → junto a Signal Colors en layout de 2 columnas
3. Signal Colors     → columna derecha junto a Accent Colors
4. "Pop" Colors      → fila inferior
```

#### Reglas de swatches

| Tipo de color | Tamaño del swatch |
|---|---|
| Color principal de categoría | Grande: 80 × 80 px |
| Variante de opacidad (80%, 60%, etc.) | Pequeño: 48 × 48 px, apilado debajo del principal |
| Signal colors (rojo, amarillo, verde) | Pequeño: 32 × 32 px |

#### Datos que debe mostrar cada entrada de color

```
Nombre del color     (ej: "Blue")
Hex: #xxxxxx
RGB: R, G, B
```

Si el color tiene variante (p.ej. "80% Blue"), mostrarla debajo del swatch principal con su propio nombre, Hex y RGB.

#### Cómo usar la plantilla

1. Copiar `assets/palette-template.html` con el nombre del proyecto (ej: `mi-proyecto-palette.html`)
2. Reemplazar `{{PROJECT_NAME}}` con el nombre del producto
3. Sustituir cada sección con los colores reales: actualizar `background:`, nombre, Hex y RGB
4. Añadir o eliminar entradas de color según la paleta generada
5. Mantener el layout CSS tal como está; solo cambiar los valores de color y textos

Para la tabla de tokens y convenciones de nomenclatura, ver [`references/color-psychology.md`](references/color-psychology.md).

## Checklist de entrega

- [ ] Archivo HTML generado con secciones: Primary, Accent, Signal, "Pop"
- [ ] Cada color muestra nombre, Hex y RGB
- [ ] Variantes de opacidad apiladas debajo del color base con swatch pequeño
- [ ] Signal colors con swatch 32 × 32 px en columna derecha junto a Accent
- [ ] `{{PROJECT_NAME}}` reemplazado con el nombre del producto
- [ ] Ratios de contraste WCAG verificados
- [ ] Connotaciones culturales validadas para la audiencia objetivo
