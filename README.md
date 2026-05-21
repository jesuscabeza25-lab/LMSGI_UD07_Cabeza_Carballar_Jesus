# LMSGI_UD07_Cabeza_Carballar_Jesus

# Manual Técnico de Explotación - WillmanTech S.L.
Documento redactado bajo el estándar internacional ISO/IEC/IEEE 26514:2022.

## 1. Introducción y Arquitectura
El actual sistema ERP/CRM de WillmanTech S.L. optimiza los flujos de ventas y facturación. El sistema presenta una arquitectura basada en microservicios y está desplegado bajo un entorno dockerizado. La topología lógica se orquesta a través de un archivo `docker-compose.yml`, aislando en contenedores separados el servidor de aplicaciones web y el Sistema Gestor de Bases de Datos (SGBD).

## 2. Guía de Instalación y Reinstalación
Para levantar el entorno productivo desde cero, asegúrese de tener Docker y Docker Compose instalados.
1. Clone el repositorio oficial de configuración.
2. Defina las variables de entorno necesarias en el archivo `.env` (ej. `POSTGRES_USER`, `POSTGRES_PASSWORD`, `WEB_PORT`).
3. La dependencia del SGBD (PostgreSQL) se resolverá automáticamente al ejecutar el despliegue.
4. Ejecute el comando de orquestación: `docker-compose up -d`.

## 3. Seguridad y Control de Acceso
El sistema se rige por un modelo de Control de Acceso Basado en Roles (RBAC):
* **Administrador:** Acceso total a configuración técnica, módulos y bases de datos.
* **Contable:** Privilegios de lectura y escritura sobre el módulo de facturación, acceso a extractos y exportación UBL/JSON.
* **Comercial:** Permisos limitados a la creación de presupuestos, lectura de catálogo de productos y cartera de clientes propia.
* *Políticas de contraseñas:* Se exige una longitud mínima de 12 caracteres, alfanumérica y rotación cada 90 días.

## 4. Procedimiento de Backup y Restauración
Para garantizar la sostenibilidad operativa, los respaldos completos deben ejecutarse diariamente.
* **Backup del SGBD (Relacional):** Utilice el siguiente comando para generar un volcado de seguridad:
    `docker exec -t [nombre_contenedor_db] pg_dump -U [usuario] -F c -b -v -f /ruta/backup/db_backup.dump [nombre_bd]`
* **Restauración:**
    `docker exec -i [nombre_contenedor_db] pg_restore -U [usuario] -d [nombre_bd] -1 /ruta/backup/db_backup.dump`

## 5. Flujo Operativo de Facturación e Informes
Cuando un usuario confirma un presupuesto en la interfaz, el ERP genera un objeto de Factura. Al solicitar el informe, el flujo del pipeline es el siguiente:
1. El motor **QWeb** compila la plantilla XML (como `report_invoice_willmantech.xml`) combinándola con los datos de la base de datos, generando un documento **HTML**.
2. El sistema envía este código HTML a la utilidad binaria subyacente llamada **wkhtmltopdf**.
3. Dicha utilidad renderiza el contenido visual y genera el archivo **PDF** final, el cual se descarga o se adjunta automáticamente al correo electrónico del cliente.
