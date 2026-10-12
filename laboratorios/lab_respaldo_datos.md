# Laboratorio de respaldos automatizados con EBS y AWS Backup 
## caso de uso 
una empresa de consultoria de marketing actualmente utiliza multiples scripts personalizados para acer copias de seguriad de sus aplicaciones, para estos scripts se estan volviendo cada vez mas dificiles de mantener. la empresa necesita hacer copias de seguridad de su almacenamiento de host bastion, almacenamiento de servidor web centralizado y snapshots de base de datos de diferentes servicios de AWS. Quieren una solucion de copia de seguridad unifacda y automatizada que agilice la gestion y proporcione una proteccion de datos consistente. 

## objetivos de aprendizaje

- Cree un plan de respaldo para los volumenes de Amazon EBS mediante etiquetas.
- Realizar un respaldo bajo demanda.

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




```

