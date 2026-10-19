# Fede_Demucs

Sitio web oficial y **planificador local** de Demucs Studio: una interfaz para
Linux (GTK4 + libadwaita) que separa canciones en batería, bajo, voces y otros
instrumentos usando [demucs](https://github.com/adefossez/demucs) y PyTorch.

Este repositorio es la web que acompaña a la aplicación de escritorio: presenta
la app, documenta modelos, presets y formatos, y ofrece un planificador que
calcula tiempos, pesos y rutas reales a partir de tus propios archivos, sin
subir nada a ningún servidor.

---

## ¿Qué hace y qué no hace?

| Hace | No hace |
| --- | --- |
| Lee la duración de tus archivos de audio en el navegador (metadatos locales). | **No separa audio.** La separación ocurre en la app de escritorio. |
| Estima tiempo de proceso, tamaño de salida y rutas exactas con las fórmulas reales de la app. | No sube archivos a ningún servidor ni usa cuentas. |
| Exporta un plan en Markdown y guarda tus ajustes en `localStorage`. | No necesita backend, base de datos ni API. |

> **Importante:** el planificador es una herramienta de previsualización. Para
> obtener las cuatro pistas hay que instalar y ejecutar Demucs Studio en Linux.

---

## Stack

- **React 19** + **TypeScript** + **Vite 6**.
- CSS propio (sin frameworks) con tipografías autoalojadas **Manrope** y
  **JetBrains Mono** (`@fontsource-variable`).
- Iconos: [`lucide-react`](https://lucide.dev/).
- Sin backend. El cálculo se ejecuta íntegramente en el cliente.

---

## Requisitos

- **Node.js 22** (ver `.nvmrc`) y npm.
- Navegador moderno con soporte de `IntersectionObserver` y decodificación de
  audio.

```bash
# Con nvm
nvm use

# Instalar dependencias
npm install
```

---

## Scripts

| Comando | Descripción |
| --- | --- |
| `npm run dev` | Servidor de desarrollo con recarga en caliente. |
| `npm run build` | Chequeo de tipos (`tsc --noEmit`) y build de producción en `dist/`. |
| `npm run preview` | Sirve el build de producción para comprobarlo localmente. |

---

## Despliegue en Vercel

El proyecto ya incluye [`vercel.json`](./vercel.json) con el framework, el
comando de build, el directorio de salida, las cabeceras de seguridad y el
`rewrite` de SPA.

### Opción A — Desde el panel

1. Sube el repositorio a GitHub, GitLab o Bitbucket.
2. En Vercel, **Add New → Project** e importa el repositorio.
3. Vercel detecta Vite automáticamente. Si no, usa:
   - **Framework preset:** Vite
   - **Build command:** `npm run build`
   - **Output directory:** `dist`
4. Pulsa **Deploy**.

### Opción B — Desde la CLI

```bash
npm install -g vercel
vercel          # previsualización
vercel --prod   # producción
```

No hay variables de entorno ni servicios externos que configurar. El sitio es
100 % estático.

---

## Estructura

```
Fede_Demucs/
├── index.html              # Metadatos SEO, Open Graph y JSON-LD
├── vercel.json             # Configuración de despliegue
├── public/
│   ├── captura.png         # Captura real de la app de escritorio
│   ├── favicon.svg
│   ├── og.png / og.svg     # Imagen para redes sociales
│   └── robots.txt
└── src/
    ├── App.tsx             # Estado global, ingesta de archivos y composición
    ├── components/         # Nav, Hero, Planner, Presets, Formats, Models, Install, FAQ…
    ├── data/catalog.ts     # Modelos, presets, formatos y todas las fórmulas
    ├── hooks/              # Persistencia en localStorage y reveal on scroll
    ├── lib/                # Utilidades de navegador y lectura de archivos
    └── styles/global.css   # Sistema visual (negro, rojo y grises finos)
```

---

## Cómo se calculan las estimaciones

Todo vive en `src/data/catalog.ts` y replica la lógica de
`demucs_studio/engine.py` de la app de escritorio:

```
segundos = duración × coste_relativo(modelo, shifts) × coste_unidad(dispositivo) × 1,05
```

- **`coste_relativo`** = `bag × max(1, shifts)`. Cada submodelo y cada *shift*
  implican una pasada completa por la canción.
- **`coste_unidad`** (calibración de fábrica): `CPU 0,55` · `GPU (CUDA) 0,10`
  segundos de cómputo por segundo de audio, medidos en un Ryzen 7 4750G y una
  GTX 1660 Ti.
- **`× 1,05`**: 5 % de margen para carga del modelo y codificación.

La app de escritorio guarda los tiempos reales de cada separación y afina esa
calibración; el planificador web usa siempre la base de fábrica y lo indica en
la interfaz.

**Tamaño de salida** (4 pistas, estéreo, 44,1 kHz):

| Formato | Cálculo |
| --- | --- |
| WAV | `44100 × 2 × bytes_muestra × segundos` |
| FLAC | WAV × `0,55` |
| MP3 / M4A | `kbps × 1000 ÷ 8 × segundos` |
| OGG / Opus | `kbps (según VBR) × 1000 ÷ 8 × segundos` |

**Ejemplo real** (canción de 4 minutos, preset Equilibrada, CPU):
`240 × 1 × 0,55 × 1,05 ≈ 138,6 s` → `2 min 19 s`. En GPU: `25 s`.

---

## Privacidad

- Los archivos **no salen del equipo**. La duración se lee con un elemento
  `<audio>` local y un `ObjectURL` que se revoca al terminar.
- Los ajustes se guardan en `localStorage` del navegador.
- No hay analítica, cookies de seguimiento ni peticiones a terceros.

---

## Origen de los datos

Los modelos, presets, formatos, textos y capturas provienen del proyecto de
escritorio original (`../`). Las tablas de `catalog.ts` son la traducción
tipada de `demucs_studio/presets.py`, `demucs_studio/formats.py` y
`demucs_studio/engine.py`, para que las cifras mostradas coincidan con la app.

---

## Licencia y créditos

Esta web forma parte del proyecto Demucs Studio. El motor de separación,
[demucs](https://github.com/adefossez/demucs) y sus modelos pertenecen a Meta AI
Research y se distribuyen bajo licencia MIT. La interfaz es código propio del
proyecto.

Antes de publicar, añade el archivo de licencia que corresponda a tu proyecto en
la raíz del repositorio.
