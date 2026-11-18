# STORE MOLANO — Cloudflare Pages con usuarios y administrador

Se mantienen los seis usuarios del ZIP, sus contraseñas y vencimientos, el panel
administrativo y el administrador fijo original. Los cambios siguen guardándose
mediante commits en el bloque const USERS de script.js en GitHub. No se usa D1.

## 1. Actualizar el repositorio

Descomprime este ZIP y sube el CONTENIDO de store-molano-main a la raíz del repositorio
que vas a conectar a Cloudflare. No subas el ZIP ni otra carpeta por encima.
Reemplaza los archivos antiguos y elimina, si existen:
- node_modules/
- netlify/
- netlify.toml
- package-lock.json antiguo

functions/, lib/, package.json y build.mjs deben estar en la raíz del repositorio.
No subas claves reales ni archivos .env o .dev.vars.
Si ya cambiaste usuarios después de generar el ZIP original, conserva esos cambios
en el bloque USERS de tu script.js actual al aplicar las rutas nuevas de esta versión.

## 2. Crear el proyecto Pages

Workers & Pages > Create application > Continue to Pages (si aparece) >
Import an existing Git repository. Selecciona el repositorio donde subiste este código.
No uses un proyecto Workers ni el modo de arrastrar archivos al panel.

| Configuración | Valor |
|---|---|
| Production branch | main |
| Framework preset | None / ninguno |
| Build command | npm run build |
| Build output directory | dist |
| Root directory | Vacío: raíz del repositorio |

Mantén activos los despliegues automáticos de main. El build copia los archivos
públicos a dist; Cloudflare compila functions/ por separado desde la raíz.
No necesitas flags Node.js ni un comando de despliegue wrangler en el panel Pages.

## 3. Las siete variables

Settings > Variables and Secrets > Add, entorno Production:

| Nombre exacto | Valor | Tipo |
|---|---|---|
| GMAIL_CLIENT_ID | Client ID de Google para este proyecto | Texto |
| GMAIL_CLIENT_SECRET | Client Secret del mismo cliente OAuth | Secreto / Encrypt |
| GMAIL_REFRESH_TOKEN | Refresh Token correspondiente al cliente y cuenta Gmail | Secreto / Encrypt |
| GITHUB_TOKEN | Token real con Contents: Read and write sobre el repositorio | Secreto / Encrypt |
| GITHUB_REPO | virtualformacion/store-molano | Texto |
| GITHUB_FILE_PATH | script.js | Texto |
| GITHUB_BRANCH | main | Texto |

El valor GITHUB_REPO debe coincidir con el repositorio que publica ESTA página.
Si creaste store-molano-cloudflare, usa virtualformacion/store-molano-cloudflare.
Si usas otra cuenta o nombre, escribe propietario/repositorio según corresponda.
El token debe tener acceso a ese repositorio y permiso de escritura de contenido.

Cada nombre y cada valor se escriben en campos separados, sin = ni comillas.
Después de guardar las variables, vuelve a desplegar desde Deployments.
No necesitas ADMIN_PASSWORD ni otras variables: el administrador permanece en
functions/api/updateusers.js, en FIXED_ADMIN, con su contraseña original.
También se aceptan REPO_OWNER, REPO_NAME, FILE_PATH y BRANCH como alternativa,
pero los nombres GITHUB_* de la tabla tienen prioridad.

## 4. Administrador y usuarios

Abre https://TU-PROYECTO.pages.dev/admin.html y entra con los mismos datos del
administrador original. No añadí un botón nuevo a la portada: el enlace flotante
original a WhatsApp permanece como estaba.

Importante: se conservó el vencimiento original del administrador: 2026-12-16.
Después de ese vencimiento el panel rechazará operaciones hasta que actualices
FIXED_ADMIN.expiresAt en el código. No se extendió silenciosamente esa fecha.

Prueba el flujo:
1. Entra al panel y comprueba los seis usuarios originales.
2. Crea un usuario de prueba con fecha futura.
3. Confirma que GitHub recibe un commit que actualiza script.js.
4. Espera el despliegue de ese commit en Cloudflare, recarga la página principal
   e inicia sesión con el usuario nuevo.
5. Edita y elimina ese usuario y confirma el mismo recorrido.
6. Comprueba la consulta de Gmail con un mensaje reciente admitido.

El panel lee los usuarios directamente desde GitHub; el login del navegador usa
el script.js publicado. Por eso los cambios aparecen primero en el panel y pasan
a la página cuando finaliza el despliegue. Las pestañas abiertas deben recargarse.
Cada modificación de usuarios que active un despliegue consume un build.
Si GitHub responde 409 por cambios simultáneos, vuelve a listar y repite la operación.

## 5. Alcance y validación

Se conservan los 50 mensajes de la búsqueda original, sus asuntos multilingües,
la ventana original de 10 minutos, los dos recorridos, la prioridad de códigos,
las funciones de extracción y los enlaces admitidos.
Se conservan el diseño, la lista USERS sin campos nuevos y la administración en GitHub.

Adaptaciones de ejecución:
- Rutas /api/getLastEmail y /api/updateusers en lugar de /.netlify/functions/.
- Entrada onRequest y variables env para Cloudflare.
- Gmail REST con renovación OAuth y descarga por lotes de hasta 20 mensajes,
  reutilizados en el segundo recorrido; no se añaden filtros de búsqueda.
- Decodificación Base64URL de Gmail (-, _, relleno opcional y UTF-8).
- Lectura de USERS como datos, sin ejecutar new Function, que no es compatible
  durante solicitudes de Cloudflare. Aquí emails es opcional, como requiere el original.
- Se elimina la pausa aleatoria artificial y los logs de cuerpos de correo.
- El navegador muestra el error de la API cuando existe.

Se mantiene el modelo original de autenticación: usuarios y contraseñas en
JavaScript público y comprobación de acceso en el navegador. El administrador
permanece en el repositorio según lo solicitado. No se añadió autenticación de
sesión en el servidor ni cambios de base de datos.

npm test ejecuta seis pruebas con respuestas simuladas. Compara Gmail con la
función original para todos sus asuntos y enlaces; comprueba Base64URL, fechas,
destinatarios, prioridad, posición 50 y errores. También compara el resultado
de listar, crear, editar, renombrar y eliminar usuarios con la función original,
incluyendo permisos de administrador, duplicados y conflictos de commit.
Verifica que se conserva exactamente el administrador y su vencimiento.

npm run build prepara los archivos públicos; Wrangler comprueba la compilación
para Cloudflare. La conexión a Gmail y GitHub reales debe probarse en tu cuenta:
no se usaron credenciales reales ni se modificó tu repositorio en línea.

Documentación oficial:
https://developers.cloudflare.com/pages/functions/get-started/
https://developers.cloudflare.com/pages/functions/bindings/
https://developers.cloudflare.com/pages/configuration/build-configuration/
