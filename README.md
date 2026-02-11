# 🚀 Azure Serverless Lab: Intelligent Payment Processing System
### NTT DATA Bootcamp - Cloud-Native Specialization (Quarkus + Azure)

Este laboratorio práctico consiste en la construcción de un pipeline de procesamiento de información financiera altamente escalable, utilizando una arquitectura orientada a eventos (**Event-Driven Architecture**).

## 📋 Escenario del Negocio
El banco (**NTT_BANK**) requiere procesar archivos masivos de clientes para extraer su historial de pagos y enviar notificaciones automáticas. El reto es construir un sistema **Serverless** que no dependa de servidores permanentes y que sea capaz de integrarse con servicios externos de forma eficiente y segura.

## 🏗️ Arquitectura de la Solución

El sistema se compone de tres micro-funciones desarrolladas en **Quarkus**, desacopladas mediante servicios de mensajería de Azure para garantizar resiliencia y escalabilidad:

1.  **Ingestor (Blob Trigger):** Detecta la carga del archivo `.txt` en Azure Blob Storage, lo procesa línea por línea y envía los DNIs a un **Event Hub**.
2.  **Collector (Event Hub Trigger):** Escucha los eventos, consume un microservicio conectado a **Airtable** para recuperar el historial de pagos y datos del cliente.
3.  **Notifier (Service Bus Trigger):** Recibe el paquete de datos procesado y ejecuta la lógica de notificación final (Email/Log).



---

## 🛠️ Stack Tecnológico
* **Lenguaje:** Java 17+ (LTS)
* **Framework:** [Quarkus](https://quarkus.io/) (Extensiones: `azure-functions`, `rest-client-jackson`, `smallrye-fault-tolerance`)
* **Infraestructura (Azure Free Tier / $200 Credit):**
    * **Azure Blob Storage:** Almacenamiento de archivos de entrada.
    * **Azure Event Hubs:** Ingesta de eventos masivos.
    * **Azure Service Bus:** Cola de mensajes para notificaciones confiables.
    * **Azure Functions:** Cómputo serverless bajo demanda.
* **Base de Datos Externa:** Airtable (vía API REST).

---

## 📂 Materiales del Laboratorio
Los archivos necesarios se encuentran en la raíz de este repositorio:
* `/data/clientes.txt`: Listado de DNIs para pruebas.
* `/docs/airtable_schema.json`: Estructura técnica de la tabla en Airtable.
* `MOCK_API_URL`: URL del microservicio de consulta (proporcionada por el instructor).

---

## 🚀 Guía de Implementación

### Fase 1: Ingesta Eficiente (Blob a Event Hub)
El alumno debe configurar el `BlobTrigger`. 
* **Reto:** No cargar todo el archivo en memoria. Debe leerse como un Stream para soportar archivos de gran tamaño.
* **Configuración:** Crear un Namespace de Event Hubs (Nivel Basic es suficiente).

### Fase 2: Enriquecimiento de Datos (Event Hub a Service Bus)
Esta es la función principal. Debe usar el `RestClient` de Quarkus para conectar con el microservicio de Airtable.
* **Valor Agregado:** Implementar `@Retry` o `@CircuitBreaker` para manejar fallos de la API externa.
* **Salida:** Enviar un JSON estructurado a una cola de **Azure Service Bus**.

### Fase 3: Notificación y Monitoreo
La última función debe procesar la cola de Service Bus.
* **Creatividad:** Diseñar un formato de salida (Log o Email) que presente el historial de pagos de forma amigable para el usuario final.
* **Observabilidad:** Configurar **Application Insights** para visualizar el flujo completo de la transacción.

---

## 🏁 Criterios de Evaluación
1.  **Ejecución Exitosa:** El flujo debe completarse desde la subida del `.txt` hasta la notificación final.
2.  **Manejo de Errores:** El sistema no debe romperse si el microservicio de Airtable devuelve un error 404 o 500.
3.  **Performance:** El uso de memoria de las funciones debe ser optimizado (Quarkus Best Practices).
4.  **Desafío Pro:** (Opcional) Compilar la función de consulta en **Modo Nativo** para minimizar el Cold Start.

---

## 💻 Comandos de Referencia

**Crear el scaffold del proyecto:**
```bash
mvn io.quarkus.platform:quarkus-maven-plugin:create \
    -DprojectGroupId=com.nttdata.bcp.bootcamp \
    -DprojectArtifactId=payment-processor-lab \
    -Dextensions="azure-functions,rest-client-jackson,smallrye-fault-tolerance"
