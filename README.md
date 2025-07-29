## Sistema de Punto de Venta Tienda-Ropa


<div align="center">
  integrantes
López Cruz David Eduardo - Frontend
  
Salinas Montesinos Cintia Yadai - backend

Grupo: VSI

Equipo: 11

</div>

---
## Introducción

Este proyecto Laravel implementa un sistema completo para la gestión de usuarios y productos (ropa), con funcionalidades CRUD (Crear, Editar, Actualizar, Eliminar). Además, incluye el uso de migraciones, modelos, controladores, y rutas para una estructura modular, robusta y escalable.

El objetivo principal del sistema es demostrar cómo Laravel permite el desarrollo rápido y organizado de aplicaciones utilizando las mejores prácticas.

---
##  Objetivo del Proyecto

 - Autenticación con roles
 - CRUDs para usuarios y productos
 - Consumo de API desde Angular
 - Buen diseño UI/UX y seguridad básica
   
---
## Dependencias y Configuración

Para ejecutar correctamente el proyecto, se recomienda contar con los siguientes requisitos:

PHP: 8.2 o superior

Composer: 2.x

Base de datos: MySQL

Servidor web: Apache 

Framework: Laravel 12

Sistema operativo recomendado: Linux, Windows 10+ o macOS

Node.js y npm (solo si usas frontend con Vite/React/Vue):

Node.js 18.x 

npm 9.x o superior


---
## Estructura del Proyecto
El proyecto sigue una estructura MVC bien definida:

1.-Modelos: Representan las tablas en la base de datos y encapsulan la lógica de los datos.

2.-Controladores: Gestionan la lógica de negocio y conectan los modelos con las vistas o API.

3.-Migraciones: Definen y versionan las estructuras de las tablas de la base de datos.

4.-Rutas: Conectan las solicitudes HTTP con los métodos de los controladores.


---
## Gestión de Usuarios

La gestión de usuarios utiliza el modelo User y el controlador UserController. Este módulo implementa:

---
## Creación de Usuarios


Los usuarios se crean mediante el método store en el controlador. Este método valida y guarda los datos en la tabla users. Se espera recibir:

name: Nombre del usuario.
email: Dirección de correo único.
password: Contraseña cifrada.

Codigo para poder hacer las migraciones a la base de datos


``` PHP
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('nombre');
    $table->string('apellidos');
    $table->string('email')->unique();
    $table->string('password');
    $table->enum('rol', ['admin', 'cliente'])->default('cliente');
    $table->timestamps();
});

```


``` TS
crearUsuario(usuario: any): Observable<any> {
  return this.http.post(`${this.apiUrl}/usuarios`, usuario);
}

```
----
##  Visualización de Usuarios 


CODIGO

``` php
// Listar todos los usuarios
public function index()
{
    return response()->json(User::all());
}

// Mostrar un usuario específico por ID
public function show($id)
{
    $usuario = User::find($id);

    if (!$usuario) {
        return response()->json(['message' => 'Usuario no encontrado'], 404);
    }

    return response()->json($usuario);
}

```

Explicación:

index()

Utiliza User::all() para recuperar todos los usuarios registrados en la base de datos.

Retorna la respuesta en formato JSON, lista para consumir desde Angular.

show($id)

Usa User::find($id) para buscar un usuario por su ID.

Si no se encuentra, retorna un mensaje de error 404.

Si se encuentra, retorna los datos del usuario en JSON.


---

## Actualización de Usuarios


CODIGO

```php

public function update(Request $request, $id)
{
    $user = User::find($id);

    if (!$user) {
        return response()->json(['message' => 'Usuario no encontrado'], 404);
    }

```


User::find($id)

Localiza el usuario que se va a modificar.

$request->validate()

Asegura que los datos cumplan con las reglas antes de aplicar cambios.

Usa sometimes para permitir actualizaciones parciales.

Encriptación Condicional de Contraseña

Solo si se manda una nueva contraseña se actualiza con Hash::make().

$user->update($datos)

Aplica los cambios al registro correspondiente en la base de datos.


---


## Registro de Productos

Los productos se registran en la base de datos mediante el método store. Se espera recibir:

nombre
descripción
precio
stock
imagen
categoría
oferta 


CODIGO

```php
public function store(Request $request)
{
    $request->validate([
        'nombre'      => 'required|string|max:255',
        'descripcion' => 'required|string|max:500',
        'precio'      => 'required|numeric|min:0',
        'stock'       => 'required|integer|min:0',
        'imagen'      => 'required|url',
        'categoria'   => 'required|in:hombre,mujer',
        'oferta'      => 'boolean'
    ]);


```

---

## Edición de Productos


El método update permite modificar un producto existente en la base de datos. Recibe los nuevos datos a través de $request y los aplica al producto identificado por su id.


CODIGO


```php
public function update(Request $request, $id) {
    $producto = Producto::find($id);
    $producto->update($request->all());
    return $producto;
}


```

---

## Eliminación de Productos

CODIGO

```php

public function destroy($id) {
    $producto = Producto::find($id);
    $producto->delete();
    return response()->json(['message' => 'Producto eliminado']);
}

```

Producto::find($id): busca el producto por su ID.
$producto->delete(): elimina el producto de la base de datos.
return response()->json(...): devuelve un mensaje de confirmación.

Este método se usa en el API para resolver a solicitudes DELATE



---


## Migraciones: Creación de Tablas en la Base de Datos


Las migraciones garantizan que las tablas se definan y actualicen correctamente.


## Migración de Usuarios

La tabla user incluye
id
nombre
apellidos
email
password: Contraseña cifrada
rol
CODIGO

```PHP
Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('nombre');
    $table->string('apellidos');
    $table->string('email')->unique();
    $table->string('password');
    $table->enum('rol', ['admin', 'cliente'])->default('cliente');
    $table->timestamps();
});

```

## Migración de Productos

la tabla incluye
id
nombre 
descripción
precio
stock
imagen
categoría = HOMBRE /  MUJER
oferta



CODIGO

```PHP

Schema::create('productos', function (Blueprint $table) {
    $table->id();
    $table->string('nombre');
    $table->string('descripcion');
    $table->decimal('precio', 8, 2);
    $table->integer('stock');
    $table->string('imagen');
    $table->enum('categoria', ['hombre', 'mujer']);
    $table->boolean('oferta')->default(false);
    $table->timestamps();
});

```


---
##  Rutas: Conexión entre Componentes

FRONTEND (ANGULAR)

CODIGO

```ts

const routes: Routes = [
  { path: '', component: LoginComponent },
  { path: 'inventario', component: InventarioComponent }, // Vista admin
  { path: 'agregar-producto', component: AgregarProductoComponent },
  { path: 'editar-producto/:id', component: EditarProductoComponent },
  { path: 'tienda', component: TiendaComponent }, // Vista cliente
  { path: 'carrito', component: CarritoComponent },
  { path: 'mis-pedidos', component: MisPedidosComponent },
  { path: 'seguimiento/:id', component: SeguimientoPedidoComponent },
  { path: '**', redirectTo: '' }
];

```

BACKEND (LARAVEL) 

CODIGO

```PHP
// Autenticación
Route::post('/login', [AuthController::class, 'login']);

// Usuarios (solo admin)
Route::middleware(['auth:sanctum', 'role:admin'])->group(function () {
    Route::get('/usuarios', [UserController::class, 'index']);
    Route::post('/usuarios', [UserController::class, 'store']);
    Route::put('/usuarios/{id}', [UserController::class, 'update']);
    Route::delete('/usuarios/{id}', [UserController::class, 'destroy']);
});

// Productos (admin y cliente)
Route::middleware('auth:sanctum')->group(function () {
    Route::get('/productos', [ProductoController::class, 'index']);
    Route::post('/productos', [ProductoController::class, 'store']);
    Route::put('/productos/{id}', [ProductoController::class, 'update']);
    Route::delete('/productos/{id}', [ProductoController::class, 'destroy']);
});

// Pedidos (cliente autenticado)
Route::middleware('auth:sanctum')->group(function () {
    Route::post('/pedidos', [PedidoController::class, 'store']);
    Route::get('/mis-pedidos', [PedidoController::class, 'misPedidos']);
    Route::get('/pedido/seguimiento/{id}', [PedidoController::class, 'seguimiento']);
});


```


Las rutas vinculan las solicitudes HTTP con los métodos de los controladores.


```php
Route::get('/productos', [ProductoController::class, 'index']);        // Obtener productos
Route::post('/productos', [ProductoController::class, 'store']);       // Crear producto
Route::put('/productos/{id}', [ProductoController::class, 'update']);  // Editar producto
Route::delete('/productos/{id}', [ProductoController::class, 'destroy']); // Eliminar producto

```


 Rutas con Usuarios (solo administrador)

```php
Route::get('/usuarios', [UserController::class, 'index']);
Route::post('/usuarios', [UserController::class, 'store']);
Route::put('/usuarios/{id}', [UserController::class, 'update']);
Route::delete('/usuarios/{id}', [UserController::class, 'destroy']);

```

 Rutas para Usuario Normal 

```php
Route::middleware(['auth:sanctum'])->group(function () {
    Route::get('/productos', [ProductoController::class, 'index']);
    Route::get('/mis-pedidos', [PedidoController::class, 'misPedidos']);
    Route::post('/pedidos', [PedidoController::class, 'store']);
    Route::get('/pedido/seguimiento/{id}', [PedidoController::class, 'seguimiento']);
});

```


 Rutas protegidas

```php
Route::middleware(['auth:sanctum', 'role:admin'])->group(function () {
    Route::post('/usuarios', [UserController::class, 'store']);
});

```
---

## Acontinuacion te mostrare la ejecucion del navegador 

***en esta parte mostrarmos el login del administrador el cual puede agregarr, editar y eliminar productos***

<img width="993" height="766" alt="image" src="https://github.com/user-attachments/assets/c7242ea9-27e3-46b9-a2ba-6b69acfb11cd" />

imagen de todos los productos 

<img width="936" height="739" alt="image" src="https://github.com/user-attachments/assets/8aa27a06-a254-4e10-917f-7b1586c493ad" />

EDITAR

<img width="925" height="725" alt="image" src="https://github.com/user-attachments/assets/23001f0d-226e-4ea3-9f5e-81e0fee84aa4" />

AGREGAR

<img width="942" height="731" alt="image" src="https://github.com/user-attachments/assets/7b30811b-cd25-4616-adbf-1b2de5c05d50" />

 VER TODOS LOS PRODUCTOS COMPRADOS 

 <img width="991" height="511" alt="image" src="https://github.com/user-attachments/assets/58d3e087-6398-478e-a0e7-58029065dcf9" />
 
---
## Parte del cliente mostrada en navegador 
ROL CLIENTE

<img width="955" height="726" alt="image" src="https://github.com/user-attachments/assets/f1c28472-3772-43c6-9682-b6a00b275b29" />

si no tienes cuenta te puedes registrar 

<img width="1161" height="770" alt="image" src="https://github.com/user-attachments/assets/6c25e512-774f-4e7a-8126-4f4ee5abce08" />


INICIO

<img width="1016" height="773" alt="image" src="https://github.com/user-attachments/assets/69cb149c-2146-4761-a664-d096a86699c0" />


CATEGORIA 
EN ELLA SSE FILTRAN POR DOS CATEGORIAS HOMBRE Y MUJER 
 
<img width="1014" height="770" alt="image" src="https://github.com/user-attachments/assets/3f759c82-8ab4-4e44-9acb-ef63257ca683" />

CATEGORIA Hombre 

<img width="1121" height="734" alt="image" src="https://github.com/user-attachments/assets/9e278c57-7e91-428a-808e-1880d2ac019f" />

Al agregar una prenda/articulo nos vamos al carrito y vemos que se envio al carrito exitosamente 

<img width="1101" height="721" alt="image" src="https://github.com/user-attachments/assets/ed9427f6-c727-4ce1-9804-8feda32e25a6" />

para ello le damos continuar con el pago 

<img width="1114" height="734" alt="image" src="https://github.com/user-attachments/assets/7639a4c5-1b69-4408-a963-2923db677be6" />

 En automatico te da el correo exitosamente 
 <img width="1391" height="704" alt="image" src="https://github.com/user-attachments/assets/74a7223b-84e4-42f2-a2c6-77c256f74c6e" />

Te da un PDF 

<img width="1280" height="693" alt="image" src="https://github.com/user-attachments/assets/6dd66443-0ae6-4315-a635-4eef737d92d0" />

asi mismo no da ver el seguimiento del pedido

<img width="1140" height="513" alt="image" src="https://github.com/user-attachments/assets/17c98dab-f231-49db-851e-03553e0a5007" />

<img width="1097" height="615" alt="image" src="https://github.com/user-attachments/assets/fa512094-e99f-4a54-ac56-5dffaf428f8e" />

<img width="1117" height="587" alt="image" src="https://github.com/user-attachments/assets/cede43ad-e7b2-4e32-8a2c-f074fda370ef" />

Tambien tenemos el apartado de ofertas 

<img width="1179" height="783" alt="image" src="https://github.com/user-attachments/assets/df4a318d-f77f-4d58-98f5-96a67f47d28d" />



## Conclusión

El desarrollo del sistema web de tienda de ropa utilizando Angular en el frontend y Laravel en el backend permitió cumplir de manera efectiva con todos los requisitos funcionales y técnicos establecidos en el proyecto. Se implementaron las funcionalidades principales como el login con autenticación y control de roles (admin y cliente), gestión completa de productos y usuarios, sistema de carrito de compras, pedidos y seguimiento, así como validaciones y medidas de seguridad en ambas capas de la aplicación.


Gracias al uso de Laravel Sanctum, se garantizó una comunicación segura entre el frontend y la API, restringiendo el acceso a rutas según el tipo de usuario. El administrador cuenta con control total sobre los registros, mientras que el usuario cliente puede explorar productos, realizar pedidos y consultar el estado de sus compras. Además, el sistema cuenta con un diseño responsivo, navegación estructurada y modularización del código Angular, lo que facilita su escalabilidad y mantenimiento.

En conclusión, el proyecto no solo logró integrar correctamente las tecnologías requeridas, sino que también demostró la capacidad de construir aplicaciones web modernas, seguras y funcionales, cumpliendo con buenas prácticas de programación y diseño orientado al usuario
