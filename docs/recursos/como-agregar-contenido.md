# Cómo agregar contenido

Guía rápida para sumar contenido nuevo al sitio durante la cursada.

## 1. Elegir la sección y el nombre de archivo

Todo el contenido vive en `docs/`, organizado por sección. Usar nombres de
archivo en minúsculas, con guiones (`kebab-case`), sin espacios ni tildes:

```text
docs/
  resumenes/unidad-2.md
  teoria/normalizacion.md
  practica/guia-3.md
  ejercicios/practica.md   (agregar una sección "## Ejercicio N" nueva)
  materiales/clases.md
  recursos/links.md
```

## 2. Escribir el archivo en Markdown

Cada página es un archivo `.md` normal. Elementos disponibles:

- **Títulos**: `#`, `##`, `###`
- **Código**: bloques con tres backticks y el lenguaje (` ```python `)
- **Tablas**: sintaxis estándar de Markdown
- **Fórmulas**: `\(a^2 + b^2 = c^2\)` (inline) o `\[ ... \]` (bloque), vía MathJax
- **Notas / advertencias**:

```markdown
!!! note "Título"
    Contenido de la nota.

!!! tip "Tip"
    Contenido del tip.

!!! warning "Atención"
    Contenido de la advertencia.
```

- **Imágenes**: guardarlas junto al contenido relacionado (por ejemplo
  `docs/teoria/img/`) y referenciarlas con `![descripción](img/archivo.png)`

Ver [Resúmenes → Unidad 1](../resumenes/unidad-1.md) como referencia con
todos estos elementos usados a la vez.

- **Archivos adjuntos (PDF, slides)**: guardarlos en la carpeta
  `archivos/` de la sección correspondiente (por ejemplo
  `docs/materiales/archivos/`), con nombre en `kebab-case`. Todo link a un
  material cargado (PDF u otro archivo) **debe abrir en una pestaña nueva**,
  agregando `{: target="_blank" }` después del link:

  ```markdown
  [Nombre del material](archivos/archivo.pdf){: target="_blank" }
  ```

## 3. Registrar la página en la navegación

MkDocs no descubre páginas automáticamente: hay que agregar la ruta en el
bloque `nav` de `mkdocs.yml`, dentro de la sección correspondiente.

```yaml
- Resúmenes:
    - resumenes/index.md
    - "Unidad 1": resumenes/unidad-1.md
    - "Unidad 2": resumenes/unidad-2.md   # nueva línea
```

## 4. Previsualizar localmente

```bash
mkdocs serve
```

Abre el sitio en `http://127.0.0.1:8000` con recarga automática al guardar.

## 5. Commitear y pushear

Al pushear a `main`, GitHub Actions reconstruye y publica el sitio
automáticamente (ver `.github/workflows/deploy.yml`). No hace falta ningún
paso manual de build/deploy.
