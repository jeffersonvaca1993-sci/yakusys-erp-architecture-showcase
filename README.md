Yaku-Sys: Core Financiero y ERP TransaccionalAutoría y Diseño: Jefferson. Este repositorio expone la arquitectura lógica y el diseño de base de datos de un sistema de grado de producción bajo acuerdos de confidencialidad.💧 Resumen EjecutivoYaku-Sys es un ecosistema ERP diseñado específicamente para la gestión de servicios básicos y cobranza. A diferencia de soluciones genéricas, este sistema traslada la inteligencia de negocio al motor de base de datos (PostgreSQL), garantizando integridad financiera absoluta incluso si las capas superiores (Frontend/API) se ven comprometidas.🛠 Stack TecnológicoDatabase: PostgreSQL 15+ (Esquema privado con blindaje total).Backend: Supabase (Auth, Storage, RLS).Logic Layer: PL/pgSQL (Funciones SECURITY DEFINER).Frontend: Next.js 14 + Tailwind CSS.Data Integrity: TypeScript (Tipado estricto).🏛 Pilares Arquitectónicos (Conceptos de Jefferson)1. El "Smart Database Pattern"El sistema implementa una capa de abstracción donde el esquema público está vacío y solo expone funciones RPC. Esto garantiza que:Ningún usuario puede editar tablas directamente (Lockdown total).La lógica de Partida Doble se ejecuta en una transacción atómica dentro de la base de datos.2. Composición Mediante SatélitesEn lugar de una tabla de cuentas monolítica, se utiliza una Tabla Madre (operaciones_financieras) vinculada a Satélites Especializados (cartera, billetera, bancos). Esto permite que el sistema crezca en funcionalidades sin degradar el rendimiento de las consultas core.3. Inmutabilidad y AuditoríaLa tabla archivo_documental_inmutable actúa como una "Fuente de Verdad". Una vez que un documento (Factura/Recibo) es generado, sus datos se congelan en un JSONB, protegiendo el registro histórico de cambios futuros en tarifas o identidades.📊 Diagrama de Entidades y Relaciones (ERD)erDiagram
    IDENTIDADES ||--o{ USUARIOS_SISTEMA : "es_actor_de"
    IDENTIDADES ||--o{ LIBRO_DIARIO : "responsable_de"
    IDENTIDADES ||--o{ CONTRATOS_AGUA : "posee"
    IDENTIDADES ||--o{ ESTADOS_MEDIDOR : "lector_de"

    PLAN_CUENTAS ||--o{ DETALLES_ASIENTO : "imputa_a"
    PLAN_CUENTAS ||--o{ OPERACIONES_FINANCIERAS : "vinculo_unico"
    LIBRO_DIARIO ||--|{ DETALLES_ASIENTO : "contiene"
    OPERACIONES_FINANCIERAS ||--o{ DETALLES_ASIENTO : "afecta_saldo"
    OPERACIONES_FINANCIERAS ||--o{ HISTORICO_CIERRES : "genera_snapshot"
    
    OPERACIONES_FINANCIERAS ||--o| SATELITE_CARTERA : "extension_datos"
    OPERACIONES_FINANCIERAS ||--o| SATELITE_BILLETERA : "extension_datos"
    IDENTIDADES ||--o{ SATELITE_CARTERA : "pertenece_a"
    IDENTIDADES ||--o{ SATELITE_BILLETERA : "pertenece_a"

    CONTRATOS_AGUA ||--o{ ESTADOS_MEDIDOR : "tiene_lecturas"
    CICLOS_FACTURACION ||--o{ ESTADOS_MEDIDOR : "agrupa_lecturas"
    CICLOS_FACTURACION ||--o{ EMISIONES_ESTADO_CUENTA : "genera"
    LIBRO_DIARIO ||--o| ESTADOS_MEDIDOR : "contabiliza"
    IDENTIDADES ||--o| ESTADO_MOROSIDAD_CLIENTE : "monitoreo"

    PLANTILLAS_IMPRESION ||--o{ ARCHIVO_DOCUMENTAL_INMUTABLE : "define_formato"
    ARCHIVO_DOCUMENTAL_INMUTABLE ||--o{ METADATOS_DOCUMENTO : "extiende_info"
    ARCHIVO_DOCUMENTAL_INMUTABLE ||--o{ TRAZABILIDAD_DOCUMENTAL : "bitacora_estados"
    ARCHIVO_DOCUMENTAL_INMUTABLE ||--o{ EMISIONES_ESTADO_CUENTA : "es_respaldo_de"
    ARCHIVO_DOCUMENTAL_INMUTABLE ||--o| SATELITE_ACUERDOS_PAGO : "es_contrato_de"
    
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
🚀 Hoja de Ruta de Despliegue (Migraciones)El sistema sigue una estructura de despliegue secuencial obligatoria para garantizar la integridad de las dependencias:01_base/: Definición de esquemas, tipos enumerados (Enums) y funciones auxiliares core.02_tablas/: Creación de tablas por módulos (Identidad -> Financiero -> Operación -> Documentos).03_conexiones/: Inyección de llaves foráneas y triggers de integración cruzada.📸 Capturas de Pantalla del SistemaVisualización de KPIs de recaudación diaria y estados de cartera.Explorador de archivos inmutables y trazabilidad de estados.© 2026 Jefferson - Todos los derechos reservados.
