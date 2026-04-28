#  Configuración Completa de un Servidor Web con Apache y PHP  
##  Guía detallada paso a paso

Esta guía explica cómo instalar, configurar y verificar un servidor web funcional utilizando **Apache** y **PHP** en un sistema basado en Debian/Ubuntu.  
Incluye comandos, explicaciones, comprobaciones y notas importantes para evitar errores comunes.

---

##  1. Instalación de Apache

Apache será el servidor encargado de atender las peticiones HTTP.

###  Pasos a seguir
1. Actualizar los repositorios del sistema:

``` bash
sudo apt update
```

2. Instalar Apache:
``` bash
sudo apt install apache2 -y
```

3. Verificar que el servicio está activo:

``` bash
systemctl status apache2
```

4. (Opcional) Habilitar Apache para que arranque automáticamente:

``` bash
sudo systemctl enable apache2
```


###  Resultado esperado
- Apache instalado correctamente  
- Servicio activo y escuchando en el puerto 80  
- Acceso disponible desde el navegador  

---

##  2. Instalación de PHP

PHP permitirá ejecutar páginas dinámicas y procesar scripts del lado del servidor.

###  Pasos a seguir
1. Instalar PHP y el módulo necesario para integrarlo con Apache:


``` bash
sudo apt install php libapache2-mod-php -y
```

2. Verificar la versión instalada:


``` bash
php -v
```


3. (Opcional) Instalar módulos adicionales de PHP:

``` bash
sudo apt install php-mysql php-cli php-curl php-zip php-xml php-mbstring -y
```


###  Resultado esperado
- PHP instalado y funcionando  
- Apache capaz de interpretar archivos `.php`  
- Módulos adicionales disponibles si se instalaron  

---

##  3. Crear una página web de prueba

Para confirmar que PHP funciona correctamente, se creará un archivo de prueba.

###  Pasos a seguir
1. Crear el archivo `index.php` en el directorio raíz del servidor:

``` bash
sudo nano /var/www/html/index.php
```


2. Añadir el siguiente contenido:

``` bash
<?php
phpinfo();
?>
```


3. Guardar y cerrar el archivo.

###  Resultado esperado
- Archivo `index.php` creado  
- El servidor ejecuta código PHP sin errores  

---

##  4. Prueba de acceso desde el navegador

###  Pasos a seguir
1. Abrir un navegador web.
2. Acceder a:
   - `http://localhost`
   o
   - `http://IP_DEL_SERVIDOR`
3. Comprobar que aparece la página de información de PHP.

###  Resultado esperado
- Se muestra la página generada por `phpinfo()`  
- Apache y PHP funcionando correctamente  

---

##  5. Comprobaciones adicionales recomendadas

###  Verificar que Apache escucha en el puerto 80

``` bash
sudo ss -tulnp | grep apache
```


###  Comprobar el estado del firewall

``` bash
sudo ufw status
```


Si Apache está bloqueado:

``` bash
sudo ufw allow 'Apache'
```


###  Reiniciar Apache tras cambios

``` bash
sudo systemctl restart apache2
```


---

##  Checklist final

- [ ] Apache instalado  
- [ ] PHP instalado correctamente  
- [ ] Archivo `index.php` creado  
- [ ] Página accesible desde navegador  
- [ ] Firewall configurado  
- [ ] Servicio Apache activo  

---

##  Notas importantes

- Si no puedes acceder al servidor, revisa:
  - La IP del equipo  
  - El firewall  
  - Que Apache esté en ejecución  
- Si editas archivos de configuración, recuerda reiniciar Apache.  
- El archivo `index.php` debe tener permisos adecuados.
