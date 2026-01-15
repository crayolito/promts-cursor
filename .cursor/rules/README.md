# Reglas de Proyecto - Cursor IDE

Este directorio contiene las reglas de proyecto que Cursor IDE aplicará automáticamente al trabajar en el código.

## 📋 Reglas Disponibles

### 1. **regla1-basica.mdc** - Reglas Generales

- **Aplicación**: Siempre activa (`alwaysApply: true`)
- **Alcance**: Todos los archivos `.ts`, `.js`, `.tsx`, `.jsx`
- **Contenido**:
  - Idioma español obligatorio
  - Estructura de comentarios (PASOS/FASES)
  - Nomenclatura descriptiva

### 2. **regla2-basica.mdc** - Reglas CSS Profesional

- **Aplicación**: Siempre activa (`alwaysApply: true`)
- **Alcance**: Todos los archivos `.css`, `.scss`, `.sass`
- **Contenido**:
  - Mobile First obligatorio
  - Uso de variables CSS
  - Mapa de navegación en archivos largos
  - Estructura de media queries

### 3. **regla3-backend.mdc** - NestJS Clean Architecture

- **Aplicación**: Automática según archivos (`alwaysApply: false`)
- **Alcance**: Archivos `.ts` y `.js` en proyectos backend
- **Contenido**:
  - Estructura de carpetas (núcleo, adaptadores, infraestructura)
  - Nomenclatura de archivos
  - Reglas de importación
  - Ejemplos de entidades, casos de uso, controladores

### 4. **regla4-frontend.mdc** - Angular + NgRx + Shopify

- **Aplicación**: Automática según archivos (`alwaysApply: false`)
- **Alcance**: Archivos `.ts` y `.html` en proyectos frontend
- **Contenido**:
  - Angular moderno (standalone, signals)
  - NgRx moderno (createFeature, createActionGroup)
  - Filosofía Shopify (secciones dinámicas)
  - Estructura feature-based

## 🚀 Cómo Funcionan

Las reglas se aplican automáticamente cuando:

- **`alwaysApply: true`**: Siempre están activas en el contexto de Cursor
- **`alwaysApply: false`**: Se activan cuando editas archivos que coinciden con los `globs` especificados

## 📝 Formato de Archivos

Los archivos `.mdc` (Markdown Cursor) tienen:

- **Metadatos YAML** al inicio (description, globs, alwaysApply)
- **Contenido Markdown** con las reglas detalladas

## 🔧 Personalización

Para agregar o modificar reglas:

1. Crea un nuevo archivo `.mdc` en esta carpeta
2. Define los metadatos YAML
3. Escribe las reglas en Markdown
4. Cursor las detectará automáticamente

## 📚 Notas

- Los archivos `.txt` vacíos están ignorados por git (ver `.gitignore`)
- Las reglas se sincronizan con el repositorio remoto si existe
- Cada proyecto puede tener sus propias reglas en `.cursor/rules/`
