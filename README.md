# TechBankNet

TechBankNet es una plataforma bancaria digital diseñada con una arquitectura moderna, altamente escalable y orientada a eventos. Este proyecto adopta de forma rigurosa los principios de **Domain‑Driven Design (DDD)**, el patrón **CQRS (Command Query Responsibility Segregation)** y **Event Sourcing**, apoyándose en **Apache Kafka** como broker de mensajes para garantizar asincronía, resiliencia y procesamiento confiable de eventos. Para la óptima organización y legibilidad del proyecto se han seguido las mejores prácticas de arquitectura de puertos y adaptadores, también conocida como **Arquitectura Hexagonal**.

---

## 🧩 Arquitectura General

La arquitectura de TechBankNet se basa en varios pilares fundamentales:

### **1. Domain‑Driven Design (DDD)**

La plataforma está organizada alrededor del dominio bancario, lo que permite:

* Modelos ricos y expresivos.
* Bounded contexts claramente delimitados.
* Entidades y agregados que representan con precisión comportamientos reales del negocio.
* Independencia entre módulos para facilitar la evolución del sistema.

El dominio bancario, incluyendo cuentas, transacciones, movimientos y auditorías, se encapsula cuidadosamente para evitar acoplamientos innecesarios.

---

## ⚙️ CQRS: Separación de Comandos y Consultas

El uso del patrón **CQRS** permite dividir las operaciones en dos flujos independientes:

### **Comandos (Órdenes de Escrituras contra la BBDD)**

* Representan intenciones de cambio de estado: creación de cuentas, depósitos, retiros, transferencias, etc.
* Se validan dentro de los agregados siguiendo las reglas del dominio.
* Generan eventos que representan lo ocurrido en el sistema.

### **Consultas (Peticiones de Lectura a BBDD)**

* Optimizadas para un acceso rápido y escalable.
* Alimentadas de forma asíncrona por los eventos generados en el modelo de escritura.
* Materializadas en una base de datos NoSQL para lecturas eficientes y distribuidas.

### Beneficios de CQRS en un banco digital

* **Escalabilidad independiente**: las consultas suelen ser más frecuentes que los comandos. Al separarlas, cada componente escala según su demanda.
* **Modelos optimizados**: el modelo de lectura puede estructurarse para rendimiento, mientras que el de escritura se centra en preservar la lógica del dominio.
* **Auditabilidad**: cada cambio se registra como un evento, permitiendo reconstruir el historial completo de cualquier entidad.

---

## 🔄 Event Sourcing + Kafka

TechBankNet no almacena solo estados finales sino que también almacena eventos que describen lo que ha ocurrido en el tiempo. Estos eventos son persistidos y publicados mediante **Apache Kafka**.

### Razones para utilizar Kafka como broker

* **Garantía de entrega y durabilidad**.
* **Procesamiento distribuido** con particiones.
* **Alta capacidad de throughput**, ideal para flujos transaccionales bancarios.
* **Reproducción de eventos** para reconstrucción del modelo de lectura o análisis posteriores.

### Beneficios en una entidad bancaria

* Consistencia eventual garantizada por eventos inmutables.
* Flujos completamente auditables y trazables.
* Integración sencilla con sistemas externos sin acoplar el dominio principal.
* Capacidad de soportar millones de operaciones diarias de forma distribuida.

---

## 🏛️ Bases de Datos: SQL para Comandos y NoSQL para Consultas

TechBankNet utiliza:

* **SQL**: para el modelo de escritura y persistencia de eventos.
* **NoSQL**: para el modelo de lectura materializado, optimizado para consultas rápidas y escalables.

Esta decisión se alinea con la filosofía CQRS, donde cada base de datos está completamente ajustada al tipo de carga que recibe.

---

## 🚀 Flujo de Trabajo Interno

1. El cliente realiza una operación (ej. depósito o retiro).
2. Se genera un **comando** que valida reglas en el dominio.
3. El agregado produce un **evento** que es almacenado y enviado a Kafka.
4. Los **consumidores** procesan el evento para actualizar el modelo de lectura NoSQL.
5. Las consultas posteriores utilizan este modelo materializado para responder en milisegundos.

---

## 🏦 Ventajas Clave para una Entidad Bancaria Digital

* **Alto rendimiento y elasticidad** gracias a Kafka y la separación CQRS.
* **Resiliencia**: los eventos pueden reprocesarse ante fallos.
* **Evolución del sistema** sin interrumpir operaciones críticas.
* **Mejor experiencia de usuario** con consultas instantáneas.
* **Trazabilidad completa** de cualquier acción realizada.

---

## 📋 Prerrequisitos

Para ejecutar TechBankNet en entorno local, necesitas:

* **Docker** y **Docker Compose** instalados.
* **Java 17+**.
* **Maven 3.8+**.
* Puertos disponibles (por defecto):

    * 5432 (PostgreSQL)
    * 27017 (MongoDB)
    * 9092, 29092, 2181 (Kafka y Zookeeper)

---

## 🛠️ Requisitos Técnicos

* Arquitectura basada en microservicios siguiendo los principios SOA (Services Oriented Architecture).
* Comunicación asíncrona mediante Kafka.
* Persistencia híbrida SQL + NoSQL.
* Implementación de CQRS + Event Sourcing.
* Desarrollo orientado al dominio.

---

## ▶️ Ejecución del Proyecto

Para levantar las bases de datos y los brokers de Kafka simplemente ejecuta:

```bash
docker compose up -d
```

Esto iniciará:

* PostgreSQL (modelo de escritura)
* MongoDB (modelo de lectura)
* Zookeeper
* Apache Kafka (brokers y topics necesarios)

Una vez levantada la infraestructura, puedes iniciar los microservicios con:

```bash
mvn clean install
mvn spring-boot:run
```

---

## 🧩 Patrones de Diseño Utilizados

TechBankNet adopta varios patrones de diseño que fortalecen la arquitectura, mejoran la mantenibilidad y aseguran la flexibilidad del sistema.

### **Repository Pattern**

Separa la lógica de acceso a datos del dominio, proporcionando:

* Abstracción clara entre modelo y persistencia.
* Facilita pruebas unitarias mediante mocks.
* Cambios de tecnología (SQL, NoSQL) sin afectar el dominio.

### **Mediator Pattern**

El mediator centraliza y coordina el flujo de comandos:

* Recibe comandos como **OpenAccountCommand**, **DepositFundsCommand**, **WithdrawFundsCommand** y **CloseAccountCommand**.
* Resuelve el handler correspondiente sin acoplar la capa de aplicación.
* Simplifica la orquestación y eliminación de dependencias circulares.

### **Builder Pattern**

Usado para la construcción segura y clara de objetos complejos del dominio:

* Mejora la legibilidad.
* Minimiza errores en la creación de entidades.
* Útil en agregados con múltiples reglas de negocio.

### **Dependency Injection (DI)**

La solución utiliza DI para desacoplar componentes:

* Permite sustituir implementaciones fácilmente.
* Facilita testing.
* Mejora la organización de módulos y servicios.

## 🧑‍💻 Autor

Este proyecto, **TechBankNet**, ha sido diseñado y desarrollado íntegramente como un proyecto personal para explorar y demostrar el potencial de las arquitecturas reactivas y orientadas a eventos en el contexto bancario moderno.
