# Manual Técnico de Explotación - WillmanTech ERP/CR

**Version:** 1.0.0
**Fecha de emeision:** 2025-05-15
**Elaborado por:** Departamento de Desarrollo DAW/DAM
**Jefe:** Álvaro López De San Román 
**Estandar aplicado:** ISO/IEC/IEE 26514:_2022 - *Requirements for designers and dev of user documentation*

---

## Tabla de Contenidos 

1. [Introducción y Arquitectura del Sistema](#1-introducción-y-arquitectura-del-sistema)
2. [Guía de Instalación y Reinstalación](#2-guía-de-instalación-y-reinstalación)
3. [Seguridad y Control de Acceso](#3-seguridad-y-control-de-acceso)
4. [Procedimiento de Backup y Restauración](#4-procedimiento-de-backup-y-restauración)
5. [Flujo Operativo de Facturación e Informes](#5-flujo-operativo-de-facturación-e-informes)
   
## 1. Introducción y Arquitectura del Sistema
### 1.1 Proposito del Docmuento 

Este manual Tecnico de explotacion describe los procdiemientos oprativos, tanto configuracion y mantenimiento del sistema ERP/CRM implnatado para WIllmanTech S.L. ha sido redactado conform a las directrices del estandar ISO/IEC/IEE 26514:_2022, en donde se estabalce los requisitos de calidad  y usabilidad para la documentacion tecnica del sistema de software en entornos productivos 

Este docuemnto sera dirigdo a dos perfiles de usuario:
-**Administradores de sistemas**
-**Usuarios operativos**(contables y comeraciales): ERP

### 1.2 Descripcioón General del Sistema

WillmanTech ERP es una solucójn de getsión empresarial basada en **Odooo Community Edition 17**, enj dond eun ERP open-source abierto libremente utilizando un sistema empresarial europeo.

## 1.3 Arquitectura de Despliegue — Docker Compose

El sistema está desplegado mediante **Docker Compose**, una herramienta de orquestación de contenedores que permite definir y levantar múltiples servicios en un único fichero de configuración (`docker-compose.yml`).

<img width="1153" height="715" alt="image" src="https://github.com/user-attachments/assets/79b4e863-ae1f-45c6-8e09-418afd4a5a75" />

---

## 2. Guía de Instalación y Reinstalación

### 2.1 Requisitos Previos del Sistema

| Componente | Requisito mínimo | Recomendacion |
| --- | --- | --- |
| SO | Ubuntu 22.04 LTS| Ubuntu 22.04 LTS |
| CPU | 2 vCPU x86_64| 4 vCPU |
| RAM | 4 GB| 8 GB |
| Disco | 40 GB SSD | 100 GB SSD |

## 3. Seguridad y Control de Acceso

### 3.1 Arquitectura de Roles en Odoo

Odoo implementa un sistema de control de acceso basado en (roles). Cada usuario pertenece a uno o varios grupos que determinan qué puede ver y hacer en el sistema.

Los tres roles principales configurados para WillmanTech son:

#### Rol: Administrador del Sistema

| Parámetro | Valor |
|---|---|
| Nombre del grupo | `base.group_system` |
| Acceso | Total a todos los módulos |
| Gestión de usuarios| Sí |
| Configuración técnica | Sí modo desarrollador |
| Exportación de datos | Sí |
| **Asignación** | Solo 1-2 personas en producción|

#### Rol: Contable

| Parámetro | Valor |
|---|---|
| Nombre del grupo| `account.group_account_manager`|
| Módulos accesibles | Contabilidad, Facturación |
| Crear facturas | Sí |
| Confirmar/Validar | Sí |
| Configuración contable | Sí, plan de cuentas, impuestos|
| Ver nóminas | No |

#### Rol: Comercial

| Parámetro | Valor |
|---|---|
| Nombre del grupo   | `sales_team.group_sale_salesman`|
| Módulos accesibles | Ventas, CRM, Clientes |
| Crear presupuestos | Sí |
| Confirmar pedidos  | Sí |
| Ver contabilidad   | Solo facturas propias |
| Configuración      | No |


### 3.2 Política de Contraseñas

Odoo no impone políticas de contraseña por defecto. Para WillmanTech se ha configurado la siguiente política mediante el módulo `auth_password_policy`:

| Parámetro| Valor requerido |
|----|---|
| Longitud mínima | 12 caracteres |
| Mayúsculas | Mínimo 1 |
| Minúsculas | Mínimo 1 |
| Números | Mínimo 2 |
| Caracteres especiales | Mínimo 1 |
| Caducidad de contraseña | 90 días |
| Reutilización | No repetir últimas 5 |

### 3.4 Hardening de Seguridad Adicional

```bash
#  Deshabilitamos el gestor de bases de datos público (/web/database/manager)
docker exec -it willmantech_erp bash
echo "list_db = False" >> /etc/odoo/odoo.conf
exit
docker restart willmantech_erp

# Verificamos que el puerto 5432 (PostgreSQL) NO es accesible desde exterior
apt install fail2ban -y
# Regla para Odoo en /etc/fail2ban/jail.local
```
---
## 4. Procedimiento de Backup y Restauración

### 4.1 Estrategia de Backup

WillmanTech utiliza una estrategia de backup **3-2-1** consultada por google como se manejan las garndes empresas en estas situaciones :
- **3** copias de los datos
- en **2** soportes diferentes (disco local + almacenamiento en la nube)
- con **1** copia fuera del sitio (offsite / cloud)

| Tipo de backup | Frecuencia | Retención | Destino |
|---|---|---|---|
| Backup completo | Diario  | 30 días | `/backups/` + S3 |
| Backup incremental | Cada 6 horas | 7 días | `/backups/incremental/` |

# 5. Flujo Operativo de Facturación e Informes

### 5.1 Creación de una Factura de Cliente — Paso a Paso

Esta sección describe el flujo completo que sigue un usuario con perfil **Contable** para generar una factura:

 **FLUJO DE FACTURACIÓN — WillmanTech ERP**                     

[1] Menú: Contabilidad -> Clientes -> Factura

[2] Rellenar cabecera del documento:  
  -> Cliente: buscar por nombre/NIF
  ->Fecha de factura: automática (hoy) o manual 
  -> Término de pago: seleccionar 
  
[3] Añadir líneas de detalle:
  -> Clic en "Añadir una línea"
  -> eleccionar producto 
  -> Ajustar cantidad y precio si difiere del catálogo 
  -> Aplicar descuento si procede (campo Descuento %) 
  
[4] Guardar el borrador: -> Botón "Guardar" — Estado: BORRADOR    
[5] Validar / Confirmar la factura: -> Botón "Confirmar" — Estado: PUBLICADA 
[6] Imprimir o enviar el PDF                             

<img width="528" height="332" alt="image" src="https://github.com/user-attachments/assets/796c8f3f-1769-48a3-9860-9332d7bdd1d0" />

  

                                                           

