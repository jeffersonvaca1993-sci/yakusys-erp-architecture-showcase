Yaku-Sys: Core Financiero y ERP TransaccionalNota de Autoría: Este proyecto y su arquitectura lógica han sido concebidos y diseñados íntegramente por Jefferson. El código fuente se mantiene en un repositorio privado por acuerdos de confidencialidad; este documento sirve como exposición de la arquitectura y los patrones de diseño implementados.💧 Resumen EjecutivoYaku-Sys es un ERP de grado industrial optimizado para la gestión de servicios públicos (agua potable) en el contexto legal y fiscal de Ecuador. A diferencia de los sistemas tradicionales que confían la lógica al backend/frontend, Yaku-Sys implementa un "Smart Database Pattern", donde la integridad financiera y las reglas de negocio están blindadas a nivel de motor de base de datos.🛠 Stack TecnológicoCore: PostgreSQL 15+ (PostGIS ready).Backend Serverless: Supabase (Auth, Storage, Realtime).Lógica de Negocio: PL/pgSQL (Funciones autoejecutables y Triggers de integridad).Frontend: Next.js 14 (App Router), React, Tailwind CSS.Contrato de Datos: TypeScript (Tipado estricto extremo).🏛 Paradigmas de Arquitectura1. Motor Contable "Hard-Wired"El sistema no "registra" asientos; genera asientos como consecuencia atómica de acciones operativas.Partida Doble Nativa: Implementación de un catálogo de cuentas donde cada transacción cumple con el principio de dualidad antes de que el COMMIT sea exitoso.Running Balances: Optimización de lectura mediante saldos vivos en nodos operativos, sincronizados mediante triggers inyectados.2. Blindaje Documental (Inmutabilidad)La confianza es el pilar del sistema.Snapshotting: Cada factura o recibo se congela en un objeto JSONB inmutable. Si la lógica de tarifas cambia en el futuro, el documento histórico permanece fiel a lo que el cliente recibió.Cadena de Trazabilidad: Cada cambio de estado (GENERADO -> FIRMADO -> ANULADO) queda registrado con sello de tiempo UTC inalterable.3. Seguridad de Nivel Bancario (Lockdown)Esquema Privado: El 100% de las tablas residen en un esquema privado inaccesible vía REST API estándar.Funciones Proxy: Solo se exponen funciones específicas con SECURITY DEFINER, actuando como una "aduana" de datos que valida permisos y sanitiza inputs antes de tocar las tablas core.📊 Diagrama de Entidades y Relaciones (ERD)erDiagram
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
📸 Evidencia de ImplementaciónVista general del estado de cartera y métricas de morosidad.Ejemplo de cómo una lectura de medidor genera automáticamente la partida doble en el Libro Diario.© 2026 Jefferson - Todos los derechos reservados.
