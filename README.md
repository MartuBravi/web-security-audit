# Auditoría de Seguridad Web

**Auditora:** Martina Bravi

## Resumen Ejecutivo
El presente reporte detalla los resultados de una auditoría de seguridad de "Caja Negra" realizada sobre un entorno web. El propósito fue evaluar la postura de seguridad frente a vectores de ataque comunes, identificando exposiciones críticas en la configuración del servidor y vulnerabilidades derivadas de la falta de mantenimiento del software. 

## Aviso de Confidencialidad y Ética
**IMPORTANTE:** Para proteger la integridad de la infraestructura analizada y cumplir con las mejores prácticas de la industria, la URL y las direcciones IP reales han sido anonimizadas. Todos los hallazgos son producto de una auditoría autorizada.

## Alcance del Proyecto
la evaluación se centró en:
* **Identificación de superficie de ataque:** Mapeo de servicios expuestos.
* **Enumeración de vulnerabilidades:** Análisis de CMS y plugins.
* **Evaluación de autenticación:** Pruebas de resiliencia ante ataques de fuerza bruta y enumeración de usuarios.

---

## 🛠️ Metodología y Hallazgos

## 🛡️ Fase 1: Reconocimiento e Identificación (Footprinting)

El objetivo de esta fase fue obtener la identidad técnica del servidor y su dirección física en la red.

* **Identificación de IP:** Se utilizaron los comandos `nslookup` para resolver la dirección IP del objetivo y `ping` para comprobar su conectividad.

* **Escaneo de Servicios:** Con `nmap -sV [IP]`, se identificaron los puertos abiertos y las versiones de los servicios corriendo.

La siguiente tabla detalla todos los servicios detectados y su función:

| Puerto | Protocolo | Servicio | Función |
| :--- | :--- | :--- | :--- |
| **21** | TCP | FTP | Transferencia de archivos |
| **22** | TCP | SSH | Gestión remota |
| **25, 587, 2525** | TCP | SMTP | Envío de correos electrónicos |
| **80, 443** | TCP | HTTP/HTTPS | Servidor Web (LiteSpeed) |
| **110, 995** | TCP | POP3/POP3s | Recepción de correos |
| **143, 993** | TCP | IMAP/IMAPs | Sincronización de correos |
| **465** | TCP | SMTPs | Envío de correos cifrado |

> **Análisis de superficie de ataque:**
> La presencia de múltiples servicios de correo electrónico junto con los servicios de administración remota (SSH, FTP) indica que el objetivo opera en un entorno de hosting compartido. Esto incrementa significativamente la superficie de ataque, ya que una vulnerabilidad en cualquier componente (ej. una versión desactualizada de `Exim` o `Dovecot`) podría comprometer la integridad del servidor.

<details>
  <summary>📸 Click para ver la evidencia técnica (Captura de Nmap)</summary>
  
  <img width="540" height="413" alt="image" src="https://github.com/user-attachments/assets/dcbe1d49-d9c0-40db-8b32-cf008349a27f" />
</details>

###  **Identificación de Stack Tecnológico:** 
El proceso de identificación se llevó a cabo de manera iterativa, documentando la evolución desde la detección pasiva hasta la verificación activa.

 #### **Intento 1: Detección Pasiva (`whatweb`)**
  Se utilizó `whatweb` para obtener una visión rápida del entorno.
    * **Resultado:** La herramienta devolvió un error `403 Forbidden` y no logró identificar el CMS específico, limitándose a reconocer el servidor web `LiteSpeed`.
    * **Análisis:** Las configuraciones de seguridad del servidor impidieron una huella digital pasiva precisa.

<details>
  <summary>📸 Click para ver la evidencia técnica</summary>
<img width="1058" height="104" alt="image" src="https://github.com/user-attachments/assets/2367228c-615c-45ae-a0d8-bfde1987d7d1" />
</details>

 #### **Intento 2: Verificación Activa (`curl`)**
  Ante la falta de resultados concluyentes, se optó por una consulta directa a un endpoint crítico de WordPress.
    * **Comando:** `curl https://[TARGET_URL]/wp-login.php`
    * **Evidencia:** El servidor respondió con la estructura HTML de acceso administrativo de WordPress, confirmando el CMS utilizado.

<details>
  <summary>📸 Click para ver la evidencia</summary>
  
  <img width="893" height="124" style="margin-bottom: 20px;" alt="image" src="https://github.com/user-attachments/assets/021bb478-5d2e-4675-8af4-283da9af2cbd" />
  <br>
  <img width="895" height="45" alt="image" src="https://github.com/user-attachments/assets/acf2d9f6-1505-478f-9cac-8df9b89d71e7" />
</details>

> **Nota de Auditoría:** Este proceso resalta la importancia de no depender de una única herramienta. La capacidad de realizar consultas manuales permitió superar las restricciones de seguridad que afectaron a las herramientas automatizadas.

---

## 🛡️ Fase 2: Descubrimiento de Contenido (Directory Fuzzing)

### **Mapeo de Directorios y Estructura**
Inicialmente, se realizaron pruebas de descubrimiento de contenido utilizando herramientas de fuzzing como `gobuster` y `dirb`, con el objetivo de identificar directorios ocultos.

Sin embargo, durante estas pruebas se detectó que el servidor web (`LiteSpeed`) implementa respuestas personalizadas que devuelven códigos `200 OK` para recursos inexistentes (páginas tipo 404 con contenido customizado), lo que generó una alta tasa de falsos positivos y redujo la confiabilidad de los resultados.

<details>
  <summary>📸 Click para ver la evidencia</summary>
<img width="699" height="447" alt="Captura de pantalla 2026-04-20 223825" src="https://github.com/user-attachments/assets/a352fc79-b630-4102-9922-ad21d9a9d41a" />
</details>

### **Enumeración Dirigida** 
En base a este comportamiento, se ajustó la estrategia hacia una enumeración dirigida, apoyada en el conocimiento de la arquitectura de WordPress. Este enfoque permitió obtener resultados más precisos y reducir la interacción innecesaria con el servidor, evitando bloqueos por mecanismos de defensa como *Rate Limiting* o WAF.

Se validó la existencia de directorios críticos como `/wp-admin/`, `/wp-login.php`, `/wp-json/` y `/xmlrpc.php`, los cuales son componentes nativos de la arquitectura del sitio.
<details>
  <summary>📸 Click para ver la evidencia</summary>
<img width="440" height="391" alt="image" src="https://github.com/user-attachments/assets/551b82c3-a8d9-4b10-800a-68aea3082f86" />
</details>

> **Nota de Auditoría:** En escenarios donde el CMS no es identificable o la estructura no es conocida, el uso de fuzzing con diccionarios específicos continúa siendo una técnica válida y recomendada.

---

## 🛡️ Fase 3: Análisis Específico de CMS (WordPress)

### Detección de vulnerabilidades
Al confirmar el uso de WordPress, se procedió a una auditoría especializada para detectar vulnerabilidades en el núcleo y complementos.

* **Comando inicial:** `wpscan --url [TARGET_URL] --enumerate u`
* **Evasión de WAF:** El servidor bloqueaba peticiones automáticas.

  Se aplicó:
    * `--random-user-agent`: Para disfrazar el tráfico como un navegador común.
    * `--throttle 1000`: Para reducir la velocidad y no alertar al firewall.

Comando aplicado: 
`wpscan --url [TARGET_URL] --enumerate vp --random-user-agent --throttle 1000`
<details>
  <summary>📸 Click para ver la evidencia</summary>
  <img width="1000" height="341" alt="image" src="https://github.com/user-attachments/assets/b8e289a7-c9f8-4e63-87b8-b6c3a16649bd" />
</details>

### **Hallazgos de Infraestructura**

  * **xmlrpc.php:** Activo. Se confirmó con `curl -X POST` que el servicio acepta estructuras XML. Esto representa un riesgo de fuerza bruta masiva mediante el método de `system.multicall`.
  * **readme.html:** Expuesto (revela información de instalación).

  <details>
         <summary>📸 Click para ver la evidencia</summary>
    <img width="838" height="459" alt="image" src="https://github.com/user-attachments/assets/ed9e241f-b19a-487e-9e81-bf64ed9a3d86" />
     </details>

---

## 🛡️ Fase 4: Análisis de Vulnerabilidades (Vulnerability Assessment)

###  **Exploración de plugins:** 
Se buscaron plugins desactualizados en la página para descubrir si existe alguna vulnerabilidad utilizando:
`wpscan --url [TARGET_URL] -e ap,u --random-user-agent --throttle 1000`

Se identificaron 5 plugins desactualizados (WooCommerce, Contact Form 7, etc.).

<details>
  <summary>📸 Click para ver la evidencia (Plugins identificados)</summary>
<img width="792" height="761" alt="image" src="https://github.com/user-attachments/assets/e668a085-e8a0-44a9-b9fb-60165545a503" />
<img width="799" height="137" alt="image" src="https://github.com/user-attachments/assets/8af1dda4-5dcc-46d9-8da7-d5617b516b38" />
</details>

* **Verificación de Exploits:** Se utilizó `searchsploit` para buscar vulnerabilidades específicas.
* **Resultado:** No se encontraron exploits públicos directos.

###  **Profundización en Análisis de Vulnerabilidades (Vulnerability Assessment Avanzado):**

El uso de herramientas genéricas demostró ser insuficiente para un entorno WordPress, donde la seguridad depende estrechamente de la versión específica de cada plugin. Por lo tanto, se optó por una metodología de enumeración dirigida mediante `WPScan` con integración de API, permitiendo cruzar el stack tecnológico detectado contra bases de datos de vulnerabilidades actualizadas en tiempo real (WPVulnDB).

Comando utilizado:
`wpscan --url [TARGET_URL] --api-token [TOKEN] --enumerate vp --random-user-agent `

Este proceso permitió identificar vulnerabilidades (CVEs) que no estaban contempladas en las bases de datos locales, transformando un hallazgo de 'Riesgo Medio' por software desactualizado en una identificación concreta de vectores de ataque explotables.

<details>
  <summary>📸 Click para ver la evidencia (Análisis dinámico)</summary>
<img width="798" height="745" alt="image" src="https://github.com/user-attachments/assets/80eff48e-5f51-44c4-b3f2-f5c1bef498a9" />
</details>

###  Vulnerabilidades Identificadas

Se detectaron tres vulnerabilidades críticas en el plugin `smtp-mail` (v1.2.13), las cuales representan un riesgo directo para la seguridad de la plataforma:

* **CVE-2023-3092 (XSS Stored):** Permite la inyección de scripts maliciosos en la configuración del plugin, los cuales se ejecutan en la sesión del administrador.
* **CVE-2024-25914 (CSRF):** Posibilita la ejecución de acciones no autorizadas mediante la manipulación de peticiones del usuario.
* **CVE-2025-62762 (CSRF):** Vulnerabilidad adicional de *Cross-Site Request Forgery* que permite la modificación no consentida de la configuración del plugin.

Se detectó una vulnerabilidad crítica en el plugin `page-views-count` (CVE-2025-63034): 
* **Fallo de Missing Authorization en la versión 2.8.7:** permite a usuarios con privilegios mínimos (suscriptores) modificar configuraciones críticas del plugin.

### Evaluación de riesgos

| ID | Hallazgo | Severidad | Impacto |
| :--- | :--- | :--- | :--- |
| CVE-2023-3092/etc | Vulnerabilidades críticas en plugin `smtp-mail` (XSS/CSRF) | Alta | Ejecución de scripts / Configuración comprometida |
| CVE-2025-63034 | Missing Authorization en `page-views-count` | Media | Escalada de privilegios |
| Configuración | Exposición de `xmlrpc.php` | Media | Fuerza bruta / denegación de servicio |
| Información | Exposición de `readme.html` | Baja | Enumeración de versiones |

## 🛡️ Fase 5: Pruebas de Autenticación (Exploitation Attempt)

Se intentó validar el acceso al área administrativa mediante vectores detectados previamente.

* **Enumeración de Usuarios:** `wpscan --enumerate u` arrojó los usuarios `buscalotodo` y `mb`.
  <details>
           <summary>📸 Click para ver la evidencia (Fase 5)</summary>
           <img width="654" height="152" alt="image" src="https://github.com/user-attachments/assets/40c32cf9-45ab-4931-8a1d-330a3e75b9eb" />

</details>

* **Identificación de Usuario Real:** Mediante la observación de indicadores en el sitio, se halló un correo electrónico que el sistema de login validó como "existente" (aunque la contraseña fuera errónea).
* **Ataque de Fuerza Bruta:** Se creó un diccionario personalizado y se lanzó un ataque vía XML-RPC.
    * *Resultado:* Negativo. Las credenciales no se encuentran en los diccionarios utilizados.
   <details>
           <summary>📸 Click para ver la evidencia (Fase 5)</summary>
          <img width="719" height="120" alt="image" src="https://github.com/user-attachments/assets/4e5f8042-f779-4447-b7c7-a959565e7e40" />

</details>

---

## 🛡️ Fase 6: Priorización y Recomendaciones

### Hallazgos Críticos Priorizados:

Tras el análisis de vulnerabilidades mediante inteligencia de amenazas, se definieron las prioridades de remediación basadas en el riesgo real detectado
1. **Vulnerabilidades en `smtp-mail` (CVE-2023-3092, CVE-2024-25914, CVE-2025-62762):** Se detectaron fallas de *XSS Stored* y *CSRF* en la versión 1.2.13. Estos vectores permiten la manipulación de la configuración del servidor de correo y la ejecución de código en la sesión del administrador.
2. **Vulnerabilidad en `page-views-count` (CVE-2025-63034):** Fallo de *Missing Authorization* en la versión 2.8.7 que permite a usuarios con privilegios mínimos (suscriptores) modificar configuraciones críticas del plugin.
3. **Exposición de Servicios (XML-RPC):** El endpoint `/xmlrpc.php` se encuentra activo y plenamente funcional, facilitando ataques de fuerza bruta automatizados.

### Plan de Acción Recomendado:

* **Mitigación Inmediata (Prioridad Alta)**
    * Actualizar de forma obligatoria los plugins `smtp-mail` (a versión 1.3.51+) y `page-views-count` (a versión 2.9.0+) para corregir los CVEs identificados.
    * Deshabilitar el endpoint `xmlrpc.php` si no se utiliza para servicios de terceros.
* **Hardening de Seguridad:**
    * Borrar archivos de instalación legados como `readme.html` para evitar la enumeración de versiones.
    * Implementar un *Web Application Firewall* (WAF) que filtre peticiones dirigidas a la API de usuarios (`wp-json`) para evitar la enumeración de cuentas.

<details>
  <summary>📸 Click para ver la evidencia (Priorización y Plan de acción)</summary>
  <img src="URL_DE_TU_IMAGEN_859872" alt="Recomendaciones y plan de acción" />
</details>

### 🏁 Conclusión Final

El análisis realizado permitió identificar múltiples vectores de ataque, destacándose vulnerabilidades en plugins desactualizados y exposición de endpoints críticos. Si bien no se logró comprometer el acceso administrativo, la superficie de ataque actual representa un riesgo significativo ante actores maliciosos. Se recomienda la aplicación inmediata de las medidas de mitigación propuestas.
