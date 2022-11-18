Queremos que en función de la variable force_recreate se generen o no unas claves ssh o se lean de un ficheros si es que el fichero existe. Si no existe, pasamos del force_recreate 

```
module "misclaves" {
  source = "..."
  algorithm = ""
  configuracion = ""
  force_recreate = boolean
  #Si false, comprueba si ya existen los ficheros... y los carga, y su contenido es lo que asigna en los outputs.
  #Si no existen los ficheros o la variable está en true, se generan nuevas claves. Si file destination no es nulo, se vencan a fichero y se meten en los outputs. 
  
  
  variable "file_destination" {
    description = "Directorio donde generar los ficheros"
    type        = string 
    nullable    = false
    default     = "."
    validation {
        condition     = length(regexall("^[.]?[\\/]?([a-zA-Z0-9_-]+[\\/]?)*$", var.file_destination))==1
        error_message = "El directorio no es válido"
    }
}



--- 

main.tf 

terraform {
  required_providers {
    docker = {
              source = "hashicorp/tls"
  }
}

resource "tls_private_key" "claves" {
  count = #Permitirá generar o no el recurso  
  
  algorithm = var.algoritmo.nombre
  ecdsa_curve = var.algoritmo.nombre == "ECDSA" ? var.algoritmo.configuracion : null   #Si el algoritmo es ECDSA, pondremos eso
  rsa_bits = var.algoritmo.nombre == "RSA" ? var.algoritmo.configuracion : null #Si el algoritmo es RSA, pondremos eso
}

#Pero así lo estamos generando siempre, que no es lo que queremos. 
```

En otra carpeta crearán el script que use este módulo 
En el main.tf tenemos que meter los recursos (en el main.tf o en otro en realidad, da igual). Hay que meter algo parecido a un tls private key terraform.  
Pero NO siempre queremos las claves, no siempre queremos que se generen, necesitamos saber si queremos que se nos genern o no. Además del tipo de recurso, hay que especificar los outputs. 

Con un count podemos modular que se cree o no el recurso. 

```
outputs.tf

output "privateKey" {
    value = {
        pem = tls_private_key.claves.private_key_pem
        open_ssh = tls.private_key.claves.private_key_openssh
    }
}


output "publicKey" {
   value = {
        pem = tls_private_key.claves.private_key_pem
        open_ssh = tls.private_key.claves.private_key_openssh
    }
}


```

La creación de claves sólo debe ocurrir cuando se crea el recurso, porque si no creamos el recurso, debemos usar los que ya están.  
En caso de que ya haya sido creado, y se hayan generado las claves durante la creación, que se consulten en el fichero

Necesitamos un sitio en el que almacenar si los ficheros existen o no.  
Esa información a nivel conceptual se puede guardar en una variable, pero no una variable de las de variable.tf 
Es una variable que calculamos nosotros --> locals

```
outputs.tf
---

output "privateKey" {
    value = length( tls_private_key.claves ) == 1 ? { #Miramos si existe el recurso
        pem = tls_private_key.claves[0].private_key_pem
        open_ssh = tls.private_key.claves[0].private_key_openssh
    }
    : #Si existe, que mire lo que haya en el fichero
}


output "publicKey" {
    value = length( tls_private_key.claves ) == 1 ? { #Miramos si existe el recurso
        pem = tls_private_key.claves[0].private_key_pem
        open_ssh = tls.private_key.claves[0].private_key_openssh
    }
    : #Si existe, que mire lo que haya en el fichero
}


```


LOCALS

Locals permite generar variables internas, nuestras, para manejar en el código.
Con las funciones built-in endswith comprobamos si la ruta termina con /, en caso contrario lo añadimos 
Y fileexists si el archivo existe 
La función file permite leer el contenido del fichero y devolverlo como texto, podemos almacenar su contenido 

```
Ejemplo.

locals {

  directorioficheros = endswith(var.file_destination, "/") ? var.file_destination : "${var.file_destination}/" #Añadirá una barra / al final si no la tiene
  
  existe_fichero_clave_privada_pem      = fileexists( "${locals.directorio_ficheros}privateKey.pem" )
  existe_fichero_clave_privada_openssh  = fileexists( "${locals.directorio_ficheros}privateKey.openssh" )
  existe_fichero_clave_publica_pem      = fileexists( "${locals.directorio_ficheros}publicKey.pem" )
  existe_fichero_clave_publica_openssh  = fileexists( "${locals.directorio_ficheros}publicKey.openssh" )

#En caso de que exista, almacenamos el contenido, en caso contrario null: 
  contenido_fichero_clave_privada_pem      = existe_fichero_clave_privada_pem ? file( "${locals.directorio_ficheros}privateKey.pem" ) : null
  contenido_fichero_clave_privada_openssh  = existe_fichero_clave_privada_openssh ? file( "${locals.directorio_ficheros}privateKey.openssh" ) : null
  contenido_fichero_clave_publica_pem      = existe_fichero_clave_publica_pem ? file( "${locals.directorio_ficheros}publicKey.pem" ) : null
  contenido_fichero_clave_publica_openssh  = existe_fichero_clave_publica_openssh ? file( "${locals.directorio_ficheros}publicKey.openssh" ) : null
  
  existen_todos_los_ficheros_de_claves  = existe_fichero_clave_privada_pem     && 
                                          existe_fichero_clave_privada_openssh &&
                                          existe_fichero_clave_publica_pem     &&
                                          existe_fichero_clave_publica_openssh
}

```
 
 Si existen los ficheros de las claves, ¿hay que generar el recurso? Depende, hay otra variable en juego (force_recreate). Por tanto, si existen los ficheros de las claves y no me piden que fuerce un recreado (force_receate = false)  
 no creamos el recurso. Si existen ficheros anteriores de clavces pero sí se fuerza el recreado, entonces da igual 
 
 ```
 resource "tls_private_key" "claves" {
  count = locals.existen_todos_los_ficheros_claves && ! force_recreate ? 0 : 1 #Permitirá generar o no el recurso  
  
  algorithm = var.algoritmo.nombre
  ecdsa_curve = var.algoritmo.nombre == "ECDSA" ? var.algoritmo.configuracion : null   #Si el algoritmo es ECDSA, pondremos eso
  rsa_bits = var.algoritmo.nombre == "RSA" ? var.algoritmo.configuracion : null #Si el algoritmo es RSA, pondremos eso
}

```

A partir del contenido de los ficheros podemos terminar de determinar los valores de las variables de las claves  

```

output "privateKey" {
    value = length( tls_private_key.claves ) == 1 ? {
        pem = tls_private_key.claves[0].private_key_pem
        open_ssh = tls.private_key.claves[0].private_key_openssh   
    } : {
        pem = locals.contenido_fichero_clave_privada_pem
        open_ssh = locals.contenido_fichero_clave_privada_open_ssh
}


output "publicKey" {
   value = length( tls_public_key.claves ) == 1 ? {
        pem = tls_public_key.claves[0].public_key_pem
        open_ssh = tls.public_key.claves[0].public_key_openssh   
    } : {
        pem = locals.contenido_fichero_clave_publica_pem
        open_ssh = locals.contenido_fichero_clave_publica_open_ssh
}


```

Crearemos el recurso en base a si existen los ficheros y no tengo que recrear, no crearé el recurso, en caso contrario sí. Sí voy a crear las claves.  
Para crear ficheros, se necesita un provisionador local que ejecutará un comando 

```

  provisioner "local-exec" {
    command = "echo '${self.private_key_pem}' > "${locals.directorio_ficheros}privateKey.pem"
    }
    
    
  provisioner "local-exec" {
    command = "echo '${self.public_key_pem}' > "${locals.directorio_ficheros}publicKey.pem"
    }
    
    
    
  provisioner "local-exec" {
    command = "echo '${self.private_key_openssh}' > "${locals.directorio_ficheros}privateKey.openssh"
    }
    
    
    
  provisioner "local-exec" {
    command = "echo '${self.public_key_openssh}' > "${locals.directorio_ficheros}publicKey.openssh"
    }
    
    
```

Podemos meter todos estos provisioners en uno con EOT

```
    
  provisioner "local-exec" {
    command = << EOT 
                  "echo '${self.private_key_pem}' > "${locals.directorio_ficheros}privateKey.pem"
                  "echo '${self.public_key_pem}' > "${locals.directorio_ficheros}publicKey.pem"
                  "echo '${self.private_key_openssh}' > "${locals.directorio_ficheros}privateKey.openssh"
                  "echo '${self.public_key_openssh}' > "${locals.directorio_ficheros}publicKey.openssh"
    }

``` 

Añadimos un mkdir como comprobación para que en caso de que no exista el directorio, lo cree (con -p regulamos que no explote ne caso de que exista)  

```
main.tf 
-----
resource  "tls_private_key" "claves" {
  count = locals.existen_todos_los_ficheros_claves && ! force_recreate ? 0:1
  provisioner "local-exec" {
    command = << EOT 
                  mkdir -p ${locals.directorio_ficheros}
                  "echo '${self.private_key_pem}' > ${locals.directorio_ficheros}privateKey.pem"
                  "echo '${self.public_key_pem}' > ${locals.directorio_ficheros}publicKey.pem"
                  "echo '${self.private_key_openssh}' > ${locals.directorio_ficheros}privateKey.openssh"
                  "echo '${self.public_key_openssh}' > ${locals.directorio_ficheros}publicKey.openssh"
    }
}
```

Para probar si funciona el script, creamos un nuevo directorio, en el que añadimos un archivo main.tf, en el que importamos el módulo creado.  

```
main.tf 

terraform {
  required_providers {
      tls = {
              source = "hashicorp/tls"
      }
  }
}

module "claves_prueba" {
      modulo_claves_tls = {
                            nombre        = "RSA"
                            configuracion = "2048"


1h....
