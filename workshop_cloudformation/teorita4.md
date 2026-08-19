# Pseudo parameters

## Resumen

en este laboratorio, aprenderemos a como utilizar los pseudo parametros. para escribri templates reusables.

## Temas tratados.

para el final de este laboratorio, podras: 

- aprovechar los *Pseudoparametros* para las mejores practicas de portabilidad de plantillas
- identifique ejemplos de casos de uso para aprovechar los *Pseudoparametros* 

al trabajar con plantill de **CloudFormation. ima de las cosas que debemos aspirar es a escribir plantailla modulares y reutilizables, para facilitar el reuso en otras cuentas y regiones de AWS. Ademas de los parametros de CloudFormation, puede optar por idesar su plantilla para utilizar los pseudoparametros, los pseudoparametros son parametros predefinidos por cloudformation

puedes utilizar un pseudo parametro en la misma manera de un paramtro, como por ejemplo, como un argumento en la *Ref intrinsic function*

- AWS::AccountId - regresa el ID de la cuenta que tu elegiste para crear tu template.
- AWS::Region - regresa la region de AWS donde tu levantaras tu stack.
- AWS::Partition - regresas el nombre de la partition. 

## pasos para el laboratorio 

en el siguiente ejemplo, utilizaremos **AWS System Manager Parameter Store** para almacenar centralmente toda la informacion de la configuracion. como el username de la base de datos, para estos describimos un recursos llamado *AWS::SSM::Parameter*¨entu CloudFormation template where you will store the user name. 

