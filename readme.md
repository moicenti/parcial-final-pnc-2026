# [Moisés Marco Centi Zepeda] [00054123]

## Indicaciones

Recientemente, se utilizó AI para crear un sistema de gestion de una biblioteca, el cual ha generado varios errores, su trabajo es arreglarlo. Dado el siguiente caso de uso, explique y/o resuelva cada problema según se le pida.

---

## Consideraciones

La libreria crea automaticamente un correo con los nombres de la persona

---

## Problemas

### 1. Filtro por autor y género (10%)

QA ha reportado que el endpoint para obtener los libros puede filtrar por **autor** y por **género**, o por cualquiera de los dos de manera individual.

Actualmente:

- Filtrar únicamente por autor funciona correctamente.
- Filtrar únicamente por género funciona correctamente.
- Filtrar por **autor y género al mismo tiempo** provoca que el servidor falle.

**Instrucción:** Explique la causa del problema y resuélvalo.

Bookservice linea 44, dos cosas, primero los parametros se llaman en orden equivocado, primero debería ir el autor y luego el genero según
el Repository. Segundo error, el metodo del repository debería buscar usando un Gender, no un string ya que JPA no puede buscar
por genero si se manda un String por discrepancia entre los tipos, por eso el metodo solo de buscar genero funciona, pues
busca utilizando un Gender, no un String, tambien para evitar problemas, se hizo que el genre se haga uppercase así evitar problemas
con mayusculas y minusculas. 

---

### 2. Error al volver a prestar un libro (10%)

Un usuario reportó que al pedir prestado el libro **The Selfish Gene**, devolverlo e intentar pedirlo prestado nuevamente, el servidor falla.

**Instrucción:** Explique la causa del problema y resuélvalo.

Este error ocurre, pues en la función del servicio de movimientos, se contempla poner como no disponible en caso de que se hayan quedado
sin copias, pero al devolver una copia el estado no se modifica como disponible de vuelta, por lo que si se acaban los libros el libro
deja de estar disponible e independientente de que se retorne el estado seguirá apareciendo como no disponible

---

### 3. Cantidad de libros por género (10%)

Existe un endpoint que devuelve la cantidad de libros disponibles por género. Sin embargo, actualmente dicho endpoint falla.

**Instrucción:** Explique la causa del problema y resuélvalo.

--- El problema es que hay algunos libros con genero vacío y el metodo crashea al intentar obtener un nombre de un Genero nulo,
se agrega una validación para que no se tomen en cuenta los Libros sin Género

### 4. Error al consultar un libro por ID (10%)

Un miembro del equipo de frontend reporta que la siguiente llamada falla:


```http
GET /books?id=ed16ed1e-7017-4697-a08a-d28c09a74acf
```

**Instrucción:** Explique la causa del problema.

Esta consulta está fallando porque el controller tiene la id como una PathVariable, en cambio lo están enviando como
RequestParam , la forma correcta para realizar la petición es 
GET /books/UUID 


---

### 5. Error al crear un libro (10%)

QA ha reportado que el siguiente payload enviado al endpoint `POST /books` provoca un error:

```json
{
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "genre": "classic",
  "isbn": "978-0132350884",
  "available": true,
  "availableCount": 5
}
```

**Instrucción:** Explique la causa del problema.

--- En el método de crear un libro, al momento de asignar el género toma directamente el genre y busca un Enum como ese string
el problema de este payload, es que classic no encuentra un Enum, CLASSIC si lo hace entonces el payload debe tener los generos 
en mayusculas o hacer que el service los fuerce a ser mayusculas.

### 6. Devolución de libros no prestados (20%)

QA ha reportado que un usuario es capaz de devolver libros que nunca ha solicitado en préstamo.

**Instrucción:**

- Confirme si este comportamiento es realmente posible.
- Si es posible, explique la causa y resuelva el problema.
- Si no es posible, explique por qué, haciendo referencia al código correspondiente.

---
El problema si es posible, ya que nunca se verifica que la persona haciendo la petición sea la misma persona que había 
pedido prestado el libro entonces hay que verificar si existe un movimiento con ese Lector y con ese Libro agregando un metodo
al repocitorio que verifique si existe un movimiento con ese Lector y con ese Libro en caso que no exista bloquea el acceso
