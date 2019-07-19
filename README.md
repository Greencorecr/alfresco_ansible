# alfresco_ansible [![Build Status](https://travis-ci.org/Greencorecr/alfresco_ansible_.svg?branch=master)](https://travis-ci.org/Greencorecr/alfresco_ansible)

Recetas de ansible para Alfresco

## Descripción

Recetas para:

- Instalación y actualización automática de Alfresco 5.2
- Instalación automática de Alfresco 6.1 (en progreso)

## Uso

El rol de Ansible para alfresco ha sido creado de forma tal que las tareas se encuentran divididas en **tags**, lo que permite en una sola ejecución secuencial, escoger cuales porciones deseamos que se vayan ejecutando, de forma tal que el mismo rol puede de igual forma instalar Alfresco en un equipo nuevo, como actualizar a la versión que se indique en el inventario.


### Instalación desde cero

#### Requerimientos para Ansible

En un equipo con Ubuntu 18.04 físico o virtualizado, con espacio suficiente para la instalación de Alfresco y el respaldo de sus datos, donde necesita primero activar el servicio de ssh, copiar una llave de SSH hacia el usuario que utilizará para la conexión, y crear un permiso de sudo, creando con el usuario apropiado el archivo **``/etc/sudoers.d/ansible``**:

```
%sudo ALL=(ALL) NOPASSWD: ALL
Defaults:greencore !requiretty
```

Luego instalamos algunos paquetes necesarios para la conexión inicial, tanto de la distribución como de Python:

```
# Paquetes de Ubuntu
sudo apt install -y python3-minimal python3-pip

# Para Mysql
sudo apt install -y python3-pymysql

# Para PostreSQL
python3-psycopg2 libpq-dev postgresql libpostgresql-jdbc-java

# Paquetes de Python
sudo pip3 install ansible psutil # Puede omitir "ansible" en el equipo remoto, solo es necesario en el equipo controlador
```

### Configuración de inventario

Solamente debe modificar un único archivo, que es el **inventario** donde vamos a definir cual es el IP del servidor donde vamos a instalar o actualizar Alfresco, y ahí definir los parámetros que son utilizados para la configuración de Alfresco.

Por ejemplo, en ``inventory/hosts.yml``:

```
---
  alfresco:
    hosts:
      10.xx.xx.xx:                                                                                       # Cambiar IP de servidor de Alfresco
        ansible_user: greencore                                                                          # Usuario a utilizar en SSH
        alfresco_installer: alfresco-community-installer-201707-linux-x64.bin                            # Cual version del instalador, descomentar
        # vieja
        #alfresco_installer: alfresco-community-installer-201602-linux-x64.bin
        alf_glob_prop_path: /opt/alfresco_community/tomcat/shared/classes/alfresco-global.properties     # Ruta de archivo alfresco-global.properties
        alf_root: /opt/alfresco_community/                                                               # Raíz de Alfresco
        dir_root: /opt/alfresco_community/alf_data                                                       # Directorio alf_data
        solr4_root: /opt/alfresco_community/alf_data/solr4/index                                         # Directorio de Solr
        installer_delay: 190                                                                             # Tiempo a esperar que complete instalador
        mysql_root_pass: root
```

Antes de continuar, comprobamos el inventario y la ejecución de ansible en general, ejecutando:

```
ansible -i inventory/hosts.yml -m ping alfresco
10.xx.xx.xx | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}

```

Solamente para comprobar la disponibilidad de archivos de configuración, archivos de descarga, módulos de Python requeridos para módulos de Ansible, etc, primero vamos a ejecutar ansible en modo **dry-run**, lo que quiere decir que no realiza ningún cambio.


```
ansible-playbook -C -i inventory/hosts.yml install_alfresco52-mysql_no_docker.yml --tags install,mysql_install,mysql
```

Si no detectamos ningún problema, ahora podemos ejecutar sin la opción ``-C`` para realizar la instalación de Alfresco:

```
ansible-playbook -i inventory/hosts.yml install_alfresco52-mysql_no_docker.yml --tags install,mysql_install,mysql
```

En caso de utilizar directamente PostgreSQL, puede utilizar:

```
ansible-playbook -i inventory/hosts.yml install_alfresco52-pgsql_no_docker.yml --tags install,pgsql
```

### Actualización de Alfresco

Antes de iniciar se recomienda comprobar los servicios del equipo antes de actualizar, seguir las normas de solicitud de cambio, utilizar equipos de prueba y siempre realizar respaldos de equipos en producción.

En caso de estar realizando el laboratorio completo de instalación y actualización, es posible que necesite cambiar la versión de alfresco a ser instalada según el archivo de inventario.

En este caso, los tags adicionales de **mysql_preupgrade** y **mysql_postupgrade** se encargan de realizar respaldo de datos, de mover el directorio de Alfresco, y re-importan los datos en MySQL así como el dir.root para ser aplicados en la actualización.

```
ansible-playbook -i inventory/hosts.yml install_alfresco52-mysql_no_docker.yml --tags install,mysql_preupgrade,mysql,mysql_postupgrade
```

Ahora solo falta comprobar el servicio, que el contenido se encuentra en su lugar, y observar los archivos de logs por posibles mensajes de error.


## Depuración

Si algo falla, por ejemplo que el tiempo de espera del instalador sea muy bajo, puede continuar luego de esa tarea, usando como base este comando:

```
ansible-playbook -i inventario playbook.yml --tags a,b,c --start-at-task="Nombre completo de la tarea"
```
