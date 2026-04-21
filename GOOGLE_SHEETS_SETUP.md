# Configuración de Google Sheets para captura de leads

## Paso 1 — Crear el Google Sheet

1. Ir a https://sheets.google.com → crear una hoja nueva
2. Nombrarla "Leads Calculadora Hidráulica"
3. En la fila 1, poner estos encabezados exactos:
   
   | A          | B       | C        | D         | E     | F          | G        | H   |
   |------------|---------|----------|-----------|-------|------------|----------|-----|
   | Timestamp  | Nombre  | Empresa  | Teléfono  | Email | Ubicación  | Proyecto | App |

4. Copiar el ID de la hoja desde la URL:
   `https://docs.google.com/spreadsheets/d/ESTE_ES_EL_ID/edit`


## Paso 2 — Crear el Apps Script

1. En el Google Sheet: menú **Extensiones → Apps Script**
2. Borrar todo el código que aparece y pegar esto:

```javascript
function doPost(e) {
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Hoja 1');
    // Si tu hoja se llama diferente, cambiar 'Hoja 1' por el nombre correcto
    
    var data = JSON.parse(e.postData.contents);
    
    sheet.appendRow([
      data.timestamp || new Date().toISOString(),
      data.nombre || '',
      data.empresa || '',
      data.telefono || '',
      data.email || '',
      data.ubicacion || '',
      data.proyecto || '',
      data.app || ''
    ]);
    
    // Opcional: enviar notificación por email a Agraria
    try {
      MailApp.sendEmail({
        to: 'jroca@agraria.com.uy',
        subject: 'Nuevo lead: ' + (data.nombre || 'Sin nombre') + ' — ' + (data.app || ''),
        body: 'Nuevo lead desde la calculadora:\n\n' +
              'Nombre: ' + (data.nombre || '-') + '\n' +
              'Empresa: ' + (data.empresa || '-') + '\n' +
              'Teléfono: ' + (data.telefono || '-') + '\n' +
              'Email: ' + (data.email || '-') + '\n' +
              'Ubicación: ' + (data.ubicacion || '-') + '\n' +
              'Proyecto: ' + (data.proyecto || '-') + '\n' +
              'App: ' + (data.app || '-') + '\n' +
              'Fecha: ' + (data.timestamp || '-')
      });
    } catch(mailErr) {
      // Si falla el mail, no bloquear — el dato ya se guardó
    }
    
    return ContentService
      .createTextOutput(JSON.stringify({status: 'ok'}))
      .setMimeType(ContentService.MimeType.JSON);
      
  } catch(err) {
    return ContentService
      .createTextOutput(JSON.stringify({status: 'error', message: err.toString()}))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

// Necesario para que funcione con mode:'no-cors' desde el navegador
function doGet(e) {
  return ContentService
    .createTextOutput(JSON.stringify({status: 'ok', message: 'API activa'}))
    .setMimeType(ContentService.MimeType.JSON);
}
```


## Paso 3 — Publicar como web app

1. En Apps Script: menú **Implementar → Nueva implementación**
2. Tipo: **Aplicación web**
3. Configurar:
   - Descripción: "Leads calculadora"
   - Ejecutar como: **Yo** (tu cuenta)
   - Quién tiene acceso: **Cualquier persona**
4. Click en **Implementar**
5. **Autorizar** con tu cuenta de Google cuando te lo pida
6. Copiar la URL que te da (es algo como):
   `https://script.google.com/macros/s/AKfycbx.../exec`


## Paso 4 — Pegar la URL en la app

En el archivo `index.html` de la calculadora, buscar esta línea:

```javascript
const GSHEET_URL = 'PEGAR_URL_DE_GOOGLE_APPS_SCRIPT_AQUI';
```

Y reemplazar por:

```javascript
const GSHEET_URL = 'https://script.google.com/macros/s/TU_ID_REAL/exec';
```


## Paso 5 — Probar

1. Subir el `index.html` actualizado a Netlify
2. Hacer click en "Imprimir PDF" o "Enviar por mail"
3. Completar los datos del formulario
4. Verificar que aparecen en el Google Sheet


## Notas

- Los datos se envían con `mode: 'no-cors'` para evitar errores de CORS en el navegador
- Si la conexión falla, el usuario NO se bloquea — puede seguir usando la app
- Los datos de contacto se guardan en localStorage para no pedir repetidamente
- Cada vez que el usuario imprime/envía con un proyecto diferente, se registra una nueva fila
- La notificación por email es opcional — si no la querés, borrá el bloque `MailApp.sendEmail`
- Podés usar la misma URL para la app de riego si querés capturar leads también ahí
