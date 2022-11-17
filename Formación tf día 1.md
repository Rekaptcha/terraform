#Terraform

Es una herramienta de Hashicorp que permite crear y gestionar infraestructura desde código, automatizarla
//Vagrant permite crear y gestionar máquinas virtuales con todo lo que necesitan

Terraform nos ofrece:
-Un lenguaje declarativo, HCL (Hashicorp Language)
-Un intérprete con el que realizar comandos sobre los ficheros que escribimos en HCL

En definitiva, sirve para provisionar (adquirir, liberar, gestionar, mantener) recursos contra unos proveedores.
Los proveedores serán habitualmente los clouds, y los recursos de tipo infraestructura.
El principal uso de terraform es la gestión automatizada de infraestructuras en clouds. Pero no es el único uso que se le puede dar a terraform.

Estaremos escribiendo programas, software

#¿Qué es DevOps?
Una filosofía, cultura, es el movimiento en pro de la automatización en todos los pasos de la construcción de un software

Fases de un proyecto, ¿cuáles son automatizables?
```
#Plan X
#Desarrollo X  -> Almacenan el código en GIT, SVN, Mercury...
#Empaquetado: V
    JAVA: Maven, gradle, sbt
    .NET: msbuild, dotnet, nuget
    PYTHON: poetry
    JS: yarn, npm, webpack
 
Estas herramientas nos ayudan a automatizar el trabajo, ¿quién las configura? Desarrollo
El que sabe qué necesito para empaquetar, qué encoding necesito, lo hace desarrollo
 
#Pruebas --> la definición no es automatizable, la ejecución si
*Selenium, appium, katalon, karate, soapUi, postman, JMeter, Loadrunner...*  
De ello se encarga testing Q&A  

¿Dónde se realizan esas pruebas? En un entorno efímero.
No lo haremos en la máquina del dev, porque son entornos que están maleados, no tenemos confianza en ellos. Lo hacemos en un entorno efímero, de usar y tirar. Para evitar que se queden valores en variables de entorno, residuos que puedan intoxicar ejecuciones posteriores. Por ello se automatiza la disposición (creación y eliminación) de ese entorno. Para ello se usan las herramientas como Vagrant, Terraform, Kubernetes  

Terraform, Vagrant y Kubernetes nos permiten disponer de entornos
Ansible, Puppet, Chef, Salt nos permiten configurar esos entornos, el aprovisionamiento de dichos entornos (configurarlos, instalarles cosas..)

Esto lo ha hecho toda la vida el sys admin (es el que sabe crear una máquina, una red...) 

<------------ Si conseguimos llegar hasta aquí, automatizar todo esto, tendríamos integración continua (generar un entorno de pruebas automatico, instalar en él la última versión que han realizado los desarrolladores de forma automática, y someter dicha aplicación a un montón de pruebas automatizadas)
¿Quién llama a todos esos programas, quién orquesta esos programas? La orquestación de dichos programas la realizan los servidores de automatización CI/CD: 
*Azure DevOps, Jenkins, TravisCI, Bamboo, TeamCity, Gitlab CI, Github Actions..*  Esto es lo que monta el DevOps


#Liberación de una release
<------------ Entrega Continua
#Despliegue
<------------ Despliegue continuo (una cosa es poner la última versión del software en manos del cliente, y otra instalársela)

#Operación
#Monitorización  

 ```
 
Todo entorno de producción debe tener:

-Alta disponibilidad.
  Tratar de cumplir con un objetivo de tiempo de servicio (SLA): 99%, 99,9%, 99,99%
  
-Escalabilidad.
  Capacidad de ajustar la infra a las necesidades de cada momento
  Escalabilidad vertical:
  Escalabilidad horizontal:
  


---
Clouds: El conjunto de servicios (TIC) que un proveedor ofrece:
1- A través de intenret
2-Mediante procesos automatizados y desatendidos
3-Con cobro por uso

Los servicios pueden ser de varios tipos:
 -Infraestructura (almacenamiento / computo) : IaaS
 -Plataforma (bases de datos) : PaaS    
 Yo no alquilo una maquina, alquilo directamente una base de datos, un mysql, esa base de datos tendra una infra por detras pero yo no alquilo eso, sino la bbdd, para una serie de usuarios, te alquilo la bbdd, si hay que mantener la infra, la mantiene el proveedor, si hay que actualizarla, se encarga, a mi que me de el servicio de bbdd. Se alquila no la infra, sino una serie de programas instalados encima. No son programas destinados a un usuario final, sino programas que me sirven a mi para dar la solución final que tendrá un usuario.
 -Software  (cloud9)     SaaS 
 Nos da un entorno, o una aplicación, que como usuario final puedo utilizar.
 
 ---
 
Terraform providers
Los providers son una abstracción lógica de una API. Se encarga de entender las interacciones de API y exponer los recursos.  Los providers no son solo cloud, por ejemplo, puedes tener un proveedor de dns, terraform no solo sirve para infra, terraform permite automatizar nuestra parte del proceso de adquisición y liberación de servicios en un proveedor (aunque normalmente son servicios de infra, y normalmente en un cloud)

Estamos escribiendo un programa en un lenguaje declarativo, con una sintaxis mezcla de JSON y YAML
Es un lenguaje declarativo para la ejecución de programas (creación de scripts)
Para la creación/mantenimiento de unos recursos en un proveedor, o eliminación de esos recursos

Llamamos lenguaje a lo que usamos para programar porque tienen su gramática, su sintaxis y su semántica (Java, C, ADA..)
Podemos usar los lenguajes de distintas formas a la hora de expresar conceptos en el mundo de la programación, a esas formas de expresarnos las llamamos paradigmas de programación:
-Imperativo
-Procedural
-Funcional
-Orientada a Objetos
-Declarativa

En el lenguaje natural tenemos tipos de sentencias también (frases): afirmativas, interrogativas, imperativas, declarativas  
"Pon la silla debajo de la ventana" Tipo de oración: IMPERATIVA : ORDEN
Habitualmente, cuando montamos scripts, usamos lenguaje imperativo  
Bash, PowerShell (PS1), Python..
Estructuras/vocablos tipicos de lenguaje imperativo: IF, ELSE, FOR, WHILE...  

"La silla debe estar debajo de la ventana" Tipo de oración: DESIDERATIVA/DECLARATIVA
Da igual el estado de lque partamos en un momento dado, al ejecutar el script siempre debe quedar en el mismo estado.

Herramientas que usan lenguaje declarativo:
-Terraform
-Ansible
-Kubernetes
-Openshift

Al final en un fichero HCL vamos a declarar recursos que queremos en un proveedor.
Declararemos infraestructura a crear en un cloud, en un fichero de código: Infraestructura como código.  

En los ficheros HCL, haremos uso intesivo de:
=
{}
8 palabras: terraform, provider, variable, output, resource, data, provisioner, module  


#Ejemplo. Queremos un servidor con 16GBs de RAM y 4cpuS, 2tb de HDD, pinchado en una red conectado a Internet  
Si primero lo queremos para AWS y luego para Azure, habrá que reescribirlo entero, no será ni parecido.    

Ventajas de Terraform:  
-Necesito aprender 1 unica sintaxis  
-Usa un lenguaje declarativo  
-Cada proveedor de terraform tiene su documentación, y un listado de todos los servicios disponibles en ese proveedor

AWS tiene su propia herramienta cliente para automatizar trabajos en AWS, para desplegar su infraestructura -> aws cli  
Utiliza lenguaje imperativo.


---
Un fichero HCL siempre lleva una marca: terraform  
```
terraform {
    #Establezco la configuración de terraform: la versión requerida de terraform y los providers que son necesarios
    #Para ello se utiliza el bloque required_providers
    required_providers {
        nombre_proveedor = {
            source = "repositorio" #De este repositorio se descargará el programa del proveedor
            }
}   
```
```
terraform {
    required_providers {
        aws = {
            source = "hashicorp/aws"
            }
            
 }
```
Cuando instalamos terraform no viene por defecto con todos los proveedores, los tenemos que instalar.  
Cualquier provider que queramos usar lo vamos a tener que instalar, vamos a tener que especificar los providers que son necesarios.  

Para consultar cómo instalarlo (el bloque de código especificado más arriba) accedemos al registry de terraform y dentro del apartado del proveedor "How to use this provider" viene el ejemplo que podemos copiar y usar  

Cada proveedor lleva su configuración específica, que se establece en un bloque externo llamado provider  

```
provider "nombre_proveedor" {
    propConfiguracion1 = valor1
}
```

Cuando escribimos un script de terraform, ese script será procesado por terraform, quien le enviará instrucciones a un provider. Un provider es un programa desarrollado para terraform que permite comunicar con un proveedor. En caso de amazon ese programa habla con aws cli, que será quien hable con aws  

script terraform -> terraform -> provider -> aws cli -> aws  

La información de credenciales se puede pasar en la configuración, pero es una mala práctica, se pasa en el punto del aws cli

```
Ejemplo de configuración de provider  

provider "aws" {
    region = "us-east-1"
}
```
Cada proveeddor tendrá sus propios parámetros configurables. Esto ya no es terraform, esto está fuera del bloque de terraform, esto es algo del proveedor.  

Cada recurso de cada proveedor tiene sus propias propiedades configurables, se debe consultar siempre la documentación disponible del registry.  

Al final en un fichero HCL lo que voy a hacer es declarar los recursos que quiero en un proveedor.
Si declaro infraestructura a crear, en un fichero de código: Infraestructura como código.  

---

Un script de terraform es un conjunto de archivos con extensión: .tf  
El nombre de los archivos da igual. Un script de terraform son muchos ficheros que tienen la extensión .tf, da igual como se llamen. Esos archivos estarán en una carpeta, separada para ese script: cada script tiene su propia carpeta en terraform, no puedo tener dos scripts diferentes en una misma carpeta.  
De esa carpeta, terraform coge TODOS los archivos con extensión .tf y los une en uno solo que es el que ejecuta. Podemos tener las cosas separadas en todos los ficheros .tf que queramos, y cuando ejecutemos terraform todos los ficheros .tf de esa carpeta en la que lanzamos el comando se juntarán. Da igual el orden, terraform es insensible al orden.  

A un script de terraform le podemos pedir varias cosas:  
-Que se inicialice:   $ terraform init  
Descargamos todso los proveedores en local que se van a usar  
-Que se valide: $terraform validate  
Que terraform valide la sintaxis
-Que haga un plan: $ terraform plan 
qué tareas son necesarias llevar a cabo para conseguir los recursos/infra que están ahí declarados  
-Que aplique un plan: $ terraform apply    
Que cree, borre o actualice los recursos existentes para que cuadren con lo que tenemos establecidos en el fichero
-Que elimine todos los recursos: $ terraform destroy   


---

Vamos a trabajar en los ejemplos con el proveedor Docker. Le podemos pedir que nos cree recursos: imagens, contenedores (volumenes, redes...).   
  
#Conceptos básicos de contenedores:    
Un contenedor es un entorno aislado dentro de un SO con Kernel Linux donde ejecutar procesos.  
Se pueden ejecutar contenedores en Windows? Realmente cuando lo hacemos necesitamos levantar un subsistema Linux (una vm en nuestro SO de MAC O Windows con Linux) para que ejecute los contenedores. En MAC pasa lo mismo.  
Los contenedores hacen uso de funcionalidades que existen en programas que se incluyen en el kernel de Linux para generar entornos aislados dentro de los cuales ejecutar procesos.  

Entorno aislado -->  Un contenedor tiene:  
-Su propia configuración de red, y por ende su propia IP(s)  
-Sus propias variables de entorno  
-Su propio sistema de archivos (filesystem)  
-Y puede tener limitaciones de acceso al hardware del host  

Los contenedores los creamos desde una Imagen de contenedor.  
Una imagen de contenedor Es un fichero comprimido que contiene un programa YA INSTALADO y PRECONFIGURADO por alguien. Además, en ese fichero comprimido, se incluyen librerías, dependencias y comandos adicionales que puedan requerirse o ser de ayuda.  

Si queremos montar SQL Server en el pc:  
-Comprobamos los requisitos  
-Descargamos un instalador de SQL Server   
-Ejecutamos el instalador  
Esto da lugar, junto con una configuración que apliquemos, da lugar a una instalación de SQL Server  

Si estamos en una máquina Windows, supongamos que esa instalación acaba em--> C:/Archivos de programa/SQLServer  
Si esta carpeta la metemos en un zip y la enviamos por email, ese zip es una imagen de contenedor.  
Es un fichero comprimido que contiene un programa YA INSTALADO y PRECONFIGURADO por alguien. Además, en ese fichero comprimido, se incluyen librerías, dependencias y comandos adicionales que puedan requerirse o ser de ayuda.  
 
Cuando descargamos una imagen de contenedor, no es como cuando descargado un instalador, sino que descargamos un programa ya instalado por alguien y con una configuración que ya ha hecho.  


Un contenedor ejecuta su propio SO? Una imagen de contenedor lleva un SO? NO.  
Los contenedores no ejecutan un SO propio, nunca. Solo hay 1 kernel de SO: el del host, esa es la gran diferencia a las máquinas virtuales. Por eso los contenedores tienen ventaja frente a las vm.  

```
Instalar App1 + App2 + App3
----------------------------
           SO
-----------------------------
        HARDWARE
```
Problemas de este tipo de instalación: Conflictos de dependencias o de configuración a nivel de SO, requerimientos diferentes, seguridad (app1 puede acceder a archivos de app2), si app1 se vuelve loca (100% CPU), app1 pasa a estar offline, pero se cae todo, app2 y app3 también.  

Por eso se usaban vm en los entornos de producción. Con las vm se instala por encima del SO el hipervisor, que permite definir VM. Y ese hipervisor limitará la cantidad de recursos que puede consumir cada una del hardware real.   
Aunque estas máquinas no son reales, a efectos sí lo son, y encima tenemos que instalar un SO, y ya encima de esta capa, la app

```
App1         |   App2 + App3
----------------------------
so1         |      so2
----------------------------
 VM1         |      VM2
----------------------------
HIPERVISOR: HyperV, KVM, VMware, Citrix
----------------------------
           SO
-----------------------------
        HARDWARE
```

Con ello se resuelven los problemas de conflictos, de requerimientos y de seguridad, están en máquinas distintas. Si app1 se vuelve loca y coge el 100% de la CPU sólo afecta a los recursos definidos para esa VM.  
Problemas de trabajar con VM: desperdicio de recursos (muchos SO, licencias...), configuración compleja y más costosa de mantener, rendimiento de las apps (al tener 2 SO ya se tarda más, nunca va tan rápido como tenerlo instalado directamente sobre el SO host).  

Los contenedores surgen para sovlentar esto, en lugar de un hipervisor necesitamos un gestor de contenedores: Docker.  

El programa Docker tiene dos partes: cliente y servidor, el servidor es un daemon que está corriendo en el sistema. El daemon es dockerd, y el programa docker no es si no un cli que usamos para atacar a dockerd  

Docker Compose simplemente es otro cli para trabajar con dockerd  

```
App1         |   App2 + App3
----------------------------
 C1         |      C2
----------------------------
Gestor de Contenedores: Podman, Docker, Crio, Containerd
----------------------------
         SO Linux
-----------------------------
        HARDWARE
```
En entornos locales usamos Podman y Docker, en entornos de producción generalmente Clio y Containerd.  
Podman es el que viene a día de hoy de serie con las máquinas de la familia redhat (fedora, centos..)    

Los gestores de contenedores permiten crear contenedores, entornos aislados en donde crear procesos. Y en esos contenedores podemos ejecutar directamente los procesos, que estarán en comunicación directa con el kernel del SO que tenemos en el host. Resuelve prácticamente los mismos problemas que las vm (no todos, pero casi) pero sin ninguno de los problemas que tenemos con las vm.  

Kubernetes y Openshift: Orquestadores de gestores de contenedores.  
Nos sirven para controlar un cluster de gestores de contendores para su uso en un entorno de producción.  

Cluster de Kubernetes:  
Tengo 3 máquinas VM y en cada una un gestor de contenedores.  
Hoy en día K8S admite 2 gestores de contenedores: Crio y Containerd.  
Ya no soportan Docker.  

```
Cluster de K8S:  
    Máquina 1:  
        crio | containerd 
    Máquina 2: 
        crio | containerd 
    Máquina 3: 
        crio | containerd
```

K8S cuando le pida que despliegue un contenedor, contactará con el gestor de contenedores de una máquina y le pedirá a ese gestor que cree un contenedor. K8S y Openshift son orquestadores de gestores, le pedirán a unos gestores de contenedores instalados en unas vm que creen o eliminen contenedores.


---  
Servidor web más usado en el mundo hoy en día: nginx
Si queremos crear un contenedor con un servidor nginx, lo primero que debemos hacer es descargar una imagen nginx  

``` 
docker image pull nginx:latest  
```

Se ha descargado el zip y lo ha descomprimido. Ahora, con esa imagen, se puede construir el contenedor. 

```
docker container create --name minginx nginx:latest  
``` 

Arrancamos el contenedor  

```
docker start minginx  
```

Ya tenemos un nginx funcionando en el pc.  
Un nginx funciona por defecto en el puerto 80, para comprobar que está arrancado podemos ejecutar curl  
curl http://80
curl localhost 
