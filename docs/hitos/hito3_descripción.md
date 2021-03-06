
# Documentación del hito 3: Provisionamiento de máquinas virtuales

---

# Tabla de contenidos

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
<!-- END doctoc -->

- [Introducción](#introducci%C3%B3n)
- [Tecnología empleada](#tecnolog%C3%ADa-empleada)
  - [VirtualBox](#virtualbox)
  - [Vagrant](#vagrant)
  - [Ansible](#ansible)
    - [ansible.cfg](#ansiblecfg)
    - [ansible_hosts](#ansible_hosts)
    - [ansible_playbook](#ansible_playbook)
- [Prueba de despliegue de la infraestructura y aprovisionamiento en local](#prueba-de-despliegue-de-la-infraestructura-y-aprovisionamiento-en-local)
- [Prueba de aprovisionamiento en azure](#prueba-de-aprovisionamiento-en-azure)
  - [Comprobaciones de aprovisionamiento del hito3](#comprobaciones-de-aprovisionamiento-del-hito3)
  - [Avance en la aplicación con respecto al hito anterior incluyendo servicios adicionales.](#avance-en-la-aplicaci%C3%B3n-con-respecto-al-hito-anterior-incluyendo-servicios-adicionales)
    - [Descripción del microservicio de tareas](#descripci%C3%B3n-del-microservicio-de-tareas)
    - [Test del microservicio de tareas](#test-del-microservicio-de-tareas)
    - [Guía de uso del microservicio](#gu%C3%ADa-de-uso-del-microservicio)
      - [Añadir tareas](#a%C3%B1adir-tareas)
      - [Mostrar tareas](#mostrar-tareas)
      - [Modificar tareas](#modificar-tareas)
      - [Eliminar tareas](#eliminar-tareas)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

---

# Introducción

El objetivo de este hito es realizar un aprovisionamiento de una máquina virtual que
instale todo lo necesario para que se pueda desplegar correctamente la aplicación.

---

# Tecnología empleada

## VirtualBox

Como software de virtualización se ha utilizado **[VirtualBox](https://www.virtualbox.org/)**.

Se ha elegido virtualbox porque es un software de código abierto, gratuito y muy sencillo de usar. Además, parto con cierta experiencia en el uso de este software y es totalmente compatible con el resto de tecnologías que se van a utilizar en este hito.

## Vagrant

Aunque en este hito no era necesario utilizar esta herramienta, consideré muy útil poder crear, eliminar... un conjunto de máquinas virtuales ejecutando una simple orden, además de poder ejecutar directamente el aprovisionamiento de ansible a la vez que se cree la máquina virtual.

Además, el objetivo de la asignatura es poder desplegar de forma automática un servicio en la nube, y en este caso, vagrant sería el primer botón rojo que se pulsaría para crear el conjunto de infraestructuras necesarias para poder desplegar nuestra aplicación.

En primer lugar se ha instalado Vagrant y creado un directorio dentro del proyecto donde se ha realizado `vagrant init`.

Posteriormente se ha definido el archivo **[Vagrantfile](https://github.com/jmv74211/Proyecto-cloud-computing/blob/master/vagrant/Vagrantfile)** donde se ha especificado que la imagen del sistema operativo sea **Ubuntu 16.04LTS**.

Los motivos de elegir esta versión de sistema operativo como base para ejecutar la aplicación son los siguientes:
- Encontrar soluciones a los problemas de Ubuntu es mucho más fácil que otras versiones debido a que existe una gran comunidad de usuarios de ubuntu y a una extensa documentación.

- El servidor Ubuntu tiene una gran cantidad de soporte para implementaciones en contenedores y en la nube.

- Por lo general, los paquetes de software de Ubuntu están más actualizados que en Debian.

- Las nuevas tecnologías llegan antes a Ubuntu debido a las colaboraciones de Canonical y otras compañías.

- Estabilidad a largo plazo. Ubuntu ofrece soporte prolongado en sus versiones LTS para que se puedan seguir manteniendo y actualizando durante un largo periodo de tiempo.

El motivo de elegir la versión 16.04LTS ha sido porque he utilizado esta versión en otras prácticas de asignaturas y no he tenido ningún problema, y consultando foros hablan positivamente acerca de esta versión.

Los siguientes enlaces argumentan parcialmente la decisión que he tomado

- **[enlace 1](https://www.hostinger.es/tutoriales/centos-vs-ubuntu-elegir-servidor-web/#gref)**
- **[enlace 2](https://www.linuxadictos.com/debian-vs-ubuntu.html)**

Dicha especificación del SO la he realizado mediante la siguiente línea:

      config.vm.box = "ubuntu/xenial64"

A continuación se ha definido el nombre del host de la máquina, en este caso lo he llamado *smartage* debido al nombre de la aplicación que propuse.

      config.vm.hostname = "smartage"

Por último, en el Vagrantfile se ha especificado que se aprovisione la máquina con ansible y que ejecute nuestro playbook.

    config.vm.provision "ansible" do |ansible|
      ansible.playbook = "../provision/playbook_principal.yml"
    end

## Ansible

En este caso, se ha elegido ansible como software para aprovisionar nuestra máquina virtual. El principal motivo de esta elección, es que es muy sencillo de usar, basta con instarlo y configurar una serie de archivos que se describirán a continuación. Otro de los principales motivos de esta elección, es porque se realizó un seminario con esta herramienta, e incluso se mostraba como podía integrarse utilizando vagrant, por lo tanto, esta decisión ha sido la acertada.

Bien es cierto que también se ha podido utilizar otro software como por ejemplo Chef o Puppet, que ya veremos más adelante en un seminario, aunque para este hito lo más adecuado (en mi opinión) ha sido utilizar ansible.

Tras instalar ansible con `pip install ansible` (*anécdota: Inicialmente lo instalé con apt y me instaló una versión más antigua que no me funcionaba correctamente, tras instalarlo con pip me instaló la última versión y se arregló*), creé un directorio llamado provision (tal y como se indica en el hito) y dentro guardé los siguientes archivos:

### ansible.cfg

En este archivo nos permite modificar la configuración básica de ansible. En este caso este archivo tiene el siguiente contenido:

    [defaults]
    host_key_checking = False      
    inventory = ./ansible_hosts

`host_key_checking = False`: está a false para evitar la comprobación de de clave de ssh de forma que se puedan usar diferentes máquinas con la misma MAC sin que haya problema

`inventory = ./ansible_hosts`: Especifica la ubicación y nombre del fichero de host con el que se va a trabajar.

### ansible_hosts

Fichero con formato parecido a los init file de configuración que vamos a utilizar para definir lo siguiente:

      [vagrantboxes]
      smartage ansible_ssh_port=2222 ansible_ssh_host=127.0.0.1 ansible_ssh_private_key_file=./.vagrant/machines/default/virtualbox/private_key

      [vagrantboxes:vars]
      ansible_ssh_user=vagrant

En la sección de `[vagrantboxes]` vamos a definir el conjunto host que tenemos (en este caso solo uno) y a especificar la siguiente información para cada uno de ellos:

- `ansible_ssh_port`: Puerto ssh mediante el cuál vamos a acceder a la máquina. En este caso es 2222 porque vagrant por defecto utiliza este puerto.

- `ansible_ssh_host`: Se especifica la dirección de acceso para comunicarnos con la máquina. En este caso es la dirección 127.0.0.1 ya que estamos utilizando la dirección localhost.

- `ansible_ssh_private_key_file`: Para acceder a la máquina mediante ssh es necesaria tener la clave privada, y en este parámetro se especifica la dirección donde se encuentra dicha clave.

En la sección de `[vagrantboxes:vars]` vamos a definir la información global que se van a utilizar para todas las máquinas. En este caso tenemos la siguiente línea:

- `ansible_ssh_user`: Aquí se especifica el usuario que vamos a utilizar para acceder por ssh. En este caso se utiliza el usuario vagrant para todas las máquinas.

### ansible_playbook

En este archivo vamos a definir un archivo en formato .yaml para realizar las instrucciones de aprovisionamiento. En este caso se ha definido una serie de roles (que a su vez contienen otros playbook) que van a ejecutar una labor en concreta.

En este caso el playbook principal es el siguiente:

    - hosts: all
      gather_facts: False
      become: yes

      roles:
      - base
      - python3

Como breve resumen de este playbook, lo que vamos a hacer es que para todos los hosts (en este caso uno solo) vamos a ejecutar como superusuario los siguientes roles. Sinceramente, no sé exactamente (tras haber investigado sobre ello)el valor de gather facts. Tiene el valor de false porque inicialmente me daba error al ejecutar y estableciéndolo a false me funcionó. Según he visto (pero no entiendo muy bien) estableciéndolo a true sirve para llamar al módulo setup como primera tarea del playbook ([enlace](https://stackoverflow.com/questions/34485286/ansible-gathering-facts-with-filter-inside-a-playbook)).

Se puede consultar los archivos relacionados:

- [playbook_principal](https://github.com/jmv74211/Proyecto-cloud-computing/blob/master/provision/local/playbook_principal.yml): Describe el conjunto de roles que se van a usar para el aprovisionamiento.
- [Playbook_base](https://github.com/jmv74211/Proyecto-cloud-computing/blob/master/provision/local/roles/base/tasks/main.yml): Instala python mínimo, git y repositorio; además de configurar unas variables de entorno.
- [Playbook_python](https://github.com/jmv74211/Proyecto-cloud-computing/blob/master/provision/local/roles/python3/tasks/main.yml): Instala python 3.6, pip3 y las dependencias necesarias para ejecutar correctamente la aplicación.

---

# Prueba de despliegue de la infraestructura y aprovisionamiento en local

Utilizando la configuración de vagrant y ansible del apartado anterior es super sencillo la creación de la máquina virtual y aprovisonamiento para que se pueda ejecutar nuestra aplicación.

Simplemente hay que lanzar `vagrant up` desde el directorio de vagrant (dentro del proyecto).

![img](https://raw.githubusercontent.com/jmv74211/Proyecto-cloud-computing/master/images/hito3/vagrant_up.png)

A continuación se creará la máquina virtual y se instalará todo lo necesario sin tener que hacer nada.

![img](https://raw.githubusercontent.com/jmv74211/Proyecto-cloud-computing/master/images/hito3/provision.png)


Comprobamos que se ha creado la máquina virtual.

![img](https://raw.githubusercontent.com/jmv74211/Proyecto-cloud-computing/master/images/hito3/mv_virtualbox.png)


Accedemos a dicha máquina virtual por `vagrant ssh`.

![img](https://raw.githubusercontent.com/jmv74211/Proyecto-cloud-computing/master/images/hito3/vagrant_ssh.png)

Probamos lanzar la aplicación directamente, y comprobamos que se ejecuta correctamente.

![img](https://raw.githubusercontent.com/jmv74211/Proyecto-cloud-computing/master/images/hito3/ejecutar_app.png)

A continuación abrimos otra terminal y volvemos a hacer `vagrant ssh`, y probamos que el servicio web esté funcionando.

Prueba 1

![img](https://raw.githubusercontent.com/jmv74211/Proyecto-cloud-computing/master/images/hito3/app_test1.png)

Prueba 2

![img](https://raw.githubusercontent.com/jmv74211/Proyecto-cloud-computing/master/images/hito3/app_test2.png)

---

# Prueba de aprovisionamiento en azure

Para poder desplegar y ejecutar la aplicación en un sistema cloud se ha elegido la plataforma **[Azure](https://azure.microsoft.com/es-es/)**, ya que se nos ha proporcionado saldo para poder ir experimentando en dicho sistema. El proceso de creación de la máquina virtual donde se va a desplegar la aplicación ha sido muy sencillo, basta con seguir los pasos que se propone **[aquí](https://docs.microsoft.com/es-es/azure/virtual-machines/linux/quick-create-portal?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)**

Se ha creado una máquina virtual básica, con el mismo sistema operativo que el usado en la máquina local: Ubuntu 16.04 LTS.

Para poder aprovisionar dicha máquina con el software que se necesita para ejecutar la aplicación, se ha modificado la configuración del archivo **ansible_hosts** respecto a la configuración de la máquina virtual local, añadiendo la dirección IP del servidor remoto, el puerto de acceso a ssh y la ruta para encontrar la clave privada para poder conectarnos por ssh.

La configuración queda de la siguiente forma:

    [vagrantboxes]
    smartage ansible_ssh_port=22 ansible_ssh_host=137.116.210.191 ansible_ssh_private_key_file=~/.ssh/id_rsa

    [vagrantboxes:vars]
    ansible_ssh_user=jmv74211

(Podemos encontrar los archivos relacionados en este **[enlace](https://github.com/jmv74211/Proyecto-cloud-computing/tree/master/provision/azure)**).

Posteriormente se ha ejecutado el playbook de ansible para azure con la orden `ansible-playbook playbook_principal.yml`:

![img](https://raw.githubusercontent.com/jmv74211/Proyecto-cloud-computing/master/images/hito3/azure_ansible.png)

Podemos comprobar que todo el aprovisionamiento se ha ejecutado correctamente. Finalmente vamos a ejecutar el servicio para comprobar su correcto funcionamiento.

Ejecutamos la aplicación con **[gunicorn](https://gunicorn.org/)** en el puerto 5000, ya que se ha establecido una configuración para poder redireccionar dicho puerto al 80 (ha dado muchos problemas intentar ejecutarlo en el puerto 80 directamente).

![img](https://raw.githubusercontent.com/jmv74211/Proyecto-cloud-computing/master/images/hito3/azure_app.png)


Finalmente, realizamos peticiones al servicio a través del puerto 80 y obtenemos el resultado esperado.

![img](https://raw.githubusercontent.com/jmv74211/Proyecto-cloud-computing/master/images/hito3/azure_app_test.png)

La dirección IP del servidor web es la siguiente **MV: 137.116.210.191**

---

## Comprobaciones de aprovisionamiento del hito3

- Comprobación de [@jmv74211](https://github.com/jmv74211) al aprovisionamiento de [@gecofer](https://github.com/Gecofer) disponible en este [enlace](https://github.com/jmv74211/Proyecto-cloud-computing/blob/master/docs/hitos/correcci%C3%B3n_a_%40Gecofer.md).

- Comprobación de [@gecofer](https://github.com/jmv74211) al aprovisionamiento de [@jmv74211](https://github.com/Gecofer) disponible en este [enlace](https://github.com/Gecofer/proyecto-CC/blob/master/docs/corrección_a_%40jmv74211.md).

---

## Avance en la aplicación con respecto al hito anterior incluyendo servicios adicionales.

### Descripción del microservicio de tareas

Para el desarrollo de este hito se ha añadido una nuevo microservicio relacionado con las tareas.

El objetivo de este microservicio es que los usuarios puedan gestionar un conjunto de tareas incluyendo los siguientes datos:

- **task_id**: Identificador de la tarea.

- **user**: Usuario que ha creado dicha tarea.

- **name**: Nombre de la tarea.

- **description**: Descripción de la tarea.

- **estimation**: Número de días estimados para realizar la tarea.

- **difficulty**: Dificultad estimada (valor de 1-10).

- **max_date**: Fecha máxima para realizar la tarea.

Una vez descrito su estructura de datos, se ha creado una colección de datos en **[mlab](https://www.mlab.com/)** donde se almacenan los datos y tareas de los usuarios.

Como medida adicional, se ha utilizado otra API para manejar el acceso y manejo de los datos. En el microservicio de login y registro se utilizó **[Flask MongoAlchemy](https://pythonhosted.org/Flask-MongoAlchemy/)**, y en este caso he utilizado **[Pymongo](https://api.mongodb.com/python/current/)** por cambiar de API, ya que pymongo me ha parecido más útil al haber mucha más documentación y ayuda en la web, y además, parece que está más estandarizada.

La funcionalidad realizada por la aplicación es la siguiente:

- **Añadir tareas:** Permite que los usuarios añadan nuevas tareas a través de una petición **PUT**.

- **Modificar tareas:** Permite que los usuarios modifiquen tareas a través de una petición **POST**.

- **Eliminar tareas:** Permite que los usuarios eliminen tareas a través de una petición **DELETE**.

- **Mostrar tareas:** Permite que mostrar a los usuarios el conjunto de tareas que existen. En el siguiente hito se desarrollará este apartado más en profundidad, permitiendo enlazar a los usuarios registrados e identificados en el sistema utilizando el microservicio de login y registro, y posteriormente accediendo al microservicio de tareas donde puedan consultar y gestionar sus propias tareas.

Los archivos fuentes relacionados se pueden consultar accediendo al siguiente **[enlace](https://github.com/jmv74211/Proyecto-cloud-computing/tree/master/src/app/task_service)**

---

### Test del microservicio de tareas

Conjuntamente al desarrollo del microservicio de tareas se han realizado sus respectivos test para probar que el servicio funciona correctamente.

Los test que se han realizado son los siguientes:

- **Test unitario:** Se han realizado test para probar cada una de las funciones que se han implementado en el modelo de la aplicación. Se puede consultar dicho archivo accediendo a este **[enlace](https://github.com/jmv74211/Proyecto-cloud-computing/blob/master/src/test/test_functions_task_service.py)**.

- **Test de integración:** Se han realizado test para comprobar el correcto funcionamiento del servicio, probando las principales funcionalidades que se han desarrollado mediante el uso de peticiones GET, PUT, POST Y DELETE. Se puede consultar dicho archivo accediendo a este **[enlace](https://github.com/jmv74211/Proyecto-cloud-computing/blob/master/src/test/test_task_service.py)**.

---

### Guía de uso del microservicio de tareas (NUEVO versión 3.0)

Para ejecutar esta aplicación basta con lanzarla mediante la orden `python3 task_service.py` o  `gunicorn -b :3000 task_service:app` dentro del directorio /src/app/task_service.

Si accedemos al raíz de la aplicación nos mostrará el siguiente contenido:

    status	"OK"

Para realizar las distintas peticiones **PUT, POST, DELETE, GET** se va a utilizar la herramienta **[postman](https://www.getpostman.com/)**

#### Añadir tareas

Para añadir una tarea vamos a realizar una petición **PUT**, añadiendo la información en formato JSON. Como se puede observar en la siguient imagen, no es necesario añadir el *task_id*, ya que el servicio lo inserta automáticamente.

![img](https://raw.githubusercontent.com/jmv74211/Proyecto-cloud-computing/master/images/hito3/demo_1.png)

Como resultado nos devuelve que se ha insertado con id = 0 y un código de estado de 201.

#### Mostrar tareas

Para mostar una tarea, vamos a utilizar la petición **GET**.

![img](https://raw.githubusercontent.com/jmv74211/Proyecto-cloud-computing/master/images/hito3/demo_2.png)

Como se puede observar, nos muestra la tarea que añadimos en el apartado anterior con un código de estado = 200.


#### Modificar tareas

En primer lugar vamos a añadir otra tarea utilizando la orden PUT descrita anteriormente. La tarea la vamos a relacionar con una supuesta práctica de IC.

![img](https://raw.githubusercontent.com/jmv74211/Proyecto-cloud-computing/master/images/hito3/demo_3.png)

Ahora vamos a proceder a modificar dicha tarea mediante la petición **POST**. Por ejemplo vamos a modificar la estimación temporal y la fecha máxima de entrega.

**Atención**: En este caso si es necesario introducir el *task_id*, para poder identificar la tarea que deseamos modificar.

![img](https://raw.githubusercontent.com/jmv74211/Proyecto-cloud-computing/master/images/hito3/demo_4.png)

#### Eliminar tareas

Para proceder a eliminar una tarea vamos a utilizar la petición **DELETE**. Simplemente basta con emplear el identificador de la tarea que se desea eliminar, como se puede observar en la siguiente imagen:

![img](https://raw.githubusercontent.com/jmv74211/Proyecto-cloud-computing/master/images/hito3/demo_5.png)

Como se puede observar, nos devuelve el código de estados 204, correspondiente a que no hay contenido.

Volvemos a comprobar la lista de tareas mediante la petición GET para verificar que la tarea se ha eliminado correctamente.

![img](https://raw.githubusercontent.com/jmv74211/Proyecto-cloud-computing/master/images/hito3/demo_6.png)
