# 📋 Plan de Implementación: Aplicación "BARBERIA" (Flutter + Firebase)

> ⚠️ **Nota preliminar:** De acuerdo con tu solicitud, este documento **no contiene código**. Es un procedimiento paso a paso enfocado en arquitectura, configuración, flujo de trabajo y decisiones de diseño para desarrollar la aplicación de manera ordenada y escalable.

---

## 🛠️ Fase 0: Preparación del Entorno y Herramientas Requeridas
1. **Instalación del SDK:** Descargar e instalar Flutter SDK y Dart SDK oficiales. Verificar instalación con `flutter doctor`.
2. **IDE Recomendado:** Utilizar **VS Code** (estándar y mejor soportado para Flutter). Instalar extensiones oficiales: `Flutter`, `Dart`, `Firebase`, `Awesome Flutter Snippets`, `Error Lens`, `Pubspec Assist`. 
   - *Nota:* "Antigravity" no es un IDE reconocido en el ecosistema Flutter. Si te refieres a Android Studio o IntelliJ, el flujo es similar, pero VS Code es más ligero y ampliamente documentado.
3. **Entornos de Prueba:** Configurar emuladores Android (Android Studio Device Manager) y simulador iOS (macOS). Habilitar depuración USB para dispositivos físicos.
4. **Control de Versiones:** Inicializar repositorio Git, crear ramas (`main`, `develop`, `feature/*`), configurar `.gitignore` oficial de Flutter.
5. **Inicialización del Proyecto:** Crear proyecto Flutter con nombre `barberia_app` y organizar estructura de carpetas desde el inicio (`lib/`, `assets/`, `test/`, `web/`).

---

## 🎨 Fase 1: Diseño UI/UX
1. **Identidad Visual:** Definir paleta de colores (ej. tonos oscuros, acentos dorados/cobre para evocar elegancia masculina), tipografía legible y estilo de iconografía minimalista.
2. **Arquitectura de Información:** Mapear flujos de usuario:
   - Cliente: Login/Registro → Home → Catálogo de Servicios → Agendar Cita → Historial → Perfil
   - Barbero/Admin: Dashboard → Gestión de Horarios → Validación de Citas → Reportes
3. **Prototipado:** Diseñar wireframes y mockups en Figma/Adobe XD. Validar contraste, jerarquía visual y accesibilidad (tamaños de toque ≥ 44px, navegación por gestos y botones claros).
4. **Sistema de Componentes:** Definir reutilizables: `AppBar` personalizado, `BottomNavigationBar`, `ServiceCard`, `AppointmentRow`, `FormFields`, `StateLoaders`, `EmptyStateWidgets`.
5. **Responsive & Multiplataforma:** Planificar adaptaciones para móviles (vertical) y tablets/web (grid de 2-3 columnas, navegación lateral opcional).

---

## 📦 Fase 2: Arquitectura y Dependencias (`pubspec.yaml`)
1. **Estructura de Capas (Clean/Feature-Based):**
   - `lib/core/` (utilidades, constantes, temas, rutas)
   - `lib/data/` (servicios Firebase, repositorios, modelos DTO)
   - `lib/domain/` (entidades, casos de uso, interfaces)
   - `lib/presentation/` (pantallas, widgets, providers)
2. **Dependencias Esenciales (`pubspec.yaml`):**
   - `firebase_core`, `firebase_auth`, `cloud_firestore` → Backend y autenticación
   - `provider` → Gestión de estado
   - `go_router` o `auto_route` → Navegación segura y tipada
   - `intl` → Formateo de fechas, horas y monedas
   - `cached_network_image` → Optimización de imágenes de servicios/perfiles
   - `flutter_local_notifications` → Recordatorios de citas
   - `flutter_secure_storage` → Persistencia segura de tokens/sesiones locales
   - `image_picker` o `file_picker` → Carga de foto de perfil
   - `flutter_form_builder` + `formz`/`validators` → Validación de formularios
3. **Configuración Inicial:** Ejecutar `flutter pub get`, activar linter estricto en `analysis_options.yaml`, configurar `flutter_native_splash` y `flutter_launcher_icons` para branding.

---

## 🔥 Fase 3: Configuración de Firebase (Auth + Firestore)
1. **Consola Firebase:** Crear proyecto, registrar apps Android, iOS y Web.
2. **Archivos de Configuración:** Descargar `google-services.json` (Android) y `GoogleService-Info.plist` (iOS). Colocarlos en rutas correctas según documentación oficial de FlutterFire.
3. **Autenticación:** Habilitar método "Correo electrónico/Contraseña". Configurar políticas de contraseña y activar email de verificación (opcional pero recomendado).
4. **Firestore Database:** Crear base de datos en modo producción. Definir colecciones:
   - `users` (id, email, nombre, rol, foto, teléfono)
   - `barbers` (id, nombre, especialidades, horarios, rating)
   - `services` (id, nombre, duración, precio, imagen, barbero_asignado)
   - `appointments` (id, cliente_id, barbero_id, servicio_id, fecha, estado, notas)
5. **Reglas de Seguridad:** Configurar acceso por rol. Solo el dueño del documento o un admin puede modificar ciertos campos. Validar tipos y límites en reglas antes de pasar a producción.
6. **Inicialización en Flutter:** Configurar `Firebase.initializeApp()` en el entry point antes de cualquier interacción con servicios.

---

## 🔐 Fase 4: Autenticación (Email/Password)
1. **Formularios de Acceso:** Diseñar pantallas de Login y Registro con validación en tiempo real (formato email, longitud contraseña, confirmación).
2. **Flujo de Credenciales:** Implementar llamadas a Firebase Auth para `signInWithEmailAndPassword` y `createUserWithEmailAndPassword`.
3. **Manejo de Estados:** Diferenciar visualmente y lógicamente: cargando, éxito, error (usuario no encontrado, contraseña incorrecta, email ya registrado).
4. **Persistencia de Sesión:** Aprovechar el manejo nativo de sesiones de Firebase. Implementar verificación de estado en inicio de app para redirigir automáticamente a Dashboard o Auth.
5. **Recuperación de Acceso:** Añadir flujo de "¿Olvidaste tu contraseña?" que dispare email de restablecimiento vía Firebase Auth.
6. **Asignación de Rol Inicial:** Tras registro exitoso, crear documento en `users/{uid}` con rol por defecto (`cliente`). Validar rol antes de mostrar funcionalidades específicas.

---

## 🔄 Fase 5: Gestión de Estado con Provider
1. **Creación de Providers:**
   - `AuthProvider`: estado de sesión, usuario actual, métodos de auth, logout.
   - `ServiceProvider`: lista de servicios, filtros por categoría/barbero, cache local.
   - `AppointmentProvider`: citas activas, historial, estados (pendiente, confirmada, cancelada).
2. **Inyección en Árbol de Widgets:** Envolver la app con `MultiProvider` en el entry point. Inicializar providers con datos estáticos o listeners de Firestore según corresponda.
3. **Optimización de Rebuilds:** Usar `Consumer`, `Selector` o `context.watch`/`context.read` de forma estratégica. Evitar escuchar streams innecesarios en pantallas que no los requieren.
4. **Separación de Responsabilidades:** Los providers solo deben contener lógica de estado y orquestación. Las llamadas directas a Firebase deben encapsularse en servicios/repositorios inyectados o instanciados dentro de los providers.
5. **Manejo de Errores Globales:** Implementar interceptores o listeners en providers para capturar excepciones de red, permisos o sesiones expiradas, y mostrar feedback consistente.

---

## 💾 Fase 6: Integración con Firestore (Base de Datos)
1. **Modelado de Datos:** Definir clases Dart puras que representen documentos de Firestore. Incluir métodos mentales de serialización/deserialización (`toMap`, `fromMap`) para mantener coherencia.
2. **Capa de Repositorios:** Crear servicios aislados para cada colección (`UserRepository`, `ServiceRepository`, `AppointmentRepository`). Centralizar consultas, escrituras y escuchas.
3. **Operaciones CRUD:**
   - Lectura: usar `snapshots()` para actualizaciones en tiempo real (ej. disponibilidad de horarios).
   - Escritura: transacciones para reservar citas (evitar double-booking), validando disponibilidad antes de commit.
   - Actualización/Eliminación: soft-delete o campo `estado` para mantener historial auditado.
4. **Consultas Eficientes:** Usar índices compuestos en Firestore para combinaciones frecuentes (`barbero_id + fecha`, `cliente_id + estado`). Evitar consultas sin `where` limitadas.
5. **Paginación y Caché:** Implementar `startAfterDocument` para listas largas (historial de citas). Usar caché local o `Provider` para evitar llamadas redundantes a la red.
6. **Seguridad en Cliente:** Nunca confiar únicamente en validación UI. Replicar validaciones críticas en el backend (Firebase Security Rules + Cloud Functions si es necesario).

---

## 🗺️ Fase 7: Navegación y Estructura de Pantallas
1. **Ruteo Centralizado:** Configurar rutas nombradas o declarativas con `go_router`. Definir redirecciones por estado de autenticación.
2. **Protección de Rutas:** Implementar guardias que verifiquen `currentUser != null` y `role` antes de permitir acceso a pantallas de gestión.
3. **Splash Screen:** Mostrar pantalla de carga inicial mientras se verifica `FirebaseAuth.instance.authStateChanges`. Redirigir según resultado.
4. **Navegación Inferior/Tab:** Organizar secciones principales: Inicio, Citas, Servicios, Perfil. Mantener estado de tabs al cambiar de pantalla.
5. **Deep Links & Notificaciones:** Preparar rutas para abrir citas específicas desde notificaciones push o enlaces externos.
6. **Accesibilidad y Localización:** Preparar estructura para `flutter_localizations` y `intl` si se planea multiidioma. Usar semántica en widgets clave.

---

## ✅ Fase 8: Pruebas, Optimización y Despliegue
1. **Pruebas Unitarias:** Validar lógica de providers, serialización de modelos y validaciones de formularios.
2. **Pruebas de Integración:** Simular flujos completos: registro → login → crear cita → verificar estado en Firestore → cancelar cita.
3. **Pruebas en Dispositivos Reales:** Verificar rendimiento en gama baja/media, consumo de memoria, manejo de pérdida de red y reconexión.
4. **Optimización:** Activar `--release` para builds finales. Usar `const` donde sea posible, evitar rebuilds innecesarios, comprimir assets, habilitar `split-debug-info`.
5. **Monitoreo en Producción:** Integrar Firebase Crashlytics y Analytics. Configurar alertas para errores críticos y métricas de retención.
6. **Generación de Builds:** 
   - Android: `flutter build appbundle`
   - iOS: `flutter build ipa`
   - Web: `flutter build web --release`
7. **Publicación:** Subir a Google Play Console, Apple App Store Connect y hosting estático (Firebase Hosting/Vercel/Netlify). Cumplir políticas de privacidad y permisos solicitados.
8. **Ciclo de Vida:** Mantener changelog, versionado semántico, y pipeline CI/CD básico (GitHub Actions/CodeMagic) para automatizar pruebas y builds.

---

## 📌 Anexo: Recomendaciones Críticas para "BARBERIA"
- **Seguridad:** Nunca expongas claves de API en el cliente. Usa reglas estrictas de Firestore. Valida roles tanto en cliente como en servidor.
- **Experiencia de Usuario:** Implementa estados vacíos amigables, loaders contextuales y feedback háptico/visual claro. La reserva de citas debe sentirse instantánea y confiable.
- **Escalabilidad:** Diseña los providers y repositorios para soportar futuras características (pagos, reseñas, múltiples sucursales, chat interno).
- **Documentación:** Mantén un `README.md` con arquitectura, flujos, decisiones técnicas y guías de contribución. Usa commits semánticos y PRs con revisión de código.
- **Legal:** Añade política de privacidad, términos de uso y consentimiento de datos, especialmente si se almacenan fotos o números de teléfono.

---
✅ **Siguiente paso recomendado:** Una vez validado y aprobado este plan, se puede proceder a la implementación fase por fase, comenzando por la configuración del entorno, dependencias en `pubspec.yaml` y la estructura de carpetas base. Si deseas, puedo generar un checklist ejecutable por sprints o un diagrama de flujo de arquitectura en texto plano para tu tablero de gestión.
