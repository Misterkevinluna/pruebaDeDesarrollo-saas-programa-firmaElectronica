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


## Parte 2 - Revisión de código.
Con el poco conocimiento que tengo de nest pero con el conocimiento que tengo de node veo varias faltas.


### 1. ¿Qué problemas de seguridad ves?

* **Contraseña almacenada en texto plano**
  `password: data.password`
  No se deben de guardar las contraseñas en texto plano, se deben de cifrar.


* **jwt secret hardcodeado**
  AQUÍ:
  ```ts
  jwt.sign(
    ...
    'super-secret-key'

  ```
  TAMBIEN AQUÍ:
  ```TS
    jwt.verify(token, 'super-secret-key');
  ```
  La llave secreta jamas debe de estar escrita en el codigo, esta proviene del .env o de ahi se debe de extraer.


  * **Si no existe token también deja pasar**
    AQUÍ:
    ```TS
      if (!token) {
        next();
        return;
      }
    ```
    El middleware no esta protegiando absolutamente nada.


* **No existe validación del Body**
  ```ts
  @Body() body:any
  
  ```
  El body puede estar recibiendo cualquier cosa, un string, numero, arrays, ts, se deberia de especificar que espera recibir, puede ser un tipo de dato o una estructura como un json, clases o dto.


* **Se usa any**
  Se usa 'any' y muchas veces, eso le quita la seguridad que brinda ts.
  `body:any`
  `users:any[]`
  `data:any`
  Falta de tipación.


* **Se expone la contraseña al cliente**
  Al retornar

  ```ts
  return {

    user,

    token

  }
  ```
  El objeto `user` contiene el `password` osea el json con la estructura completa incluyendo el password:

  ```ts
  {
    "id":"...",
    "email":"...",
    "password":"123456",
    "role":"user"
  }

  ```

  Nunca se debe de responder con la contrseña.


* **IDs inseguros**
  ```ts
  Math.random().toString()
  ```
  No garantiza unicidad.
  Puede repetirse.
  Debe usarse `uuid`.


* **No existe expiración del usuario**
  El jwt dura 30 dias, pero no existe mecanismo de revocación, blacklist, refresh token.


* **No existe autorización**
  El jwt contiene `role` pero nunca se valida, no existen wards.



### 2. ¿Qué errores de arquitectura encuentras?
El código presenta varios problemas de arquitectura que afectan la mantenibilidad, escalabilidad y el cumplimiento de los principios SOLID.

 
 * **El Service actúa como Base de Datos**

Actualmente el servicio almacena los usuarios en memoria:

```ts
private users: any[] = [];
```

Esto rompe el principio de responsabilidad única (**SRP**), ya que el servicio no debería encargarse de persistir información.

Lo correcto sería que la persistencia estuviera delegada a un **Repository** o a una capa de acceso a datos utilizando herramientas como:

- TypeORM
- Prisma
- Mongoose
- Sequelize

---

* **El Service tiene demasiadas responsabilidades**

El `UserService` actualmente:

- Valida datos.
- Crea el usuario.
- Almacena el usuario.
- Genera el JWT.

Esto viola el principio **Single Responsibility Principle (SRP)**.

Cada responsabilidad debería estar separada:

- **UserService** → Gestión de usuarios.
- **AuthService** → Autenticación y generación de tokens.
- **Repository** → Persistencia de datos.

---

* **No existe una capa de persistencia**

La aplicación no tiene un Repository.

Actualmente el controlador depende directamente del Service y el Service almacena los datos en memoria.

La arquitectura debería incluir una capa de persistencia.

```text
Controller
      │
      ▼
Use Case / Service
      │
      ▼
Repository
      │
      ▼
Database
```

---

* **No existe un dominio bien definido**

No existen elementos propios de una arquitectura limpia como:

- Entidades (Entities)
- Casos de uso (Use Cases)
- Value Objects
- Interfaces de repositorio

Toda la lógica está concentrada dentro del mismo servicio.

---

* **Alto acoplamiento**

El controlador depende directamente de una implementación concreta (`UserService`).

En proyectos grandes es recomendable depender de abstracciones (interfaces o puertos) para facilitar:

- Pruebas unitarias.
- Sustitución de implementaciones.
- Escalabilidad.

---

### 3. ¿Qué malas prácticas de NestJS hay?

El proyecto no aprovecha varias características que NestJS ofrece para desarrollar aplicaciones escalables y mantenibles.

* **Uso de `any`**

Se utilizan múltiples tipos `any`.

```ts
@Body() body: any
```

```ts
private users: any[] = [];
```

```ts
async createUser(data: any)
```

```ts
req: any
```

Esto elimina la seguridad de tipos que proporciona TypeScript.

Lo correcto sería definir DTOs e interfaces.

---

* **No utiliza DTOs**

Actualmente el controlador recibe cualquier estructura.

```ts
@Post()
async createUser(@Body() body: any)
```

Lo recomendable es:

```ts
@Post()
async createUser(@Body() dto: CreateUserDto)
```

---

* **No utiliza ValidationPipe**

No existe validación automática de los datos recibidos.

NestJS recomienda utilizar:

```ts
app.useGlobalPipes(new ValidationPipe());
```

---

* **No utiliza class-validator**

El DTO debería validar los datos mediante decoradores.

Ejemplo:

```ts
export class CreateUserDto {

    @IsEmail()
    email: string;

    @MinLength(8)
    password: string;

}
```

---

* **No utiliza ConfigModule**

La llave secreta está escrita directamente en el código.

```ts
'super-secret-key'
```

Lo correcto sería obtenerla mediante:

```ts
ConfigService.get('JWT_SECRET')
```

---

* **No utiliza JwtModule**

Se importa directamente la librería:

```ts
import * as jwt from 'jsonwebtoken';
```

NestJS proporciona:

- JwtModule
- JwtService

Los cuales permiten una mejor integración con el framework.

---

* **No utiliza Passport**

NestJS recomienda implementar autenticación utilizando:

- Passport
- JwtStrategy
- AuthGuard

En lugar de implementar la autenticación manualmente mediante Middleware.

---

* **No utiliza Guards**

La autenticación está implementada mediante Middleware.

Sin embargo, la autorización en NestJS debe implementarse mediante Guards.

Ejemplo:

```ts
@UseGuards(AuthGuard('jwt'))
```

---

### 4. ¿Cómo lo reestructurarías?

Separaría las responsabilidades en módulos independientes, mas o menos así:

```text
src
│
├── modules
│   ├── auth
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── auth.module.ts
│   │   ├── jwt.strategy.ts
│   │   ├── guards
│   │   └── dto
│   │
│   ├── users
│   │   ├── user.controller.ts
│   │   ├── user.service.ts
│   │   ├── user.repository.ts
│   │   ├── user.entity.ts
│   │   └── dto
│   │
│   └── common
│       ├── decorators
│       ├── exceptions
│       ├── filters
│       ├── interceptors
│       └── pipes
│
├── config
└── database
```

También reorganizaría el flujo de dependencias.

```text
Controller
      │
      ▼
Use Case
      │
      ▼
Repository
      │
      ▼
Database
```

Además dividiría las responsabilidades.

| Componente | Responsabilidad |
|------------|-----------------|
| UserService | Gestión de usuarios |
| AuthService | Login, JWT, Hash de contraseñas |
| Repository | Persistencia |
| Controller | Exposición de endpoints |

---

### 5. ¿Qué riesgos tendría esto en producción?

Si este código llegara a producción tendría riesgos importantes.

#### Riesgos de seguridad

- Contraseñas almacenadas en texto plano.
- Secret JWT expuesto en el código fuente.
- Acceso permitido incluso con tokens inválidos.
- Ausencia de validación de entradas.
- Exposición de la contraseña en la respuesta.

---

#### Riesgos operativos

- Todos los usuarios se perderían al reiniciar el servidor.
- No existe persistencia en base de datos.
- No soporta múltiples instancias del servidor.
- No existe control de concurrencia.

---

#### Riesgos de escalabilidad

- No es posible distribuir la aplicación horizontalmente.
- No existe separación entre autenticación y gestión de usuarios.
- Alto acoplamiento entre componentes.

---


### 6. ¿Cómo escalarías este módulo?

El crecimiento debería realizarse por etapas.

#### Primera etapa

Implementaría:

- DTOs.
- ValidationPipe.
- Repositories.
- Base de datos.
- ConfigModule.
- JwtModule.

---

#### Segunda etapa

Agregaría:

- Refresh Tokens.
- Roles.
- Permissions.
- Guards.
- Exception Filters.
- Logging.
- Rate Limiting.

---

#### Tercera etapa

Para soportar mayor carga:

- RabbitMQ.
- Microservicios.
- Balanceadores de carga.
- Cache distribuida.

---

### 7. ¿Qué mejorarías primero?

Priorizaría los cambios en el siguiente orden:

1. Hash de contraseñas utilizando `bcrypt`.
2. Mover la llave JWT al archivo `.env`.
3. Corregir el Middleware para bloquear accesos no autorizados.
4. Implementar DTOs.
5. Agregar `ValidationPipe`.
6. Reemplazar `throw new Error()` por excepciones propias de NestJS.
7. Separar `AuthService` de `UserService`.
8. Crear una capa Repository.
9. Persistir los usuarios en una base de datos.
10. Reemplazar el Middleware por `JwtStrategy` y `AuthGuard`.

---

### Otra Observación

Existe un error importante en la validación del token.

Actualmente se obtiene el encabezado completo:

```ts
const token = req.headers.authorization;
```

El encabezado normalmente llega así:

```text
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR...
```

Sin embargo, el código intenta verificar directamente ese valor.

```ts
jwt.verify(token, 'super-secret-key');
```

`jwt.verify()` espera únicamente el JWT, no la cadena `"Bearer <token>"`.

Lo correcto sería:

```ts
const authHeader = req.headers.authorization;

if (!authHeader || !authHeader.startsWith('Bearer ')) {
    throw new UnauthorizedException();
}

const token = authHeader.substring(7);

const decoded = jwt.verify(token, jwtSecret);
```
