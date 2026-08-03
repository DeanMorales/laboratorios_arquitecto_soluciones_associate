# Instrinsic Functions

este laboratorio muestra como utilizar las *Funciones intrinsecas* en tu template. 

Las funciones intrinsecas son funciones integradas que le ayudan a administrar sus pilas. sin ellos, estaras limitado a plantillas muy basicas. similares al la un template basico de un **S3**

## Temas tratados

- utilizar la **Referencia**  *!Ref* para asignar dinamicamente los valores a una propiedad de recurso.
- etiqueta una instancia con Fn::join
- agregar una etiqueta a la instancias usando funciones.

### inicio laboratorio

> en los laboratorios pasados habiamos hardcodeado* las AMI ID directamente para levantar una EC2

ahora crearemos una funcion con referencia para hacerlo mas dinamico y flexible, y pasar esta variable al runtime. 

