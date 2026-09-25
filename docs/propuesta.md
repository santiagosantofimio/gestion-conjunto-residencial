# Propuesta de proyecto

**Asignatura:** Bases de Datos I — Ingeniería de Sistemas, Universidad El Bosque
**Integrantes:** Santiago Santofimio, Jairo Esteban, Santiago Silva, Andres Hernandez, Jorge Pinilla
**SGBD:** MySQL 8

## 1. Título

Base de datos para la gestión de cartera, reservas de zonas comunes y control de visitantes en un conjunto residencial sometido al régimen de propiedad horizontal.

## 2. Contexto

En Colombia, los conjuntos residenciales se rigen por la Ley 675 de 2001, que regula la propiedad horizontal. De acuerdo con esta ley, cada propietario contribuye a las expensas comunes, como la cuota de administración, en proporción al coeficiente de copropiedad de su unidad (art. 25); el retraso en el pago causa intereses de mora (art. 30), y el administrador debe llevar la contabilidad de la copropiedad y el registro de propietarios y residentes (art. 51).

Además del cobro de las cuotas, la administración de un conjunto gestiona el uso de las zonas comunes, como el salón comunal o la zona de BBQ, y, junto con el personal de vigilancia, controla el ingreso de visitantes. Este último registro contiene datos personales, por lo que su tratamiento está sujeto a la Ley 1581 de 2012 de protección de datos personales.

## 3. Planteamiento del problema

El proyecto toma como caso simulado el Conjunto Residencial Los Cerezos, ubicado en Bogotá, con tres torres de ocho pisos, 96 apartamentos, un salón comunal, una zona de BBQ y una cancha múltiple. En la situación actual, la información se lleva en hojas de cálculo, cuadernos y mensajes de WhatsApp, lo que genera las siguientes dificultades:

- La cartera se controla en una hoja de cálculo por mes. En cada fila se repiten la torre, el apartamento, el nombre y el teléfono del propietario y el coeficiente, junto con los cobros y pagos del periodo, de modo que los datos del propietario quedan duplicados y se desactualizan cuando cambian.
- Los pagos se registran a mano a partir de los soportes de consignación que envían los residentes. No es posible saber con rapidez cuánto debe cada apartamento, desde qué periodo, ni cuánto se ha recaudado frente a lo facturado.
- Expedir un paz y salvo, que el propietario solicita, por ejemplo, para vender su inmueble, exige revisar varias hojas de cálculo.
- Las reservas de zonas comunes se solicitan por WhatsApp y se anotan en un cuaderno, lo que ha ocasionado reservas cruzadas para una misma fecha y no deja registro del uso ni del cobro de la tarifa.
- Los visitantes se registran a mano en un cuaderno de portería. Ante un incidente de seguridad no es posible consultar quién visitó un apartamento en una fecha determinada, y los datos personales de los visitantes quedan a la vista de cualquier persona que llega a la portería.
- El registro de residentes y vehículos está desactualizado, por lo que el vigilante no puede verificar si una persona o un vehículo pertenece al conjunto.

Estas deficiencias afectan el flujo de caja de la copropiedad, dificultan la rendición de cuentas del administrador ante la asamblea y el consejo de administración, y exponen a los residentes a riesgos de seguridad y de uso indebido de sus datos personales.

## 4. Objetivos

### 4.1 Objetivo general

Diseñar e implementar en MySQL una base de datos relacional, normalizada hasta la tercera forma normal, que centralice la información de apartamentos, propietarios, residentes, cobros, pagos, reservas de zonas comunes y visitas de un conjunto residencial, con el fin de facilitar el control de la cartera y la seguridad de la copropiedad.

### 4.2 Objetivos específicos

1. Identificar los requerimientos de información y las reglas de negocio de la administración de un conjunto residencial.
2. Construir el modelo entidad-relación con sus entidades, atributos, relaciones y cardinalidades.
3. Normalizar la planilla de cartera original hasta la tercera forma normal, evidenciando cada transformación.
4. Implementar el esquema en MySQL con llaves primarias, llaves foráneas y restricciones que hagan cumplir las reglas de negocio.
5. Cargar datos de prueba y desarrollar consultas SQL que respondan preguntas operativas de la administración y del consejo de administración.

## 5. Alcance

**Incluye:** torres y apartamentos con su coeficiente de copropiedad, propietarios, residentes y sus vehículos, conceptos de cobro, cargos por periodo y pagos, zonas comunes y sus reservas, visitantes y visitas, y empleados de administración y vigilancia.

**No incluye:** contabilidad general, presupuesto ni estados financieros, nómina, facturación electrónica, asambleas y votaciones, peticiones, quejas y reclamos (PQR), asignación de parqueaderos, pagos en línea ni interfaz gráfica.

## 6. Usuarios del sistema

| Usuario | Uso principal |
|---------|---------------|
| Administrador | Registra propietarios y residentes, genera los cobros de cada periodo, registra pagos y reservas |
| Vigilante de portería | Registra el ingreso y la salida de visitantes y consulta residentes y vehículos autorizados |
| Consejo de administración | Consulta reportes de cartera, recaudo y uso de zonas comunes |

## 7. Entidades candidatas

| Entidad | Descripción |
|---------|-------------|
| torre | Edificio del conjunto |
| apartamento | Unidad privada de una torre, con su área y coeficiente de copropiedad |
| propietario | Persona dueña de uno o más apartamentos |
| residente | Persona que habita un apartamento, sea propietario, arrendatario o familiar |
| vehiculo | Vehículo autorizado para ingresar, asociado a un apartamento |
| concepto_cobro | Tipo de cobro: administración, cuota extraordinaria, multa, intereses de mora o uso de zona común |
| cargo | Cobro de un concepto a un apartamento en un periodo, con su valor y fecha de vencimiento |
| pago | Dinero recibido de un apartamento y aplicado a sus cargos pendientes |
| zona_comun | Espacio común reservable, con su capacidad y tarifa de uso |
| reserva | Uso de una zona común solicitado por un residente para una fecha y franja horaria |
| visitante | Persona externa que ingresa al conjunto |
| visita | Ingreso de un visitante a un apartamento, con hora de entrada y de salida |
| empleado | Personal de administración y vigilancia que registra pagos y visitas |

## 8. Preguntas que la base de datos podrá responder

- ¿Qué apartamentos están en mora, cuánto deben y desde qué periodo?
- ¿Cuánto se facturó y cuánto se recaudó por mes y por torre?
- ¿Qué apartamentos están a paz y salvo a una fecha determinada?
- ¿Qué zonas comunes se reservan con mayor frecuencia y cuánto ingreso generan?
- ¿Qué visitantes ingresaron a un apartamento en un rango de fechas y qué vigilante registró cada ingreso?
- ¿Cuántos residentes y vehículos registrados tiene cada torre?

## 9. Justificación del SGBD

Se elige MySQL por ser un sistema gratuito y de amplio uso, con soporte de integridad referencial mediante el motor InnoDB, validación de restricciones `CHECK` desde la versión 8.0.16 y herramientas como MySQL Workbench para diagramar el modelo y ejecutar consultas.
