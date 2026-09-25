# Análisis de requerimientos

## 1. Descripción del caso

El Conjunto Residencial Los Cerezos, ubicado en Bogotá, está sometido al régimen de propiedad horizontal de la Ley 675 de 2001. Su administración está a cargo de un administrador y un auxiliar administrativo, y la portería funciona las 24 horas con vigilantes por turnos. La base de datos debe apoyar tres frentes: la cartera de la copropiedad, la reserva de zonas comunes y el control de visitantes.

Parámetros de operación del caso:

- El conjunto tiene tres torres de ocho pisos, con cuatro apartamentos por piso. El número de cada apartamento se forma con el piso y la posición: el 804 es el apartamento 4 del piso 8.
- Cada apartamento tiene un área privada y un coeficiente de copropiedad fijados en el reglamento de propiedad horizontal.
- La cuota ordinaria de administración se causa el primer día de cada mes y vence el día 10. Su valor es el presupuesto mensual aprobado por la asamblea multiplicado por el coeficiente del apartamento.
- Las zonas comunes reservables son el salón comunal, la zona de BBQ y la cancha múltiple. Se reservan por franjas: mañana (8:00 a 12:00), tarde (13:00 a 17:00) y noche (18:00 a 22:00). El salón comunal y la zona de BBQ tienen tarifa; la cancha no tiene costo.
- La administración no recibe pagos en efectivo: los residentes pagan por consignación, transferencia o PSE y envían el soporte.

### 1.1 Definiciones

| Término | Definición |
|---------|------------|
| Cargo | Valor que se cobra a un apartamento por un concepto, como la cuota de administración, una multa o el alquiler de una zona común |
| Saldo de un cargo | Valor del cargo menos la suma de los pagos aplicados a él |
| Fecha de corte | Fecha a la que se calcula el estado de la cartera |
| Cargo vencido | Cargo con saldo mayor que cero cuya fecha de vencimiento es anterior a la fecha de corte |
| Apartamento en mora | Apartamento que tiene al menos un cargo vencido |
| Paz y salvo | Condición de un apartamento cuyos cargos causados hasta la fecha de corte tienen saldo cero |
| Residente vigente | Residente que no tiene fecha de salida registrada |

El saldo no se almacena en ninguna tabla: se calcula en las consultas a partir de los cargos y los pagos, para evitar un dato derivado que pueda quedar desactualizado.

## 2. Usuarios del sistema

| Usuario | Funciones | Información que registra | Información que consulta |
|---------|-----------|--------------------------|--------------------------|
| Administrador | Dirige la operación y responde ante la asamblea y el consejo | Personas, apartamentos, residentes, vehículos, conceptos de cobro, cargos, zonas comunes y reservas | Cartera, recaudo, paz y salvo, uso de zonas comunes |
| Auxiliar administrativo | Apoya la gestión de cartera | Pagos | Cargos pendientes y pagos de cada apartamento |
| Vigilante de portería | Controla el ingreso al conjunto | Visitantes y visitas | Residentes vigentes, vehículos autorizados y visitantes que siguen dentro |
| Consejo de administración | Supervisa la gestión del administrador | Ninguna | Reportes de cartera, recaudo y uso de zonas comunes |

Los propietarios, residentes y visitantes no acceden directamente al sistema. Son los titulares de los datos personales que se registran y realizan sus solicitudes, como reservas o certificados de paz y salvo, a través de la administración o de la portería.

## 3. Información que se almacena

| Entidad | Información |
|---------|-------------|
| torre | Nombre y número de pisos |
| apartamento | Torre, número, área privada, coeficiente de copropiedad y propietario |
| persona | Tipo y número de documento, nombres, apellidos, teléfono y correo |
| residente | Persona, apartamento que habita, tipo de residente (propietario, arrendatario o familiar), fecha de ingreso y fecha de salida |
| vehiculo | Placa, tipo (carro o moto), marca, color y apartamento al que está autorizado |
| concepto_cobro | Nombre y descripción del concepto |
| cargo | Apartamento, concepto, fecha de causación, fecha de vencimiento y valor |
| pago | Cargo al que se aplica, empleado que lo registra, fecha, valor, medio de pago y referencia bancaria |
| zona_comun | Nombre, capacidad y tarifa por franja |
| reserva | Zona común, residente que la solicita, fecha de solicitud, fecha de uso, franja y cargo asociado cuando la zona tiene tarifa |
| visitante | Tipo y número de documento, nombres y apellidos |
| visita | Visitante, residente que autoriza el ingreso, vigilante que lo registra, fecha y hora de ingreso, fecha y hora de salida y placa del vehículo |
| empleado | Tipo y número de documento, nombres, apellidos, teléfono y rol |

Los propietarios y los residentes se registran en una sola entidad, `persona`, porque un mismo individuo puede cumplir ambos papeles. Registrarlo en dos tablas duplicaría sus datos y permitiría que quedaran inconsistentes. La propiedad se representa con la referencia del apartamento a su propietario, y la residencia con la entidad `residente`, que vincula a una persona con el apartamento que habita.

## 4. Procesos

### P1. Registro de propietarios, residentes y vehículos

Cuando una persona compra un apartamento o llega a vivir en el conjunto, el administrador la registra en `persona` si aún no existe. Si es propietaria, la asigna como propietaria del apartamento; si va a vivir en él, la registra como residente con su tipo y fecha de ingreso. Luego registra los vehículos autorizados del apartamento. Cuando un residente se muda, se registra su fecha de salida en lugar de eliminarlo, para conservar las visitas y reservas asociadas.

### P2. Causación de cargos

El primer día de cada mes, el administrador genera un cargo de cuota ordinaria de administración para cada apartamento, con vencimiento el día 10. De forma ocasional registra cuotas extraordinarias aprobadas por la asamblea y multas impuestas por el consejo de administración.

### P3. Registro de pagos

El residente paga por consignación, transferencia o PSE y envía el soporte. El auxiliar administrativo verifica el soporte y registra el pago sobre el cargo correspondiente, con su referencia bancaria. Si un mismo pago cubre varios cargos, se registra un pago por cada cargo con la misma referencia. El dinero se aplica primero a los intereses y luego a los cargos más antiguos, de acuerdo con el artículo 1653 del Código Civil.

### P4. Liquidación de intereses de mora

Al cierre de cada mes, el administrador liquida los intereses de mora sobre los saldos vencidos de las expensas y los registra como un cargo del concepto "Intereses de mora". La tasa, que no puede superar una y media veces el interés bancario corriente (Ley 675 de 2001, art. 30), se calcula fuera del sistema.

### P5. Expedición de paz y salvo

Cuando un propietario solicita un paz y salvo, el administrador consulta si el apartamento tiene cargos con saldo pendiente a la fecha. Si no los tiene, expide el certificado.

### P6. Reserva de zonas comunes

Un residente vigente solicita una zona común para una fecha y franja, con al menos un día de anticipación. El administrador verifica que la franja esté libre y registra la reserva. Si la zona tiene tarifa, genera en la cuenta del apartamento un cargo del concepto "Alquiler de zona común" y lo asocia a la reserva. Si el residente cancela, se elimina la reserva y su cargo, siempre que el cargo no tenga pagos.

### P7. Control de visitantes

Cuando llega un visitante, el vigilante solicita su documento y lo registra si es su primera visita. Luego llama por citófono al apartamento; si un residente autoriza el ingreso, el vigilante registra la visita con la fecha y hora de ingreso y, si llega en vehículo, la placa. Cuando el visitante sale, registra la hora de salida.

### P8. Reportes al consejo de administración

Cada mes, el administrador presenta al consejo el estado de la cartera, lo facturado frente a lo recaudado y el uso de las zonas comunes.

## 5. Reglas de negocio

La columna "Mecanismo" indica cómo se garantiza cada regla en la base de datos. Las restricciones `CHECK` de MySQL solo pueden evaluar columnas de la misma fila, por lo que las reglas que relacionan varias tablas se verifican con consultas de control o mediante el procedimiento de la administración.

### 5.1 Estructura del conjunto

| Código | Regla | Mecanismo |
|--------|-------|-----------|
| RN-01 | Cada torre tiene un nombre único y un número de pisos mayor que cero. | `UNIQUE`, `CHECK` |
| RN-02 | Cada apartamento pertenece a una sola torre, y su número no se repite dentro de la misma torre. | `FOREIGN KEY`, `NOT NULL`, `UNIQUE` compuesto |
| RN-03 | El número del apartamento se forma con el piso (1 a 8) y la posición (1 a 4). El piso no se almacena por separado, porque se deduce del número. | `CHECK` |
| RN-04 | El área privada es mayor que cero y el coeficiente de copropiedad está entre 0 y 100 %. El coeficiente lo fija el reglamento, por lo que se registra como dato propio del apartamento y no se calcula a partir del área. | `CHECK` |

### 5.2 Personas, residentes y vehículos

| Código | Regla | Mecanismo |
|--------|-------|-----------|
| RN-05 | Cada persona se registra una sola vez y se identifica por su tipo y número de documento, aunque sea propietaria y residente a la vez. | `UNIQUE` compuesto |
| RN-06 | Los tipos de documento admitidos son CC, CE, TI, PA y PPT. La regla aplica también a visitantes y empleados. | `CHECK` |
| RN-07 | El correo electrónico, cuando se registra, debe tener un formato válido. | `CHECK` |
| RN-08 | Cada apartamento tiene exactamente un propietario registrado; una persona puede ser propietaria de varios apartamentos. | `FOREIGN KEY`, `NOT NULL` |
| RN-09 | Un residente es una persona que habita un apartamento como propietario, arrendatario o familiar, y no puede registrarse dos veces en el mismo apartamento. | `FOREIGN KEY`, `CHECK`, `UNIQUE` compuesto |
| RN-10 | Si el tipo de residente es propietario, la persona debe ser la propietaria registrada del apartamento. | Consulta de control |
| RN-11 | La fecha de salida de un residente, si existe, es posterior a su fecha de ingreso. | `CHECK` |
| RN-12 | Cada vehículo tiene una placa única, es carro o moto y está autorizado para un solo apartamento. | `UNIQUE`, `CHECK`, `FOREIGN KEY` |
| RN-13 | La placa sigue el formato colombiano: tres letras y tres números para carros (ABC123), y tres letras, dos números y una letra para motos (ABC12D). | `CHECK` |

### 5.3 Cobros y pagos

| Código | Regla | Mecanismo |
|--------|-------|-----------|
| RN-14 | Cada concepto de cobro tiene un nombre único. | `UNIQUE` |
| RN-15 | Todo cargo corresponde a un apartamento y a un concepto, su valor es mayor que cero y su fecha de vencimiento es posterior a la de causación. | `FOREIGN KEY`, `NOT NULL`, `CHECK` |
| RN-16 | La cuota ordinaria vale el presupuesto mensual multiplicado por el coeficiente del apartamento. Se almacena el valor facturado, porque el presupuesto cambia cada año y el cargo debe conservar lo que efectivamente se cobró. | Procedimiento (P2) |
| RN-17 | Un apartamento no puede tener dos cargos del mismo concepto con la misma fecha de causación, lo que impide cobrar dos veces la cuota de un mes. | `UNIQUE` compuesto |
| RN-18 | Cada pago se aplica a un solo cargo, y un cargo puede recibir varios pagos parciales. | `FOREIGN KEY`, `NOT NULL` |
| RN-19 | El valor de un pago es mayor que cero, el medio de pago es consignación, transferencia o PSE, y la referencia bancaria es obligatoria. | `CHECK`, `NOT NULL` |
| RN-20 | La suma de los pagos de un cargo no puede superar el valor del cargo. | Consulta de control |
| RN-21 | Todo pago queda asociado al empleado que lo registró, que debe tener rol de administrador o auxiliar. | `FOREIGN KEY`, consulta de control |
| RN-22 | Los pagos se aplican primero a los intereses y luego a los cargos más antiguos (Código Civil, art. 1653). | Procedimiento (P3) |
| RN-23 | Los intereses de mora se liquidan cada mes sobre los saldos vencidos y se registran como un cargo del concepto "Intereses de mora". | Procedimiento (P4) |
| RN-24 | No se pueden eliminar apartamentos, personas, conceptos ni cargos que tengan registros asociados, para conservar el historial financiero. | `FOREIGN KEY` con `ON DELETE RESTRICT` |

### 5.4 Zonas comunes y reservas

| Código | Regla | Mecanismo |
|--------|-------|-----------|
| RN-25 | Cada zona común tiene un nombre único, una capacidad mayor que cero y una tarifa por franja mayor o igual a cero; la tarifa cero indica que la zona no tiene costo. | `UNIQUE`, `CHECK` |
| RN-26 | Las reservas se hacen por franja: mañana, tarde o noche. | `CHECK` |
| RN-27 | Una zona común no puede reservarse dos veces para la misma fecha y franja. | `UNIQUE` compuesto |
| RN-28 | Toda reserva la solicita un residente, con al menos un día de anticipación. | `FOREIGN KEY`, `NOT NULL`, `CHECK` |
| RN-29 | Si la zona tiene tarifa, la reserva queda asociada al cargo de alquiler generado en la cuenta del apartamento, y un cargo corresponde como máximo a una reserva. | Procedimiento (P6), `FOREIGN KEY`, `UNIQUE` |
| RN-30 | Cancelar una reserva elimina su registro y su cargo, siempre que el cargo no tenga pagos. | Procedimiento (P6) |

### 5.5 Portería

| Código | Regla | Mecanismo |
|--------|-------|-----------|
| RN-31 | Cada visitante se identifica por su tipo y número de documento, y solo se registran los datos necesarios para identificarlo, en aplicación del principio de finalidad de la Ley 1581 de 2012 (art. 4). | `UNIQUE` compuesto |
| RN-32 | Toda visita la autoriza un residente vigente del apartamento visitado. El apartamento se obtiene a través del residente, por lo que no se almacena en la visita. | `FOREIGN KEY`, `NOT NULL`, procedimiento (P7) |
| RN-33 | Toda visita la registra un empleado con rol de vigilante. | `FOREIGN KEY`, consulta de control |
| RN-34 | La hora de salida, si existe, es posterior a la de ingreso. Una visita sin hora de salida indica que el visitante sigue dentro del conjunto. | `CHECK` |
| RN-35 | La placa del vehículo del visitante es opcional y se registra en la visita, porque un mismo visitante puede llegar en vehículos distintos. | Columna que admite `NULL` |

### 5.6 Empleados

| Código | Regla | Mecanismo |
|--------|-------|-----------|
| RN-36 | Cada empleado se identifica por su tipo y número de documento, y su rol es administrador, auxiliar o vigilante. | `UNIQUE` compuesto, `CHECK` |

## 6. Requerimientos de consulta

Numeración preliminar; la definitiva se fija en `sql/03_consultas.sql`.

| Código | Pregunta | Usuario | Cláusulas previstas |
|--------|----------|---------|---------------------|
| C1 | ¿Qué apartamentos están en mora a una fecha de corte, cuánto deben y desde qué fecha? | Administrador, consejo | `JOIN`, `LEFT JOIN`, `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY` |
| C2 | ¿Cuánto se facturó y cuánto se recaudó por mes y por torre? | Consejo | `JOIN`, `GROUP BY`, `ORDER BY` |
| C3 | ¿Qué apartamentos están a paz y salvo a una fecha de corte? | Administrador | `JOIN`, subconsulta, `WHERE` |
| C4 | ¿Cuántas reservas tuvo cada zona común en un periodo y cuánto ingreso generó? | Consejo | `LEFT JOIN`, `WHERE`, `GROUP BY`, `ORDER BY` |
| C5 | ¿Qué visitantes ingresaron a un apartamento en un rango de fechas, quién autorizó cada ingreso y qué vigilante lo registró? | Administrador, vigilante | `JOIN`, `WHERE ... BETWEEN`, `ORDER BY` |
| C6 | ¿Cuántos residentes vigentes y vehículos registrados tiene cada torre? | Administrador | `LEFT JOIN`, `WHERE`, `GROUP BY` |
| C7 | ¿Qué propietarios tienen más de un apartamento en el conjunto? | Administrador | `JOIN`, `GROUP BY`, `HAVING`, `ORDER BY` |
| C8 | ¿Qué visitantes siguen dentro del conjunto? | Vigilante | `JOIN`, `WHERE ... IS NULL`, `ORDER BY` |

Además, las reglas RN-10, RN-20, RN-21 y RN-33, que no pueden garantizarse con restricciones, se verificarán con consultas de control que deben devolver cero filas. Su resultado sirve como evidencia en la sección de resultados del informe.

## 7. Requerimientos no funcionales

- **SGBD:** MySQL 8.0.16 o superior, con motor InnoDB para la integridad referencial y juego de caracteres `utf8mb4` para almacenar tildes y la letra ñ.
- **Ejecución:** los scripts se ejecutan en orden (`01_schema.sql`, `02_datos_prueba.sql`, `03_consultas.sql`) sobre un servidor sin la base de datos creada y sin errores.
- **Nombres:** tablas y columnas en español, en singular y en `snake_case`; llaves primarias con la forma `id_<tabla>`.
- **Protección de datos:** solo se almacenan los datos personales necesarios para cada proceso, y no se registran datos sensibles como biometría o información de salud.
- **Valores monetarios:** se expresan en pesos colombianos.

## 8. Supuestos y exclusiones

- Cada apartamento tiene un único propietario registrado. Si hay copropietarios, se registra a quien figure como principal.
- No se conserva el historial de propietarios: cuando se vende un apartamento, se actualiza su propietario.
- Un pago se aplica a un solo cargo; cuando una consignación cubre varios cargos, se registra un pago por cada uno con la misma referencia.
- La tasa de interés de mora se calcula fuera del sistema; en la base de datos solo se registra el cargo resultante.
- Cancelar una reserva equivale a eliminarla, lo que libera la franja.
- Los datos de prueba incluyen una muestra de los apartamentos del conjunto, no los 96.
