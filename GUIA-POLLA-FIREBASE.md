# Guía — Conectar la Polla a Firebase (gratis)

Estado: el demo (`_polla-demo.html`) ya está listo y aprobado en diseño/flujo. Guarda solo en el
celular de cada persona (localStorage). Para que sea **real y compartida** (que todos vean el mismo
ranking de polleros y la racha), hay que enchufarla a un backend. Recomendado: **Firebase Firestore**
(plan gratis Spark: 50.000 lecturas y 20.000 escrituras por día — nuestro uso real es <2%).

Cuando vuelvas, con esto lo dejamos funcionando en minutos.

---

## Qué necesito de ti (2 cosas)

1. **La config de Firebase** (5 min, ver Paso 1–3). Me la pegas y yo hago el resto.
2. **Un token de GitHub** (Personal Access Token) para subir los cambios. Sin eso no puedo publicar.

Si prefieres, seguimos en modo demo por ahora y esto queda pendiente.

---

## Paso 1 — Crear el proyecto (gratis)

1. Entra a https://console.firebase.google.com y crea un proyecto (ej. `inpadel-polla`).
   Puedes desactivar Google Analytics.
2. Menú **Build → Firestore Database → Crear base de datos**. Elige región y parte en
   **modo de prueba** (después ponemos reglas).

## Paso 2 — Registrar la app web

1. En **Configuración del proyecto → Tus apps → Web (</>)**, registra una app.
2. Copia el objeto `firebaseConfig` que te muestra. Se ve así:

```js
const firebaseConfig = {
  apiKey: "AIza...",
  authDomain: "inpadel-polla.firebaseapp.com",
  projectId: "inpadel-polla",
  storageBucket: "inpadel-polla.appspot.com",
  messagingSenderId: "123...",
  appId: "1:123...:web:abc..."
};
```

## Paso 3 — Pegarme eso

Me pasas ese `firebaseConfig` y yo lo integro a `polla.html`.

---

## Modelo de datos (lo que voy a usar)

- **players/{deviceId}** → `{ name, points, streak, updated }`
  Un registro por celular. `deviceId` es un id anónimo que guardo en el navegador (sin login).
- **bets/{ronda__deviceId}** → `{ round, deviceId, name, oro:{n,c}, chupa:{n,c}, ts }`
  La polla de cada persona para esa fecha. Editable hasta el cierre.
- **rounds/{fecha}** → `{ closesAt, result:{oro,chupa}|null, settled }`
  La fecha, su cierre (miércoles 21:00) y el resultado cuando se carga.

## Cómo funciona (sin Cloud Functions, 100% gratis)

- **Sin login**: eliges tu nombre de la lista; guardo un candado por dispositivo. Editas hasta las 21:00.
- **Nombre duplicado**: si dos celulares distintos juegan con el mismo nombre, salta un aviso a todos
  ("detectamos 2 personas jugando como X") para evitar suplantación.
- **Liquidación automática**: cuando cargas el resultado de la fecha (desde una vista de organizador,
  protegida con `#admin`), el navegador lee todas las apuestas, calcula el pago de cada uno
  (cada acierto paga su cuota), actualiza puntos y racha, y marca la fecha como pagada.
  No usa funciones en la nube → no requiere tarjeta.
- **Racha 🔥**: acertar al menos 1 de los 2 mercados la mantiene; fallar (o no jugar) la corta.
- **Cuotas automáticas**: se calculan del ranking (favorito barato, tapados caros). Ya está hecho.

## Reglas de seguridad (las pongo yo, para que no borren datos ajenos)

```
rules_version = '2';
service cloud.firestore {
  match /databases/{db}/documents {
    match /players/{id} { allow read: if true; allow write: if true; }
    match /bets/{id}    { allow read: if true; allow write: if true; }
    match /rounds/{id}  { allow read: if true; allow write: if true; }
  }
}
```
Nota: para una polla de amigos con puntos (sin plata real) esto basta. Si algún día se abusa,
subimos una capa simple (código por WhatsApp o clave del organizador para liquidar).

---

## Deploy (cuando esté lista)

`polla.html` se publica junto a `index.html` en GitHub Pages y se enlaza desde el sitio
(un botón "🎟️ Polla de la fecha" arriba). Comandos (los corro yo con tu token):

```
git add polla.html index.html
git commit -m "Polla de la fecha (Firebase) + enlace desde el ranking"
git push
```
GitHub Pages actualiza solo en ~1 min. Queda en https://fabmarti15.github.io/inpadel-ranking/polla.html
