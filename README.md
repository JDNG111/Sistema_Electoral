# 🗳️ Sistema de Votaciones Electrónicas

Sistema completo de votaciones en línea con autenticación segura, temporizador automático, contadores optimizados y generación de certificados. Desarrollado con Firebase, JavaScript vanilla y Bootstrap.

![Estado](https://img.shields.io/badge/Estado-Producción-success)
![Versión](https://img.shields.io/badge/Versión-2.0-blue)
![Licencia](https://img.shields.io/badge/Licencia-MIT-green)

---

## 📋 Tabla de Contenidos

- [Características](#-características)
- [Demo](#-demo)
- [Tecnologías](#-tecnologías)
- [Arquitectura](#-arquitectura)
- [Instalación](#-instalación)
- [Configuración](#-configuración)
- [Estructura de Datos](#-estructura-de-datos)
- [Reglas de Seguridad](#-reglas-de-seguridad)
- [Optimizaciones](#-optimizaciones)
- [Uso](#-uso)
- [API Reference](#-api-reference)
- [Contribuir](#-contribuir)
- [Licencia](#-licencia)

---

## ✨ Características

### 🔐 Seguridad
- **Autenticación con Google Sign-In** vía Firebase Authentication
- **Reglas de Firestore estrictas** que validan:
  - Solo incrementos de +1 por voto
  - Un solo candidato modificado por transacción
  - Validación de horarios en servidor
  - Prevención de doble votación
- **Listeners en tiempo real** sin consumo adicional de lecturas
- **Sanitización de emails** para IDs seguros

### ⏱️ Temporizador Automático
- **Cuenta regresiva en tiempo real** (días, horas, minutos, segundos)
- **Tres estados visuales:**
  - 🟡 **En Espera** - Antes del inicio
  - 🟢 **Activo** - Durante la votación
  - 🔴 **Cerrado** - Después del cierre
- **Recarga automática** al cambiar de estado
- **Validación doble**: Tiempo expirado + Estado manual = Resultados

### 📊 Optimización de Lecturas
- **Reducción del 99%** en operaciones de lectura de Firestore
- **Contadores agregados** en lugar de contar documentos
- **Datos estáticos** para candidatos y firmas (0 lecturas)
- **Listeners eficientes** con `onSnapshot`
- **Estimación**: 4,650 lecturas vs 79,500 (sistema tradicional)

### 📜 Certificados
- **Generación automática** de certificados de participación
- **Código de verificación** único por voto
- **Descarga en PNG** (vía html2canvas)
- **Descarga en PDF** (vía jsPDF)
- **Datos del votante** incluidos en el certificado

### 👨‍💼 Panel de Administración
- **Configuración de horarios** con selector de fecha/hora
- **Abrir/Cerrar votaciones** manualmente
- **Sincronizar contadores** (recalcular desde votos)
- **Resetear votos** (con doble confirmación)
- **Vista previa en tiempo real** del estado

### 📈 Resultados en Tiempo Real
- **Gráficos de barras** animados
- **Porcentajes calculados** automáticamente
- **Ordenamiento** por número de votos
- **Estadísticas**: Total votos, candidatos, participación
- **Acceso público** una vez cerradas las votaciones

---

## 🎥 Demo

### Estados del Sistema

**Antes de Inicio:**
```
┌─────────────────────────────────┐
│  ⏳ Las Votaciones Comenzarán  │
│     [03d 12h 45m 23s]          │
│  Inicio: 15/12/2025 08:00      │
└─────────────────────────────────┘
```

**Durante Votación:**
```
┌─────────────────────────────────┐
│  ✅ ¡Votaciones Activas!       │
│     [00d 05h 15m 42s]          │
│  Cierre: 15/12/2025 18:00      │
└─────────────────────────────────┘
```

**Resultados:**
```
┌─────────────────────────────────┐
│       📊 Resultados Oficiales   │
│  Total Votos: 1,234            │
│                                 │
│  1. Opción A: 45.2% (558)      │
│  2. Opción B: 32.1% (396)      │
│  3. Opción C: 22.7% (280)      │
└─────────────────────────────────┘
```

---

## 🛠️ Tecnologías

| Tecnología | Versión | Uso |
|------------|---------|-----|
| **Firebase SDK** | 12.6.0 | Backend (Auth + Firestore) |
| **Bootstrap** | 5.3.0 | Framework CSS |
| **Bootstrap Icons** | 1.10.0 | Iconografía |
| **html2canvas** | 1.4.1 | Captura de pantalla |
| **jsPDF** | 2.5.1 | Generación de PDFs |
| **JavaScript ES6+** | - | Lógica del cliente |

### Servicios de Firebase Utilizados
- ✅ **Firebase Authentication** (Google Provider)
- ✅ **Cloud Firestore** (Base de datos NoSQL)
- ✅ **Firebase Hosting** (opcional)

---

## 🏗️ Arquitectura

### Diagrama de Flujo

```
┌─────────────┐
│   Usuario   │
└──────┬──────┘
       │
       ▼
┌─────────────────────┐
│  Google Sign-In     │
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐      ┌──────────────────┐
│  Verificar Horario  │─────▶│  Temporizador    │
└──────┬──────────────┘      └──────────────────┘
       │
       ├──▶ Antes: Mostrar cuenta regresiva
       │
       ├──▶ Durante: Permitir votar
       │        │
       │        ▼
       │   ┌─────────────────┐
       │   │ Registrar Voto  │
       │   └────────┬────────┘
       │            │
       │            ▼
       │   ┌─────────────────────┐
       │   │ Incrementar         │
       │   │ Contadores (+1)     │
       │   └────────┬────────────┘
       │            │
       │            ▼
       │   ┌─────────────────────┐
       │   │ Generar Certificado │
       │   └─────────────────────┘
       │
       └──▶ Después: Mostrar resultados
```

### Estructura del Proyecto

```
voting-system/
│
├── index.html              # Aplicación principal (SPA)
│
├── firestore.rules         # Reglas de seguridad
│
├── README.md               # Este archivo
│
└── config/
    └── firebase-config.js  # Configuración (separar en producción)
```

---

## 📦 Instalación

### Requisitos Previos

- Cuenta de [Firebase](https://console.firebase.google.com/)
- Navegador moderno (Chrome, Firefox, Safari, Edge)
- Editor de código (VS Code recomendado)

### Paso 1: Clonar el Repositorio

```bash
git clone https://github.com/tu-usuario/sistema-votaciones.git
cd sistema-votaciones
```

### Paso 2: Crear Proyecto en Firebase

1. Ve a [Firebase Console](https://console.firebase.google.com/)
2. Crea un nuevo proyecto
3. Habilita **Authentication** → Google Sign-In
4. Crea una base de datos **Firestore**
5. Copia las credenciales del proyecto

### Paso 3: Configurar Credenciales

Edita `index.html` y reemplaza:

```javascript
const FIREBASE_CONFIG = {
  apiKey: "TU_API_KEY_AQUI",
  authDomain: "tu-proyecto.firebaseapp.com",
  projectId: "tu-proyecto-id",
  storageBucket: "tu-proyecto.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abcdef123456"
};

const ADMIN_EMAIL = 'tu-email@gmail.com'; // Email del administrador
```

### Paso 4: Configurar Reglas de Firestore

Copia las reglas desde `firestore.rules` a tu proyecto en Firebase Console.

### Paso 5: Servir Localmente

#### Opción A: Live Server (VS Code)
```bash
# Instalar extensión "Live Server"
# Click derecho en index.html → "Open with Live Server"
```

#### Opción B: Python
```bash
python -m http.server 8000
# Abrir http://localhost:8000
```

#### Opción C: Firebase Hosting
```bash
npm install -g firebase-tools
firebase login
firebase init hosting
firebase deploy
```

---

## ⚙️ Configuración

### Candidatos

Edita el objeto `CANDIDATOS_DEMO`:

```javascript
const CANDIDATOS_DEMO = {
  'id_unico_1': {
    nombre: 'Nombre del Candidato',
    foto: 'https://url-imagen.jpg',
    descripcion: 'Descripción breve'
  },
  'id_unico_2': {
    nombre: 'Otro Candidato',
    foto: 'https://url-imagen.jpg',
    descripcion: 'Otra descripción'
  },
  // ... más candidatos
  'voto_blanco': {
    nombre: 'VOTO EN BLANCO',
    foto: 'https://url-blanco.jpg',
    descripcion: '',
    ocultarNombre: false
  }
};
```

### Horarios

Los horarios se configuran desde el **Panel de Administrador**:

1. Inicia sesión con el email de admin
2. Ve al panel de administración
3. Selecciona fecha y hora de inicio
4. Selecciona fecha y hora de cierre
5. Click en "Guardar Horario"

### Número de Votantes Habilitados

Para habilitar votantes, agrega documentos a la colección `voters`:

```javascript
// En Firebase Console o vía script
elections/
  └── eleccion2025/
      └── voters/
          └── email_sanitizado/
              ├── email: "usuario@ejemplo.com"
              ├── nombre: "Nombre Completo"
              └── cedula: "123456789"
```

**Script de ejemplo para carga masiva:**

```javascript
// cargar-votantes.js
const votantes = [
  { email: 'user1@ejemplo.com', nombre: 'Usuario 1', cedula: '111' },
  { email: 'user2@ejemplo.com', nombre: 'Usuario 2', cedula: '222' }
];

votantes.forEach(async (votante) => {
  const emailSanitized = votante.email.replace(/[@.]/g, '_');
  await setDoc(doc(db, 'elections', ELECTION_ID, 'voters', emailSanitized), votante);
});
```

---

## 💾 Estructura de Datos

### Firestore Collections

```
elections/
│
└── eleccion2025/                    # Documento principal
    ├── open: boolean                # Estado de la votación
    ├── adminUID: string             # UID del administrador
    ├── createdAt: timestamp         # Fecha de creación
    ├── startTime: string (ISO)      # Inicio de votaciones
    ├── endTime: string (ISO)        # Cierre de votaciones
    ├── totalVotos: number           # Contador total (optimizado)
    └── resultados: {                # Contadores por candidato
          "opcion_1": 0,
          "opcion_2": 0,
          ...
        }
    │
    ├── votes/                       # Subcolección de votos
    │   └── email_sanitizado/        # Un documento por votante
    │       ├── Email: string
    │       ├── Candidato: string
    │       ├── Candidato_Nombre: string
    │       ├── Fecha: string
    │       └── Codigo_Verificacion: string
    │
    └── voters/                      # Subcolección de votantes habilitados
        └── email_sanitizado/
            ├── email: string
            ├── nombre: string
            └── cedula: string
```

---

## 🔒 Reglas de Seguridad

### Características de las Reglas

✅ **Validación de incrementos exactos (+1)**
✅ **Un solo candidato por voto**
✅ **Campos protegidos** (no modificables por usuarios)
✅ **Validación de horarios** en servidor
✅ **Prevención de doble votación**
✅ **Admin con permisos totales**

### Ejemplo de Regla Crítica

```javascript
// Validar que totalVotos incremente en exactamente +1
&& request.resource.data.totalVotos == resource.data.totalVotos + 1

// Validar que solo UN candidato tenga +1
&& validarIncrementoUnCandidato(
     resource.data.resultados, 
     request.resource.data.resultados
   )
```

---

## ⚡ Optimizaciones

### Lecturas de Firestore Reducidas

| Acción | Antes | Después | Ahorro |
|--------|-------|---------|--------|
| Ver candidatos | 5 lecturas | 0 lecturas | 100% |
| Ver resultados (1500 votos) | 1500 lecturas | 1 lectura | 99.93% |
| Login + verificar | 3 lecturas | 3 lecturas | 0% |
| **Total (50 usuarios)** | **79,500** | **4,650** | **94.15%** |

### Técnicas Aplicadas

1. **Contadores Agregados**
   ```javascript
   // ❌ Mal (leer todos los votos)
   const votos = await getDocs(votesCollection);
   const total = votos.size;
   
   // ✅ Bien (leer contador)
   const info = await getDoc(electionDoc);
   const total = info.data().totalVotos;
   ```

2. **Datos Estáticos**
   ```javascript
   // Candidatos hardcodeados en el cliente = 0 lecturas
   const CANDIDATOS = { /* ... */ };
   ```

3. **Listeners Eficientes**
   ```javascript
   // onSnapshot no consume lecturas adicionales
   // Solo la lectura inicial
   onSnapshot(electionRef, (snap) => {
     // Reaccionar a cambios sin leer de nuevo
   });
   ```

4. **Increment Atómico**
   ```javascript
   // Firebase maneja la concurrencia
   await updateDoc(ref, {
     totalVotos: increment(1)
   });
   ```

---

## 🎮 Uso

### Como Usuario

1. **Acceder al sistema**
   - Abrir la URL del sistema
   - Ver temporizador y estado actual

2. **Votar**
   - Click en "Acceder al Sistema"
   - Autenticarse con Google
   - Seleccionar candidato
   - Confirmar voto

3. **Descargar certificado**
   - Automático tras votar
   - Opciones: PNG o PDF
   - Código de verificación único

### Como Administrador

1. **Configurar elección**
   - Iniciar sesión con email de admin
   - Configurar horario de inicio y cierre
   - Guardar configuración

2. **Monitorear**
   - Ver estado en tiempo real
   - Temporizador automático
   - Contadores actualizados

3. **Gestionar votación**
   - Abrir/Cerrar manualmente
   - Sincronizar contadores si es necesario
   - Resetear votos (con confirmación)

4. **Ver resultados**
   - Automático al cerrar
   - Gráficos y estadísticas
   - Exportables

---

## 📚 API Reference

### Funciones Principales

#### `startTimer(info)`
Inicia el temporizador con actualización cada segundo.

**Parámetros:**
- `info` (Object): Datos de la elección con `startTime` y `endTime`

**Retorna:** `void`

#### `isVotingAllowed(info)`
Verifica si las votaciones están activas.

**Parámetros:**
- `info` (Object): Datos de la elección

**Retorna:** `boolean`

#### `drawCandidates(candidatos)`
Renderiza las tarjetas de candidatos en el DOM.

**Parámetros:**
- `candidatos` (Object): Objeto con candidatos

**Retorna:** `void`

#### `selectCandidate(card)`
Marca un candidato como seleccionado.

**Parámetros:**
- `card` (HTMLElement): Elemento DOM de la tarjeta

**Retorna:** `void`

### Eventos Firebase

```javascript
// Escuchar cambios en la elección
onSnapshot(electionRef, (snapshot) => {
  const data = snapshot.data();
  // Reaccionar a cambios
});

// Autenticación
onAuthStateChanged(auth, (user) => {
  if (user) {
    // Usuario logueado
  } else {
    // Usuario no logueado
  }
});
```

---

## 🤝 Contribuir

### Proceso

1. Fork el proyecto
2. Crea una rama (`git checkout -b feature/NuevaCaracteristica`)
3. Commit cambios (`git commit -m 'Agregar nueva característica'`)
4. Push a la rama (`git push origin feature/NuevaCaracteristica`)
5. Abre un Pull Request

### Guía de Estilo

- **JavaScript**: ES6+, camelCase
- **CSS**: BEM o utility-first
- **Commits**: Conventional Commits
- **Documentación**: JSDoc para funciones

### Roadmap

- [ ] Soporte multi-idioma
- [ ] Tema oscuro
- [ ] Exportar resultados a Excel
- [ ] Firma digital de certificados
- [ ] Modo offline (PWA)
- [ ] Dashboard de estadísticas avanzadas
- [ ] Encriptación de votos

---

## 📄 Licencia

Este proyecto está bajo la Licencia MIT.

```
MIT License

Copyright (c) 2025 [Julian]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## 👨‍💻 Autor

**[Julian]**

- GitHub: [@tu-usuario](https://github.com/JDNG111)
- Conoce más sobre mi: [Tu Perfil](https://jdng111.github.io/Portafolio/)

---

## 🙏 Agradecimientos

- Firebase por la infraestructura backend
- Bootstrap por el framework CSS
- La comunidad de código abierto

---

## 📞 Soporte

¿Encontraste un bug? ¿Tienes una sugerencia?

- 🐛 [Reportar Bug](https://github.com/jdng111/Sistema-Electoral/issues)
- 💡 [Solicitar Feature](https://github.com/jdng111/Sistema-Electoral/issues)
- 📧 [Contacto](navarroestudiante1010@gmail.com)

---

<div align="center">

**⭐ Si este proyecto te fue útil, considera darle una estrella ⭐**

Hecho con ❤️ y ☕

</div>
