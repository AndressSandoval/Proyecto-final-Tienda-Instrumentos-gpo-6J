¡Excelente iniciativa! Vamos a estructurar este proyecto bajo el estándar de **Agente Global (.agents)**. Esta metodología asegura que el desarrollo sea modular, escalable y que cada "habilidad" (diseño, código, scraping) esté bien definida.

---

## 1. Estructura de la Habilidad Agente Global: `.agents`

Primero, creamos la jerarquía de carpetas necesaria para organizar el conocimiento y los recursos del agente.

```text
proyectomusicman/
├── .agents/
│   ├── SKILL.md             # Definición de capacidades y flujos
│   ├── scripts/             # Automatizaciones de terminal
│   ├── ejemplos/            # Snippets de código de referencia
│   └── resources/           # Activos (iconos, fuentes, etc.)
├── lib/                     # Código fuente Flutter
├── assets/                  # Imágenes de instrumentos
└── pubspec.yaml             # Dependencias del proyecto
```

### Contenido de `.agents/SKILL.md`
Este archivo sirve como el "cerebro" del agente para entender qué debe hacer:
> **Skill Name:** MusicMan Store Manager
> **Capabilities:** > - **UI Design:** Implementación de pantallas con Material 3.
> - **Code Logic:** CRUD funcional con Firebase.
> - **Scraping:** Obtención de metadatos de instrumentos (estructuras base).

---

## 2. Prerrequisitos y Configuración del Entorno

Antes de escribir código, validamos que tu sistema esté listo. Abre tu terminal en **VS Code** o **Antigravity**.

### Paso A: Verificación e Instalación de Herramientas
Ejecuta estos comandos para asegurar que todo esté en orden:
1.  **Flutter:** `flutter --version` (Si no está, descárgalo en flutter.dev).
2.  **Firebase CLI:** `firebase --version`.
    * *Si no está instalado:* `npm install -g firebase-tools`.

### Paso B: Conectividad con Firebase
1.  **Login:** `firebase login` (Sigue el enlace en el navegador).
2.  **FlutterFire CLI:** Para vincular Flutter con Firebase de forma automática:
    `dart pub global activate flutterfire_cli`
3.  **Configurar Proyecto:** `flutterfire configure`
    * Selecciona tu proyecto de Firebase Console.
    * Selecciona **Android** y **Web/iOS** según necesites. Esto generará el archivo `firebase_options.dart`.

---

## 3. Configuración del Proyecto (`pubspec.yaml`)

Añade las dependencias necesarias para Firestore y el manejo de estado:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^3.0.0
  cloud_firestore: ^5.0.0
  cupertino_icons: ^1.0.6
```
Ejecuta: `flutter pub get`

---

## 4. Desarrollo del Proyecto: `tiendainstrumentos`

### Estructura de Archivos `.dart`
Generaremos los archivos siguiendo una secuencia lógica de navegación y datos.

#### A. Modelo de Datos (`lib/models/product_model.dart`)
```dart
class Product {
  String id;
  String nombre;
  double precio;
  String tipo; // Guitarra, Piano, etc.

  Product({required this.id, required this.nombre, required this.precio, required this.tipo});

  Map<String, dynamic> toMap() => {
    "nombre": nombre,
    "precio": precio,
    "tipo": tipo,
  };

  static Product fromSnapshot(Map<String, dynamic> data, String id) {
    return Product(
      id: id,
      nombre: data['nombre'] ?? '',
      precio: (data['precio'] ?? 0).toDouble(),
      tipo: data['tipo'] ?? '',
    );
  }
}
```

#### B. Servicio de Firestore (`lib/services/firebase_service.dart`)
Este archivo maneja el **CRUD**.

```dart
import 'cloud_firestore/cloud_firestore.dart';

final FirebaseFirestore _db = FirebaseFirestore.instance;

// CREATE
Future<void> addProduct(String nombre, double precio, String tipo) async {
  await _db.collection('productos').add({'nombre': nombre, 'precio': precio, 'tipo': tipo});
}

// READ (Stream para tiempo real)
Stream<QuerySnapshot> getProducts() {
  return _db.collection('productos').snapshots();
}

// UPDATE
Future<void> updateProduct(String id, String newNombre, double newPrecio) async {
  await _db.collection('productos').doc(id).update({'nombre': newNombre, 'precio': newPrecio});
}

// DELETE
Future<void> deleteProduct(String id) async {
  await _db.collection('productos').doc(id).delete();
}
```



#### C. Interfaz de Usuario (UI)

**Pantalla de Inicio (`lib/screens/home_screen.dart`):**
Contiene la lista de instrumentos y el botón para añadir.

```dart
import 'package:flutter/material.dart';
import '../services/firebase_service.dart';

class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("MusicMan Store"), backgroundColor: Colors.deepPurple),
      body: StreamBuilder(
        stream: getProducts(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          var docs = snapshot.data!.docs;
          return ListView.builder(
            itemCount: docs.length,
            itemBuilder: (context, i) {
              var data = docs[i].data() as Map<String, dynamic>;
              return ListTile(
                title: Text(data['nombre']),
                subtitle: Text("\$${data['precio']}"),
                trailing: IconButton(
                  icon: Icon(Icons.delete, color: Colors.red),
                  onPressed: () => deleteProduct(docs[i].id),
                ),
                onTap: () => _showEditDialog(context, docs[i].id, data['nombre']),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _showAddDialog(context),
        child: Icon(Icons.add),
      ),
    );
  }

  // Aquí irían los métodos _showAddDialog y _showEditDialog con formularios sencillos
}
```

---

## 5. Verificación de Funcionamiento

1.  **Firestore:** Asegúrate de que las reglas de seguridad en Firebase Console permitan lectura/escritura (en modo prueba para desarrollo).
2.  **Ejecución:** Conecta un emulador o dispositivo físico y lanza:
    `flutter run`

**¿Deseas que profundice en el código de los cuadros de diálogo (Dialogs) para completar el ciclo de "Crear" y "Actualizar" o prefieres configurar el Skill de Scraping primero?**
