Perfecto, vamos a construir esto bien organizado y funcional desde cero, siguiendo una secuencia lógica y clara. Te voy a dejar:

1. Estructura completa del agente `.agents`
2. Archivos base (SKILL.md, scripts, ejemplos, resources)
3. Preparación del entorno (Flutter + Firebase)
4. Creación del proyecto `proyectoferreteria`
5. Configuración Firebase + Firestore
6. CRUD completo de productos en Flutter
7. Pantallas UI básicas

---

# 🔹 1. Estructura del agente global `.agents`

```
.agents/
│
├── SKILL.md
│
├── scripts/
│   ├── check_env.sh
│   ├── setup_flutter.sh
│   └── firebase_setup.sh
│
├── ejemplos/
│   ├── crud_producto.dart
│   └── firebase_service.dart
│
└── resources/
    ├── firestore_rules.txt
    └── pubspec_base.yaml
```

---

# 🔹 2. Archivo principal: SKILL.md

```md
# Skill: Desarrollo Flutter + Firebase (Ferretería)

## Descripción
Este agente permite crear aplicaciones Flutter con integración a Firebase (Firestore),
incluyendo CRUD completo y UI básica.

## Capacidades
- Verificar entorno Flutter
- Configurar Firebase
- Crear estructura de proyecto
- Generar CRUD en Flutter
- Crear UI básica (pantallas + navegación)

## Requisitos
- Flutter instalado
- Flutterbase CLI (opcional)
- Cuenta Firebase activa

## Flujo de trabajo
1. Verificar herramientas
2. Configurar Firebase
3. Crear proyecto Flutter
4. Integrar Firestore
5. Implementar CRUD
6. Crear UI

## Skills incluidos
- skill_diseño → UI/UX Flutter
- skill_codigo → lógica Dart
- skill_scraping → opcional (datos externos)
```

---

# 🔹 3. Scripts de verificación

## `scripts/check_env.sh`

```bash
echo "Verificando Flutter..."
flutter doctor

echo "Verificando Firebase CLI..."
firebase --version

echo "Verificando Dart..."
dart --version
```

---

## `scripts/firebase_setup.sh`

```bash
echo "Iniciando login Firebase..."
firebase login

echo "Inicializando proyecto..."
firebase init
```

---

# 🔹 4. Preparación del entorno

En terminal:

```bash
flutter doctor
```

Si falta algo, te lo dirá.

Instalar Firebase CLI:

```bash
npm install -g firebase-tools
```

Login:

```bash
firebase login
```

---

# 🔹 5. Crear proyecto Flutter

```bash
flutter create proyectoferreteria
cd proyectoferreteria
code .
```

---

# 🔹 6. Configurar `pubspec.yaml`

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^2.0.0
  cloud_firestore: ^4.0.0
```

Luego:

```bash
flutter pub get
```

---

# 🔹 7. Configurar Firebase en el proyecto

1. Ir a Firebase Console
2. Crear proyecto
3. Agregar app Android
4. Descargar `google-services.json`
5. Colocarlo en:

```
android/app/google-services.json
```

---

# 🔹 8. Inicializar Firebase en Flutter

## `main.dart`

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'screens/home.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Ferretería',
      home: HomePage(),
    );
  }
}
```

---

# 🔹 9. Modelo de Producto

## `models/producto.dart`

```dart
class Producto {
  String id;
  String nombre;
  double precio;

  Producto({required this.id, required this.nombre, required this.precio});

  Map<String, dynamic> toMap() {
    return {
      'nombre': nombre,
      'precio': precio,
    };
  }

  factory Producto.fromMap(String id, Map<String, dynamic> map) {
    return Producto(
      id: id,
      nombre: map['nombre'],
      precio: map['precio'],
    );
  }
}
```

---

# 🔹 10. Servicio Firestore

## `services/producto_service.dart`

```dart
import 'package:cloud_firestore/cloud_firestore.dart';
import '../models/producto.dart';

class ProductoService {
  final CollectionReference productos =
      FirebaseFirestore.instance.collection('productos');

  Future<void> crearProducto(Producto producto) {
    return productos.add(producto.toMap());
  }

  Stream<List<Producto>> obtenerProductos() {
    return productos.snapshots().map((snapshot) =>
        snapshot.docs.map((doc) =>
            Producto.fromMap(doc.id, doc.data() as Map<String, dynamic>)
        ).toList());
  }

  Future<void> actualizarProducto(Producto producto) {
    return productos.doc(producto.id).update(producto.toMap());
  }

  Future<void> eliminarProducto(String id) {
    return productos.doc(id).delete();
  }
}
```

---

# 🔹 11. Pantalla principal (CRUD)

## `screens/home.dart`

```dart
import 'package:flutter/material.dart';
import '../services/producto_service.dart';
import '../models/producto.dart';

class HomePage extends StatelessWidget {
  final ProductoService servicio = ProductoService();

  final TextEditingController nombreCtrl = TextEditingController();
  final TextEditingController precioCtrl = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Ferretería")),
      body: Column(
        children: [
          TextField(controller: nombreCtrl, decoration: InputDecoration(labelText: "Nombre")),
          TextField(controller: precioCtrl, decoration: InputDecoration(labelText: "Precio")),

          ElevatedButton(
            onPressed: () {
              servicio.crearProducto(
                Producto(id: '', nombre: nombreCtrl.text, precio: double.parse(precioCtrl.text)),
              );
            },
            child: Text("Agregar"),
          ),

          Expanded(
            child: StreamBuilder<List<Producto>>(
              stream: servicio.obtenerProductos(),
              builder: (context, snapshot) {
                if (!snapshot.hasData) return CircularProgressIndicator();

                final productos = snapshot.data!;

                return ListView.builder(
                  itemCount: productos.length,
                  itemBuilder: (context, index) {
                    final p = productos[index];

                    return ListTile(
                      title: Text(p.nombre),
                      subtitle: Text("\$${p.precio}"),
                      trailing: Row(
                        mainAxisSize: MainAxisSize.min,
                        children: [
                          IconButton(
                            icon: Icon(Icons.delete),
                            onPressed: () => servicio.eliminarProducto(p.id),
                          ),
                          IconButton(
                            icon: Icon(Icons.edit),
                            onPressed: () {
                              nombreCtrl.text = p.nombre;
                              precioCtrl.text = p.precio.toString();

                              servicio.actualizarProducto(p);
                            },
                          ),
                        ],
                      ),
                    );
                  },
                );
              },
            ),
          )
        ],
      ),
    );
  }
}
```

---

# 🔹 12. Reglas básicas Firestore

## `resources/firestore_rules.txt`

```txt
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /productos/{docId} {
      allow read, write: if true;
    }
  }
}
```

---

# 🔹 13. Resultado final

Tendrás:

✔ App Flutter funcional
✔ CRUD completo
✔ Firebase conectado
✔ UI básica
✔ Sistema escalable

---

# 🔹 14. Siguiente paso (te recomiendo)

Después de esto puedes mejorar con:

* Login de usuarios
* Categorías de productos
* Carrito de compras
* Diseño más avanzado (Material 3)

---

Si quieres, en el siguiente paso puedo:
👉 Separarte el proyecto en arquitectura profesional (MVC / Clean Architecture)
👉 Mejorarte la UI tipo app real de tienda
👉 O agregarte autenticación con Firebase

Solo dime 👍
