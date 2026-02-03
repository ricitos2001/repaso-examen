# EXAMEN PRÁCTICO: DESARROLLO DE APLICACIONES WEB CLIENTE (ANGULAR)
**Proyecto:** Sistema de Gestión de Incidencias Técnicas (TechTicket)
**Duración estimada:** 4 - 5 horas
**Tecnología:** Angular 17+ (Standalone Components), RxJS, Signals, HttpClient.

---

## CONTEXTO
Una empresa de soporte técnico necesita un SPA (Single Page Application) para gestionar las incidencias de sus equipos informáticos. Se requiere una aplicación moderna, reactiva y optimizada que permita listar, filtrar, crear y detallar incidencias.

## PREPARACIÓN DEL ENTORNO
Se proporciona un archivo `db.json` inicial para levantar una API simulada con `json-server` en el puerto 3000.

```json
/* db.json */
{
  "incidencias": [
    { "id": "1", "titulo": "Fallo disco duro", "tipo": "hardware", "prioridad": "alta", "estado": "abierta", "descripcion": "Ruido mecánico...", "fecha": "2024-12-01" },
    { "id": "2", "titulo": "Licencia caducada", "tipo": "software", "prioridad": "media", "estado": "resuelta", "descripcion": "Office pide clave...", "fecha": "2024-12-02" }
  ],
  "usuarios": [ { "id": 1, "nombre": "Admin", "token": "fake-jwt-token" } ]
}
```

---

## BLOQUE 1: ESTRUCTURA, RUTAS Y NAVEGACIÓN (Fase 4)
**Objetivo:** Crear el esqueleto de navegación de la SPA.

1.  **Configuración de Rutas:**
    *   `/home`: Página de bienvenida (Pública).
    *   `/login`: Formulario de acceso simulado (Pública).
    *   `/panel`: Layout principal del área privada (Lazy Loading).
        *   `/panel/incidencias`: Listado de incidencias.
        *   `/panel/incidencias/nueva`: Formulario de creación.
        *   `/panel/incidencias/:id`: Detalle de la incidencia (Usar **Resolver** para precargar datos).
    *   `**`: Página 404 personalizada.

2.  **Seguridad (Guards):**
    *   Implementar un `authGuard` funcional (CanActivate) que proteja la ruta `/panel`. Si el usuario no está logueado (simular con un booleano en `AuthService` o localStorage), redirigir a `/login`.
    *   Implementar un `pendingChangesGuard` (CanDeactivate) en el formulario de creación para evitar salir si el formulario está "dirty" sin guardar.

3.  **Breadcrumbs:**
    *   Implementar un componente de migas de pan dinámico que lea la data de las rutas (`data: { breadcrumb: '...' }`).

---

## BLOQUE 2: UI, DOM Y EVENTOS (Fase 1)
**Objetivo:** Dotar de interactividad y personalización a la interfaz.

1.  **Theme Switcher:**
    *   Crear un servicio o lógica en el `AppComponent` que detecte la preferencia del sistema (`prefers-color-scheme`).
    *   Implementar un botón que cambie entre modo Claro/Oscuro.
    *   Usa `Renderer2` para inyectar/remover la clase `.dark-theme` en el `<body>` o `<html>`.
    *   Persistir la elección en `localStorage`.

2.  **Menú Lateral (Sidebar):**
    *   En el layout de `/panel`, crear un menú lateral que se pueda colapsar/expandir.
    *   Usar un booleano `isOpen` y animaciones CSS (o clases condicionales).
    *   Cerrar el menú automáticamente si se hace clic fuera de él (usar `@HostListener` o eventos en el backdrop).

3.  **Modal de Confirmación:**
    *   Crear un componente modal reutilizable accesible mediante `@ViewChild` desde el componente padre.
    *   Debe permitir cerrarse con la tecla `ESC` (evento de teclado).

---

## BLOQUE 3: SERVICIOS Y COMUNICACIÓN (Fase 2)
**Objetivo:** Desacoplar lógica y comunicar componentes.

1.  **Servicio de Notificaciones (Toasts):**
    *   Implementar un `ToastService` con un `BehaviorSubject`.
    *   Crear un componente `ToastComponent` que se suscriba y muestre mensajes flotantes (éxito, error) con temporizador de auto-cierre.

2.  **Comunicación entre Hermanos:**
    *   Crear un componente "Filtro" (sidebar izquierda en el listado) y un componente "Lista" (centro).
    *   Usar un `FilterService` para comunicar los cambios en los filtros (por ejemplo, filtrar por prioridad: Alta/Media/Baja) hacia la lista sin usar `@Input/@Output` directos (cadena larga), sino mediante el servicio compartido.

3.  **Loading Global:**
    *   Implementar un `LoadingService` y un componente Spinner que tape la pantalla cuando haya peticiones HTTP en curso.

---

## BLOQUE 4: HTTP Y DATOS (Fase 5)
**Objetivo:** Consumir la API simulada con buenas prácticas.

1.  **Servicio HTTP Base:**
    *   Configurar `provideHttpClient` en `app.config.ts`.
    *   Crear `IncidenciasService` usando `HttpClient`.
    *   Implementar métodos: `getAll()`, `getById(id)`, `create(data)`, `delete(id)`.
    *   Tipar todas las respuestas con interfaces (`Incidencia`, `ApiResponse`).

2.  **Interceptores:**
    *   **AuthInterceptor:** Debe inyectar un header `Authorization: Bearer fake-token` en todas las peticiones a `/incidencias`.
    *   **ErrorInterceptor:** Capturar errores HTTP. Si es un 404 o 500, mostrar un Toast de error automáticamente.

3.  **Visualización:**
    *   En el listado, usar el `AsyncPipe` para suscribirse a los datos.
    *   Manejar estados de: "Cargando", "Error" y "Lista vacía".

---

## BLOQUE 5: FORMULARIOS REACTIVOS AVANZADOS (Fase 3)
**Objetivo:** Crear el formulario de "Nueva Incidencia" (`/panel/incidencias/nueva`).

1.  **Validaciones Síncronas:**
    *   Campos: Título (Req, min 5), Tipo (Select), Prioridad (Radio), Descripción.
    *   **Validación Cross-Field:** Si la prioridad seleccionada es "Alta", la descripción debe tener obligatoriamente más de 20 caracteres.
    *   **Validador Personalizado:** Crear `nifValidator` para un campo de "DNI del solicitante".

2.  **Validación Asíncrona:**
    *   Campo "Código de Activo" (inventario).
    *   Crear un validador `assetExistsValidator` que simule una llamada al servidor (usar `timer` y `switchMap`) para verificar si el código introducido es válido. Mostrar "Verificando..." mientras escribe.

3.  **FormArray (Dinámico):**
    *   Sección "Piezas necesarias".
    *   Permitir añadir/eliminar dinámicamente líneas de piezas (Nombre pieza, Precio estimado).
    *   Mostrar el coste total estimado sumando las filas en tiempo real.

4.  **Feedback Visual:**
    *   Mostrar errores solo cuando el campo es `touched` o `dirty`.
    *   Deshabilitar el botón de envío si el form es inválido o `pending`.

---

## BLOQUE 6: GESTIÓN DE ESTADO Y OPTIMIZACIÓN (Fase 6)
**Objetivo:** Modernizar la gestión de datos y el rendimiento.

1.  **State Management con Signals:**
    *   Crear un `IncidenciasStore` (servicio) usando **Signals** (`signal`, `computed`).
    *   El store debe manejar el array de incidencias, el estado de loading y el término de búsqueda actual.
    *   El listado debe renderizarse leyendo `store.incidencias()`.

2.  **Buscador en Tiempo Real:**
    *   Implementar un input de búsqueda conectado a un `FormControl`.
    *   Usar RxJS (`valueChanges`, `debounceTime(500)`, `distinctUntilChanged`) para actualizar el Signal del filtro en el store sin machacar al servidor (o filtrando en local si se prefiere).

3.  **Optimización:**
    *   Usar `ChangeDetectionStrategy.OnPush` en el componente de tarjeta de incidencia.
    *   Implementar `trackBy` en el `*ngFor` del listado de incidencias.

---

## CRITERIOS DE EVALUACIÓN (RÚBRICA)

| Sección | Criterio | Puntos |
| :--- | :--- | :--- |
| **DOM & UI** | Theme switcher funcional con persistencia y uso de Renderer2. | 1.0 |
| **Rutas** | Lazy loading, Guards (Auth/Dirty) y Resolvers implementados correctamente. | 1.5 |
| **Componentes** | Comunicación correcta entre hermanos (Service) y uso de Toasts. | 1.5 |
| **HTTP** | Interceptores configurados, tipado fuerte y manejo de errores centralizado. | 2.0 |
| **Formularios** | Validaciones custom (síncrona y asíncrona), Cross-field y FormArray funcional. | 2.5 |
| **Estado/Opt** | Uso de Signals para el estado, Debounce en búsqueda y OnPush/TrackBy. | 1.5 |
| **Total** | | **10.0** |

---

### Entregables
1.  Repositorio Git con el código fuente.
2.  Archivo `README.md` documentando:
    *   Estructura de módulos/componentes.
    *   Explicación de la estrategia de estado (Signals vs Subjects).
    *   Guía de cómo levantar el proyecto y el servidor JSON.
