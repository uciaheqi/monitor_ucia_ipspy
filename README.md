# Panel UCI · Google Sheets · GitHub Pages

Este proyecto ilustra cómo publicar un panel analítico ligero para una Unidad de Cuidados Intensivos
utilizando **GitHub Pages** como frontend estático y **Google Sheets** como backend de datos, sin
Google Apps Script.

## Arquitectura

- Frontend estático (HTML, CSS, JavaScript puro) alojado en GitHub Pages.
- Autenticación y acceso a Google Sheets mediante:
  - **Google Identity Services** (OAuth 2.0 del lado del cliente).
  - **Google Sheets API v4** (biblioteca `gapi`).

Toda la lógica está en el navegador, por lo que **no se debe exponer ninguna credencial de cuenta de
servicio** en el código. En su lugar, se utilizan:

- Un **Client ID OAuth 2.0 Web**.
- Una **API Key** del mismo proyecto de Google Cloud.

## Requisitos previos

1. Proyecto en Google Cloud Console.
2. Habilitar la API:
   - `Google Sheets API`.
3. Configurar credenciales:
   - Crear un **OAuth 2.0 Client ID** de tipo `Web application`.
   - Crear una **API Key**.
4. Configurar la pantalla de consentimiento OAuth.

## Configuración de OAuth para GitHub Pages

Asumiendo que desplegará en una URL del tipo:

- `https://USUARIO.github.io/REPO/`

en la consola de Google Cloud:

1. En **Credenciales → ID de cliente OAuth 2.0**:
   - Añada en *Authorized JavaScript origins* la URL:
     - `https://USUARIO.github.io`
2. No es necesario configurar redirect URIs para el flujo de token implícito usado por
   Google Identity Services.

## Configuración del proyecto

1. Edite el archivo `assets/js/app.js` y reemplace:

   ```js
   const CLIENT_ID = "REEMPLAZAR_POR_CLIENT_ID_WEB.apps.googleusercontent.com";
   const API_KEY = "REEMPLAZAR_POR_API_KEY";
   ```

   por los valores reales de su proyecto.

2. Verifique que las constantes:

   ```js
   const SHEET_ID = "1r_OmMJirLBC33Gjtl-mzlAxYKodnK-ld1ARlei7ut7k";
   const SHEET_NAME = "BASE_UCI";
   ```

   coinciden con el libro y la hoja que desea utilizar.

3. Asegúrese de que la hoja `BASE_UCI` tenga encabezados en la primera fila. El código intenta
   detectar automáticamente columnas típicas como:

   - `fecha_de_ingreso`
   - `fecha_de_egreso`
   - `edad`
   - `sexo`
   - `condicion_al_egreso`
   - `diagnostico_principal` o `diagnostico`

4. Ajuste nombres de columnas o la lógica en `buscarColumna` si fuera necesario.

## Despliegue en GitHub Pages

1. Inicialice un repositorio en la carpeta del proyecto.
2. Suba todos los archivos a GitHub.
3. En la configuración del repositorio, habilite **GitHub Pages** apuntando a la rama principal.
4. Acceda a la URL publicada, pulse en **Conectar con Google** y otorgue permisos a su cuenta
   para leer y escribir en el Google Sheet.

## Notas de seguridad

- **No suba archivos JSON de cuentas de servicio al repositorio ni los incruste en el frontend.**
- Si ya ha expuesto una clave privada de cuenta de servicio, genere y descargue una nueva clave
  desde la consola de Google Cloud y elimine o deshabilite la anterior.
- Limite el ámbito de acceso a los estrictamente necesarios. En este ejemplo se usa
  `https://www.googleapis.com/auth/spreadsheets` porque se requiere lectura y escritura.

## Flujo de trabajo en la interfaz

- Tras conectar con Google, el panel:
  - Lee los datos de la hoja `BASE_UCI`.
  - Calcula KPIs básicos (total de pacientes, edad media, proporción de mujeres y tasa de mortalidad).
  - Genera:
    - Un gráfico de **ingresos por mes**.
    - Un histograma de **grupos de edad**.
  - Muestra la base en una tabla filtrable y con búsqueda simple.
- El formulario inferior permite añadir un nuevo paciente, que se inserta como nueva fila al final
  de la hoja respetando el orden de las columnas.

A partir de esta estructura puede extender el panel, añadir nuevas métricas, gráficos específicos
o exportaciones adicionales según las necesidades de su servicio de UCI.
