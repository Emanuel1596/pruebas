# Diseño de navegación y estado

Este documento define cómo se organizarán la navegación y el estado de OpenBooks antes de continuar con su implementación.

La propuesta parte del MVP del proyecto y busca dejar claro qué pantallas existen, cómo se relacionan entre sí, qué información utiliza cada una y dónde debería vivir cada dato para evitar duplicaciones o inconsistencias.

---

## 1. Mapa de navegación

Los flujos principales de OpenBooks son:

- Inicio → lista de libros → Detalle del libro.
- Buscar → Resultados → Detalle del libro.
- Mis libros → libro guardado → Detalle del libro.

A partir de esos flujos, la navegación queda organizada de la siguiente manera:

### Inicio

- Seleccionar un libro → **Detalle del libro**
- Realizar búsqueda → **Resultados**
  - Carga
  - Contenido
    - Seleccionar un libro → **Detalle del libro**
  - Sin resultados
  - Error
- Ir a **Mis libros**

### Mis libros

- Seleccionar un libro guardado → **Detalle del libro**
- Lista vacía
  - Buscar libros → **Inicio**

### Detalle del libro

- Regresar → pantalla desde la que se abrió el detalle
- Guardar libro → actualizar libros guardados → **Mis libros**
- Eliminar de Mis libros → actualizar libros guardados → **Mis libros**

Los estados **Carga**, **Sin resultados** y **Error** no se consideran pantallas independientes. Son estados de la pantalla **Resultados**.

De la misma forma, **Lista vacía** pertenece a **Mis libros** y **Sin portada** es una condición de presentación de un libro, no una pantalla adicional.

La navegación principal entre **Inicio** y **Mis libros** se mantiene mediante la barra inferior definida en el diseño del proyecto.

---

## 2. Información por pantalla

### Pantalla principal · Inicio

**Propósito:**  
Permitir al usuario comenzar una búsqueda, consultar la lista inicial de libros y acceder a Mis libros.

**Muestra:**

- Nombre de la aplicación.
- Campo de búsqueda.
- Lista inicial de libros.
- Título y autor de cada libro.
- Portada cuando esté disponible.
- Accesos a Inicio y Mis libros.

**Recibe:**

- Texto actual de búsqueda.
- Libros que se mostrarán en la lista inicial.

**Modifica:**

- Texto de búsqueda cuando el usuario escribe.
- Flujo de navegación cuando el usuario busca, selecciona un libro o entra a Mis libros.

**Necesita conservar:**

- El texto de búsqueda mientras el usuario pasa de Inicio a Resultados.

---

### Resultados

**Propósito:**  
Mostrar el resultado de una búsqueda realizada en Open Library.

**Muestra:**

- Texto de la búsqueda.
- Lista de libros encontrados.
- Estado de carga.
- Estado sin resultados.
- Estado de error.
- Acción para volver a intentar la búsqueda.

**Recibe:**

- Texto de búsqueda.
- Resultados obtenidos.
- Estado actual de la búsqueda.

**Modifica:**

- Texto de búsqueda si el usuario lo cambia.
- Estado de búsqueda cuando se realiza o se repite una consulta.
- Flujo de navegación cuando se selecciona un libro.

**Necesita conservar:**

- Texto de búsqueda.
- Resultados obtenidos.
- Estado de la búsqueda mientras el usuario entra a un detalle y posteriormente regresa.

---

### Detalle del libro

**Propósito:**  
Mostrar la información del libro seleccionado y permitir guardarlo o eliminarlo de Mis libros.

**Muestra:**

- Título.
- Autor o autores.
- Portada cuando esté disponible.
- Año de publicación cuando esté disponible.
- Acción Guardar libro o Eliminar de Mis libros.

**Recibe:**

- Libro seleccionado.

**Modifica:**

- Estado compartido de los libros guardados.

**Necesita conservar:**

- La información del libro mientras el detalle permanezca abierto.
- El cambio realizado sobre la colección de libros guardados debe mantenerse al navegar a otras pantallas.

La pantalla de detalle no debe crear una segunda copia independiente del estado de guardado. El estado real debe provenir de la misma fuente de verdad utilizada por Mis libros.

---

### Mis libros

**Propósito:**  
Mostrar los libros que el usuario ha guardado localmente.

**Muestra:**

- Lista de libros guardados.
- Título y autor de cada libro.
- Portada cuando esté disponible.
- Estado de lista vacía.
- Acción Buscar libros cuando la lista esté vacía.

**Recibe:**

- Colección compartida de libros guardados.

**Modifica:**

- No necesita modificar directamente la colección desde la lista.
- Modifica únicamente la navegación al seleccionar un libro o regresar a Inicio.

**Necesita conservar:**

- La colección de libros guardados.
- Esta información debe mantenerse entre pantallas y, posteriormente, persistirse localmente para conservarse entre ejecuciones de la aplicación.

---

## 3. Organización del estado

No toda la información que aparece en una vista debe convertirse en estado. Los datos de un libro, como título, autor, año o disponibilidad de portada, forman parte del modelo. En cambio, los datos que pueden cambiar y modificar lo que muestra la interfaz sí deben organizarse como estado.

La organización propuesta es la siguiente:

| Dato | Quién lo utiliza | Quién lo modifica | Dónde debería vivir | Justificación |
| --- | --- | --- | --- | --- |
| Texto de búsqueda | Inicio y Resultados | Inicio y Resultados | En un nivel superior compartido por ambas vistas | Las dos pantallas leen y modifican el mismo texto. Tener una copia en cada pantalla podría provocar valores diferentes. |
| Resultados de búsqueda | Resultados y Detalle | La lógica encargada de realizar la búsqueda | Estado compartido del flujo de búsqueda | Deben seguir disponibles si el usuario abre un detalle y después regresa a Resultados. |
| Estado de búsqueda: loading, content, noResults o error | Resultados | La lógica de búsqueda | Estado compartido del flujo de búsqueda | La interfaz de Resultados depende directamente de este valor. Al cambiar, cambia lo que se muestra. |
| Libros guardados | Mis libros y Detalle | Detalle al guardar o eliminar | Estado compartido por encima de ambas vistas | Varias vistas necesitan leer el mismo dato y Detalle necesita modificarlo. Debe existir una sola fuente de verdad. |
| Estado de guardado de un libro | Detalle y Mis libros | Detalle | Debe obtenerse de la colección de libros guardados | Evita mantener un `isSaved` independiente en varias vistas y reduce el riesgo de inconsistencias. |
| Libro seleccionado | Detalle | La pantalla desde la que se selecciona el libro | Se pasa durante la navegación | Solo se necesita para abrir el detalle correspondiente; no es necesario mantener otra copia global permanente. |
| Sección principal activa: Inicio o Mis libros | Barra inferior y vista raíz | Usuario al cambiar de sección | Estado local de la vista que contiene ambas secciones | La vista raíz es la responsable de esta decisión de navegación. |
| Título, autor, año y portada | Inicio, Resultados, Detalle y Mis libros | Open Library o los datos locales de prueba | Modelo `Book` | Son datos propios del libro. Las vistas los reciben y los muestran, pero no necesitan crear una copia de estado por pantalla. |

### Estado local y estado compartido

Se considera **estado local** cuando el dato pertenece a una sola vista y no necesita ser conocido por otras.

Se considera **estado compartido** cuando varias vistas necesitan leerlo o cuando una vista distinta de la propietaria necesita modificarlo.

Para OpenBooks:

- La colección de libros guardados debe ser compartida.
- Los resultados y el estado de una búsqueda deben mantenerse durante el flujo de búsqueda.
- El texto de búsqueda debe compartirse entre Inicio y Resultados.
- La selección de la sección principal puede pertenecer a la vista raíz.
- El libro seleccionado puede viajar como información de navegación hacia Detalle.

### Single Source of Truth

La aplicación debe evitar que un mismo dato tenga varios dueños.

El caso más importante es **Mis libros**. Si la lista de guardados y la pantalla de detalle mantuvieran copias diferentes de la información de guardado, podrían dejar de coincidir.

Por eso, la colección de libros guardados debe ser la fuente de verdad. La pantalla Detalle la modifica y Mis libros la lee.

Mientras el proyecto sea pequeño, el estado puede mantenerse por encima de las vistas que lo necesitan y compartirse con `@Binding` cuando una vista hija deba editarlo.

Si el estado compartido crece y comienza a ser utilizado por varias pantallas, puede organizarse en un objeto `@Observable` y proporcionarse a las vistas descendientes mediante `@Environment`. Cuando una vista necesite crear bindings hacia propiedades de ese objeto observable, puede utilizarse `@Bindable`.

La idea es que cada dato viva en el nivel más bajo desde el que pueda compartirse correctamente, sin duplicar la fuente de verdad.

### Persistencia

La colección de libros guardados deberá persistirse localmente porque forma parte del MVP.

La persistencia y el estado de la interfaz tienen responsabilidades diferentes:

- El estado compartido permite que las pantallas trabajen con la misma colección durante la ejecución.
- La persistencia local permite recuperar esa colección cuando la aplicación se vuelva a abrir.

La implementación concreta de persistencia se realizará posteriormente. En esta etapa solo se define dónde debe vivir la información y cómo debe compartirse.

---

## 4. Estrategia de navegación

Para el flujo de OpenBooks se propone utilizar los mecanismos de navegación de SwiftUI que mejor se ajustan a la estructura del proyecto.

### NavigationStack

`NavigationStack` puede actuar como contenedor del flujo de navegación de una sección.

Su función es mantener una pila de vistas. Cuando se abre una pantalla nueva, esta se agrega sobre la anterior y el usuario puede regresar a la pantalla previa.

Esto resulta adecuado para recorridos como:

- Inicio → Detalle.
- Inicio → Resultados → Detalle.
- Mis libros → Detalle.

Además, evita tener que guardar manualmente una copia de la pantalla anterior solo para saber a dónde regresar.

### NavigationLink

`NavigationLink` puede utilizarse cuando el usuario toca directamente un elemento para abrir otra vista.

En OpenBooks puede utilizarse para:

- Abrir el detalle de un libro desde Inicio.
- Abrir el detalle de un resultado.
- Abrir el detalle de un libro guardado.

La información que necesita la pantalla siguiente debe viajar con la navegación. En estos casos, Detalle necesita recibir el libro seleccionado.

### navigationDestination

`navigationDestination` permite relacionar un valor o tipo de navegación con la vista que debe mostrarse.

Para OpenBooks resulta útil porque **Detalle del libro es una misma pantalla que puede abrirse desde diferentes lugares**: Inicio, Resultados y Mis libros.

En lugar de diseñar tres pantallas de detalle diferentes, se mantiene una sola pantalla y se le proporciona la información del libro seleccionado.

También puede utilizarse para organizar destinos del flujo de búsqueda cuando la navegación dependa del estado o de una acción realizada por el usuario.

### Navegación entre Inicio y Mis libros

Inicio y Mis libros son las dos secciones principales del proyecto.

La barra inferior definida en los wireframes puede cambiar entre ambas secciones. La sección activa debe pertenecer a la vista raíz porque es la vista que contiene y coordina esas dos opciones.

Este cambio de sección es diferente a abrir un detalle sobre una pantalla. Por eso, la selección de Inicio o Mis libros se mantiene como estado de la raíz, mientras que los recorridos hacia Resultados o Detalle pueden organizarse mediante `NavigationStack`.

### Regreso entre pantallas

Cuando el usuario abra Detalle mediante una navegación apilada, el regreso puede ser manejado por `NavigationStack`.

Los casos principales son:

- Inicio → Detalle → regresar a Inicio.
- Resultados → Detalle → regresar a Resultados.
- Mis libros → Detalle → regresar a Mis libros.

### Guardar o eliminar un libro

Cuando el usuario guarde o elimine un libro:

1. Detalle modifica la fuente de verdad de libros guardados.
2. Mis libros utiliza ese mismo estado compartido.
3. La interfaz de Mis libros refleja el cambio.
4. De acuerdo con el flujo definido para el proyecto, el usuario puede continuar hacia Mis libros.

No se necesita crear una copia diferente del libro guardado para cada pantalla.

---

## 5. Relación con el prototipo actual

El prototipo actual de OpenBooks ya permite cambiar entre Inicio, Resultados, Detalle y Mis libros mediante un `enum` que representa la pantalla actual y varios valores `@State` almacenados en `ContentView`.

También utiliza `@Binding` para compartir el texto de búsqueda con vistas hijas.

Ese prototipo permite validar el flujo y las pantallas. Sin embargo, conforme avance el proyecto, conviene separar con mayor claridad:

- El estado de navegación.
- El estado compartido de búsqueda.
- La colección de libros guardados.
- Los datos propios de cada libro.

En esta etapa no es necesario modificar el código. El objetivo es dejar definida la organización que seguirá la implementación posterior.

---

## 6. Decisiones principales

1. **Inicio será la pantalla inicial.**
2. **Resultados será una sola pantalla con diferentes estados**, no una pantalla diferente para carga, error o sin resultados.
3. **Detalle del libro será reutilizable** sin importar si se abre desde Inicio, Resultados o Mis libros.
4. **Mis libros vacío será un estado de Mis libros**, no una pantalla independiente.
5. **Sin portada será una condición del libro**, no una pantalla.
6. **Los libros guardados tendrán una sola fuente de verdad compartida**.
7. **El texto de búsqueda y los resultados deberán mantenerse durante el flujo de búsqueda**.
8. **El libro seleccionado viajará hacia Detalle como información de navegación**.
9. **`NavigationStack`, `NavigationLink` y `navigationDestination` son los mecanismos previstos para organizar el flujo en SwiftUI**.
10. **Estas decisiones se documentan antes de llevarlas a la implementación**.
