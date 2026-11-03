# Canal privado
 
PWA instalable para intercambiar mensajes de texto entre dos dispositivos (por ejemplo, una tablet y un celular Android) sin usar apps de mensajería. Los mensajes se cifran en el dispositivo antes de enviarse.
 
## Características
 
- Solo mensajes de texto, en tiempo real.
- Instalable como app (Android, Chrome) y con soporte offline para la interfaz.
- Cifrado de extremo a extremo con AES-256-GCM. La clave se deriva de una frase secreta compartida (PBKDF2-SHA256, 310.000 iteraciones) y se guarda en el dispositivo como `CryptoKey` no extraíble.
- Acceso restringido a dos cuentas mediante reglas de Firestore.
- Nombre personalizado por dispositivo y alias local para el otro dispositivo.
- La pantalla se difumina cuando la app pasa a segundo plano.
- Opciones para cerrar sesión, olvidar la clave y borrar todos los mensajes.
## Estructura
 
| Archivo | Descripción |
|---|---|
| `index.html` | Interfaz y lógica de la app (HTML, CSS y JS en un solo archivo) |
| `manifest.webmanifest` | Datos de instalación de la PWA |
| `sw.js` | Service worker (cachea solo archivos propios, nunca datos de Firebase) |
| `firestore.rules` | Reglas de seguridad de Firestore |
| `firebase.json` | Configuración de Hosting y Firestore |
| `icon-192.png`, `icon-512.png` | Íconos de la app |
 
## Requisitos
 
- Cuenta de Google y un proyecto en [Firebase](https://console.firebase.google.com) (plan Spark, gratuito).
- [Node.js](https://nodejs.org) y Firebase CLI: `npm install -g firebase-tools`.
## Configuración
 
1. **Crear el proyecto** en Firebase. Se recomienda un proyecto exclusivo para esta app.
2. **Authentication:** activar el proveedor *Correo/Contraseña*, crear dos usuarios (uno por dispositivo) y luego desactivar los registros nuevos.
3. **Firestore:** crear la base de datos.
4. **Reglas:** copiar los UID de los dos usuarios y reemplazar `UID_TABLET` y `UID_CELULAR` en `firestore.rules`.
5. **Configuración web:** en *Configuración del proyecto → Tus apps → Web*, copiar el objeto de configuración y pegarlo en `index.html` en la constante `firebaseConfig`.
## Despliegue
 
```bash
firebase login
firebase use --add        # elegir el proyecto
firebase deploy
```
 
Firebase devuelve una URL HTTPS (requisito para instalar una PWA).
 
## Instalación y uso
 
1. Abrir la URL en Chrome desde cada dispositivo.
2. Menú ⋮ → **Instalar app**.
3. Iniciar sesión con la cuenta de ese dispositivo.
4. Ingresar la **misma frase secreta** en ambos (mínimo 16 caracteres).
5. En **Ajustes**, definir el nombre de este dispositivo y el alias del otro.
## Seguridad
 
- Firestore solo guarda `uid`, `ts` y el contenido cifrado (`iv`, `ct`). El texto y el nombre del remitente viajan dentro del bloque cifrado.
- Las reglas permiten leer, crear y borrar únicamente a los dos UID autorizados; no se permite editar mensajes.
- La app aplica una política de seguridad de contenido (CSP) estricta y muestra los mensajes como texto plano.
- La configuración de `firebaseConfig` no es secreta: la protección depende de las reglas de Firestore y de la autenticación. No subas contraseñas ni la frase secreta al repositorio.
## Limitaciones
 
- No hay notificaciones push: los mensajes llegan en tiempo real solo con la app abierta.
- Si se pierde la frase secreta, los mensajes anteriores no se pueden recuperar.
- El cifrado no protege un dispositivo desbloqueado en manos de otra persona: usar bloqueo de pantalla.
- Plan Spark: 20.000 escrituras y 50.000 lecturas por día en Firestore, más que suficiente para uso personal.
## Desarrollo local
 
```bash
firebase emulators:start --only hosting
```
 
O servir la carpeta con cualquier servidor estático (`npx serve .`). El service worker requiere `localhost` o HTTPS.
