# Dieguinni's Projects — Prohub

Workspace personal para gestionar proyectos, fases, recursos, biblioteca de prompts/herramientas, diario y metas.

Migrado de `localStorage` (JSON) a **SQLite** (vía sql.js) para tener una base de datos portable que puedes guardar en un repo y abrir desde cualquier lugar.

---

## Estructura del repo

```
prohub/
├── prohub.html      Interfaz (abrir en el navegador)
├── sql-asm.js       Motor SQLite compilado a JS (sin WASM, funciona vía file://)
├── prohub.db        Tu base de datos (se crea al "Guardar como .db")
└── README.md
```

## Cómo usarlo

### Primera vez

1. Abre `prohub.html` con doble clic (Chrome o Edge recomendado).
2. La app arranca con datos de ejemplo. Si tenías datos del `prohub_23.html` original abierto en este mismo navegador, se migran automáticamente desde `localStorage`.
3. En la barra lateral, sección **Base de datos**, click en **"Guardar como .db"** y elige `prohub.db` en esta misma carpeta.
4. A partir de ahora cada cambio se escribe automáticamente al archivo.

### Próximas veces

- Abre `prohub.html`. Si Chrome/Edge recuerda el permiso, se reconecta solo.
- Si no se reconecta automáticamente, click en **"Conectar prohub.db"** y selecciona el archivo.
- Mientras no esté conectado a un archivo, los cambios quedan en el **caché del navegador** (IndexedDB). Cuando conectes el .db, el siguiente guardado actualiza el archivo.

### En otra computadora

1. Clona / copia el repo completo.
2. Abre `prohub.html`.
3. Click en **"Conectar prohub.db"** y selecciona el archivo que viaja con el repo.

---

## Estado de conexión

En la sidebar verás un indicador:

| Estado | Significado |
|---|---|
| **prohub.db** (verde) | Conectado al archivo. Cada cambio se escribe a disco. |
| **Solo caché** (ámbar) | Los datos viven solo en IndexedDB de este navegador. Conecta un archivo para persistencia portable. |
| **Sin conectar** (gris) | Estado inicial antes de cargar. |

---

## Compatibilidad de navegadores

| Navegador | Auto-guardar al archivo | Importar/Exportar .db |
|---|---|---|
| Chrome / Edge / Opera | ✅ Sí (File System Access API) | ✅ |
| Firefox / Safari | ❌ No (no implementan la API) | ✅ usa "Descargar .db" + "Importar .db" |

En Firefox/Safari el flujo es manual:
1. Trabaja normalmente — los datos quedan en caché.
2. Cuando quieras guardar al repo, **"Descargar .db"** → reemplaza el archivo del repo.
3. Al abrir en otra máquina: **"Importar .db"** → selecciona el archivo del repo.

---

## Botones de la sección "Base de datos"

- **Conectar prohub.db** — Selecciona el archivo .db. Una vez conectado, los guardados son automáticos.
- **Desconectar archivo** — Vuelve al modo "solo caché".
- **Guardar como .db** — Crea un nuevo archivo .db (úsalo la primera vez).
- **Importar .db** — Reemplaza la base actual con un archivo .db existente.
- **Descargar .db** — Descarga la base actual como `prohub-YYYY-MM-DD.db`.
- **Backup JSON / Importar JSON** — Backup legible en JSON (incluye proyectos, biblioteca, diario y metas). Útil para diffs en Git o migración a otro formato.
- **Reiniciar datos** — Borra todo y vuelve a los valores de ejemplo.

---

## Inspeccionar el archivo .db

`prohub.db` es un archivo SQLite estándar. Puedes abrirlo con:

- [**DB Browser for SQLite**](https://sqlitebrowser.org/) (Windows/Mac/Linux, gratis)
- [**SQLiteStudio**](https://sqlitestudio.pl/)
- [**Beekeeper Studio**](https://www.beekeeperstudio.io/)
- Extensión [**SQLite Viewer**](https://marketplace.visualstudio.com/items?itemName=qwtel.sqlite-viewer) en VS Code

### Esquema

```sql
projects (id, name, category, status, priority, data, updated_at)
settings (key, value)
diario (id, date, data)
metas (id, titulo, periodo, data)
plantillas (id, nombre, data)
bib_items (id, type, title, "collection", fav, data)
bib_collections (name, position)
```

`data` es JSON con el objeto completo. Las columnas extra (name, category, type, etc.) son para queries directas.

Ejemplos de SQL útiles:

```sql
SELECT name, status FROM projects WHERE category='negocio';
SELECT title FROM bib_items WHERE fav=1;
SELECT date, json_extract(data,'$.content') AS texto FROM diario ORDER BY date DESC;
```

---

## Versionar con Git

```bash
cd "C:\Dieguinnis Proyects\prohub"
git init
git add prohub.html sql-asm.js prohub.db README.md
git commit -m "Initial commit"
```

> **Nota:** `prohub.db` es binario, así que `git diff` no es útil sobre él. Si quieres revisar cambios en GitHub usa **"Backup JSON"** ocasionalmente y commitea ese archivo también.

### .gitignore sugerido

```
# Nada — queremos versionar prohub.db
# Pero ignora backups temporales
*.tmp
prohub-*.db
dieguinni-projects-backup-*.json
```

---

## Migración desde la versión anterior (prohub_23.html en localStorage)

Si abres `prohub.html` en **el mismo navegador** donde tenías el `prohub_23.html` original con datos, se detecta y migra automáticamente las 3 claves de localStorage:

- `prohub_v3` → tabla `projects` + `settings`
- `prohub_extras_v1` → tablas `diario`, `metas`, `plantillas`
- `prohub_bib_v1` → tablas `bib_items` + `bib_collections`

Verás una notificación **"Datos migrados desde localStorage ✓"**. Después puedes hacer "Guardar como .db" para persistirlo al archivo.

Si la versión vieja está en otro navegador / máquina, exporta desde ella con **"Exportar backup"** (JSON), copia el archivo, y aquí usa **"Importar JSON"**.
