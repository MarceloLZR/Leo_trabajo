# 🚀 Guía de Implementación Completa

## 📋 Pasos para Implementar la Aplicación

### 1. Preparación del Entorno

```bash
# Instalar Firebase CLI globalmente
npm install -g firebase-tools

# Verificar instalación
firebase --version
```

### 2. Configurar Firebase

1. **Ir a Firebase Console:**
   - Visita: https://console.firebase.google.com/
   - Inicia sesión con tu cuenta de Google

2. **Crear nuevo proyecto:**
   - Clic en "Crear un proyecto"
   - Nombre: `campana-donacion-sangre`
   - Desactivar Google Analytics (opcional)
   - Clic en "Crear proyecto"

3. **Configurar Firestore:**
   - En el menú lateral: "Firestore Database"
   - Clic en "Crear base de datos"
   - Seleccionar "Comenzar en modo de prueba"
   - Elegir ubicación (recomendado: southamerica-east1)

4. **Configurar Hosting:**
   - En el menú lateral: "Hosting"
   - Clic en "Comenzar"
   - Seguir los pasos (se configurará después)

### 3. Estructura del Proyecto Local

```bash
# Crear carpeta del proyecto
mkdir campana-donacion-sangre
cd campana-donacion-sangre

# Inicializar Firebase
firebase login
firebase init

# Durante firebase init seleccionar:
# ✅ Firestore
# ✅ Hosting
# - Project: seleccionar el que creaste
# - Firestore rules: firestore.rules
# - Firestore indexes: firestore.indexes.json
# - Public directory: public
# - Single-page app: Yes
# - Automatic builds: No
```

### 4. Obtener Credenciales de Firebase

1. **En Firebase Console:**
   - Clic en ⚙️ (configuración del proyecto)
   - Pestaña "General"
   - Scroll hacia abajo: "Tus aplicaciones"
   - Clic en "</>" (Web)

2. **Registrar app:**
   - Nombre: `Campaña Donación Web`
   - Habilitar Firebase Hosting: ✅
   - Clic en "Registrar app"

3. **Copiar configuración:**
   ```javascript
   const firebaseConfig = {
     apiKey: "AIza...", // Tu API Key
     authDomain: "campana-donacion-sangre.firebaseapp.com",
     projectId: "campana-donacion-sangre",
     storageBucket: "campana-donacion-sangre.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abcdef"
   };
   ```

### 5. Configurar Archivos

**Crear estructura de carpetas:**
```bash
public/
├── index.html
├── manifest.json
└── icons/
```

**Archivo: `firestore.rules`**
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /groups/{groupId} {
      allow read: if true;
      allow create: if request.auth == null &&
                    resource == null &&
                    isValidGroupData(request.resource.data);
      allow update: if request.auth == null &&
                    resource != null &&
                    isValidDonationUpdate(request.resource.data, resource.data);
    }
    
    function isValidGroupData(data) {
      return data.keys().hasAll(['name', 'members', 'points', 'donations']) &&
             data.name is string &&
             data.name.size() > 0 &&
             data.name.size() <= 100 &&
             data.members is list &&
             data.members.size() > 0 &&
             data.members.size() <= 20 &&
             data.points == 0 &&
             data.donations is list &&
             data.donations.size() == 0;
    }
    
    function isValidDonationUpdate(newData, oldData) {
      let pointsDiff = newData.points - oldData.points;
      let donationsDiff = newData.donations.size() - oldData.donations.size();
      
      return newData.name == oldData.name &&
             newData.members == oldData.members &&
             pointsDiff > 0 &&
             pointsDiff <= 50 &&
             donationsDiff == 1;
    }
  }
}
```

**Archivo: `firestore.indexes.json`**
```json
{
  "indexes": [
    {
      "collectionGroup": "groups",
      "queryScope": "COLLECTION",
      "fields": [
        {
          "fieldPath": "points",
          "order": "DESCENDING"
        }
      ]
    }
  ]
}
```

**Archivo: `firebase.json`**
```json
{
  "hosting": {
    "public": "public",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ]
  },
  "firestore": {
    "rules": "firestore.rules",
    "indexes": "firestore.indexes.json"
  }
}
```

### 6. Personalizar la Aplicación

1. **Reemplazar credenciales en `public/index.html`:**
   - Buscar `firebaseConfig`
   - Reemplazar con tus credenciales reales

2. **Opcional - Personalizar colores:**
   ```css
   /* Cambiar colores principales */
   :root {
     --primary-color: #dc3545;    /* Rojo para sangre */
     --secondary-color: #667eea;   /* Azul degradado */
     --success-color: #28a745;     /* Verde éxito */
   }
   ```

### 7. Probar Localmente

```bash
# Servir la aplicación localmente
firebase serve

# La aplicación estará en: http://localhost:5000
```

### 8. Desplegar a Producción

```bash
# Desplegar todas las configuraciones
firebase deploy

# Solo hosting (más rápido para actualizaciones)
firebase deploy --only hosting

# Solo reglas de Firestore
firebase deploy --only firestore:rules
```

### 9. Configuración de Dominio Personalizado (Opcional)

1. **En Firebase Console:**
   - Hosting → "Agregar dominio personalizado"
   - Seguir instrucciones para configurar DNS

## 🔧 Comandos Útiles

```bash
# Ver logs de la aplicación
firebase functions:log

# Ver estado del proyecto
firebase projects:list

# Cambiar proyecto activo
firebase use proyecto-id

# Ver reglas actuales
firebase firestore:rules get

# Backup de datos (requiere configuración adicional)
gcloud firestore export gs://tu-bucket/backup

# Ver métricas de uso
firebase database:usage
```

## 🧪 Datos de Prueba

Para probar la aplicación, puedes crear estos grupos manualmente:

```javascript
// Grupo 1
{
  name: "Familia García",
  members: ["Juan García", "María García", "Pedro García"],
  points: 0,
  donations: [],
  createdAt: firebase.firestore.FieldValue.serverTimestamp()
}

// Grupo 2  
{
  name: "Equipo TechCorp",
  members: ["Ana López", "Carlos Ruiz", "Diana Torres", "Eduardo Vega"],
  points: 0,
  donations: [],
  createdAt: firebase.firestore.FieldValue.serverTimestamp()
}
```

## 🔒 Seguridad y Mejores Prácticas

1. **Reglas de Firestore:**
   - ✅ Ya configuradas para este caso de uso
   - ✅ Validación de datos en el servidor
   - ✅ Prevención de eliminación accidental

2. **Límites de Rate:**
   - Firebase maneja automáticamente
   - Plan gratuito: suficiente para 30 usuarios

3. **Monitoreo:**
   - Firebase Console → "Usage and billing"
   - Configrar alertas de cuota

## 🎯 Próximos Pasos Opcionales

1. **Notificaciones Push:**
   - Firebase Cloud Messaging
   - Avisos de nuevas donaciones

2. **Autenticación:**
   - Firebase Auth
   - Login con Google/Email

3. **Analytics:**
   - Google Analytics 4
   - Métricas de uso

4. **Exportar datos:**
   - Cloud Functions
   - Reportes automáticos

## 🆘 Solución de Problemas Comunes

### Error: "Firebase not initialized"
```javascript
// Verificar que las credenciales estén correctas
// Verificar que se carguen los scripts de Firebase
```

### Error: "Permission denied"
```bash
# Verificar reglas de Firestore
firebase firestore:rules get

# Redesplegar reglas
firebase deploy --only firestore:rules
```

### Aplicación no carga
```bash
# Verificar que esté en la carpeta public/
# Verificar firebase.json
# Probar localmente primero
firebase serve
```

### Datos no se guardan
- Verificar conexión a internet
- Verificar credenciales de Firebase
- Revisar consola del navegador (F12)

## 📞 Soporte

- **Documentación Firebase:** https://firebase.google.com/docs
- **Stack Overflow:** Etiquetar con `firebase` y `javascript`
- **GitHub Issues:** Para problemas específicos del código