# Mezclador de Pinturas - Especificaciones

## 1. Concepto & Visión

**Mezclador de Pinturas** es una herramienta web para artistas y pintores que necesitan replicar colores digitales usando física real. A diferencia de mezcladores teóricos, este incorpora correcciones de espectro físico que simulan cómo se comportan los pigmentos reales. La experiencia es técnica pero accesible, como un laboratorio de arte digitalizado.

---

## 2. Design Language

### Aesthetic Direction
Estilo "dashboard científico modernizado" - limpio, profesional, con acentos de color azul corporativo que transmite confianza técnica.

### Color Palette
| Rol | Color | Hex |
|-----|-------|-----|
| Primary | Azul corporativo | `#4166F5` |
| Primary Dark | Azul profundo | `#2A52BE` |
| Background | Gradiente gris azulado | `#f5f7fa` → `#e4edf5` |
| Surface | Blanco puro | `#ffffff` |
| Success | Verde esmeralda | `#2ecc71` / `#27ae60` |
| Warning | Ámbar | `#f39c12` |
| Text Primary | Gris oscuro | `#333` |
| Text Secondary | Gris medio | `#666` |

### Typography
- **Fuente principal**: Segoe UI, Tahoma, Geneva, Verdana, sans-serif
- **Títulos**: 2.5rem (h1), 1.4rem (section titles)
- **Cuerpo**: 1rem
- **Detalles técnicos**: monospace para códigos HEX/RGB

### Spatial System
- Padding contenedor principal: 30px
- Border radius: 10px (elementos), 20px (container principal)
- Gap entre elementos: 10-15px
- Sombras: `0 15px 40px rgba(0,0,0,0.12)` (container), `0 4px 12px rgba(0,0,0,0.05)` (internos)

### Motion Philosophy
- Transiciones suaves de 0.2-0.3s ease para interacciones
- Hover: `translateY(-2px)` + intensificación de sombra
- Spinner de carga durante cálculos del algoritmo genético
- Scale en variaciones de color al hover (1.1x) y selección (1.15x)

### Visual Assets
- **Iconos**: Font Awesome 6.4.0 (CDN)
  - `fa-palette` - header
  - `fa-eye-dropper` - selección de color
  - `fa-boxes-stacked` - inventario
  - `fa-clipboard-list` - receta
  - `fa-calculator` - botón calcular
  - `fa-history` - historial
  - `fa-info-circle` - tips
  - `fa-spinner` - loading

---

## 3. Layout & Structure

### Arquitectura de Página
```
┌─────────────────────────────────────────┐
│ HEADER (gradient azul, título + subtítulo) │
├───────────────────┬─────────────────────┤
│ COLOR SELECTION   │ RESULTS             │
│ (flex: 1, 350px+) │ (flex: 1, 300px+)   │
│                   │                     │
│ - Preview color   │ - Receta mezclas    │
│ - Variaciones     │ - Info gotas        │
│ - Color picker    │ - Precisión         │
│ - HEX input       │ - Comparación       │
│ - RGB inputs      │ - Historial sesión  │
│ - Botón calcular  │                     │
│ - Inventario      │                     │
├───────────────────┴─────────────────────┤
│ FOOTER (copyright, créditos)            │
└─────────────────────────────────────────┘
```

### Responsive Strategy
- **Desktop (>768px)**: Layout de dos columnas lado a lado
- **Mobile (≤768px)**:
  - Columnas apiladas verticalmente
  - RGB inputs cambian a columna
  - Color match pasa a columna única

---

## 4. Features & Interactions

### 4.1 Selección de Color
**Inputs sincronizados:**
- Color picker nativo (`input[type="color"]`)
- Campo HEX manual (`#RRGGBB`)
- Tres campos RGB (R, G, B: 0-255)

**Sincronización:** Cualquier cambio en uno actualiza los demás instantáneamente.

**EyeDropper API:**
- Preview es clickeable
- Activa el gotero del sistema (Chrome/Edge)
- Cancela silenciosamente si el usuario cierra

### 4.2 Previsualización y Variaciones
**Preview:**
- Caja de 100px altura
- Muestra código HEX
- Fondo del color seleccionado
- Cursor pointer + tooltip

**Variaciones de color:**
- Barra horizontal scrolleable
- 7 variaciones HSL: `−S−L`, `−L`, `−S`, `Original`, `+S`, `+L`, `+S+L`
- **Barra física:** Sugerencia de ajuste para pigmentos reales (ancho 350px)
- Click en variaciones actualiza preview pero NO calcula receta
- Click en "Original" o botón dedicado calcula receta

### 4.3 Inventario de Pinturas Físicas
**Grid 3x3:**
| Fila 1 | Fila 2 | Fila 3 |
|--------|--------|--------|
| Blanco | Rojo | Cyan |
| Negro | Verde | Magenta |
| *(vacío)* | Azul | Amarillo |

**Estados de botón:**
- Default: borde gris claro, fondo blanco
- Hover: fondo azul claro, borde más oscuro, elevación
- Active (seleccionado): fondo azul pálido, borde azul, inset shadow, texto más bold

**Colores base:**
```javascript
Blanco:   { r: 255, g: 255, b: 255, activo: true }
Negro:    { r: 0,   g: 0,   b: 0,   activo: true }
Rojo:     { r: 255, g: 0,   b: 0,   activo: true }
Verde:    { r: 0,   g: 181, b: 0,   activo: true }
Azul:     { r: 0,   g: 71,  b: 171, activo: true }
Cyan:     { r: 0,   g: 237, b: 255, activo: false }
Magenta:  { r: 255, g: 0,   b: 171, activo: false }
Amarillo: { r: 255, g: 237, b: 0,   activo: true }
```

### 4.4 Algoritmo Genético
**Parámetros:**
- Población: 400 individuos
- Generaciones máx: 600
- Elite: 50 mejores
- Iteraciones paralelas: 10
- Mutación: 15% (si similitud <97%) o 35%

**Función de aptitud:**
- Delta E 2000 (CIE Lab) para similitud perceptual
- Bonus por menor cantidad de gotas
- Penalización si >30 gotas

**Casos especiales:**
- Blanco casi puro (luminosidad >0.95): usa solo blanco
- Negro casi puro (luminosidad <0.05): usa solo negro

### 4.5 Receta de Mezcla
**Lista de pinturas:**
- Muestra cada color base usado
- Sample de color + nombre + descripción técnica
- Número de gotas destacado

**Info adicional:**
- Total de gotas
- Porcentaje de precisión (verde si ≥98%, ámbar si <98%)
- Comparación visual: objetivo, resultado digital, sugerido físico

**Botón "Usar este":** Aplica el color sugerido físico al selector.

### 4.6 Historial de Sesión
- Máximo 20 entradas
- Cada entrada muestra: color objetivo, mezcla lograda, receta, precisión, gotas
- Click en entrada recarga el color
- Scroll vertical si excede 300px altura

### 4.7 Utilidades Clipboard
- Botón "Copiar HEX": copia valor del input hex
- Botón "Copiar RGB": copia `rgb(r, g, b)`
- Feedback visual: borde verde temporal (500ms)

---

## 5. Component Inventory

### 5.1 Color Preview
| Estado | Apariencia |
|--------|------------|
| Default | Fondo del color, texto HEX centrado, sombra sutil |
| Hover | Cursor pointer |
| Activo (gotero) | Tooltip "Haz clic para activar el gotero..." |

### 5.2 Variation Item
| Estado | Apariencia |
|--------|------------|
| Default | 50x50px, border-radius 8px, scale 1 |
| Hover | scale 1.1 |
| Highlighted (físico) | scale 1.15, sombra removida |
| Original | scale 1.1 |

### 5.3 Inventory Button
| Estado | Apariencia |
|--------|------------|
| Default | Borde `#e9eef2`, fondo blanco, dot color |
| Hover | Fondo `#f5f9ff`, borde `#cbd5e1`, elevación |
| Active | Fondo `#edf2ff`, borde `#4166F5`, inset shadow, translateY(1px) |

### 5.4 Calculate Button
| Estado | Apariencia |
|--------|------------|
| Default | Fondo `#4166F5`, texto blanco, 100% width |
| Hover | Fondo `#2A52BE`, translateY(-2px), sombra azul |
| Loading | Texto "Ejecutando Algoritmo Genético..." + spinner |

### 5.5 Paint Item (Receta)
- Border-left 4px azul
- Hover: elevación + sombra más pronunciada
- Muestra: sample 40x40, nombre, descripción, gotas (bold, azul)

### 5.6 History Item
| Estado | Apariencia |
|--------|------------|
| Default | Borde `#dee2e6`, fondo blanco |
| Hover | Borde azul, sombra azul sutil |

### 5.7 Color Match Box
- 60x60px muestra de color
- Label bold + HEX debajo
- El tercer box (sugerido) tiene borde verde y botón "Usar este"

---

## 6. Technical Approach

### Stack
- **HTML5** single-file application
- **CSS3** embebido (no preprocesador)
- **JavaScript** vanilla ES6+ (no frameworks)
- **CDN**: Font Awesome 6.4.0, Google Fonts (no usado)

### Algoritmos Clave

**RGB ↔ HEX:**
```javascript
rgbToHex({r, g, b}) → "#RRGGBB"
hexToRgb("#RRGGBB") → {r, g, b}
```

**RGB → HSL → RGB:**
```javascript
rgbToHslSimple(r, g, b) → {h: 0-360, s: 0-100, l: 0-100}
hslToRgbSimple(h, s, l) → {r, g, b}
```

**Perceptual Color Matching:**
```javascript
rgbToLab({r, g, b}) → {l, a, b}  // CIELAB
deltaE2000(lab1, lab2) → número  // Diferencia perceptual
porcentajeSimilitud(c1, c2) → 0-100%
```

**Corrección Física (simula pigmentos reales):**
```javascript
obtenerSugerenciaFisica({r, g, b}) → {r, g, b}
// Aumenta saturación +16% y luminosidad +8%
// Ajusta tono según algoritmo HSL
```

**Algoritmo Genético:**
```javascript
generarIndividuo(basesActivas) → {color: gotas, ...}
mezclar(individuo, basesActivas) → {r, g, b}
calcularMezclaGenetica(objetivo) → {ind, color, similitud, usados}
```

### State Management
- `catalogoColores`: Object con todos los colores base y su estado activo
- `historialSesion`: Array de últimas 20 recetas calculadas
- `colorBaseVariaciones`: Color actualmente en la barra de variaciones

### Event Handling
- `input` event en color picker → sync inmediata
- `change` event en HEX → validación regex `#?[0-9A-F]{6}`
- `input` event en RGB → validación 0-255
- `click` event en botones inventario → toggle activo
- `DOMContentLoaded` → inicializa UI del inventario y variaciones

---

## 7. Mejoras Potenciales (para discusión)

1. **Persistencia**: Guardar inventario y historial en localStorage
2. **Exportar receta**: Generar PDF/PNG de la receta
3. **Inventario expandible**: Permitir agregar colores personalizados
4. **Modo oscuro**: Toggle de tema
5. **Más variaciones**: Tonos pastel, saturación extrema
6. **Mezcla sustractiva**: Opción para pigmentos reales (CMYK)
7. **PWA**: Instalable como app móvil
