# 3. Gestión del Proyecto y Flujo de Trabajo

Este documento explica nuestra metodología de trabajo en este repositorio. El objetivo es mantener un proceso organizado, donde cada cambio esté documentado y podamos integrar nuevas funcionalidades de forma segura.

## Estrategia de Ramas

Nuestro repositorio se organiza en dos ramas principales que están protegidas:

* **`main`**: Esta rama representa la versión de "producción". Es la versión más estable y final del proyecto. No se trabaja directamente sobre ella.
* **`develop`**: Esta es la rama principal de integración. Todo el nuevo trabajo (funcionalidades, arreglos, documentación) se añade a esta rama. Sirve como la versión de desarrollo más actualizada.

## Gestión de Tareas con Issues y Projects

Todo el trabajo comienza como una **"Issue"** (tarea) de GitHub.

1.  **Creación de Tareas:** Cada nueva funcionalidad o corrección se documenta primero como una Issue, lo que nos da un número de seguimiento (ej. `#12`).
2.  **Tablero Kanban:** Usamos **GitHub Projects** para visualizar el estado de todas las Issues (tareas) en un tablero (To Do, In Progress, Done).

## Proceso de Desarrollo y Ramas de Funcionalidad

Para evitar conflictos y mantener la rama `develop` estable, no trabajamos directamente en ella.

1.  **Aislamiento del Trabajo:** Cuando un desarrollador empieza a trabajar en una Issue (ej. `#12`), crea una nueva rama secundaria llamada **"feature branch"**.
2.  **Nomenclatura:** Esta rama se crea a partir de `develop` y usa el número de la Issue en su nombre para identificarla (ej. `feature/12-añadir-documentacion`).
3.  **Desarrollo:** Todo el trabajo (commits) se realiza de forma aislada en esta rama `feature`.

## Integración con Pull Requests (PRs)

Una vez que el trabajo en la rama `feature` está terminado, se propone su integración en `develop` a través de una **Pull Request (PR)**.

1.  **Solicitud de Fusión:** La PR es una solicitud formal para revisar y fusionar los cambios.
2.  **Enlace de Tareas:** En la descripción de la PR, usamos palabras clave (ej. `Closes #12`). Esto vincula la PR con la Issue y nos permite (dependiendo de la configuración) cerrar la tarea automáticamente al fusionar.
3.  **Squash and Merge:** Para mantener el historial de `develop` limpio y legible, no fusionamos todos los pequeños commits. Usamos la opción **`Squash and merge`**. Esto "aplasta" todos los commits de la rama `feature` en un solo commit antes de añadirlo a `develop`.

## Limpieza y Sincronización

Una vez que la PR ha sido fusionada en `develop`, la rama `feature` ya no es necesaria.

El proceso finaliza con una limpieza: se borra la rama `feature` (tanto en el servidor como localmente) y cada miembro del equipo sincroniza su rama `develop` local (`git pull`) para recibir los últimos cambios.