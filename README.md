# ETL Normalizador — Firebase
**Evaluación 2 · Parte 2 · Arquitectura y Almacenamiento de Datos**

## Estructura del proyecto

```
etl_firebase/
│
├── index.html               ← Dashboard web (abre directamente en el navegador)
│
├── normalizer_famosos.py    ← Script ETL para DATOS2026-2.TXT (famosos)
├── normalizer_lugares.py    ← Script ETL para DATOS2026-3.TXT (lugares)
│
├── requirements.txt         ← Dependencias Python (solo si usas los scripts .py)
└── README.md
```

---

## Opción A: Dashboard Web (recomendado, sin instalar nada)

1. Abre `index.html` en tu navegador.
2. (Opcional) Configura Firebase en el panel superior.
3. Selecciona el dataset y haz clic en **Procesar**.
4. Descarga el JSON resultante o súbelo directamente a Firestore.

### Configurar Firebase en el dashboard
Ve a [console.firebase.google.com](https://console.firebase.google.com) →
tu proyecto → **Configuración del proyecto** → pestaña **General** → scroll
hasta "Tus apps" → agrega una app Web → copia los valores de `firebaseConfig`:

```
apiKey, authDomain, projectId, storageBucket
```

Pégalos en el formulario de la sección "🔥 Configuración Firebase" y haz clic en
**Conectar Firebase**.

> **Reglas de Firestore sugeridas para desarrollo:**
> ```
> rules_version = '2';
> service cloud.firestore {
>   match /databases/{database}/documents {
>     match /{document=**} {
>       allow read, write: if true;
>     }
>   }
> }
> ```

---

## Opción B: Scripts Python

### Instalar dependencias
```bash
pip install -r requirements.txt
```

### DATOS2026-2.TXT — Famosos y fechas de nacimiento
```bash
python normalizer_famosos.py DATOS2026-2.TXT
```

Genera: `output/famosos_normalizados.json`

**Transformaciones aplicadas:**
- Unifica fechas al formato chileno `DD-MM-YYYY` (soporta YYYY/MM/DD, YYYY-MM-DD, DD/MM/YYYY, DD-MM-YYYY, etc.)
- Elimina registros duplicados (mismo nombre + misma fecha)
- Calcula la **edad** en años cumplidos al día de hoy
- Activa el flag **es_cumpleanios** si hoy coincide con DD-MM del nacimiento
- Conserva fechas históricas (a.C., "alrededor de…") sin calcular edad

### DATOS2026-3.TXT — Lugares
```bash
python normalizer_lugares.py DATOS2026-3.TXT
```

Genera: `output/lugares_normalizados.json`

**Transformaciones aplicadas:**
- Elimina duplicados (mismo nombre + misma georeferencia)
- Separa en **3 colecciones** de Firestore:
  - `lugares` → `{id, nombre}`
  - `georeferencias` → `{lugar_id, nombre, latitud, longitud}`
  - `direcciones` → `{lugar_id, nombre_calle, numero_calle, ciudad_estado_provincia, pais}`

### Subir a Firebase desde Python
Edita el script y descomenta la línea al final:
```python
guardar_en_firebase(resultado, {"credential_path": "serviceAccountKey.json"})
```

Descarga el archivo de credenciales de servicio desde:
Firebase Console → Configuración del proyecto → **Cuentas de servicio** → Generar nueva clave privada.

---

## Colecciones en Firestore

| Colección        | Campos                                                         |
|------------------|----------------------------------------------------------------|
| `famosos`        | nombre, fecha_nacimiento (DD-MM-YYYY), edad, es_cumpleanios   |
| `lugares`        | id, nombre                                                     |
| `georeferencias` | lugar_id, nombre, latitud, longitud                            |
| `direcciones`    | lugar_id, nombre, nombre_calle, numero_calle, ciudad_estado_provincia, pais |

---

## Cambios respecto al proyecto original (Flask/SQLite)

| Antes (Flask + SQLite)          | Ahora (Firebase)                        |
|---------------------------------|-----------------------------------------|
| `app.py` (servidor Flask)       | `index.html` (app web autónoma)         |
| `normalizer.py` (comunas)       | `normalizer_famosos.py` + `normalizer_lugares.py` |
| SQLite `.db` local              | Firebase Firestore (cloud)              |
| `requirements.txt` con Flask    | `requirements.txt` con firebase-admin   |
| Ruta `/Normalizar` (POST)       | Procesamiento en el navegador (JS)      |
| Descarga `.txt`                 | Descarga `.json` + subida a Firestore   |
