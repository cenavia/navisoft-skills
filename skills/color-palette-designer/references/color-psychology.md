# Referencia: Psicología del color y construcción de paletas

## Tabla de contenidos
1. [Asociaciones emocionales por color](#1-asociaciones-emocionales-por-color)
2. [Consideraciones culturales](#2-consideraciones-culturales)
3. [Roles de color y proporciones](#3-roles-de-color-y-proporciones)
4. [Esquemas cromáticos](#4-esquemas-cromáticos)
5. [Convenciones de nomenclatura y tokens](#5-convenciones-de-nomenclatura-y-tokens)
6. [Tabla de paleta de referencia](#6-tabla-de-paleta-de-referencia)
7. [Herramientas recomendadas](#7-herramientas-recomendadas)

---

## 1. Asociaciones emocionales por color

| Color | Emociones / Significado | Usos frecuentes |
|---|---|---|
| **Azul** | Confianza, profesionalidad, calma, seguridad | Finanzas, salud, tecnología, instituciones |
| **Azul claro** | Frescura, transparencia, modernidad | SaaS, apps de productividad |
| **Verde** | Naturaleza, equilibrio, salud, éxito | Bienestar, fintech sostenible, éxito/confirmación |
| **Rojo** | Urgencia, pasión, energía, peligro | CTAs, alertas de error, ofertas, deportes |
| **Naranja** | Entusiasmo, creatividad, calidez, asequibilidad | E-commerce, startups, apps juveniles |
| **Amarillo** | Optimismo, atención, alegría, advertencia | Advertencias, highlights, marcas juveniles |
| **Morado** | Creatividad, sofisticación, lujo, misticismo | Entretenimiento, belleza, marcas premium |
| **Rosa** | Ternura, romance, modernidad, feminidad | Moda, salud femenina, apps lifestyle |
| **Negro** | Elegancia, seriedad, poder, minimalismo | Lujo, tecnología premium, modo oscuro |
| **Blanco** | Pureza, claridad, neutralidad, espacio | Fondos limpios, minimalismo, salud |
| **Gris** | Neutralidad, equilibrio, sofisticación | Textos secundarios, bordes, fondos suaves |
| **Lima / Verde amarillo** | Vitalidad, jovialidad, innovación | Startups, apps de fitness, marcas disruptivas |

---

## 2. Consideraciones culturales

| Color | Occidente | Asia Oriental | Medio Oriente | Latinoamérica |
|---|---|---|---|---|
| **Blanco** | Pureza, boda | Luto (China, Japón) | Pureza | Pureza |
| **Rojo** | Peligro, amor | Suerte, prosperidad | Peligro | Pasión, energía |
| **Verde** | Naturaleza, dinero | Esperanza | Islam, sagrado | Naturaleza |
| **Amarillo** | Optimismo | Realeza (China) | Oro, felicidad | Alegría |
| **Morado** | Lujo, luto | Luto (algunos países) | Luto | Realeza, luto |

> Validar siempre los colores elegidos contra la cultura principal de la audiencia objetivo antes de entregar.

---

## 3. Roles de color y proporciones

### Regla 60-30-10

| Rol | Proporción | Descripción |
|---|---|---|
| **Primario** | ~60 % | Domina la interfaz; refleja la identidad central de la marca |
| **Secundario** | ~30 % | Complementa al primario; diferencia secciones y jerarquías |
| **Acento** | ~10 % | Atrae atención; CTA, badges, estados especiales |

### Colores neutrales

Usados en fondos, texto y bordes; no cuentan dentro del 60-30-10:
- **Grises claros** (`#F5F5F5` → `#E0E0E0`): fondos de tarjetas y páginas en modo claro
- **Grises medios** (`#9E9E9E`): texto secundario, placeholders, bordes suaves
- **Grises oscuros** (`#424242` → `#212121`): texto principal en modo claro, fondos en modo oscuro

### Colores semánticos

| Estado | Color base | Variante claro (bg) | Variante texto |
|---|---|---|---|
| Éxito | `#5FC333` / `#22C55E` | `#F0FDF4` | `#166534` |
| Advertencia | `#FFFF00` / `#EAB308` | `#FEFCE8` | `#713F12` |
| Error | `#FF0000` / `#EF4444` | `#FEF2F2` | `#991B1B` |
| Información | `#0019FF` / `#3B82F6` | `#EFF6FF` | `#1E40AF` |

---

## 4. Esquemas cromáticos

### Monocromático
- Una sola matiz con variaciones de saturación y brillo
- Crea unidad y sofisticación; fácil de mantener accesibilidad
- Ejemplo: paleta completamente en azul (`#001AFF` → `#E8EAFF`)

### Análogo
- Colores adyacentes en la rueda cromática (±30°)
- Armonioso y natural; transmite calma y coherencia
- Ejemplo: azul + azul-violeta + violeta

### Complementario
- Colores opuestos en la rueda (180°)
- Alto contraste; ideal para CTAs que contrasten con el primario
- Ejemplo: azul primario + naranja de acento

### Triádico / Tetrádico
- 3 o 4 colores equidistantes en la rueda
- Vibrante y dinámico; requiere cuidado para no saturar la UI
- Ejemplo: rojo + amarillo + azul

---

## 5. Convenciones de nomenclatura y tokens

### Escala numérica (estilo Tailwind / Material Design)

```
primary-50   → tono muy claro (fondos hover, chips)
primary-100  → claro (fondos de alerta, badges)
primary-200  → medio-claro
primary-300  → medio
primary-400  → medio-oscuro
primary-500  → base (color principal de la paleta)
primary-600  → hover en botones primarios
primary-700  → pressed / active
primary-800  → oscuro
primary-900  → muy oscuro (fondos modo oscuro)
```

Aplicar la misma escala para `secondary`, `accent`, y semánticos (`success`, `warning`, `error`, `info`).

### Tokens de diseño (CSS variables / Design Tokens)

```css
:root {
  --color-primary-500: #0019FF;
  --color-primary-600: #0014CC;
  --color-secondary-500: #6E12FF;
  --color-accent-500: #FF6200;
  --color-neutral-100: #F5F5F5;
  --color-neutral-900: #212121;
  --color-success: #5FC333;
  --color-warning: #EAB308;
  --color-error: #EF4444;
  --color-info: #3B82F6;
}
```

---

## 6. Tabla de paleta de referencia

Paleta extraída de la guía técnica; sirve como punto de partida para adaptarla a cada proyecto.

| Categoría | Token | Color | Hex | Significado psicológico |
|---|---|---|---|---|
| **Primario** | `primary-500` | Azul | `#0019FF` | Confianza, profesionalidad |
| **Primario** | `primary-600` | Azul 80 % | `#3347FF` | Variante oscura para reforzar confianza |
| **Primario** | `primary-300` | Azul claro | `#00CEFF` | Frescura, transparencia |
| **Primario** | `primary-200` | Azul claro 80 % | `#33D8FF` | Variante suave de frescura |
| **Acento** | `accent-900` | Negro | `#000000` | Elegancia, seriedad |
| **Acento** | `accent-500` | Morado | `#6E12FF` | Creatividad, sofisticación |
| **Acento** | `accent-400` | Morado 80 % | `#8B41FF` | Variante suave de creatividad |
| **Señal / Error** | `error` | Rojo | `#FF0000` | Urgencia, advertencia |
| **Señal / Advertencia** | `warning` | Amarillo | `#FFFF00` | Atención, optimismo |
| **Señal / Éxito** | `success` | Verde | `#5FC333` | Éxito, bienestar |
| **Pop / Acento alternativo** | `accent-lime` | Lima | `#C5EF14` | Vitalidad, jovialidad |
| **Neutral** | `neutral-500` | Gris | `#939598` | Neutralidad, equilibrio |
| **Neutral** | `neutral-0` | Blanco | `#FFFFFF` | Pureza, claridad |

> **Recomendaciones de uso:**
> - Indicar porcentajes de uso (60 % primario / 30 % secundario / 10 % acento) en la documentación
> - Acompañar cada color con una muestra visual en el design system
> - Documentar contraste mínimo y entornos de uso (modo claro/oscuro, hover, pressed)

---

## 7. Herramientas recomendadas

| Herramienta | Uso | URL |
|---|---|---|
| **Adobe Color** | Generación y exploración de esquemas | https://color.adobe.com |
| **Coolors** | Generación rápida de paletas | https://coolors.co |
| **Material Theme Builder** | Paletas M3 basadas en color semilla | https://m3.material.io/theme-builder |
| **WebAIM Contrast Checker** | Validación de contraste WCAG | https://webaim.org/resources/contrastchecker/ |
| **Accessible Color Matrix** | Ver todas las combinaciones en una vista | https://toolness.github.io/accessible-color-matrix/ |
| **Tailwind Color Generator** | Generar escalas 50-950 | https://uicolors.app |
