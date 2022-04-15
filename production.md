![Logo de Cuycli](https://camo.githubusercontent.com/105b60ce28ec05ae23246c58638645c12cbdab6a1f5860309eb407e0aea90545/68747470733a2f2f696d6775722e636f6d2f72696c485678412e706e67)
# **Manual de despliegue en CPANEL**

### _Contenidos_

1. [Recursos necesarios](#1-recursos-necesarios)
2. [Configuración antes de subir los archivos](#2-configuracin-antes-de-subir-los-archivos)
3. [Creación base de datos y configuración de la misma](#3-creacin-base-de-datos-y-configuracin-de-la-misma)
4. [Subir archivos a CPANEL](#4-subir-archivos-a-cpanel)
5. [Configuración de nuestra aplicación en nuestro hosting](#5-configuracin-de-nuestra-aplicacin-en-nuestro-hosting)
6. [Configuración Laravel Passport y tablas de este](#6-configuracin-passport-y-tablas-de-este)

## 1. Recursos necesarios

- Para hacer un correcto despliegue en un servidor CPANEL se necesita lo siguiente
  - Contar con una cuenta FTP
  - Contar con un sistema de gestión conexiones FTP [Ver](https://filezilla-project.org/)
  - Contar con una base de datos en MYSQL

## 2. Configuración antes de subir los archivos

- Dirigirnos a nuestro archivo **_.env_** y modificaremos las siguientes variables de entorno

```bash
  APP_ENV = production

  APP_DEBUG = false

  APP_URL = https://{url de tu sitio}
```

- En el caso que estemos utilizando **_webpack mix_**, debemos ejecutar en la consola, en la raiz de nuestro proyecto en
  siguiente comando:
  `npm run production`
- Una vez hecho esto ejecutaremos también:
  `composer dumpautoload `
- Tras esto, estamos listos para comenzar con la subida de nuestra aplicación.

## 3. Creación base de datos y configuración de la misma

- En la opción de **_cpanel_** -> **_MySQL® Databases_** ingresa
- Crea la base de datos **_dabatase_tu_app_**
- Crea un usuario **_user_tu_app_** y asígnale la base de datos creada anteriormente
- Dirígete al archivo **_.env_** y modifica los siguientes valores
- En el archivo **.env.example.prod** puedes encontrar el ejemplo de como serían los datos esperados en el entorno de
  producción

```
 DB_CONNECTION=mysql
 DB_HOST=192.168.1.26
 DB_PORT=3306
 FORWARD_DB_PORT=3306
 DB_DATABASE=dabatase_tu_app
 DB_USERNAME=user_tu_app
 DB_PASSWORD=password_tu_app
```

## 4. Subir archivos a CPANEL

- Crear una carpeta en la raíz de nuestro servidor de CPANEL
- Quedando de esta manera

```bash
|-- (home/{tu_host})
    |-- etc
    |-- laravel (Nuestra carpeta con los archivos de Laravel)
    |-- logs
    |-- mail
    |-- public_ftp
    |-- public_html (Aqui deben ir los archivos de la carpeta public de Laravel)
    |-- ssl
    |-- tmp
```

- Este paso es requerido realizarlo si no se cuenta con un usuario o cuenta **_FTP_** es dividir nuestro proyecto en dos
  - Tomaremos las carpetas de la parte superior a la carpeta **_public_** y crearemos con estos un archivo **_.zip_**
  - Luego tomaremos las carpetas de la parte inferior a la carpeta **_public_** y crearemos con estos un archivo
    **_.zip_**
  - **Nota**: Nunca tomaremos la carpeta **_public_**
- Si se cuenta con un cliente **_FTP_** no se debe realizar el paso anterior solo debes subir todos los archivos de
  nuestra aplicación **Laravel** excepto la carpeta **_public_**
- En la carpeta **_public_html_** se deben subir los archivos de la carpeta **_public_** de Laravel

## 5. Configuración de nuestra aplicación en nuestro hosting

- El siguiente paso es **muy importante**, debemos modificar nuestro archivo **_index.php_** que se encuentra en la
  carpeta **_public_html_** para indicarle donde se encuentra nuestra aplicación **laravel**, donde están las rutas,
  vistas, controladores y demás.
- En **_cpanel_** daremos click derecho a nuestro archivo **_index.php_**, seleccionamos «edit» y modificaremos las
  siguientes líneas de código
- Estructura archivo **index.php**

```php
<?php

use Illuminate\Contracts\Http\Kernel;
use Illuminate\Http\Request;

define('LARAVEL_START', microtime(true));

/*
|--------------------------------------------------------------------------
| Check If The Application Is Under Maintenance
|--------------------------------------------------------------------------
|
| If the application is in maintenance / demo mode via the "down" command
| we will load this file so that any pre-rendered content can be shown
| instead of starting the framework, which could cause an exception.
|
*/

if (file_exists(__DIR__.'/../{nuestra carpeta de laravel}/storage/framework/maintenance.php')) {
    require __DIR__.'/../{nuestra carpeta de laravel}/storage/framework/maintenance.php';
}

/*
|--------------------------------------------------------------------------
| Register The Auto Loader
|--------------------------------------------------------------------------
|
| Composer provides a convenient, automatically generated class loader for
| this application. We just need to utilize it! We'll simply require it
| into the script here so we don't need to manually load our classes.
|
*/

require __DIR__.'/../{nuestra carpeta de laravel}/vendor/autoload.php';

/*
|--------------------------------------------------------------------------
| Run The Application
|--------------------------------------------------------------------------
|
| Once we have the application, we can handle the incoming request using
| the application's HTTP kernel. Then, we will send the response back
| to this client's browser, allowing them to enjoy our application.
|
*/

$app = require_once __DIR__.'/../{nuestra carpeta de laravel}/bootstrap/app.php';

$kernel = $app->make(Kernel::class);

$response = tap($kernel->handle(
$request = Request::capture()
))->send();

$kernel->terminate($request, $response);

```

- Ahora iremos a nuestra carpeta **laravel** (o la carpeta donde se encuentren nuestros archivos laravel ), a la
  siguiente ruta dentro de la carpeta «app/providers» y editaremos el archivo «AppServiceProvider.php» para editar su
  función **register**, añadiendo el siguiente código
- En nuestro caso la carpeta _nombre-carpeta-hosting_ sería **(home/{tu_host})**

```php
public function register()
{
  $this->app->bind('path.public',function(){
  return'/home/{tu_host}/public_html';
  });
}
```

## 6. Configuración laravel passport y tablas de este

- Para que nuestra API utilice correctamente la autentificación en la API se debe configurar lo siguiente
- Configurar token de Laravel Passport
- En nuestra gestión de consola de MySQL de nuestra base de datos debes hacer lo siguiente

  - Acceder a la tabla donde se almacenan las keys de acceso

    `select * from oauth_personal_access_clients;`

  - En la siguiente tabla se almacenan los token de acceso

    `select * from oauth_clients;`

## 7. Releases

### Versiones por número.

Algo común es realizar el manejo de versiones mediante 3 números: X.Y.Z y cada uno índica una cosa
diferente:

El primero (X) se le conoce como versión mayor y nos indica la versión principal del software. Ejemplo: 1.0.0, 3.0.0 El
segundo (Y) se le conoce como versión menor y nos indica nuevas funcionalidades. Ejemplo: 1.2.0, 3.3.0 El tercero (Z) se
le conoce como revisión y nos indica que se hizo una revisión del código por algún fallo. Ejemplo: 1.2.2, 3.3.4 Ahora
que conocemos el significado de cada número, viene una pregunta importante: ¿cómo sabemos cuando cambiarlos y cuál
cambiar?

Versión mayor o X, cuando agreguemos nuevas funcionalidades importantes, puede ser como un nuevo módulo o característica
clave para la funcionalidad. Versión menor o Y, cuando hacemos correcciones menores, cuando arreglamos un error y se
agregan funcionalidades que no son cruciales para el proyecto. Revisión o Z, cada vez que entregamos el proyecto.