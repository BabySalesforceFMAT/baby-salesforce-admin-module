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
