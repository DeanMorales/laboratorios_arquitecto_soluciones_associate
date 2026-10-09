# Laboratorio: API REST con DynamoDB

## Caso de Uso

Retomando la aplicación web de alquiler de vehículos para visitantes, actualmente impulsada por una base de datos SQL relacional. Las pruebas de estrés revelan que la base de datos solo puede manejar un número limitado de usuarios concurrentes. El personal de TI busca una solución de base de datos escalable y rentable que pueda manejar altos volúmenes de tráfico sin requerir una inversión significativa en infraestructura.

---

## Objetivos

- Crear una tabla en **DynamoDB** para almacenar datos de vehículos.
- Crear una función **Lambda** para guardar registros en DynamoDB.
- Crear una **REST API** usando **API Gateway**.

---

## Arquitectura del Laboratorio

> *(Diagrama por agregar)*

---

## Conceptos Clave

| Concepto | Descripción breve |
|---|---|
| **DynamoDB** | Base de datos NoSQL totalmente administrada por AWS |
| **API** | Interfaz de programación de aplicaciones |
| **API Gateway** | Servicio para crear, publicar y gestionar APIs |
| **API REST** | Estilo arquitectónico para servicios web stateless |
| **Microservicios** | Arquitectura de componentes independientes y desacoplados |
| **SQL vs NoSQL** | Bases de datos relacionales vs no relacionales |

---

## Teoría

### Arquitectura de la Solución

La solución migra la aplicación a una arquitectura serverless utilizando tres servicios principales:

- **Amazon DynamoDB** — almacenamiento NoSQL escalable
- **AWS Lambda** — cómputo sin servidores
- **Amazon API Gateway** — punto de entrada para solicitudes HTTP

El flujo básico es:

```
Cliente → HTTP Request (JSON) → API Gateway → Lambda → DynamoDB
```

---

### Amazon API Gateway

Amazon API Gateway actúa como el punto de entrada para que las aplicaciones accedan a datos, lógica de negocios o funcionalidad de servicios de backend. Gestiona solicitudes de API a gran escala, incluyendo autenticación, throttling, caché y monitoreo.

---

### SQL vs NoSQL

| Característica | SQL (Relacional) | NoSQL (No relacional) |
|---|---|---|
| Esquema | Fijo y predefinido | Flexible y dinámico |
| Escalabilidad | Vertical | Horizontal |
| Consultas | SQL estándar | API propia por servicio |
| Casos de uso | Transacciones complejas | Apps móviles, web, gaming |

---

### Amazon DynamoDB

**Definición**: DynamoDB es una base de datos NoSQL totalmente administrada por AWS, diseñada para aplicaciones que requieren baja latencia a cualquier escala.

**Principales beneficios**:

- **Sin servidores**: no requiere aprovisionar, parchear ni administrar servidores, ni instalar software.
- **Alto rendimiento**: latencia en milisegundos de un solo dígito.
- **Escalabilidad automática**: maneja grandes volúmenes de datos sin intervención manual.
- **Alta disponibilidad**: disponibilidad y tolerancia a fallos integradas en múltiples zonas de disponibilidad (AZ), sin necesidad de diseñar esa lógica en la aplicación.
- **Flexibilidad de esquema**: permite agregar atributos a los ítems de forma independiente, agilizando el desarrollo.

---

### Escaneos en DynamoDB

Un escaneo (`Scan`) lee todos los elementos de una tabla o de un índice secundario. El proceso ocurre en tres etapas:

1. **Escaneo completo**: se recorre toda la tabla y se devuelven todos los elementos encontrados.
2. **Aplicación de filtros**: se aplica una expresión de filtro con comparaciones lógicas sobre los elementos escaneados.
3. **Resultado final**: el atributo `Count` refleja el número de elementos devueltos después del filtro.

> ⚠️ Los filtros no reducen el costo del escaneo: DynamoDB cobra por los elementos leídos **antes** de aplicar el filtro, no por los elementos devueltos.

---

## Práctica: Pasos del Laboratorio

En este laboratorio podrás:

1. Crear una tabla de Amazon DynamoDB.
2. Definir el esquema de la tabla (clave de partición y clave de ordenación).
3. Crear una función Lambda para **crear**, **actualizar** y **consultar** elementos de la tabla.
4. Exponer la función Lambda como una API RESTful mediante Amazon API Gateway.

---

## Laboratorio

Se utilizará un archivo de ejemplo escrito en **Python** para conectar la tabla DynamoDB a la API.

> El laboratorio consta de **48 pasos**; a continuación se destacan los puntos y configuraciones esenciales.

### Configuraciones esenciales a recordar

- Definir correctamente la **clave de partición** (`Partition Key`) al crear la tabla en DynamoDB.
- Asignar un **rol de ejecución IAM** a la función Lambda con permisos sobre DynamoDB.
- Configurar la **integración Lambda proxy** en API Gateway para pasar el evento HTTP directamente a la función.
- Realizar el **deploy del API** a un stage antes de probarlo.
- Habilitar **CloudWatch Logs** en API Gateway para facilitar el debugging.

---

## Recursos Adicionales

- [Documentación de Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
- [Documentación de AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
- [Documentación de Amazon API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html)
- [Integración Lambda con API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-lambda-function-handler.html)
- [DynamoDB: operaciones Scan](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Scan.html)
