# Trabajo Práctico 2 - Modelado de Datos

## Entrega V1 - Modelo de Datos

![Modelado de Datos V1](modelo-datos/Trabajo%20Practico%202%20Modelado%20de%20Datos%20V1.png)
![Modelado de Datos V1](modelo-datos/Trabajo%20Practico%202%20Modelado%20de%20Datos%20-%20Tablas%20V1.png)


# Trabajo Práctico 1 - Modelo de clases + Modelo de sistemas

Este repositorio contiene las distintas versiones del modelo de clases del sistema de estacionamiento y las respuestas a las observaciones realizadas sobre cada entrega.

![Modelo de Sistemas V3](modelo-clases/Trabajo%20Practico%201%20-%20Modelo%20de%20Sistemas%20V3.png)

```text
// RF-015
AccesoVehicular :: crearIngreso(numero: int, vehiculo: Vehiculo, personal: Usuario, estacionamiento: Estacionamiento): AccesoVehicular
{
        acceso = self.new()
        acceso.setNumero(numero)
        acceso.setFechaHora(getDate())
        acceso.setTipo(TipoAcceso.INGRESO)
        acceso.setVehiculo(vehiculo)
        acceso.setPersonal(personal)
        acceso.setEstacionamiento(estacionamiento)
        acceso.setCuenta(vehiculo.getConductor().getCuenta())
        acceso.setSincronizado(true)
        acceso.setModoLocal(false)
        return acceso
}

// RF-015 – Ingreso de vehículo registrado (incluye RF-017)
AccesoVehicularController :: registrarIngreso(patente: string, personal: Usuario, estacionamiento: Estacionamiento): AccesoVehicular
{
        vehiculo = VehiculoBD.recuperarVehiculo(patente)        // RN-001: una patente = un vehículo
        if vehiculo <> null and vehiculo.puedeIngresar() and estacionamiento.hayCupo() and estacionamiento.estaAbierto(getDate())        // RF-013
        {
                numero = AccesoVehicularBD.ultimoNumero() + 1
                acceso = AccesoVehicular.crearIngreso(numero, vehiculo, personal, estacionamiento)        // RF-015
                tarifa = estacionamiento.getTarifaVigente(getDate())        // RF-012 / RN-005
                transaccion = acceso.getCuenta().debitar(tarifa)        // RF-017 / RN-004
                acceso.setTransaccion(transaccion)
                estacionamiento.ocuparCupo()        // RF-019 / RF-020
                AccesoVehicularBD.persistir(acceso)
                return acceso
        }
        return null
}

// RF-015 (solo vehículos ACTIVOS y que no estén adentro)
Vehiculo :: puedeIngresar(): boolean
{
        if self.estado = EstadoVehiculo.ACTIVO and not self.estaAdentro()
        {
                return true
        }
        return false
}

Vehiculo :: estaAdentro(): boolean
{
        ultimo = self.AccesoVehicular -> sortedBy(a | a.fechaHora) -> last()
        if ultimo <> null and ultimo.getTipo() = TipoAcceso.INGRESO
        {
                return true
        }
        return false
}

// RF-012 / RN-005
Estacionamiento :: getTarifaVigente(fecha: date): Tarifa
{
        return self.Tarifa -> select(t | t.estaVigente(fecha)) -> first()
}

// RF-012 / RN-005
Tarifa :: estaVigente(fecha: date): boolean
{
        if self.vigenciaDesde <= fecha and self.vigenciaHasta >= fecha
        {
                return true
        }
        return false
}

// RF-017
Cuenta :: debitar(tarifa: Tarifa): Transaccion
{
        transaccion = Transaccion.crearTransaccion(TipoTransaccion.DESCUENTO, tarifa.getMonto(), tarifa, self)
        self.setSaldo(self.saldo - tarifa.getMonto())        // RN-004: el saldo puede quedar negativo
        return transaccion
}

Transaccion :: crearTransaccion(tipo: TipoTransaccion, monto: float, tarifa: Tarifa, cuenta: Cuenta): Transaccion
{
        transaccion = self.new()
        transaccion.setTipo(tipo)
        transaccion.setMonto(monto)
        transaccion.setFechaHora(getDate())
        transaccion.setEstado(EstadoTransaccion.CONFIRMADA)
        transaccion.setTarifa(tarifa)
        transaccion.setCuenta(cuenta)
        return transaccion
}

// RF-015 / RF-018
AccesoVehicular :: crearIngresoInvitado(numero: int, invitado: Invitado, personal: Usuario, estacionamiento: Estacionamiento): AccesoVehicular
{
        acceso = self.new()
        acceso.setNumero(numero)
        acceso.setFechaHora(getDate())
        acceso.setTipo(TipoAcceso.INGRESO)
        acceso.setInvitado(invitado)
        acceso.setPersonal(personal)
        acceso.setEstacionamiento(estacionamiento)
        acceso.setSincronizado(true)
        acceso.setModoLocal(false)
        return acceso        // sin cuenta ni transacción: el invitado no tiene saldo
}

// RF-018 – Ingreso de vehículo invitado
AccesoVehicularController :: registrarIngresoInvitado(patente: string, nombre: string, apellido: string, motivo: string,
                personal: Usuario, estacionamiento: Estacionamiento): AccesoVehicular
{
        if VehiculoBD.recuperarVehiculo(patente) = null and estacionamiento.hayCupo() and estacionamiento.estaAbierto(getDate())        // RN-003 / RF-013
        {
                invitado = InvitadoBD.recuperarInvitado(patente)        // RN-002: si ya vino antes, se reutiliza
                if invitado = null
                {
                        invitado = Invitado.crearInvitado(nombre, apellido, patente, motivo)        // RF-018
                }
                if not invitado.estaAdentro()
                {
                        numero = AccesoVehicularBD.ultimoNumero() + 1
                        acceso = AccesoVehicular.crearIngresoInvitado(numero, invitado, personal, estacionamiento)        // RF-015 / RF-018
                        estacionamiento.ocuparCupo()        // RF-019 / RF-020
                        AccesoVehicularBD.persistir(acceso)
                        return acceso
                }
        }
        return null
}

// RF-018
Invitado :: crearInvitado(nombre: string, apellido: string, patente: string, motivo: string): Invitado
{
        invitado = self.new()
        invitado.setNombre(nombre)
        invitado.setApellido(apellido)
        invitado.setPatente(patente)
        invitado.setMotivo(motivo)
        return invitado
}

Invitado :: estaAdentro(): boolean
{
        ultimo = self.AccesoVehicular -> sortedBy(a | a.fechaHora) -> last()
        if ultimo <> null and ultimo.getTipo() = TipoAcceso.INGRESO
        {
                return true
        }
        return false
}

// RF-013 (capacidad máxima de cupos)
Estacionamiento :: hayCupo(): boolean
{
        if self.cuposOcupados < self.cuposTotales
        {
                return true
        }
        return false
}

// RF-013 (horarios de funcionamiento)
Estacionamiento :: estaAbierto(fechaHora: datetime): boolean
{
        if fechaHora.hora >= self.horarioApertura and fechaHora.hora <= self.horarioCierre
        {
                return true
        }
        return false
}

// RF-019 / RF-020 (ocupación y cupos disponibles)
Estacionamiento :: ocuparCupo()
{
        self.setCuposOcupados(self.cuposOcupados + 1)
}
```

## Entrega V2 - Modelo de clases corregido

![Trabajo Práctico 1 - Modelo de clases (V2)](modelo-clases/Trabajo%20Pr%C3%A1ctico%201%20-%20Modelo%20de%20clases%20%28V2%20-%20Entrega%20corregida%29.png)

## Entrega V1 - Modelo de clases

![Trabajo Práctico 1 - Modelo de clases (V1)](modelo-clases/Trabajo%20Pr%C3%A1ctico%201%20-%20Modelo%20de%20clases%20%28V1%29.png)

La V1 corresponde a la entrega original y la V2 a la entrega corregida según las observaciones del profesor.

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
