# Ingeniería de Datos II — UADE

Apunte y wiki personal de la materia **Ingeniería de Datos II** de la
Universidad Argentina de la Empresa (UADE). El sitio se construye de forma
progresiva a lo largo de la cursada: resúmenes de teoría, apuntes de clase,
material de los docentes, ejercicios resueltos y recursos externos.

Publicado con [MkDocs](https://www.mkdocs.org/) y el tema
[Material for MkDocs](https://squidfunk.github.io/mkdocs-material/), desplegado
automáticamente en GitHub Pages.

## Estructura del proyecto

```text
.
├── docs/                       # Todo el contenido del sitio (Markdown)
│   ├── index.md                 # Página de inicio
│   ├── info-materia/            # Cronograma (incluye unidades), bibliografía
│   ├── resumenes/                # Resúmenes de teoría por unidad
│   ├── teoria/                   # Desarrollo de conceptos individuales
│   ├── practica/                 # Guías de trabajos prácticos
│   ├── ejercicios/               # Ejercicios resueltos y explicados
│   ├── materiales/               # Material provisto por los docentes (clases, presentaciones, archivos)
│   ├── recursos/                 # Documentación externa, herramientas, links
│   ├── stylesheets/extra.css     # Ajustes de estilo sobre el tema Material
│   └── javascripts/mathjax.js    # Configuración de fórmulas matemáticas
├── mkdocs.yml                   # Configuración del sitio y navegación
├── requirements.txt             # Dependencias Python (mkdocs-material)
└── .github/workflows/deploy.yml # Build y deploy automático a GitHub Pages
```

Cada sección tiene una página `index.md` con la lista de contenido disponible
y, en varios casos, una página de ejemplo (marcada como *placeholder*) que
muestra el formato esperado y sirve para validar que el sitio funciona.

## Cómo ejecutar el proyecto localmente

Requiere Python 3.9+.

```bash
# 1. (Recomendado) crear un entorno virtual
python -m venv .venv
.venv\Scripts\activate      # Windows
# source .venv/bin/activate # macOS/Linux

# 2. Instalar dependencias
pip install -r requirements.txt

# 3. Levantar el servidor de desarrollo (con recarga automática)
mkdocs serve
```

El sitio queda disponible en `http://127.0.0.1:8000`.

## Cómo agregar contenido nuevo

Guía completa en
[`docs/recursos/como-agregar-contenido.md`](docs/recursos/como-agregar-contenido.md)
(también visible en el sitio publicado, sección **Recursos**). Resumen:

1. Crear el archivo `.md` dentro de la carpeta de la sección correspondiente
   (`docs/resumenes/`, `docs/teoria/`, `docs/practica/`, etc.), con nombre en
   minúsculas y guiones (`unidad-2.md`, `guia-3.md`).
2. Escribir el contenido en Markdown (soporta código, tablas, fórmulas con
   `\( ... \)` / `\[ ... \]`, y notas con `!!! note "Título"`).
3. Agregar la página al bloque `nav` de `mkdocs.yml`, dentro de la sección
   correspondiente (MkDocs no descubre páginas automáticamente).
4. Verificar con `mkdocs serve` y hacer commit + push a `main`.

## Build

```bash
mkdocs build --strict
```

Genera el sitio estático en `site/` (carpeta ignorada por git, no se
versiona).

## Deploy a GitHub Pages

El deploy está automatizado con GitHub Actions
(`.github/workflows/deploy.yml`): cada push a `main` ejecuta `mkdocs build`
y publica el resultado en GitHub Pages. No requiere ningún paso manual.

**Configuración única, la primera vez:** en el repositorio de GitHub, ir a
`Settings → Pages → Build and deployment → Source` y seleccionar
**GitHub Actions**. Una vez publicado, el sitio queda disponible en:

```text
https://federico-castillo.github.io/UADE-Ingenieria-de-Datos-II/
```

## Licencia

Ver [`LICENSE`](LICENSE).
