# Arquitectura EFCEA alineada con EDA (Salesforce)

Para la gestión de los instructores (Capacitadores), se ha decidido utilizar el objeto estándar
**Contact**, agrupados bajo una **Business Account** contenedora (“Catálogo de Instructores”),
extendiendo su funcionalidad operativa mediante un **Custom Object** (`Capacitador__c`).

Se descartó el uso de **Person Accounts** para este rol, reservando dicha funcionalidad
exclusivamente para los **Alumnos (B2C)**.

---

## A. Principio de Separación de Responsabilidades (SoC)

En el modelo de negocio de Educación Continua, existen dos actores fundamentales con
ciclos de vida opuestos:

- **Alumno (Ingreso / Revenue)**  
  Es un cliente B2C. Su ciclo es **comercial**  
  `Lead → Opportunity → Enrollment`.

- **Instructor (Costo / Expense)**  
  Es un proveedor de servicios o colaborador. Su ciclo es **operativo**  
  `Contract → Assignment → Payment`.

Utilizar **Contactos** para los instructores permite mantener una **higiene de datos estricta**:

- Evitamos que los instructores entren en segmentaciones de Marketing Automation,
  Journey Builder o reportes de proyecciones de ventas destinados a alumnos.
- Prevenimos la contaminación de la base de datos de clientes.

---

## B. Patrón de diseño “Role Object” (Extensión de Objeto)

La arquitectura implementada separa la **Identidad** del **Rol**:

- **Identidad (`Contact`)**  
  Almacena datos demográficos persistentes y públicos  
  (Nombre, Email, Teléfono).

- **Rol administrativo (`Capacitador__c`)**  
  Almacena datos contractuales, sensibles y temporales  
  (Datos bancarios, RFC, Costo hora, Estatus laboral).

Este desacoplamiento permite una **gestión granular de la seguridad (Field-Level Security)**:

- Los equipos de Ventas pueden ver el **Contact** para saber quién da el curso.
- Solo **RRHH / Finanzas** tienen acceso al objeto `Capacitador__c` para ver información
  sensible de pagos.

---

## C. Estrategia “Bucket Account”

Dado que Salesforce requiere que todo **Contacto** esté vinculado a una **Cuenta**, y que
los instructores suelen ser freelancers sin empresa propia, se implementó una **Business
Account contenedora**:

- **Cuenta “Catálogo de Instructores”** (Bucket Account).

Esta cuenta actúa como un **“directorio lógico”**, que:

- Facilita la gestión masiva de instructores.
- Permite heredar permisos de forma centralizada.
- Evita crear miles de cuentas “dummy” individuales que fragmentarían el almacenamiento.

---

## 3. Alineación con Estándares de la Industria (EDA)

Esta implementación no es arbitraria; sigue los principios de arquitectura de la
**Education Data Architecture (EDA)**, el estándar oficial de Salesforce para el sector
educativo, pero adaptada a una **implementación ligera (Lightweight Implementation)**.

Estamos recreando la robustez de EDA **sin** la sobrecarga técnica de instalar el paquete
gestionado completo.

### 3.1 Tabla de alineación con EDA

| Concepto de negocio     | Implementación custom (nuestra arquitectura) | Estándar EDA (Salesforce)        | Validación    |
|-------------------------|----------------------------------------------|----------------------------------|--------------|
| Catálogo de cursos      | `Product2`                                   | `Course`                         | Alineado     |
| Instancia del curso     | `EventoEC__c` (Custom)                       | `Course Offering`                | Alineado     |
| Alumno                  | `Person Account`                             | `Contact` (Student Record Type)  | Compatible   |
| Capacitador             | `Contact` (en Bucket Account)                | `Contact` (Faculty Record Type)  | Idéntico     |
| Relación laboral        | `Capacitador__c` + Lookup                    | `Affiliation`                    | Equivalente  |
| Inscripción             | `Inscripcion__c`                             | `Course Connection` (Student)    | Alineado     |
| Asignación docente      | `Asignacion_Instructor__c`                   | `Course Connection` (Faculty)    | Alineado     |

---

## 4. Conclusión

El uso combinado de **Contactos + Custom Object** bajo una **Business Account** proporciona
una arquitectura **escalable, segura y limpia** para el manejo de proveedores de servicios
educativos.

Este enfoque permite manejar escenarios futuros complejos, como la **polimorfia del usuario**
(p. ej. un instructor que decide inscribirse también como alumno), sin duplicidad de datos ni
conflictos de permisos, y se alinea con las mejores prácticas globales de arquitectura de
datos en Salesforce.

Para más detalle sobre el modelo de datos estándar EDA, puede consultarse el ERD oficial:

- [EDA ERD (Salesforce.org)](https://sfdo-docs.s3.us-west-2.amazonaws.com/EDA_ERD.pdf)



# Validación de documentos

La funcionalidad principal de este módulo es el flujo de **Validar documentos**, que permite a la
Coordinación revisar todos los documentos asociados a una oportunidad
de inscripción y marcarlos como **Válido** o **No Válido**.

## Punto de entrada

- Aplicación: **Coordinación**
- Navegación: pestaña **“Validar documentos”**


## Flujo funcional

1. **Selección de oportunidad**

   - La Coordinación abre la pestaña **Validar documentos** en la app de Coordinación.
   - El flujo muestra una pantalla con una lista de **Oportunidades con documentos pendientes** de validación.
   - El usuario selecciona una oportunidad y hace clic en **Next**.

2. **Validación en lote de documentos**

   - El sistema carga todos los registros de `Documento__c` relacionados con la oportunidad seleccionada.
   - Se muestra una tabla con una fila por documento, con las columnas:
     - **Documento** – nombre o tipo del documento (ej. Comprobante de pago, Certificado de titulación).
     - **Decisión** – picklist con las opciones `Válido`, `No Válido` o `—`.
     - **Comentario** – texto opcional donde la Coordinación explica el motivo si marca `No Válido`.
     - **Ver** – botón/ícono de “ojo” que abre el archivo en Salesforce Files en una nueva pestaña.
   - Encima de la tabla se muestran las instrucciones de uso y la **guía visual**:
     - Filas en verde: documentos marcados como **Válidos**.
     - Filas en rojo: documentos marcados como **No Válidos**.
     - Fondo azulado: filas modificadas en la sesión actual (aún no guardadas).

3. **Revisión del archivo**

   - Para cada documento, la Coordinación hace clic en **Ver** para abrir el archivo en otra pestaña.
   - Revisa el contenido y regresa a la pantalla del flujo.

4. **Registro de la decisión**

   - En la columna **Decisión**, la Coordinación selecciona:
     - `Válido`, si el documento cumple con los requisitos.
     - `No Válido`, si el documento está incompleto, incorrecto o ilegible.
   - Si elige `No Válido`, se recomienda escribir un motivo en la columna **Comentario**  
     (por ejemplo: “INE borrosa”, “Recibo de pago vencido”, etc.).

5. **Guardado de cambios**

   - Al hacer clic en **Next**, el flujo actualiza todos los registros de `Documento__c`
     mostrados en la tabla:
     - Campo `Estado_de_validacion__c` → `Válido` o `No Válido`.
     - Campo `Comentario_de_revision__c` → texto capturado en la columna Comentario.


6. **Fin del flujo**

   - Tras guardar los cambios, el flujo muestra un mensaje de confirmación y permite regresar
     a la selección de otra oportunidad con documentos pendientes.

## Objetos y campos involucrados

- **Oportunidad (`Opportunity`)**
  - Se utiliza como contexto para agrupar los documentos que deben ser validados.

- **Documento (`Documento__c`)**
  - Lookup a `Opportunity` (`Opportunity__c`).
  - `Estado_de_validacion__c` (Picklist: Pendiente, Válido, No Válido).
  - `Comentario_de_revision__c` (Long Text).
  - Campos informativos: tipo de documento, fecha de carga, referencia a `ContentDocument`, etc.

