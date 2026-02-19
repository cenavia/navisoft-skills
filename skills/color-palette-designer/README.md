# color-palette-designer — Guía de prompts

Skill para diseñar y documentar paletas de colores para productos digitales aplicando psicología del color, accesibilidad WCAG y regla 60-30-10. La salida siempre es un archivo HTML visual con swatches agrupados por categoría.

---

## Prompts de activación

### Crear paleta desde cero (brief libre)

```
Diseña una paleta de colores para una app de finanzas personales.
La marca quiere transmitir confianza, modernidad y claridad.
El público objetivo son adultos de 25 a 40 años.
```

```
Crea la paleta de colores para mi startup de salud y bienestar.
Quiero colores que transmitan calma, naturaleza y profesionalismo.
```

```
Necesito una paleta de colores para una plataforma de e-learning
dirigida a jóvenes universitarios. La marca es vibrante y creativa.
```

---

### Crear paleta desde un PRD o documento de arquitectura

```
Genera la paleta de colores para el proyecto según este PRD: @docs/PRD.md
```

```
Basándote en este brief de identidad de marca, define la paleta de colores
con tokens de diseño y su significado psicológico: @docs/brand-brief.md
```

```
Lee el documento de arquitectura y crea la paleta de colores alineada
con los valores del producto: @architecture.md
```

---

### Definir tokens de diseño

```
Genera los tokens de diseño (CSS variables) para la paleta de colores
de mi app: primario, secundario, acento, neutrales y semánticos.
```

```
Crea una escala de tokens Primary-100 a Primary-900 para el color
azul #0019FF, incluyendo sus valores hex y RGB.
```

---

### Validar contraste y accesibilidad

```
Valida si la combinación de texto #1a1a1a sobre fondo #0019FF
cumple con WCAG 2.1 nivel AA.
```

```
Revisa si mi paleta actual cumple los estándares de accesibilidad WCAG
y sugiere ajustes donde sea necesario.
```

---

### Adaptar paleta a modo oscuro

```
Adapta esta paleta de colores para modo oscuro manteniendo
la coherencia emocional y el contraste adecuado:
- Primario: #0019FF
- Acento: #6E12FF
- Fondo: #FFFFFF
```

```
Genera las variantes dark mode para la paleta de mi app
a partir de los tokens que ya tengo definidos.
```

---

### Documentar una paleta existente

```
Documenta esta paleta de colores en formato HTML visual con swatches,
agrupados por categoría (Primary, Accent, Signal, Pop):
- Azul: #0019FF
- Azul claro: #00CFFF
- Morado: #6E12FF
- Rojo: #FF0000
- Amarillo: #FFFF00
- Verde: #5FC333
- Lima: #C5EF14
- Gris: #939598
```

```
Convierte esta paleta en un documento HTML con swatches visuales,
nombre, hex y RGB de cada color.
```

---

### Explorar psicología del color

```
¿Qué colores debo usar para una app de meditación y mindfulness?
Explícame el significado psicológico de cada opción.
```

```
Compara el uso del azul vs el verde como color primario para una
fintech latinoamericana. ¿Cuál recomiendas y por qué?
```

```
¿Qué consideraciones culturales debo tener en cuenta para definir
la paleta de una app que se lanzará en México, España y Colombia?
```

---

## Salida esperada

Para todos los casos el skill entrega:

1. **Archivo HTML** (`[proyecto]-palette.html`) con:
   - Secciones: Primary Colors · Accent Colors · Signal Colors · "Pop" Colors
   - Swatches de 80×80 px (color base) y 48×48 px (variantes de opacidad)
   - Signal colors con swatch de 32×32 px
   - Nombre, `Hex: #xxxxxx` y `RGB: R, G, B` por cada color

2. **Tabla de tokens** con escala numérica (`primary-500`, etc.) y significado psicológico

3. **Checklist de accesibilidad** con ratios de contraste verificados

---

## Archivos del skill

| Archivo | Descripción |
|---|---|
| `SKILL.md` | Flujo de trabajo e instrucciones para el agente |
| `assets/palette-template.html` | Plantilla HTML base de la salida visual |
| `references/color-psychology.md` | Asociaciones emocionales, esquemas cromáticos, tokens y herramientas |
