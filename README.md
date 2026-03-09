# Yaku-Sys: Core Financiero y ERP Transaccional

## Resumen Ejecutivo

Nota: El código fuente de este proyecto se mantiene en un repositorio privado por acuerdos de confidencialidad. Este repositorio sirve como un caso de estudio de la arquitectura implementada.

Yaku-Sys es un sistema ERP de grado de producción diseñado para gestionar la facturación, recaudación y control de morosidad de una empresa de servicios en Ecuador. El diseño se centra en la inmutabilidad de los datos y la prevención de fraudes operativos, trasladando la carga transaccional pesada directamente al motor de la base de datos.

## Paradigmas de Diseño Implementados

### 1. Motor de Partida Doble en Base de Datos

**Diseño**: Implementación de un catálogo de cuentas y un motor de contabilidad de doble entrada de forma nativa en PostgreSQL.

**Integridad ACID**: Uso intensivo de Triggers y Funciones Almacenadas (plpgsql) para garantizar que cada transacción (cobro, anulación, cargo) cuadre matemáticamente antes de realizar el commit, bloqueando cualquier intento de fraude en caja.

### 2. Arquitectura de Seguridad (RBAC & Lockdown)

Implementación de un modelo de Control de Acceso Basado en Roles (RBAC) con "Guardianes de Arquitectura" a nivel de base de datos.

Pol��ticas de Seguridad de Filas (RLS) para aislar la visibilidad de los datos según el rol operativo del usuario.

### 3. Frontend y Tipado Estricto

Desarrollo de la interfaz de cliente utilizando Next.js y React.

Modelado estricto de tipos con TypeScript para asegurar que el contrato de datos entre la API y el frontend se mantenga íntegro.

## Diagrama de Entidades y Relaciones (ERD)

```mermaid
erDiagram
    %% MÓDULO IDENTIDAD
    IDENTIDADES ||--o{ USUARIOS_SISTEMA : "es_actor_de"
    IDENTIDADES ||--o{ LIBRO_DIARIO : "responsable_de"
    IDENTIDADES ||--o{ CONTRATOS_AGUA : "posee"
    IDENTIDADES ||--o{ ESTADOS_MEDIDOR : "lector_de"

    %% MÓDULO FINANCIERO Y OPERACIÓN
    PLAN_CUENTAS ||--o{ DETALLES_ASIENTO : "imputa_a"
    PLAN_CUENTAS ||--o{ OPERACIONES_FINANCIERAS : "vinculo_unico"
    LIBRO_DIARIO ||--|{ DETALLES_ASIENTO : "contiene"
    OPERACIONES_FINANCIERAS ||--o{ DETALLES_ASIENTO : "afecta_saldo"
    OPERACIONES_FINANCIERAS ||--o{ HISTORICO_CIERRES : "genera_snapshot"
    
    %% SATÉLITES FINANCIEROS
    OPERACIONES_FINANCIERAS ||--o| SATELITE_CARTERA : "extension_datos"
    OPERACIONES_FINANCIERAS ||--o| SATELITE_BILLETERA : "extension_datos"
    IDENTIDADES ||--o{ SATELITE_CARTERA : "pertenece_a"
    IDENTIDADES ||--o{ SATELITE_BILLETERA : "pertenece_a"

    %% MÓDULO AGUA
    CONTRATOS_AGUA ||--o{ ESTADOS_MEDIDOR : "tiene_lecturas"
    CICLOS_FACTURACION ||--o{ ESTADOS_MEDIDOR : "agrupa_lecturas"
    CICLOS_FACTURACION ||--o{ EMISIONES_ESTADO_CUENTA : "genera"
    LIBRO_DIARIO ||--o| ESTADOS_MEDIDOR : "contabiliza"
    IDENTIDADES ||--o| ESTADO_MOROSIDAD_CLIENTE : "monitoreo"

    %% MÓDULO DOCUMENTAL
    PLANTILLAS_IMPRESION ||--o{ ARCHIVO_DOCUMENTAL_INMUTABLE : "define_formato"
    ARCHIVO_DOCUMENTAL_INMUTABLE ||--o{ METADATOS_DOCUMENTO : "extiende_info"
    ARCHIVO_DOCUMENTAL_INMUTABLE ||--o{ TRAZABILIDAD_DOCUMENTAL : "bitacora_estados"
    ARCHIVO_DOCUMENTAL_INMUTABLE ||--o{ EMISIONES_ESTADO_CUENTA : "es_respaldo_de"
    ARCHIVO_DOCUMENTAL_INMUTABLE ||--o| SATELITE_ACUERDOS_PAGO : "es_contrato_de"
    
    %% VÍNCULOS CRUZADOS
    ARCHIVO_DOCUMENTAL_INMUTABLE ||--o{ VINCULO_DOCUMENTAL_CONTABLE : "soporte_fisico"
    LIBRO_DIARIO ||--o{ VINCULO_DOCUMENTAL_CONTABLE : "soporte_contable"
    DETALLES_ASIENTO ||--o| CONCILIACION_BANCARIA : "requiere"

    IDENTIDADES {
        uuid id PK
        text identificacion_fiscal
        text nombre_comercial
        enum tipo_identidad
    }

    OPERACIONES_FINANCIERAS {
        uuid id PK
        uuid id_cuenta_contable FK
        numeric saldo_deudor
        numeric saldo_acreedor
        text codigo_propio
    }

    ARCHIVO_DOCUMENTAL_INMUTABLE {
        uuid id PK
        jsonb datos_congelados
        text ruta_archivo_pdf
        text codigo_verificacion
    }

    DETALLES_ASIENTO {
        uuid id PK
        uuid id_libro_diario FK
        numeric debe
        numeric haber
    }

    ESTADOS_MEDIDOR {
        uuid id PK
        uuid id_ciclo FK
        uuid id_medidor FK
        numeric lectura
    }

    TARIFAS {
        uuid id PK
        decimal precio_base
        decimal multa_atraso
    }
}
```

## 📸 Capturas de Pantalla del Sistema

(Añade aquí las imágenes de tu sistema funcionando, esquemas de la base de datos, o vistas del dashboard)