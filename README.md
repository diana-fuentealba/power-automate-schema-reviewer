# Validación y Procesamiento Automatizado de Archivos CSV

Este repositorio contiene la definición de un flujo automatizado en **Microsoft Power Automate** diseñado para monitorear, validar la estructura y procesar archivos CSV cargados en OneDrive para la Empresa.

## 📋 Descripción del Flujo

El flujo se activa automáticamente cada vez que se detecta un nuevo archivo en una carpeta específica de OneDrive. Su objetivo principal es extraer la primera línea del archivo (los encabezados) y validar si cumple exactamente con el formato requerido. Dependiendo del resultado de la validación, el flujo toma caminos distintos (procesamiento exitoso o notificación de error).

---

## 🛠️ Detalles Técnicos

* **Nombre del Flujo:** `Cuando se crea un archivo -> Copiar archivo,Enviar una notificación...`
* **ID del Flujo:** `82dd0765-cc2e-4fdf-9755-cba450c05c8b`
* **Frecuencia de Evaluación:** Cada 1 minuto (Trigger basado en Polling).
* **Conectores Utilizados:**
    * `shared_onedriveforbusiness` (OneDrive for Business)
    * `shared_office365` (Office 365 Outlook)

---

## 🔄 Arquitectura del Flujo (Paso a Paso)

### 1. Desencadenador (Trigger)
* **Acción:** `Cuando se crea un archivo` (OneDrive para la empresa).
* **Carpeta Monitoreada:** `/Datos_Input`

### 2. Acciones Iniciales
* **Obtener contenido de archivo:** Recupera los bytes del archivo recién creado utilizando su identificador único (`x-ms-file-id`).
* **Redactar (Compose):** Extrae la primera línea del archivo mediante una expresión para aislar los encabezados y validar su estructura limpia.
    ```ldif
    @first(split(body('Obtener_contenido_de_archivo'), decodeUriComponent('%0D%0A')))
    ```

### 3. Condición de Validación
El flujo evalúa si el resultado de la acción **Redactar** coincide exactamente con la siguiente cadena de texto (encabezados requeridos):
`SKU,Producto,Precio Unitario,Stock`

#### 🟢 Ramificación: SI (Estructura Correcta)
Si los encabezados coinciden perfectamente con el estándar:
1.  **Copiar archivo:** Mueve/copia el archivo original desde la carpeta de entrada hacia la ruta de destino: `/Datos_Procesados/test.csv` (sobreescribiendo si ya existe).
2.  **Enviar correo electrónico (V2):** Notifica a `diana.fuentealba.s@gmail.com` indicando que el archivo es correcto e incluye en el cuerpo del correo los datos redactados.

#### 🔴 Ramificación: NO (Estructura Incorrecta)
Si el archivo no cuenta con los encabezados exactos:
1.  **Enviar correo electrónico (V2):** Envía una alerta de error a `diana.fuentealba.s@gmail.com` solicitando la revisión de los encabezados, adjuntando la línea errónea detectada y detallando el formato esperado.

---

## 📝 Requisitos del Archivo CSV

Para que el flujo procese el archivo de forma exitosa, la primera línea de este **debe** ser textualmente:

```csv
SKU,Producto,Precio Unitario,Stock
