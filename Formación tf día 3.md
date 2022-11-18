NULL

Cuando en terraform a una propiedad de un recurso le asignamos valor null es como si no se hubiera escrito la propiedad.  
En una propiedad requerida, campo requerido, no se puede poner null. Si en un campo opcional ponemos null, es como si no hubiesemos puesto esa propiedad, no hemos asignado ningun valor a ella. 
Podemos hacer nullables (nullable=true) a nuestras variables (las definidas en variables.tf), pero no las propiedades de cada recurso. 
No es lo mismo que una variable esté desasignada que que esté asignada a null. Si está desasignada, terraform nos solicitará un valor para dicha variable durante la ejecución.  


OPERADOR TERNARIO 

Para utilizar una condición, se utiliza el operador ternario 
condicion ? valor si true : valor si false

```
Ejercicio. Modificar la validación para que la variable cuota_cpu pueda ser nula, y en caso de que no lo sea, deba ser mayor de 0 y menor de 2048

variable "cuota_cpu" {
    description = "Cuota de CPU para el contenedor"
    type        = number # string | bool | number | list() | set() | object() | map () | any
    #default     = "" # En los scripts... RARO ! NUNCA !
                     # Más adelante, cuando veamos los módulos, veremos que ahí si
    nullable    = true
                       # Indica si se le puede asignar un valor null
                       # Si no le pongo valor, se considera que tiene valor null? NOMBRE
                       # No es lo mismo una variable desasignada que asignada a null
    validation {
                    # Condicional en linea, operador ternario 
                    # condicion ? valor si true : valor si false
        condition = var.cuota_cpu == null ? true : var.cuota_cpu > 0 # Expresion lógica que si true, indica que el valor es bueno
        error_message = "El valor de cuota de cpu debe ser mayor que cero" # Que se muestra si el valor no es bueno
    }        
    validation {
        condition = var.cuota_cpu == null ? true : var.cuota_cpu <= 2048 # Expresion lógica que si true, indica que el valor es bueno
        error_message = "El valor de cuota de cpu debe ser menor o igual a 2048" # Que se muestra si el valor no es bueno
    }                       
}

```

#PUERTOS 

Cuando creamos un contenedor,  podemos solicitar a docker una redirección de puertos 

```
docker container create \
    -p ip-host:puerto-host:puerto-contenedor \
    imagen
               
```   

Mi host tiene varias IPs, concretamente: 
    - Red de amazon:         172.31.20.70
    - Red de loopback:       127.0.0.1 (localhost)
    - Red virtual de docker: 172.17.0.1


Si hacemos un ifconfig, en windows un ipconfig:
En el interfaz de loopback, tenemos en el apartado inet la dirección ip de loopback
En el interfaz de ens, tenemos en el apartado inet la dirección de ethernet (la de amazon) 
En el interfaz de docker, tenemos en el apartado inet la dirección de la red de docker

Si tuviesemos wifi tendríamos otra ip en este listado. Podemos no querer abrir el puerto en todas las ips, quizás solo en las internas. Eso se puede establecer en el -p

Se podría poner una de las IPS disponibles, pero también se puede poner una máscara
 ip_host: 0.0.0.0    Mascara de red: TODAS LAS IPS

En caso de poner esta máscara, esto es como poner todas las ips, porque todas las ips cumplen con esa máscara, en todas ellas se abriría el puerto para hacer la redirección al puento del contenedor.  


```
Ejercicio. Establecer la variable puertos_a_exponer, que la ip sea opcional, y si no nos dan la ip, ponemos por defecto el valor 0.0.0.0

variables.tf
---
variable "puertos_a_exponer" {
    description = "Puertos a exponer para el contenedor"
    type        = list(object({
                    interno = number
                    externo = number
                    ip      = optional(string, "0.0.0.0")
                  }))
    nullable    = false # true
                       # Indica si se le puede asignar un valor null
                       # Si no le pongo valor, se considera que tiene valor null? NOMBRE
                       # No es lo mismo una variable desasignada que asignada a null


main.tf 
---

resource "docker_container" "contenedor" {
    name        = var.nombre_contenedor
    image       = docker_image.imagen.image_id
                  # Variable          # Propiedad de esa variable
    #start       = false
    
    cpu_shares  = var.cuota_cpu # Le dejo usar el equivalente a un core.
                  # Y si no quiero limitarle la cpu ? 
    # Cuando en terraform, a una propiedad de un recurso le asignamos valor null
    # Es como si no hubiera escrito la propiedad
                  
    # Espera un set(string)
    env         = [ for variable in var.variables_entorno: "${variable.clave}=${variable.valor}" ]
                    # bucles en linea de python
    # Atención: DYNAMIC SOLO SIRVE PARA BLOQUES DINAMICOS.
    # Más adelante habrá otras cosas que queramos montar en bucle
    dynamic "ports"{
        for_each = var.puertos_a_exponer
                   # Un for_each en un dynamic recibe una lista
        iterator = puerto
        content {
                    internal = puerto.value["interno"]
                    external = puerto.value["externo"]
                    ip       = puerto.value["ip"]
                }
    }
    # Quiero también el puerto 8443 -> 443
}

```
--- 

# REGEX o EXPRESIONES REGULARES

Lo que hacemos es definir una REGEX: EXPRESION REGULAR.
Qué es una expresión regular? Es una secuencia(conjunto) de patrones
Qué es un patrón? Es un conjunto de caracteres seguidos de un modificador de cantidad

```
    conjunto de caracteres. Ejemplos:     Interpretación                    Texto           Encuentro el patrón en el texto?
        hola                                literalmente                    hola amigos !       SI (1)
                                                                            ^^^^
                                                                            
        [hola]                              alguno de esos caracteres       hola amigos !       SI (6)
                                                                            ^ ^  ^
                                                                             ^ ^     ^
    
        [a-z]                               algun caracter entre la a y la z
        [A-Za-záñü]
        [0-9]                               Un caracter entre el 0 y el 9
        [1-37]                              Un caracter entre el 1 y el 3 
                                            y el caracter 7
        [a-z-]                              Los caracteres de la a, a la z y el guión
        .                                   Cualquier caracter
        [.]                                 El punto
        \.
    
    modificador de cantidad 
        No poner nada                       1 vez
        ?                                   puede paracer o no          0 o 1 vez
        +                                   Al menos 1 vez              De 1 en adelante
        *                                   Puedo no paracer o aparecer las veces que quiera
                                                                        De 0 en adelante
        {4}                                 4 veces
        {3,7}                               De 3 a 7 veces
        {3,}                                Al menos 3 veces
    
    Especiales:
        ^       Comienza el texto
        $       Acaba el texto
        |       O
        ()      Agrupan
        
```

```
EJEMPLOS.  

Quiero un patrón para los números del 0 al 19

    1?[0-9]

Nombre de persona de una palabra, no compuesto

    [A-Z][a-z]{2,}

Nombre de una persona, incluyendo nombres compuestos.

    Jose
    Maria Eugenia
    Maria Antonia de los Angeles
    Maria de los Angeles
    
    [A-Z][a-z]{2,}( (([a-z]{1,3} ){0,2}[A-Z][a-z]{2,}))*

```

Para consultar regex o hacer pruebas:  regex101

En Terraform usamos INTENSIVAMENTE expresiones regulares, ya que tenemos muchas variables de texto que validar.

Por suerte, seguramente no somos la primera persona en el mundo en necesitar un determinado patrón.
Y google será vuestra salvación.

```
Patrón para validar una IP, incluyendo máscaras: 

IP.   ^((25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9][0-9]?|0)\.){3}(25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9][0-9]?|0)$

```


# Quiero un output, con las ips de los contenedores
# Y otro con la ip del balanceador



[
  "contenedor1: 172.17.0.6",
  "contenedorB: 172.17.0.7",
]

{
    "contenedor1": "172.17.0.6",
    "contenedorB": "172.17.0.7"
}

# Proveedor         Provider
# Aprovisionador    Provisioner

En terraform existen 3 tipos de Aprovisionadores

Todos ellos se usan / definen dentro de un resoruce
local_exec:     Ejecutar un programa dentro del entorno donde estoy ejecutando el script
remote_exec:    Ejecutar un programa dentro del resource en el que está definido
file:           Copiar o crear archivos/carpetas dentro del resource en el que está definido

Los provisionadores, por defecto, se ejecutan cuando un resource es creado/modificado
Puedo alternativamente configurarlos para que se ejecuten cuando el resource es destruido

Quien querriamos que instalase paquetes, crease usuarios, etc. en un servidor nuevo?
    Ansible
    Puppet
    Chef
    ---------
    Script sh
    Script ps1
    
Quien queremos que llame a esos programas? 
    Terraform? NO, ni de coña... por qué? SISTEMA CON ALTO ACOPLAMIENTO
    Servidor de automatización: PIPELINE
    
Acabo de crear un servidor con terraform.
    Querría ejecutar yo algo desde terraform dentro de la máquina? remote_exec  
        Puede ser. Qué? 
            Ansible: Python, credenciales
            Puppet:  Instalar el agente de puppet
El planchado lo haré con Ansible.


Voy de borrar un servidor con terraform.
    Querría ejecutar yo algo desde terraform dentro de la máquina? remote_exec  
        Puede ser. Qué? 
            Backup del volumen
