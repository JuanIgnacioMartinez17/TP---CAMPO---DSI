# Trabajo Práctico 1 - Modelo de clases

Este repositorio contiene el modelo de clases del sistema de estacionamiento y las respuestas a las observaciones realizadas sobre la primera entrega.

## Modelo de clases

![Modelo de clases](modelo-clases/Pasted%20Image.png)

## Respuestas a las correcciones

### 1. Revisión de multiplicidades

Se revisaron las multiplicidades de las asociaciones del modelo para representar mejor las relaciones entre las entidades del negocio.

### 2. Reportes

Los reportes no se modelan como clases. Se generan a partir de los datos almacenados en la base de datos, por lo que no forman parte del modelo de clases.

### 3. ¿Para qué está la clase `RegistroLocal`? ¿Local a qué cosa?

`RegistroLocal` no correspondía como clase. “Local” hacía referencia al dispositivo del personal, donde se guardaba el acceso cuando no había conexión (RF-028 y RF-029). Esto es un detalle de implementación y no un concepto del negocio.

Por eso se eliminó la clase. El modo sin conexión queda cubierto mediante los atributos `modoLocal` y `sincronizado` de `AccesoVehicular`.

### 4. ¿Por qué `patente` está en otra clase?

La patente pertenece únicamente a `Vehiculo`, ya que identifica al vehículo. No debe duplicarse en otras clases.

### 5. ¿Para qué está la clase `Invitado`?

`Invitado` representa a una persona ajena a la comunidad universitaria que ingresa con un vehículo que no está registrado de forma permanente. No tiene cuenta ni saldo. Sus datos son nombre, apellido y motivo de la visita; la patente corresponde a `Vehiculo`.

En la versión anterior, `Invitado` estaba desconectada del resto del modelo. Se corrigió asociándola con `AccesoVehicular`: un invitado puede tener uno o más accesos y un acceso corresponde, como máximo, a un invitado.

De esta manera, el modelo permite distinguir los accesos de invitados de los de la comunidad en los reportes operativos y de ocupación, y conserva ese historial aunque posteriormente la patente se asocie a un conductor (RN-002). Los accesos de invitados no generan `Transaccion`, porque no tienen una cuenta de la cual debitar.

### 6. Fechas de vigencia y atributo `activo`

`Tarifa` y `Descuento` tienen fechas de vigencia (`vigenciaDesde` y `vigenciaHasta`). Por ese motivo no es necesario agregar un booleano `activo`: el estado de vigencia se determina comparando la fecha actual con ese período.

### 7. Orden de las transacciones del mismo usuario en la misma fecha

Se agregó a `Transaccion` el atributo `numero`, un entero correlativo que se asigna al crear la transacción tomando el último número existente más uno.

Las transacciones de una misma cuenta se ordenan por ese número, de modo que el orden es estricto aunque coincidan la fecha y la hora. El atributo `fecha` queda como dato informativo del momento en que se registró el movimiento.
