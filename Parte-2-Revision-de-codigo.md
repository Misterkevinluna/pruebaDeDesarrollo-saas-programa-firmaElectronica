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
