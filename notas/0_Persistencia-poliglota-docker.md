# Persistencia poliglota y docker

## Persistencia poliglota

Viene de *programacion poliglota*, osea de tener una app hecha con distintos lenguajes de programacion. Tipo backend en java y front en js.

Tiene que ver con tener diferentes tipos de base de datos sql y no sql dependiendo el caso de uso.

por ejemplo:
- relacional
- columnar
- key-value
- grafos
- documentos

se decide dependiendo de
- durabilidad de la informacion (en una cache no quiero que persista para siempre)
- disponibilidad
- consistencia
- escalabilidad
- seguridad


![](attachments/Pasted%20image%2020260804162523.png)

## Docker

Plataforma para desarrolladores y administradores de sistemas, diseñada para desarrollar, implementar y ejecutar aplicaciones con contenedores

### Beneficios

![](attachments/Pasted%20image%2020260804164219.png)

### Imagenes y contenedores

La **imagen** es un paquete ejecutable que incluye todo lo necesario para ejecutar una aplicacion: codigo fuente, bibliotecas, etc

El **conenedor** es una instancia en tiempo de ejecucion de una imagen

### Contenedores vs VMs

![](attachments/Pasted%20image%2020260804164448.png)