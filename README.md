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


https://federico-castillo.github.io/UADE-Ingenieria-de-Datos-II/
```

## Licencia

Ver [`LICENSE`](LICENSE).
