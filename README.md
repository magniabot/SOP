# SOP
# SOP: Enhanced Conversions para GoHighLevel Forms & Surveys

## 📋 **Resumen Ejecutivo**

Este procedimiento permite implementar tracking de enhanced conversions para formularios y encuestas de GoHighLevel, capturando datos de usuarios (email, teléfono) para enviar de vuelta a Google Ads, Facebook Ads y GA4.

## 📊 **Resumen de Configuración Final**

### **1 Event Listener** (Custom HTML)
### **6 Variables** (Data Layer Variables + 1 UPD)
### **2 Triggers** (hl_form_submit, hl_survey_submit)  
### **4 Tags principales**:
- Google Ads UPD Event
- Google Ads Conversion (con tag sequencing)
- GA4 Lead Event  
- Meta Lead Event

**Total**: 13 elementos en GTM

## 🎯 **Alcance**
- **Aplica para**: Formularios y encuestas de GoHighLevel embebidos como iframes
- **Plataformas soportadas**: Google Ads, Facebook Ads, Google Analytics 4
- **Resultado**: Tracking automático de conversiones con datos de usuario

---

## 📋 **Requisitos Previos**

### **Accesos necesarios:**
- [ ] Acceso admin a Google Tag Manager
- [ ] Acceso a Google Ads (Conversion ID y Label)
- [ ] Acceso a Facebook Business Manager (Pixel ID)
- [ ] Acceso a Google Analytics 4 (Measurement ID)

### **Información requerida del cliente:**
- [ ] URL del sitio donde están embebidos los forms/surveys
- [ ] Dominio específico del cliente
- [ ] Tipo de conversiones a trackear (formularios, encuestas, o ambos)

### **Verificaciones:**
- [ ] Confirmar que forms/surveys están embebidos como iframes
- [ ] Deshabilitar snippets existentes de Google/Facebook que usen los mismos IDs
- [ ] Asegurar que GTM sea el único propietario de los tracking IDs

---

## 🛠️ **Implementación Paso a Paso**

### **PASO 1: Configurar Event Listener en GTM**

#### 1.1 Crear Custom HTML Tag
1. En GTM, ir a **Tags** → **New**
2. **Tag Configuration**:
   - **Tag Type**: Custom HTML
   - **Tag Name**: "GHL Forms & Surveys - Event Listener"
3. **Pegar el código JavaScript** (ver Anexo A)
4. **Modificar el dominio** en la línea ALLOWED:
   ```javascript
   'https://DOMINIO-DEL-CLIENTE.com'  // Cambiar por dominio real
   ```
5. **Triggering**: Initialization - All Pages
6. **Guardar**

#### 1.2 Verificar instalación
- Activar **Preview Mode** en GTM
- Verificar en consola que aparezca: `"GoHighLevel Forms & Surveys Tracking Script loaded"`

---

### **PASO 2: Crear Variables de Data Layer**

Crear las siguientes variables tipo **Data Layer Variable**:

| Nombre Variable | Clave Data Layer | Versión |
|---|---|---|
| DLV - transaction_id | `transaction_id` | Versión 2 |
| DLV - form_id | `form_id` | Versión 2 |
| DLV - user_email | `user_email` | Versión 2 |
| DLV - user_phone | `user_phone` | Versión 2 |
| DLV - appointment_date | `appointment_date` | Versión 2 |
| DLV - location_id | `location_id` | Versión 2 |

#### Proceso para cada variable:
1. **Variables** → **New**
2. **Variable Type**: Data Layer Variable
3. **Data Layer Variable Name**: [Usar clave de la tabla]
4. **Data Layer Version**: Version 2
5. **Guardar**

---

### **PASO 3: Configurar Variable User-Provided Data**

#### 3.1 Crear o modificar variable UPD
1. **Variables** → **New** (o editar existente)
2. **Variable Type**: User-Provided Data
3. **Type**: Manual configuration
4. **Email**: `{{DLV - user_email}}`
5. **Phone**: `{{DLV - user_phone}}`
6. **Variable Name**: "UPD - Email & Phone"
7. **Guardar**

---

### **PASO 4: Crear Triggers**

#### 4.1 Trigger para Formularios
1. **Triggers** → **New**
2. **Trigger Configuration**:
   - **Trigger Type**: Custom Event
   - **Event Name**: `hl_form_submit`
   - **Use regex matching**: ❌ (desactivado)
3. **Trigger Name**: "GHL - Form Submit"
4. **Guardar**

#### 4.2 Trigger para Encuestas
1. **Triggers** → **New**
2. **Trigger Configuration**:
   - **Trigger Type**: Custom Event
   - **Event Name**: `hl_survey_submit`
   - **Use regex matching**: ❌ (desactivado)
3. **Trigger Name**: "GHL - Survey Submit"
4. **Guardar**

---

### **PASO 5: Configurar Tags de Conversión**

#### 5.1 Google Ads - User-Provided Data Event
1. **Tags** → **New**
2. **Tag Configuration**:
   - **Tag Type**: Google Ads - User-Provided Data Event
   - **Conversion ID**: `AW-XXXXXXXXX` (insertar Conversion ID del cliente)
   - **User-data variable**: `{{UPD - Email & Phone}}`
3. **Triggering**: 
   - Para formularios: `GHL - Form Submit`
   - Para encuestas: `GHL - Survey Submit`
4. **Tag Name**: "Google Ads - UPD Event"
5. **Guardar**

#### 5.2 Google Ads - Conversion
1. **Tags** → **New**
2. **Tag Configuration**:
   - **Tag Type**: Google Ads - Conversion
   - **Conversion ID**: `AW-XXXXXXXXX`
   - **Conversion Label**: `CONVERSION_LABEL` (del cliente)
   - **Transaction ID**: `{{DLV - transaction_id}}`
3. **Triggering**: 
   - Para formularios: `GHL - Form Submit`
   - Para encuestas: `GHL - Survey Submit`
4. **Advanced Settings** → **Tag Sequencing**:
   - **Fire a tag after this tag fires**: ❌
   - **Setup tag**: ✅ → Wait for tags: "Google Ads - UPD Event"
5. **Tag Name**: "Google Ads - Conversion"
6. **Guardar**

#### 5.3 GA4 - Event
1. **Tags** → **New**
2. **Tag Configuration**:
   - **Tag Type**: Google Analytics: GA4 Event
   - **Configuration Tag**: [Tu GA4 Config Tag]
   - **Event Name**: `generate_lead`
   - **Event Parameters**:
     - `transaction_id`: `{{DLV - transaction_id}}`
     - `form_id`: `{{DLV - form_id}}`
     - `method`: `form_submission` (para forms) o `survey_submission` (para surveys)
3. **Triggering**: 
   - Para formularios: `GHL - Form Submit`
   - Para encuestas: `GHL - Survey Submit`
4. **Tag Name**: "GA4 - Lead Event"
5. **Guardar**

#### 5.4 Meta Pixel - Lead Event
1. **Tags** → **New**
2. **Tag Configuration**:
   - **Tag Type**: Meta Pixel
   - **Pixel ID**: `XXXXXXXXXXXXXXX` (Pixel ID del cliente)
   - **Event**: Lead
   - **Event Parameters** (opcional):
     - `content_name`: `Form Lead` o `Survey Lead`
     - `content_category`: `Lead Generation`
3. **Triggering**: 
   - Para formularios: `GHL - Form Submit`
   - Para encuestas: `GHL - Survey Submit`
4. **Tag Name**: "Meta - Lead Event"
5. **Guardar**

---

### **PASO 6: Testing y Validación**

#### 6.1 GTM Preview
1. **Activar Preview Mode** en GTM
2. **Cargar página** con formulario/encuesta
3. **Verificar que el Event Listener se ejecuta**:
   - Tag "GHL Forms & Surveys - Event Listener" aparece como "Fired"

#### 6.2 Completar formulario/encuesta de prueba
1. **Llenar todos los campos** requeridos
2. **Enviar formulario/encuesta**
3. **En GTM Preview verificar**:
   - Aparece evento `hl_form_submit` o `hl_survey_submit`
   - Se ejecutan tags correspondientes
   - Variables DLV están pobladas


Script "Event Listener

<script>
console.log('📋 GoHighLevel Forms & Surveys Tracking Script loaded - Unified Version');
(function () {
  var ALLOWED = [
    'https://api.leadconnectorhq.com',
    'https://widgets.leadconnectorhq.com',
    'https://widgets.gohighlevel.com',
    'https://DOMINIO-DEL-CLIENTE.com'  // CAMBIAR POR DOMINIO REAL
  ];
  var contactCache = {};
  
  window.addEventListener('message', function (msg) {
    console.log('📨 Message received:', {origin: msg.origin, data: msg.data});
    
    if (ALLOWED.indexOf(msg.origin) === -1) {
      console.log('❌ Message from disallowed origin:', msg.origin);
      return;
    }
    
    console.log('✅ Message from allowed origin:', msg.origin);
    
    var d = msg.data;
    if (typeof d === 'string') { try { d = JSON.parse(d); } catch(e){} }
    
    /* ---------- 1. store contact details (SURVEYS & FORMS) ---------- */
    if (Array.isArray(d) && d[0] === 'set-sticky-contacts' && typeof d[2] === 'string') {
      console.log('👤 Contact data received (Survey/Form):', d[2]);
      try { 
        contactCache = JSON.parse(d[2]) || {}; 
        console.log('💾 Contact cache updated:', contactCache);
      } catch(e){}
      
      var email = contactCache.email || contactCache.contact_email || '';
      var phone = contactCache.phone || contactCache.contact_phone || '';
      var customerId = contactCache.customer_id || '';
      var locationId = contactCache.location_id || contactCache.locationId || '';
      
      // Detectar si es survey o form basado en la URL del mensaje o datos
      var currentUrl = window.location.href;
      var isForm = currentUrl.indexOf('/form') !== -1 || 
                   msg.origin.indexOf('DOMINIO-DEL-CLIENTE.com') !== -1 ||
                   contactCache.fechas_de_cita ||
                   contactCache.full_address !== undefined;
      
      var eventName = isForm ? 'hl_form_submit' : 'hl_survey_submit';
      
      window.dataLayer = window.dataLayer || [];
      window.dataLayer.push({
        event: eventName,
        transaction_id: customerId,
        location_id: locationId,
        user_email: email,
        user_phone: phone,
        form_id: customerId,
        appointment_date: contactCache.fechas_de_cita || '',
        contact_data: contactCache
      });
      
      console.log('🔥 ' + eventName + ' pushed to dataLayer!', {
        customerId: customerId,
        email: email,
        phone: phone,
        locationId: locationId,
        eventType: eventName
      });
      
      return;
    }
    
    /* ---------- 2. Object-based events (FORMS) ---------- */
    if (d && typeof d === 'object' && !Array.isArray(d)) {
      // Detectar eventos de formularios con estructura de objeto
      if (d['set-sticky-contacts'] || d.customer_id || d.fechas_de_cita || d.full_address !== undefined) {
        console.log('📝 Form data received (Object format):', d);
        
        var email = d.email || d.contact_email || '';
        var phone = d.phone || d.contact_phone || d.telefono || d.celular || '';
        var customerId = d.customer_id || '';
        var appointmentDate = d.fechas_de_cita || '';
        
        window.dataLayer = window.dataLayer || [];
        window.dataLayer.push({
          event: 'hl_form_submit',
          transaction_id: customerId || 'form_' + Date.now(),
          user_email: email,
          user_phone: phone,
          form_id: customerId,
          appointment_date: appointmentDate,
          form_fields: d
        });
        
        console.log('🔥 hl_form_submit pushed to dataLayer! (Object)', {
          customerId: customerId,
          email: email,
          phone: phone,
          appointmentDate: appointmentDate
        });
        
        return;
      }
    }
    
    /* ---------- 3. legacy form/survey submissions ---------- */
    if (d && (d.action === 'formSubmitted' || d.event === 'form_success' || d.eventName === 'survey-completed')) {
      console.log('🎯 LEGACY submission detected!', d);
      
      var p = d.payload || {};
      var isLegacyForm = d.action === 'formSubmitted' || d.event === 'form_success';
      var legacyEventName = isLegacyForm ? 'hl_form_submit' : 'hl_survey_submit';
      
      window.dataLayer = window.dataLayer || [];
      window.dataLayer.push({
        event: legacyEventName,
        transaction_id: d.fingerprint || '',
        location_id: p.locationId || '',
        user_email: p.contact_email || '',
        user_phone: p.contact_phone || ''
      });
      
      console.log('🔥 ' + legacyEventName + ' pushed to dataLayer! (legacy)', d);
    }
  });
})();
</script>
```


---

**Versión**: 1.0  
**Última actualización**: Agosto 2025  
**Autor**: Gabriel Ferrés
