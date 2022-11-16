
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

**string**  *Texto*  

**number** *Número*  

**list(bool|string|number)** *Lista de cosas, de tipo boolean, string o number. Se puede acceder a cada elemento en función de su posición*  

**set(bool|string|number)** *Lista de cosas, la diferencia es que no es indexable. NO se puede acceder a cada elemento en función de su posición, sólo se puede iterar*  

**map(bool|string|number)** *Mapa, diccionario, array asociativo, conjunto pares clave-valor*  

**any** *Cualquier cosa*  

**object** *Mapa en el qeu hay una claves predeterminadas, no puedo meter cualquier cosa, existen una serie de claves ya predefinidas que debo usar y especificar.*  




---
Ejercicio:
Aplicamos los recursos especificados de imagen y conetedor.
```docker images #Muestra listado de imágenes
docker ps #Muestra listado de contenedores
```

Para eliminar lo que existe en el entorno:
```terraform destroy
```

Si volvemos a ejecutar terraform apply, creará de nuevo la imagen y el contenedor. Pero si lo ejecutamos dos veces, no lo volverá a crear una segunda vez.
Comprueba la diferencia de la infraestructura real existente y la configuración que le proporcionamos y al no encontrar diferencias, no hace nada.  

Si eliminamos el contenedor con un comando de docker:
```docker rm minginx -f
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

