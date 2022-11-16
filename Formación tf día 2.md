
#Objetivo. Crear un contenedor con NGINX
Para ello:
1. Necesitamos una imagen, que se la solicitaremos a un proveedor  
2. Creamos el contenedor. A la hora de crear ese contenedor, puede ser necesario suministrar (volúmenes, redirecciones de puertos, variables de entorno…)

Para establecer los proveedores requeridos en terraform, se utiliza la siguiente sintaxis:
```
terraform {
  required_providers {
  }}
```
El listado de proveedores de terraform se consulta en el registry de terraform. 
https://registry.terraform.io

Docker provider se utiliza para interactuar con Docker y crear contenedores e imágenes. Existe un botón "use provider" que nos proporciona el código que permite
usar el provider en nuestra configuración de terraform

```
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
      version = "2.23.0"
    }
  }}
```
Cuando utilizamos un proveedor podemos suministrar la versión, si queremos una versión mínima o asegurarnos que usamos una concreta para que
lo que queremos usar funcione. A veces no es necesario especificar la versión.

Muchos provider que existen en el registry de terraform no son oficiales, siempre está bien verificar el número de descargas y la popularidad del provider para verificar cuánto se usa y cuan recomendable es.

En required providers especificamos qué providers son REQUERIDOS, obligatorios, pero existe otro bloque, provider, en el que especificamos la configuración adicional del proveedor.

```
provider "docker" {
 #Configuración adicional del proveedor. En este caso no resulta necesario.
}
```
Dentro de la página del provider en el registry de terraform, en la barra lateral existe un apartado de "Resources". En este se especifican los recursos que se pueden solicitar a dicho proveedor.

Para crear un nuevo recurso se utiliza el bloque de resource, cuya sintaxis sigue el modelo:

``` 
resource "tipo_de_recurso" "nombre_recurso" { #El nombre del recurso se utiliza como una variable, es un nombre custom que le damos para poder identificarlo en otras partes de nuestro script, de ser necesario
}

#Ejemplo:
resource "docker_container" "contenedor" {
  name = "minginx"
  image = docker_image.imagen.image_id
}

resource "docker_image" "imagen" {
  name = "nginx:latest" #imagen:tag, descarga la última imagen disponible de nginx
}
```

Podemos consultar en la página de cada recurso (recordemos, dentro de la página del provider, en el listado de recursos disponibles), qué características se pueden configurar de dicho recurso.
Es obligatorio especificar los elementos "Required"
Existen una serie de propiedades "Read-Only" para cada recurso, la info que tenemos disponible una vez que se haya creado el recurso.   

Por ejemplo, cuando especificamos que tenenos una imagen de nginx, no sabemos con qué nombre (id) se va a crear, por ello, se utiliza una referencia del recurso:  
docker_image.imagen.image_id --> Del objeto de tipo docker_image que se llama imagen, coge su image_id
image = docker_image.imagen.image_id
variable = propiedad de esa variable


--- 
#Orden de creación de los elementos de terraform:
Los elementos definidos en un script de terraform se crean comenzando por los que no tienen dependencias.
Terraform de forma automática genera un árbol de dependencias entre los recursos y los crea en consecuencia a ello.
No importa cuál se especifique primero en los ficheros, en el momento en que existe una dependencia, como en el caso del contenedor, que tiene una dependencia en su valor
de image, al referenciar ahí el recurso docker_image.imagen, da igual el orden o si tenemos los recursos separados en distintos archivos, terraform decidirá en qué orden se irán creando las cosas en función de las dependencias.  

--- 
#Ejecución de los scripts  
Ubicados en el path en el que se encuentra el archivo main.tf, lanzamos los comandos de terraform pertinentes  

#Comandos:  
**terraform init** Comprueba qué proveedores se han especificado y en caso de que no estén disponibles, procede a descargarlos e instalarlos.
Se crea un archivo oculto .terraform.lock.hcl y una carpeta .terraform con el programa (un binario) que se conectará con el proveedor de recursos, la licencia y un changelog
Nos hemos decscargado este fichero, este programa, que usará terraform para comunicarse con docker  

El fichero .terraform.lock.hcl contiene una serie de hashes, una huella con la versión del proveedor e info relevante.
Ninguno de estos elementos se debería subir al repo de git.  

**terraform validate** Valida la sintaxis de los archivos    
**terraform plan**  
Cuando ejecutamos un plan, aparece un + delante de cada recurso que se va a crear nuevo, y un - de los que se van a eliminar.
Dentro de estos recursos, sus datos específicos (ej: id, name, cpu_shares..) aparecen especificados si se han establecido de antemano o "(known after apply)" en caso de que se generen tras crear el recurso, como así ocurre con el id de la imagen.
  
**terraform apply** Pide confirmación y aplica los cambios

--- 
#Sintaxis de terraform  

#Tipos de datos que se pueden asignar a las propiedades de los recursos:

**bool**   *Valor lógico*  

**string**  *Texto, su valor se define siempre entre ""*  

**number** *Número*  

**list(bool|string|number)** *Lista de cosas, de tipo boolean, string o number. Se puede acceder a cada elemento en función de su posición*  

**set(bool|string|number)** *Lista de cosas, la diferencia es que no es indexable. NO se puede acceder a cada elemento en función de su posición, sólo se puede iterar*  

**map(bool|string|number)** *Mapa, diccionario, array asociativo, conjunto pares clave-valor*    

**block list** o **block set** *Usa un nested schema, ese elemento tiene una serie de propiedades, algunas obligatorias y otras opcionales. 
Esta propiedad NO lleva un igual, se define de forma diferente al resto de propiedades  
```
nombre_variable_block_list {
  propiedad1 = valor1
  propiedad2 = valor2
}
```

Cuando tenemos un block set o block list, para cada bloque repetimos el elemento principal

```
ports {
  external = 8080
  internal = 80
}
ports {
  external = 8443
  internal = 443
}
```

**any** *Cualquier cosa*  

**object** *Mapa en el qeu hay una claves predeterminadas, no puedo meter cualquier cosa, existen una serie de claves ya predefinidas que debo usar y especificar.*  
  
  
---
#PROCESO
Una vez definido el script de terraform, ¿quién lo ejecutará? ¿se ejecutará manualmente? NO  
Lo ejecutará un servidor de automatización: Jenkins, Azure DevOps..   
Una vez ejecutado, y creada la infraestructura, quizás sea necesario configurar la infra desplegada con Ansible, o ejecutar unos test...  

Pero una vez ejecutado el script de terraform, terraform crará cosas, y necesitaremos información de los recursos que se han creado, datos de ellos, como la ip de la máquina, para poder lanzarle test, o su id para poder saber cuál vamos a configurar. Para ello se utilizan los outputs


---  

OUTPUTS  
Permiten extraer información de los recursos que se han creado y trasladarsela a otros recursos o a otros programas como, por ejemplo, Ansible.

Se especifican en un fichero outputs.tf  

```
output "direccion_ip" {
  value = docker_container.contenedor.ip_address
}

#Extraemos la información a posteriori de la ip del contenedor de forma sencilla, sin necesidad de meternos en el JSON
```
Para saber qué información podemos extraer de cada recurso, dentro de la página del registry del recurso de ese provider, consultamos el apartado Read-only.

Esta información se puede recuperar a posteriori mediante el comando **terraform output** en formato JSON
```
terraform output (--json | --raw) NOMBRE

terraform output #Devuelve todos los output generados
terraform output direccion_ip #Devuelve unicamente la información del output seleccionado
terraform output direccion_ip >> fichero.txt
```

---
TFSTATE
Cada vez que se realiza un apply de un fichero de configuración de infraestructura, genera un fichero .tfstate  
Se encuentra en formato JSON, detalla todos los datos de los recursos que se han creado

Apartados interesantes del tfstate:  
```
{
  "version" : 4, #Número de veces que se ha ejecutado el apply
  "terraform_version" : 1.3.4, #Versión de terraform utilizada
  "outputs" : {},
  "resources" : { #Listado de recursos desplegados
  }
}
```

En el apartado de recursos, puede consultarse la información de los recursos creados (known after apply), pero no se debe intentar obtener dicha información de esta forma, porque puede cambiar en un futuro.

---
VARIABLES  

Una vez que tenemos nuestro main.tf, si tenemos un fichero de 500 líneas, tener que entrar a modificar este fichero y cambiar un dato concreto, es incómodo. Se crea para ello otro fichero que parametrizaremos, usando variables.

Se utiliza un fichero llamado variables.tf para crear esas variables. Se podrían crear directamente en el main.tf las variables, pero las buenas prácticas lo desaconsejan totalmente.

De una variable podemos definir:  
**description** Información para saber qué define esa variable.
**type** Fuerza el tipo que debe tener esa variable.
**default** Un valor por defecto.  
**nullable** True/false, especifica si se le puede asignar un valor null. NO lo pone por defecto, en ausencia de valor.
Esta propiedad no tiene nada que ver con el valor que tiene la variable. Un dato desasignado no se asignará a null, esto sólo indica si a esa variable se le puede poner valor null. Null es un valor diferente a desasignado  

Terraform NUNCA va a funcionar con una variable desasignada. O da un error o nos pide un valor en tiempo de ejecución para esa variable.

Existen dos formas de suministrar el valor de una variable:

-Durante la ejecución del apply, usando la flag --var nombre_variable=valor  
NUNCA se debe hacer, salvo excepcionales casos, porque no deja huella, no permite controlar versiones, pierde automatizacion..  Pero, para claves o información confidencial, esa es mejor que la tengamos almacenada en un vault o 
una variable protegida de Gitlab y en el momento del apply la pipeline, el jenkins, lo administre.


```
terraform apply --var nombre_contendor=nginx
```  

-En un fichero a parte, con extensión .tfvars. El nombre del fichero es irrelevante.

```
entorno-dev.tfvars

nombre_contenedor = "nginx"  
```

Aplicamos este fichero durante la ejecución del comando apply

```
terraform apply --var-file entorno-dev.tfvars
```

Existe un tipo de fichero especial, con extensión .outo.tfvars, los valores de las variables que se definan en ese fichero se cargan por defecto, aunque serían sobreescritos por los suministrados mediante los argumentos **--var** o  **--var-file**
Si no suministramos los valores ni a través de --var ni --var-file, terraform coge todos los ficheros con extensión .outo.tfvars y saca los valores por defecto.

```
entorno-dev.outo.tfvars

nombre_contenedor = "contenedor_docker"

```

Los ficheros .outo.tfvars los coge automáticamente, los .tfvars hay que suministrárselos.


Para dividir el valor de un campo en varias variables, debemos posteriormente utilizar sintaxis de interpolación ${} (construir un texto a partir de unas variables) para referenciarlas y englobarlas dentro de un bloque de texto con ""

```
resource "docker_container" "contenedor" {
    image = var."${var.repo_imagen}:$¨{var.tag_imagen}"
}
```




---
EJERCICIOS:
Aplicamos los recursos especificados de imagen y conetedor.
```
docker images #Muestra listado de imágenes
docker ps #Muestra listado de contenedores
```

Para eliminar lo que existe en el entorno:
```
terraform destroy
```

Si volvemos a ejecutar terraform apply, creará de nuevo la imagen y el contenedor. Pero si lo ejecutamos dos veces, no lo volverá a crear una segunda vez.
Comprueba la diferencia de la infraestructura real existente y la configuración que le proporcionamos y al no encontrar diferencias, no hace nada.  

Si eliminamos el contenedor con un comando de docker:
```
docker rm minginx -f
```

La imagen sigue existiendo pero el contenedor ya no. Al lanzar terraform apply, detecta esa discrepancia y procede a crear de nuevo únicamente el contenedor.
Sin importar el estado del que partamos, siempre que ejecutemos el script la infra quedará en el mismo estado.

``` 
#Ejemplo:
resource "docker_container" "contenedor" {
  name = "minginx"
  image = docker_image.imagen.image_id
  #must_run = true #Algunos contenedores no se mantienen corriendo tras crearse, sino que lanzan un comando y se paran. 
  start = true #Especifica si el contenedor debe arrancarse tras ser creado o no
  cpu_shares = 1024  #Permitirmos que use el equivalente a un core. No tiene por qué ser un core como tal, puede estar usando dos cores al 50%, pero le dejamos usar el equivalente
}

resource "docker_image" "imagen" {
  name = "nginx:latest" #imagen:tag, descarga la última imagen disponible de nginx
}
```

```
#Ejemplo2: Añadir dos variables de entorno, VAR1 y VAR2, con un bloque, y que cuando al host se llame al puerto 8080 del host, que se redirija al puerto 80 del contenedor; y redirigir el puerto 8443 al 443

resource "docker_container" "contenedor" {
  name = "minginx"
  image = docker_image.imagen.image_id
  env = [
      "VAR1=valor1",
      "VAR2=valor2"
        ]
  ports { #Esto es una block list
      internal = 80
      external = 8080
  }
  ports { #Esto es una block list
      internal = 443
      external = 8443
  }
}

```

```
#Ejemplo4: Obtener el valor de la dirección ip del contenedor generado
# network_data es una lista de objetos, contiene ip_address, le ponemos la posición 0 para obtener el valor de ese objeto

output "direccion_ip" {
  value = docker_container.contenedor.network_data[0].ip_address
}

```
```
#Ejemplo5: Parametrizar valores de los recursos en un fichero de variables

main.tf

resource "docker_container" "contenedor" {
    name = var.nombre_contenedor
}


variables.tf

variable "nombre_contenedor {
  description = "Nombre del contenedor que se va a generar"
  type = string
  default = "contenedor_docker"
  nullable = true
}


entorno.tfvars

nombre_contenedor =  "nginx"

```

```
#Ejercicio: Dividir en dos variables el valor de la imagen, separando el nombre del tag

main.tf

resource "docker_container" "contenedor" {
    image = var."${var.repo_imagen}:$¨{var.tag_imagen}"
}


variables.tf

variable "repo_imagen"{
}

variable "tag_imagen"{
}
```
