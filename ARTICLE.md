# Cómo desplegar tu aplicación Laravel con Moss

En este artículo vamos a ver cómo desplegar una aplicación Laravel utilizando [Moss](https://moss.sh/es/). Moss es una herramienta útil para administrar tus aplicaciones y servidores de forma sencilla y segura. En particular, con Moss puedes desplegar fácilmente y con robustez a tus servidores de producción, testing, o cualquier otro entorno. Se trata de *zero-downtime deployments*, por lo que publicar una nueva release de tu aplicación no genera *downtime* para los usuarios activos que la estén usando.

Moss tiene un **Plan Gratis** muy amplio que incluye aplicaciones (aka "sitios") y servidores ilimitados. Si no tienes cuenta aún, puedes [registrate gratuitamente](https://moss.sh/es/signup/) y seguir las indicaciones de este artículo.


## Repositorio git

Para sacar partido de los despliegues sin *downtime* necesitamos que nuestra aplicación Laravel esté en un **repositorio git**. Si ya tienes un proyecto así, puedes saltarte esta sección y avanzar hasta la siguiente.

En este ejemplo vamos a crear un nuevo proyecto Laravel llamado "laraveles" usando `composer`. Una vez creado, inicializamos el repositorio git local, hacemos un *commit* de los cambios, y lo subimos al *origin* correspondiente. Para este artículo he creado un repositorio público en GitHub llamado "laraveles", por lo que mi *origin* es `git@github.com:fjros/laraveles.git` (sustitúyelo por el que corresponda en tu caso).

```
composer create-project --prefer-dist laravel/laravel laraveles
cd laraveles/
git init
git add .
git commit -m "Initial commit"
git remote add origin git@github.com:fjros/laraveles.git
git push -u origin master
```

Ya tenemos una aplicación Laravel mínima bajo control de versiones en un proveedor git. Ahora es momento de configurar el servidor web y el de base de datos para poder desplegarla.


## Crear la aplicación en Moss

Voy a asumir que ya tienes tu cuenta en Moss y al menos un servidor conectado. Si no es así, primero debes [conectar un servidor existente](https://docs.moss.sh/es/articles/3272248-conectar-un-servidor-existente) o [crear uno nuevo desde Moss](https://docs.moss.sh/es/articles/3272004-instanciar-un-nuevo-servidor-en-un-proveedor-cloud) (usando alguno de los proveedores con los que se integra en este último caso).

Una vez lo tengas, solo tienes que hacer click en "Nuevo sitio" y completar el formulario que se te presenta:

* Selecciona el servidor que has conectado.
* Elige **Laravel** como framework.
* Selecciona la **versión de PHP** que desees.
* Crea el nuevo usuario de servidor que ejecutará tu aplicación.
* Indica el nombre de dominio para tu aplicación. Si no estás en disposición de cambiar tus registros DNS en este momento, puedes usar un dominio de pruebas que Moss te proporciona.
* Indica el nombre del sitio. Este nombre te permite identificar de forma unívoca tu aplicación en el servidor, aunque más adelante cambies el nombre de dominio del sitio.
* Indica si quieres que Moss configure HTTPS. Para ello proporciona tu propio certificado o dile a Moss que emita y renueva certificados gratuitos Let's Encrypt automáticamente. Si estás haciendo pruebas puedes omitir este paso y configurar el certificado más adelante (especialmente si usas el [dominio de pruebas](https://docs.moss.sh/es/articles/2333466-visita-tu-sitio-web-usando-la-direccion-ip-de-tu-servidor) sugerido por Moss).
* Crea una nueva conexión de base de datos para tu aplicación (si es que necesita una). Para ello debes indicar el nombre de la nueva base de datos y del usuario de base de datos, además de su contraseña. Esto creará dicha base base de datos, el usuario, y los permisos asociados.

Puedes comprobar lo sencillo que es todo este proceso en el siguiente vídeo:
[Crear sitio](media/exported-laraveles-crear-sitio.mp4)

Moss inicia una **operación** en segundo plano para instalar todo el software necesario y realizar las configuraciones necesarias. Pero aún no le hemos dicho dónde está tu código para poder desplegar la aplicación. Mientras la operación anterior sigue en progreso, podemos continuar con ese paso.


## Indicar repositorio git de la aplicación

Si te diriges a la pestaña "Despliegues" verás la opción de "añadir" un **repositorio git** a la aplicación. Si aún no has asociado Moss con tu cuenta de GitLab, GitHub, o Bitbucket, éste es el momento de hacerlo (también puedes seleccionar la opción "Custom" si usas otro proveedor git).

Una vez hayas indicado el proveedor donde tienes el repo, deberás seleccionar el usuario al que pertenece dicho repo e indicar su nombre. Consejo: si tienes muchos repositorios, el filtro de búsqueda te resultará útil para encontrarlo rápidamente.

Por último, indica la rama (*branch* o *tag*) que Moss debe desplegar cuando le indiques. Normalmente será `master`, pero puedes adoptar la convención que prefieras (p.ej. `deploy`, `trunk`, etc).

En el vídeo puedes hacerte una mejor idea del proceso:
[Repositorio git](media/exported-laraveles-repo-git.mp4)

Una vez Moss haya realizado las configuraciones necesarias, ¡podemos desplegar la aplicación!

## Desplegar la aplicación

### 1-Click

Para desplegar la aplicación sólo necesitas hacer click en "Desplegar". Cuando la operación correspondiente haya acabado, puedes visitar el dominio de tu aplicación:
[Desplegar aplicación](media/exported-laraveles-desplegar.mp4)

Quizá te estés preguntando cómo ha funcionado el despliegue si no le has dicho a Moss qué comandos ejecutar para ello, ni le has pasado el contenido del fichero `.env`. La respuesta es sencilla: Moss los ha auto-generado por ti.

Al indicarle que se trata de una aplicación Laravel, Moss ha creado un script de despliegue estándar que puedes editar en cualquier momento desde el propio panel de Moss:

```
composer install --prefer-dist --no-dev --no-interaction
php artisan migrate --force
```

Además, Moss ha proporcionado un fichero `.env` con las variables apropiadas, incluyendo las de conexión a la base de datos. Al igual que antes, puedes editar dicho fichero desde Moss cada vez que lo necesites.

### Push-to-deploy

Si desplegar con un click como hemos hecho te parece útil, ¡desplegar automáticamente te gustará aún más!

Sólo tienes que habilitar esta funcionalidad en la pestaña "Flujo de despliegue" y cada vez que ejecutes `git push` en el repositorio/rama asociados, Moss iniciará un despliegue automáticamente.

[Auto-desplegar aplicación](media/exported-laraveles-auto-desplegado.mp4)


## Ventajas adicionales de usar Moss

Como has comprobado, desplegar tu aplicación Laravel con Moss es muy sencillo. Pero además tiene multitud de ventajas añadidas:

* Ya hemos comentado que todos los despliegues vía git en Moss son sin downtime. Esto también quiere decir que si tu nuevo despliegue falla por algún motivo, tus usuarios no se verán afectados porque se sigue sirviendo en todo momento tu versión anterior.
* Además recibirás una notificación por correo electrónico o Slack cuando tus despliegues finalicen correctamente o tras un error.
* Puedes crear una base de datos en un servidor remoto directamente desde Moss, y éste se encarga de usar la URL apropiada y abrir el firewall sólo a tu(s) servidor(es) de aplicación.
* Moss realiza muchas configuraciones destinadas a mejorar la seguridad de tus servidores.
* Sólo instala el software que realmente se necesita en cada servidor.
* Tiene soporte de equipos en todos sus planes (colaboradores ilimitados).
* Como has podido observar en los vídeos, ofrece visibilidad en tiempo real acerca de las operaciones que se ejecutan sobre tus servidores o sitios. Además puedes consultar el log de cada operación, lo que ayuda mucho a solucionar problemas cuando se presentan.
* Por $5/mes (USD) como máximo, puede monitorizar un servidor y los sitios que aloja y enviarte una alerta si algo empieza a ir mal.
* [Centro de Ayuda](https://docs.moss.sh/es/) en español. Los planes superiores también incluyen soporte vía chat.
* ¡Y más! Échale un vistazo porque ofrece más características y se añaden nuevas periódicamente.


## Conclusión

En este artículo hemos visto cómo desplegar tus aplicaciones Laravel sin downtime es muy sencillo con el Plan Gratis de Moss. Puedes usarlo para todas tus aplicaciones y servidores, y pasar a un plan de pago si en algún momento necesitas utilizar integraciones adicionales, desplegar con mayor frecuencia, o acceder al equipo de soporte.
