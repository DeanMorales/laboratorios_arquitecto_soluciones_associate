# laboratorio de automatizacion por CloudFormation

## actividades del laboratorio

1. crear un stack tecnologico de una EC2 y un grupo de seguridad.
2. crear una plantilla de un S3.
3. implementar un *template* para crear un *stack* de recursos.

## arquitectura 

![Imagen Arquitectura](/imagenes_laboratorios/lab_cloudformation.png "error")

## Cloudformation

permite modelar, aprovisionar y gestionar colecciones de recursos relacionados de AWS al tratar la infraestructura como codigo, haciendo las implementaciones rapidas y consistentes a lo largo del ciclo de vida del recurso. 

en este laboratorio utilizaremos un archivo JSON  o YAML para definir los recursos 

incluso en este laboratorio desplegaremos dos stacks para comprobar que lo podemos hacer N cantidad de veces. al trabajar la infraestructura como codigo, podremos trabajarlo con control de versiones

` codigo del stack `

``` 
#### Step 1 #######

Resources:
  RobotAppServer:
    Type: 'AWS::EC2::Instance'
    Properties:
      InstanceType: t2.micro
      ImageId: ami-087c17d1fe0178315

###################

#### Step 2 #######

  RobotAppSecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: Enable SSH access via port 22
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: '22'
        ToPort: '22'
        CidrIp: 0.0.0.0/0

###################

#### Step 3 #######

      SecurityGroups:
      - !Ref RobotAppSecurityGroup

###################

#### Step 4 #######

  RobotS3Bucket:
    Type: 'AWS::S3::Bucket'
    DeletionPolicy: Delete

###################

##FULL STACK CODE##

Resources:
  RobotAppServer:
    Type: 'AWS::EC2::Instance'
    Properties:
      InstanceType: t2.micro
      ImageId: ami-087c17d1fe0178315
      SecurityGroups:
      - !Ref RobotAppSecurityGroup
  RobotAppSecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: Enable SSH access via port 22
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: '22'
        ToPort: '22'
        CidrIp: 0.0.0.0/0
  RobotS3Bucket:
    Type: 'AWS::S3::Bucket'
    DeletionPolicy: Delete

```

>la principal diferencia entre YAML y Json ,es que YAML admite comentarios. 

## pasos

### paso 1 

* en este paso debemos dirigirnos a **Cloudformation** utilizando la consola 
* despues crear un nuevo stack
* seleccionamos crear un template desde *Composer* 
    * es un interfaz visual que sirve como punto de entrada sin configurar platillas **JSON o YAML** 

### paso 2 

aqui podemos jugar con el codigo de arriba, vamos a ir pegando paso  paso en *Composer* 

la seccion *Resources* es el unico elemento obligatorio en una plantilla de CloudFormation, 
* declara los recursos de AWS crear y como configurarlo.
Cada otra seccion de la plantilal como **Parameters, Outputs o Mappings es opcional** 

en el siguiente enlace se muestra doc oficial de como levantar una Instancia EC2 

[levantar una EC2](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html)

### paso 3 implementacion de la plantilla

* recordemos que para levantar una instancia es obligatorio un grupo de seguridad 
* utilizamos la propiedad *!Ref* para hacer la conexion entre recursos
* al omitir un nombre de **Bucket S3** se genera un automaticamente

* como siguiente step al finalizar nuestra plantilla, debemos dar clic en **Create Template** 

> esta plantilla se guarda de manera automatica en un bucket s3 donde almacenaremos diferentes versiones y de otros stacks.

despues podemos proceder a terminar el stack para lanzarlo inmediatamente y probar que todo salga de manera correcta. 

### paso 4 levantar infrasetructua

* continuando con el stack, debemos darle un nombre, pues nos ayuda a reconocer y comprobar los recursos.
* en este stack no hay parametro a ingresar o personalizar.

