# Transacciones en MongoDB 



## ¿Qué son las transacciones en MongoDB?

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
js
const session = db.getMongo().startSession();
```



## Paso 2: Iniciar la transacción

```js
js
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
js
session.commitTransaction();
```

Si ocurre un error por alguna razón:

```js
js
session.abortTransaction();
```



## Paso 5: Finalizar la sesión

```js
js
session.endSession();
```



## 5. Aspectos Importantes

- Todas las operaciones en la transacción deben ejecutarse dentro de la misma sesión.
- Si la red se cae o ocurre un error, puedes reintentar la transacción.
- Las transacciones pueden involucrar varias colecciones e incluso varias bases de datos dentro del mismo Replica Set.



