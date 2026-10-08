# laboratorio para implementar una API con una base de datos de DynamoDB


## caso de uso 

retomando el uso de una aplicacion web de alquiler de vehiculos para visitantes, actualmentes esta impulsada por una base de datos SQL relacional. las pruebas de estres revelan que la base de datos solo puede manejar un numero limitado de usuarios concurrentes. el personal de TI quiere una solucion de base de datos escalable y rentable que pueda manehar altos volumenes de trafico sin requerir una inversion significativa en infraestructura.

## objetivos

- Crear una tabla en DynamoDB para almacenar datos de vehiculos.
- Crear una funcion Lambda para guardar registros de DynamoDB.
- Crear una REST API usando API Gateway

## Arquitectura del laboratorio

## Conceptos claves

- DynamoDB
- API 
- API Gateway
- API REST
- Microservicios

## Teoria 

migra la aplicacion a una arquitectura sin servidor utilizando Amazon DynamoDB para almacenamiento NoSQL escalable, AWS Lambda para computacion y Amazon API Gateway para manejar solicitudes de API. 
