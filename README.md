# MongoDB Investigación



## ¿Qué son las transacciones en MongoDB? (16/07/2025)

Una **transacción en MongoDB** es una secuencia de operaciones (lecturas y escrituras) que se ejecutan como una única unidad lógica de trabajo. El principal objetivo de las transacciones es garantizar que todas las operaciones agrupadas se completen satisfactoriamente y, si alguna falla, se revierten todas, asegurando la integridad y consistencia de los datos.



## Propiedades de las transacciones (ACID)

Las transacciones en MongoDB cumplen con las propiedades **ACID**:

- **Atomicidad:** Todas las operaciones dentro de una transacción se completan o ninguna lo hace.

- **Consistencia:** La base de datos pasa de un estado válido a otro. No se permiten datos inválidos.

- **Aislamiento:** Las transacciones en proceso no afectan ni son afectadas por otras transacciones simultáneas.

- **Durabilidad:** Una vez que se confirma una transacción, los cambios son permanentes, incluso si ocurre una falla del sistema.

  

## ¿Para qué sirven?

Las transacciones son útiles cuando necesitas:

- Realizar varias actualizaciones o inserciones que deben aplicarse juntas (por ejemplo, transferencias bancarias o sistemas de pedidos).

- Garantizar que los cambios se realicen completamente o no se realicen en absoluto.

- Personalizar flujos de trabajo complejos que implican varias colecciones o documentos.

  

## Características clave

- **Multidocumento:** Se pueden agrupar operaciones sobre múltiples documentos e incluso colecciones.

- **Distribuidas:** MongoDB soporta transacciones distribuidas en diferentes bases de datos.

- **A partir de la versión 4.0:** Las transacciones multidocumento están disponibles desde MongoDB 4.0, pero solo si tu base funciona como *ReplicaSet* y no en despliegue standalone. 

  

## Ejemplo Básico (conceptual)

1. Inicias una **sesión**.

2. Comienzas una **transacción** dentro de esa sesión.

3. Ejecutas varias operaciones CRUD (create, read, update, delete).

4. Si todo sale bien, **confirma** (commit) la transacción y se aplican los cambios.

5. Si ocurre un error, puedes **revertir** (rollback/abort) la transacción y se deshace todo lo hecho dentro de ella.

   

## Ventajas

- Permiten trabajar con operaciones complejas asegurando que los datos se mantengan coherentes y correctos.
- Son especialmente útiles para aplicaciones donde la precisión de los datos es crítica (por ejemplo, aplicaciones bancarias o de gestión de inventarios).



## Ejemplo Práctico en la Consola de MongoDB (mongo shell)

Supón que tienes dos colecciones: `cuentas` (cuentas bancarias) y debes transferir dinero de una cuenta a otra.



## Paso 1: Iniciar la sesión

```js
const session = db.getMongo().startSession();
```



## Paso 2: Iniciar la transacción

```js
session.startTransaction();
```



## Paso 3: Ejecutar operaciones dentro de la sesión

```js
jsconst cuentas = session.getDatabase("miBanco").cuentas;

// Debita $100 de la cuenta origen
cuentas.updateOne(
  { nombre: "Juan" },
  { $inc: { saldo: -100 } }
);

// Acredita $100 en la cuenta destino
cuentas.updateOne(
  { nombre: "Ana" },
  { $inc: { saldo: 100 } }
);
```



## Paso 4: Confirmar (commit) o revertir (abort) la transacción

Si todo sale bien:

```js
session.commitTransaction();
```

Si ocurre un error por alguna razón:

```js
session.abortTransaction();
```



## Paso 5: Finalizar la sesión

```js
session.endSession();
```



## 5. Aspectos Importantes

- Todas las operaciones en la transacción deben ejecutarse dentro de la misma sesión.
- Si la red se cae o ocurre un error, puedes reintentar la transacción.
- Las transacciones pueden involucrar varias colecciones e incluso varias bases de datos dentro del mismo Replica Set.



_______________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________







# Introducción a Indexación de campos (17/07/2025)



## ¿Qué son Indices?



En MongoDB, los índices son estructuras de datos que almacenan una representación ordenada de los valores de uno o más campos en una colección. Estos índices se utilizan para acelerar las consultas y mejorar el rendimiento de las operaciones de búsqueda, ayudando al motor de bases de datos a encontrar y acceder a los documentos de manera más eficiente y rápida.

Un índice es como el **índice de un libro**. Si un libro tiene 1,000 páginas y quieres buscar rápidamente todos los temas que hablen de "JavaScript", es mejor tener un índice que te diga en qué páginas aparece...

En MongoDB pasa lo mismo. Si tienes una colección con muchos documentos, un índice te permite **buscar más rápido** en un campo específico. 



## Ejemplo básico sin índice

Supón que tienes esta colección:

```js
// Colección: usuarios

{
  "_id": ObjectId("..."),
  "nombre": "Carlos",
  "email": "carlos@example.com",
  "edad": 25
}
```

Tienes 10,000 documentos como ese.



Ahora haces una consulta:



```js
db.usuarios.find({ email: "carlos@example.com" })
```



Si **no tienes un índice** en el campo `email`, MongoDB va a revisar uno por uno todos los documentos hasta encontrar el que tenga ese email → esto se llama ***búsqueda secuencial*.**



## ✅ Crear un índice en el campo `email`

Ahora, si creas un índice en `email`, MongoDB crea una **estructura especial ordenada** para buscar más rápido por ese campo.



### En Mongo Shell:

```js
CopiarEditar

db.usuarios.createIndex({ email: 1 }) // 1 significa orden ascendente
```

Ahora, cada vez que consultes por `email`, MongoDB no buscará uno por uno, sino que irá directo al lugar exacto, como si buscara una palabra en un diccionario.





_______________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________







# Taller de Ejercicios (18/07/2025)



Resuelve los siguientes ejercicios usando `mongosh`.

### Ejercicio 1
- Crea un índice sobre el campo `edad` y realiza una consulta de usuarios mayores de 30 años.

  


```js
db.usuarios.createIndex({ edad:1 });

db.usuario.find({ edad:{ $gt:30} });

```



### Ejercicio 2

- Crea un índice único sobre el campo "Nombre", intenta insertar un duplicado y verifica el error. 

  


```js
db.usuarios.createIndex({nombre:1} , {unique: true}); 

db.usuarios.insertOne([nombre: "camilo"]); 

```



### Ejercicio 3,

* Agrega un campo embebido llamado `perfil` con los subcampos `ocupacion` y `nivel_estudios`. Crea un índice sobre `perfil.ocupacion` y realiza una consulta.



```js
db.usuarios.updateMany({}, { $set: { perfil: { ocupacion:"", nivel_estudios:""  }}});

db.usuarios.createIndex( { "perfil.ocupacion": 1});


db.usuarios.find( {}, {  "perfil.ocupacion":1 });
```



### Ejercicio 4,

Crea un índice multikey sobre un array llamado `habilidades` y realiza una consulta que lo use.

```js
db.usuarios.updateMany({}, { $set: { habilidades: [ "", "" , ""      ]}});

db.usuarios.createIndex( { "habilidades": 1});


db.usuarios.find( {}, {  habilidades: ""});
```





















































