# Despliegue de Tienda Demo con Git y Apache2

Esta guía explica cómo desplegar una tienda estática en una máquina virtual Ubuntu Server y abrirla desde Windows en `http://tienda.local`. Los cambios se hacen en Windows, se suben a GitHub y se aplican en Ubuntu con `git pull`.

**Repositorio de la práctica:** [ProfitCoder/Tienda_Despliegue](https://github.com/ProfitCoder/Tienda_Despliegue/)

## Requisitos

- **Windows:** Git, un editor de código, un navegador y permisos de administrador para editar el archivo `hosts`.
- **VirtualBox:** una máquina virtual Ubuntu Server con conexión a Internet, un adaptador de red en modo NAT y un usuario con acceso a `sudo`.
- **GitHub:** un repositorio público con permiso de escritura para publicar tus cambios. Puedes crear un *fork* del repositorio de la práctica.

## 1. Preparar la web en Windows

Si partes del ZIP de la actividad, descomprímelo y abre `index.html` en el navegador para revisar la tienda original. Crea un repositorio público en GitHub e inicialízalo con un README.

Si vas a utilizar el código de esta práctica, crea un *fork* desde GitHub. En ambos casos, clona tu repositorio desde PowerShell. Sustituye `TU_USUARIO` y, si has elegido otro nombre, `Tienda_Despliegue`.

```powershell
git clone https://github.com/TU_USUARIO/Tienda_Despliegue.git
cd Tienda_Despliegue
```

Si partes del ZIP, copia los archivos extraídos dentro de la carpeta clonada. `index.html` y `src/` deben quedar en la raíz del repositorio. Después sube la versión original a GitHub:

```powershell
git status
git add .
git commit -m "Subir la versión inicial de la tienda"
git push
```

Si has creado un *fork*, los archivos ya están en tu repositorio y puedes seguir con el siguiente paso.

## 2. Clonar el repositorio en Ubuntu

Instala las herramientas necesarias si todavía no están disponibles:

```bash
sudo apt update
sudo apt install git apache2 curl
```

Crea el directorio de la tienda y asigna su propiedad al usuario con el que has iniciado sesión. En la práctica se usó `profitcoder`. El directorio de destino debe estar vacío antes de clonar.

```bash
sudo mkdir -p /var/www/tienda
sudo chown "$(whoami)" /var/www/tienda
git clone https://github.com/ProfitCoder/Tienda_Despliegue.git /var/www/tienda
cd /var/www/tienda
ls -l
```

Usa en `git clone` la misma URL que usaste en Windows si quieres que Ubuntu reciba los cambios de tu repositorio o *fork*. La URL del ejemplo despliega la versión publicada en el repositorio de la práctica.

El listado debe mostrar `index.html`, `README.md` y el directorio `src/`.

## 3. Configurar los permisos

Desde `/var/www/tienda`, aplica el permiso `755` a los directorios y `644` a los archivos de la web. Los comandos excluyen el directorio interno `.git`.

```bash
find . -name .git -prune -o -type d -exec chmod 755 {} +
find . -name .git -prune -o -type f -exec chmod 644 {} +
stat -c '%a %n' . index.html README.md src
```

El resultado esperado es:

```text
755 .
644 index.html
644 README.md
755 src
```

## 4. Configurar el Virtual Host de Apache2

Crea el archivo de configuración en Ubuntu:

```bash
sudo nano /etc/apache2/sites-available/tienda.conf
```

Escribe esta configuración:

```apache
<VirtualHost *:80>
    ServerName tienda.local
    DocumentRoot /var/www/tienda
</VirtualHost>
```

En nano, guarda con **Ctrl+O**, confirma con **Enter** y sal con **Ctrl+X**. Si VirtualBox reserva el Ctrl derecho como tecla anfitrión, usa el **Ctrl izquierdo**.

Habilita el sitio y recarga Apache si la configuración supera la comprobación de sintaxis:

```bash
sudo a2ensite tienda.conf
sudo apache2ctl configtest && sudo systemctl reload apache2
```

Para verificar que Apache sirve el `index.html` correcto, compara la respuesta HTTP con el archivo de la tienda:

```bash
curl -fsS -H 'Host: tienda.local' http://127.0.0.1/ \
  | cmp - /var/www/tienda/index.html && echo 'Tienda OK'
```

Si aparece `Tienda OK`, el sitio ya funciona dentro de Ubuntu.

## 5. Abrir la tienda desde Windows

En VirtualBox, abre la configuración de red de la VM y añade esta regla en **Red → Adaptador NAT → Reenvío de puertos**:

| Campo | Valor |
| --- | --- |
| Nombre | Tienda |
| Protocolo | TCP |
| IP anfitrión | `127.0.0.1` |
| Puerto anfitrión | `80` |
| IP invitado | Vacía |
| Puerto invitado | `80` |

Si el puerto `80` está ocupado en Windows, puedes usar el `8080` como puerto anfitrión y acceder a `http://tienda.local:8080`. Para abrir la URL de la práctica sin indicar un puerto, utiliza el `80`.

Abre el **Bloc de notas como administrador**. Selecciona **Archivo → Abrir**, elige **Todos los archivos** y abre el archivo `C:\Windows\System32\drivers\etc\hosts`. Añade esta línea y guarda:

```text
127.0.0.1 tienda.local
```

Con la VM encendida, abre [http://tienda.local](http://tienda.local) en el navegador de Windows. Si estás preparando el informe, haz una captura de la tienda antes de modificar el título.

## 6. Modificar el título y subir el cambio

En la copia de Windows, abre `index.html` y añade tus iniciales al título visible de la tienda. Si también quieres mostrarlas en la pestaña del navegador, edita la etiqueta `<title>`. Si la versión del *fork* ya lleva las iniciales de la práctica, sustitúyelas por las tuyas.

Guarda el archivo y ejecuta en PowerShell desde la carpeta clonada:

```powershell
git diff -- index.html
git add index.html
git commit -m "Actualizar el título de la tienda"
git push
```

## 7. Actualizar Ubuntu y comprobar el resultado

En Ubuntu, descarga y aplica el cambio:

```bash
cd /var/www/tienda
git pull --ff-only
git log -1 --oneline
```

Vuelve a abrir [http://tienda.local](http://tienda.local) en Windows y pulsa **Ctrl+F5**. Comprueba que el título muestra tus iniciales. Para este cambio de HTML basta con actualizar el archivo.

Para el informe, guarda una captura del `git pull` correcto y otra del navegador con el título actualizado y `tienda.local` visible en la barra de direcciones.

## Incidencias y soluciones

| Incidencia | Solución |
| --- | --- |
| El primer `git push` devuelve `fetch first`. | Si inicializaste la carpeta con `git init` y GitHub ya tenía un commit, integra ambos historiales con `git fetch origin` y `git merge origin/main --allow-unrelated-histories`. Si aparece un conflicto en `README.md`, resuélvelo en el editor, ejecuta `git add README.md`, completa la fusión con `git commit` y repite `git push -u origin main`. |
| `chown` devuelve `invalid user`. | Ejecuta `whoami` para comprobar el nombre del usuario de Ubuntu y úsalo como propietario. |
| `git clone` devuelve `repository not found`. | Copia la URL HTTPS desde GitHub. El nombre original es `Tienda_Despliegue`, con guion bajo. |
| Nano devuelve `No such file or directory` al guardar. | Comprueba que la ruta sea `/etc/apache2/sites-available/tienda.conf`. |
| `curl` intenta conectar al puerto `443`. | Escribe `http://`. El Virtual Host de la práctica escucha en el puerto `80`. |
| Windows deniega el guardado de `hosts`. | Abre el Bloc de notas como administrador. Si careces de permisos, puedes probar la respuesta desde PowerShell con `curl.exe --resolve tienda.local:80:127.0.0.1 -I http://tienda.local/`. |
| `git pull` indica que sobrescribiría cambios locales en `index.html`. | Ejecuta `git diff HEAD -- index.html`. Si solo aparece un cambio de modo de `100644` a `100755`, corrígelo con `chmod 644 index.html` y repite `git pull --ff-only`. Si también hay cambios de contenido, guárdalos antes de actualizar. |


---

## Sobre Tienda Demo

Aplicación web estática de ejemplo para la práctica de despliegue de aplicaciones web de 2.º DAW.

### Estructura del repositorio

```text
Tienda_Despliegue/
├── index.html
├── README.md
├── Practica_Tienda_3.pdf
└── src/
    ├── css/
    │   └── style.css
    ├── img/
    │   └── logo.svg
    └── js/
        └── script.js
```

### Características

- Catálogo de seis productos ficticios.
- Filtros por categoría y búsqueda de productos.
- Carrito de demostración, sin procesamiento de pagos ni registro de pedidos.
- Diseño adaptable a móviles y escritorio.
- HTML, CSS y JavaScript sin dependencias de servidor.

### Probar la tienda localmente

Abre `index.html` en un navegador. Las fuentes externas requieren conexión a Internet. Si no se cargan, la tienda utiliza las fuentes alternativas del sistema.

### Desplegar y actualizar la tienda

La guía de arriba explica cómo configurar Apache con `/var/www/tienda` como `DocumentRoot` y acceder a la tienda en `http://tienda.local`.

`git clone` crea la primera copia del repositorio en Ubuntu. Después de publicar cambios en GitHub, actualiza esa copia con:

```bash
cd /var/www/tienda
git pull --ff-only
```

### Alcance de la demostración

La tienda es una maqueta educativa con datos ficticios. No incluye backend, autenticación, inventario real, pagos ni tratamiento de datos personales.
