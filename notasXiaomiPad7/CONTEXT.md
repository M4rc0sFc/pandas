# Current Project
 
Carpeta de notas manuscritas que Marcos crea en la app **Lienzos Mi** usando su tablet Xiaomi Pad 7. Las notas son principalmente sobre su tesis (hiperspectral remote sensing de bosque tropical seco) y bitácora de avances de la misma. Claude Code convierte las imágenes de notas a markdown usando la skill `ocr-notes`.
 
## Workflow
 
1. Marcos escribe notas a mano en Lienzos Mi (Xiaomi Pad 7).
2. Exporta las notas como imagen a esta carpeta.
3. Claude Code recibe la imagen y usa la skill `ocr-notes` para convertirla a markdown.
4. El archivo `.md` resultante se guarda **en esta misma carpeta**, junto a la imagen original.
 
### Skill de referencia
 
La skill `ocr-notes` se encuentra en:
 
```
C:\Users\Marco\.claude\skills\ocr-notes
```
 
Consulta su `SKILL.md` antes de procesar cualquier imagen.
 
## What good looks like
 
- El markdown refleja **fielmente** el contenido de la imagen, sin inventar ni omitir información.
- El archivo resultante sigue la naming convention definida abajo.
- La estructura del markdown usa headings, listas y tablas solo cuando la nota original lo justifica.
- El tono y lenguaje del output coinciden con lo escrito en la nota (generalmente español).
 
## What to avoid
 
- **No inventar contenido** que no esté en la imagen.
- **No interpretar** letra ilegible como texto seguro; marcar con `[ilegible]` cuando no se pueda leer con confianza.
- **No cambiar la estructura** de la nota original (si es una lista, que sea lista; si es prosa, que sea prosa).
- **No crear archivos fuera de esta carpeta.**
- **No sobreformatear**: no agregar headings, bullets o tablas que la nota original no tenga.
 
## Naming conventions
 
| Content type               | Pattern                      | Example              |
| -------------------------- | ---------------------------- | -------------------- |
| Markdown note from image   | `[nombredenota][DDMMYY].md`  | `avances300326.md`   |
 
- Nombres siempre en **minúsculas**, sin espacios ni caracteres especiales.
- Las imágenes originales también viven en esta carpeta.
 
## Contenido de la carpeta
 
Esta carpeta contiene dos tipos de archivos:
 
- **Imágenes** (.png, .jpg): notas manuscritas originales exportadas desde Lienzos Mi.
- **Markdown** (.md): transcripciones generadas por Claude Code a partir de las imágenes.
  