# check_mk-ansible-livestatus_query

## Purpose:
Generate csv report from severall check_mk remote sites using **lq** (livestatus query binary) and publishing it on apache web server using ansible. 

## Components of this project:
* inventory
* playbook
* webserver


### Inventory
Has next template. It can be customized as you need. 
```
[all]
hostname1 site_name=SITE_NAME1
hostname2 site_name=SITE_NAME2
...
hostnameN site_name=SITE_NAMEN
hostname_webserver


[all:vars]
ansible_connection=ssh
ansible_ssh_user=root
ansible_ssh_pass=your_root_password
host_key_checking=false


[check_mk_sites]
hostname1
hostname2
...
hostnameN


[webserver]
hostname_webserver
```

### Playbook
```
-
  name: 'Query and Save to file each site info'
  hosts: check_mk_sites
  vars:
    carpeta_reportes: "{{carpeta_reportes}}"
  tasks:
    -
      name: "Create Localhost temp folder for saving each query in separate files"
      file:
        path: "{{carpeta_reportes}}"
        state: directory
      delegate_to: localhost

    -
      name: 'Query each site and save result in a variable'
      shell: 
        cmd: |
          printf "GET services\nColumns: host_name description state\nOutputFormat: csv\n" | sudo -u {{site_name}} /omd/sites/{{site_name}}/bin/unixcat /omd/sites/{{site_name}}/tmp/run/live
      args:
        executable: /bin/bash 
      register: reporte

    -
      name: "Copying Query Result into temporal files"
      copy:
        content: "{{reporte.stdout_lines}}"
        dest: "{{carpeta_reportes}}/reporteSalidaServicios{{site_name}}.txt"
      delegate_to: localhost

    -
      name: "Cleaning and formating result for generate csv"
      shell: cat {{carpeta_reportes}}/reporteSalidaServicios{{site_name}}.txt | sed 's/, /\n/g' | sed 's/"//g' | sed 's/\[//g' | sed 's/\]//g' | awk '{print $0 ";{{site_name}}"}' > {{carpeta_reportes}}/reporteProcesado{{site_name}}.csv
      delegate_to: localhost
      register: procesado

    -
      name: "Merge all temporal files to newone"
      shell: cat {{carpeta_reportes}}/reporteProcesado* > {{carpeta_reportes}}/reportefinal.csv 
      delegate_to: localhost
      
    -
      name: "Adding head titles to final csv report"
      shell: sed '1ihostname;servicio;estado;site' {{carpeta_reportes}}/reportefinal.csv > {{carpeta_reportes}}/reportefinalcabecera.csv
      delegate_to: localhost

-
  name: 'Copying final csv report to remote webserver'
  hosts: webserver
  vars:
    carpeta_destino: "/var/www/html/reporte/reporte"
  tasks:
    -
      name: 'Copying File'
      copy:
        src: {{carpeta_reportes}}/reportefinalcabecera.csv
        dest: "{{carpeta_destino}}/reporte.csv"
        owner: apache
        group: apache
        mode: '644'

```

### webserver
For publishing report, I created a simple html webpage with a link to it.
```
<html>
<head>
</head>
<body>
<h1>Reporte Generado</h1>
<p>Descargar <a href="/reporte/reporte.csv">aqui</a></p>
</body>
</html>
```
Folder structure of apache site:
```
.
├── index.html
└── reporte
    └── reporte.csv
```

## Usage:
```
ansible-playbook playbook.yml -i Inventory
```

