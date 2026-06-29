<div align="center">

# Prueva Tecnica para Vacante de Desarrollo de Software
### Kevin Luna Jiménez

**Medellín, Antioquia — Colombia**  
*2026/06/30*

[![Email](https://img.shields.io/badge/Email-kevinlunasjimenez%40gmail.com-D14836?style=flat-square&logo=gmail&logoColor=white)](mailto:kevinlunasjimenez@gmail.com)

</div>

---

## 👨‍💻 Descripción

Esta es una prueba de desarrollo para aplicar a una bacante como desarrollador, es mi investigación y análisis a las preguntas y solicitudes de la prueba.

---

## Parte 1 – Diseño técnico

### 1. Diagrama simple (draw.io, Miro o incluso texto).
[Diagrama Arquitectonico](https://drive.google.com/file/d/1U00ckdSWSOp6TBhZl1Fuj_2B388MP4pc/view?usp=sharing).


### 2. Justificación de decisiones arquitectónicas.
**API Gateway:** Se implementa un API Gateway como único punto de entrada para los clientes. Su función es centralizar el acceso a los microservicios, validar la autenticación mediante JWT y enrutar las solicitudes al servicio correspondiente. Esto reduce el acoplamiento entre el cliente y los microservicios, facilitando el mantenimiento y futuras ampliaciones.

**Identity Service:** Este microservicio concentra la gestión de usuarios, autenticación y autorización mediante JWT. Se decidió unificar estas responsabilidades porque están estrechamente relacionadas y, para el tamaño del sistema planteado, no es necesario separarlas en distintos servicios.

**Document Service:** Es responsable de administrar el ciclo de vida de los documentos: carga, consulta, actualización y eliminación. Separar esta responsabilidad permite mantener una lógica de negocio clara y facilita la evolución del sistema sin afectar otros procesos.

**Signature Service:** Se encarga exclusivamente del proceso de firma electrónica y validación de documentos firmados. Mantener este proceso en un microservicio independiente favorece la separación de responsabilidades y permite escalar esta funcionalidad de forma independiente si la carga de procesamiento aumenta.

**Notification Service:** Gestiona el envío de notificaciones a los usuarios, como la confirmación de una firma o el estado de un documento. Al estar desacoplado del resto de la lógica de negocio, evita que el tiempo de envío de una notificación afecte la experiencia del usuario.

**Base de datos por servicio:** Cada microservicio dispone de su propia base de datos, evitando dependencias directas entre ellos y permitiendo que cada servicio evolucione de forma independiente sin afectar a los demás.

**S3 o equivalente:** Los documentos se almacenan en un servicio de almacenamiento de objetos (como Amazon S3 o un servicio equivalente), mientras que la base de datos únicamente conserva la información y referencia del archivo. Esta estrategia mejora el rendimiento y facilita el almacenamiento de archivos de gran tamaño.


Como posible evolución de la arquitectura, si el volumen de operaciones o la complejidad de las transacciones entre microservicios aumentara, se podría implementar el patrón Saga junto con un sistema de mensajería como RabbitMQ o Kafka. Esto permitiría coordinar procesos distribuidos de forma asíncrona, mejorar la tolerancia a fallos y reducir el acoplamiento entre los servicios.


### 5. Estrategia de seguridad.

La seguridad de la plataforma se basa en los siguientes aspectos:
* **Autenticación mediante JWT:** Cada usuario debe autenticarse para obtener un token JWT, el cual será utilizado para acceder a los recursos protegidos de la plataforma, también tendrá su respectivo refreshToken para que la persona pueda tener un tiempo determinado con la sesión abierta, si la sesión expira se actualizará mediante el refreshtoken dándole así uno nuevo.
*	**Comunicación segura:** Toda la comunicación entre clientes y microservicios se realizará mediante HTTPS, garantizando el cifrado de la información durante la transmisión.
*	**Protección de credenciales:** Las contraseñas de los usuarios se almacenarán utilizando algoritmos de hash seguros como BCrypt o Argon2, evitando guardar contraseñas en texto plano.
*	**Control de acceso por roles (RBAC):** El acceso a las funcionalidades dependerá del rol asignado al usuario, permitiendo definir permisos específicos para administradores, firmantes y clientes.
*	**Gestión de secretos:** Las credenciales de bases de datos, claves privadas y demás información sensible se almacenarán mediante variables de entorno, evitando incluir datos confidenciales dentro del código fuente.
*	**Integridad de documentos:** Se utilizará un hash SHA-256 para verificar que un documento no haya sido alterado después de haber sido firmado.


### 6. Organización del repositorio.

Se propone una estrategia Monorepo, donde cada microservicio se desarrolla de forma independiente dentro del mismo repositorio. Esta organización facilita la administración del proyecto, el versionamiento y el mantenimiento del código, sin perder la independencia de despliegue de cada servicio.

#### **Organización del repositorio**

Se propone una estrategia **Monorepo**, donde cada microservicio se desarrolla de forma independiente dentro del mismo repositorio. Esta organización facilita la administración del proyecto, el versionamiento y el mantenimiento del código, sin perder la independencia de despliegue de cada servicio.

```text
electronic-signature-platform/
│
├── services/
│   ├── api-gateway/
│   ├── identity-service/
│   ├── document-service/
│   ├── signature-service/
│   └── notification-service/
│
├── shared/
│   └── common/
│
├── infrastructure/
│   ├── docker/
│   └── kubernetes/
│
└── README.md
```

#### **Descripción**

- **services/**: Contiene cada uno de los microservicios del sistema. Cada carpeta representa una aplicación independiente con su propio ciclo de vida y despliegue.

- **shared/**: Contiene componentes reutilizables que no implementan lógica de negocio, como utilidades, constantes o librerías internas compartidas.

- **infrastructure/**: Almacena la configuración necesaria para el despliegue e infraestructura del proyecto, como Docker y Kubernetes.

- **README.md**: Documentación principal del proyecto, incluyendo instrucciones de instalación, arquitectura y ejecución.
