# Laboratorio de respaldos automatizados con EBS y AWS Backup 
---
## caso de uso 

una empresa de consultoria de marketing actualmente utiliza multiples scripts personalizados para acer copias de seguriad de sus aplicaciones, para estos scripts se estan volviendo cada vez mas dificiles de mantener. la empresa necesita hacer copias de seguridad de su almacenamiento de host bastion, almacenamiento de servidor web centralizado y snapshots de base de datos de diferentes servicios de AWS. Quieren una solucion de copia de seguridad unifacda y automatizada que agilice la gestion y proporcione una proteccion de datos consistente. 

## objetivos de aprendizaje

- Cree un plan de respaldo para los volumenes de Amazon EBS mediante etiquetas.
- Realizar un respaldo bajo demanda.
---
### respaldos de datos con AWS Backup

> vamos a utilizar AWS Backup para centralizar y automatizar la proteccion de datos en todos los servicios de AWS.
eliminando la necesidad de scrpts personalizados y procesos de respaldo manuales. 

**Descripcion de AWS backuo**

AWS Backup es un servicio administrado por Amazon, por lo que no debes preocuparte por el escalado de tus copias de seguridad.
ofrece un serivicio totalmente adminsitrado basado en politicas de AWS. 


- actualmente AWS Backup cubre servicios como:
  - Amazon EBS (elastic Block store)
  - Amazon EFS (Elastic File System)
  - Amazon FSx.
  - Amazon EC2(Elastic Compute Cloud).
  - Amazon RDS.
  - Amazon DynamoDB

  ofreces proteccion de datos de manera centralizada y automatizada, asi como brinda soporte para las regulaciones de cumplimiento o piliticas empresariales. para la proteccion de datos. junto con AWS Organizations permite deployar diferentes politicas de proteccion de datos para configurar, administrar y governar sus cuentas y con ellas los recursos y servicios que estan tengan.
  **Benefits** 

  - Centralized Management: mediante la AWS CLI, Console y APIs. de manera rapida y simple puedes configurar politicas de respaldo. aseguranto los recursos atravez de la cuenta y proteger en caso necesario de un backup, restore y retention. 
  - Process automation: basado en politicas y de manera automatizada. la retencion de mgmt, calendarizar los backup y el ciclo de vida de la administrarcion eliminando los scripts personalizados.salvando tiempo y dinero. 
  - impeocws compliance: enforce encrypt audit compliance. de manera que diseñes una estrategia para alinaer los procesos de la empresa con los requerimientos de cumplimiento. 

- en esta solucion, AWS Backup crea copias de seguridad de los volumenes de Amazon EBS adjuntos a varias instancias EC2.

-para los volumenes de Amazon EBS, esta solucion utiliza un almacen de respaldo, que es un contenedor que almacena y organiza respaldos con politicas de acceso y encriptacion. 
- un plan de respaldo define cuando y como hacercopias de seguridad de los rescursos de AWS, incluyendo la frecuencia de respaldos, los periodos de retencio y las politicas de ciclo de vida. se pueden crear multiples planes de respaldo para diferentes requisitos de reslpaldo. 
- los recursos se asignan a un plan de respaldo usando etiquetas o ID de recursos y AWS Backup automaticamente los respalda de acuerdo con el calendario y las reglas de retencion del plan. 
---

### AWS KMS 

cada boveda de respaldo requiere una clave de encriptacion de AWS Key Management Service (AWS KMS) para encriptar toos los respaldos almacenados dentro de la boveda, proporcionando seguridad de datos en reposo.

Descripcion de AWS KMS

es un servicio administrado por AWS. facilita la administracion de cifrado de datos y controla el uso del cifrado en una amplia gama de AWS. como S3, EC2 etc. es un servicio, puede generar las keys en KMS o importarlas desde su infraestructura de administracion de claves. 
 KMS trabaja bajo la FIPS 140-2 que le permite validar, aislar y proteger 

- puedes utilizar KMS para encriptar o desencriptar tu servicios, tambien tus recursos de AWS pueden utilizar tus llaves (keys) para proteger tu informacion 
- otra caracteristica importante es que las llaves de KMS nunca dejan el la boveda por lo que siempre tendras acceso para administrarlas y darles mantenimiento en un solo sitio. 
- puedes crear politicas de uso, estoy quiere decir que podemos determinar el acceso a los usuarios y a que recursos pueden acceder estos, como pueden aplicar las llaves de manera personalizada. 

- todas las llamadas al servicio de KMS  se almacenan en forma de logs en **AWS CloudTrail** por motivos de seguimiento y auditoria. 

**beneficios**

- amazon se encarga de la seguridad fisica, la durabilidad, refuerso por politicas y a demas de estar siempre disponibles 

- encriptar y datos firmados de forma digital. integra encripcion por medio de la SDK, usa llaves para encriptar las aplicaciones y usa APIs para contruir la encriptacion en sus APIs.
- ademas de incorportar la firma digital con llaves asimetricas para asegurar la integridad de la informacion. 

- Compliance and Control.- AWS KMS utilizar varios controles de calidad y segurida validados como PCI DDS,FIPS HIPAA ETC. 
- instancias DE HSM 

> para respaldar una base de datos de Amazon Aurora. esta solucion utiliza el almacen de respaldo predeterminado que AWS Backup Crea automaticamente para cada cuenta de AWS. 


## Laboratorio

### Respaldo de datos. utilizando Backup

- crearemos una boveda de respaldo personalizada 
- configure una copia de seguridad bajo demanda de los volumenes deeee Amazon EBS adjuntos a Amazon EC2 con un ID de recurso.
- Crea planes de respaldo automatizados para volumenes de EBS usando etiquetas.

#### pasos 

1. Nos dirigimos a EC2 para revisar nuestras instancias actuales, seleccionamos a la que queramos realizar el backup. 
  entonces ingresamos a la configuracion de la instancia y en la parte de etiquetas o tags, debemos actualizar la informacion
  - en el panel de configuracion de el almacenamiento de la instancia verificamos el disco EBS al que esta conectado la instancia. 
  -copiamos el volumen ID de la instancia 

  "vol-0gf0d987fg09d7h" device name "/dev/xvda" por ejemplo. 
2. nos dirigimos a la seccion **Elastic Blcok Store** y seleccionamos nuestro volumen que recien copiamos su ID.
  - dentro de sus configuracion debe estar **tags**  y damos clic en administrar 
  - por ulitmo le agregamos una tag para utilizarla al momento de realizar el backup. 
   por ejemplo (key: Backup, Value: True) y click en save. 

3. regresamos a nuestra instancia y de igual manera le agregamos el **tag** corespondiente.

4. repetimos la etiquetas para Bastion_host para agrupar todos nuestros recursos de la app para hacerles el backup. 

##### AWS Backup

1. nos dirigimos a AWS backup para crear el plan 
  - nos dirigimos a la seccion de vaules **Vaults**  y creamos uno nuevo con las siguientes especificaciones. 
   - nombre: any_company_backup
   - vault type: podremos elegir Backup Vault 
   - encrytion key: seleccionamos en la lista desplegable default AWS/backup
   - click en crear un vaul. 

2. Protected Resources
> la pagina protected resources muestra los recursos de AWS que AWS Backup ha respaldado al menos una vez. Los recursos asignados a planes de respaldo pero que aun o se han respaldado no apareceran hasta que se complete su primer trabajo de respaldo programado o bajo demanda. 

- debemos crear un respaldo bajo demanda para este laboratorio. 
- seleccionamos los settings :
  - para Resources type: EBS con su corespondiente id. 
  - Backup Window: Create backup now.
  - Total retention period: 7 days.

AWS Backup se integra con AWS indentity and Access Management IAM para controlar las operaciones de Backup. los roles de IAM definen los permisos que AWS backup necesita para accederrrrrrr y proteger los recursos en su entorno de AWS. 

- para la boveda de respaldo elija : any_company_backup
- rol de IAM: seleccionamos el rol en etes caso **backup_rol** 
- finalmente click e ncrear on-demand backup.

>backup creara copias de seguridad coherentes en caso de bloqueo de los volumenes de EBS, conectados a las instnacias de EC2 de forma predeterminada. la coherencia en caso de bloqueo significa que las instantaneas de todos los volumenes conectados a la misma instancia se acpturan en el mismo momento, lo que ayuda a mantener la integridad de los datos en cofiguraciones de varios volumenes. 

- el proceso de backup tarda al rededor de 5-6 minutos.
3. Creacion de un backup plan
dentro de las opciones de AWS backup nos encontramos a los planes de respaldo. podemos crean un plan perzonaliado. 

> un plan de respaldo es una expresion de politica que define horarios de respaldo, periodos de retencion y reglas de ciclo de vida para los recursos de AWS. Los planes de respaldo automatizan la proteccion en varios recursos y tipos de recursos, como tablas de Amazon DynamoDB, sistemas de archivos de Amazon Elastic File System y volumenes de Amazon EBS.

- selecciona **build a new plan** y con un nombre de preferencia any_company_backup_plan  
- dentro de la configuracion de reglas.definamos una regla nueva.
  - name: backup_rule
  - backup vault: any_company_backup
  - backup frecuncy: weekly - sun 
  - start time: default
  - click en create plan

> las ventanas de backup definen cuando pueden iniciarse los trabajos de backup. la configuracion predetermianda inicia los backups dentro de una ventana de 8hrs a partir de las 5:00 am UTC, con un requisito de finalizacion dentro de las 8 horas posteriores de su hora de inicio. 

> las asignaciones de recursos conectan los planes de respaldo a recursos especificos de AWS. Cada asignacion requiere un rol de IAM que otorgue a AWS backup los permisos necesarios para realizar operaciones de respaldo en los recursos asignados. 

4. **Assign resources** 
geral definimos un nombre 

- tagged:resources
- IAM role: choose an IAM role
- backup_role 
- Define resource section: include all resource type
- Refine seleccion using tags - proporcionando
  - key: backup 
  - condition: equals 
  - value: True
- clikc en asignar recursos 

>ten en cuenta que las etiquetas distinguen mayusculas y minusculas. 
como salida final nos dice una especificaciones

"seleccionando todos los recursos, protege todos los recursos de tu cuenta, tu seleccion actual podria respaldar varios de ellos e incurrir en implicaciones de costos."

aceptamos y seguimos. 

5. **Jobs** 
>los trabajos de backup representan operacioens de backup individuales. la pagina Trabajos muestra el estado de todas las operaciones de backup y restauracion, incluyendo backups a pedido y ejecuciones de planes de backups programados 

- como podemos ver el ultimo trabajo que hicimos como completado. con su propio individuales
- nos vamos a los recursos protegidos **protected resources** 
podremos ver el volumen que respaldamos anteriormente del tipo EBS. damos click sobre el. 

> las organizaciones suelen crear varios planes de respaldo para abordar diferentes objetivos de recuperacion en las cargas de trabajo; por ejemplo, las bases de datos de produccion podrian requerir respaldos diarios con una retencion de 30 dias, mientras que los recursos de desarrollo solo necesitan respaldos semanales de 7 dias . 

