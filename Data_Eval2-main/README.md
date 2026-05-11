# Base de Datos - MySQL

## Descripción
Base de datos MySQL para el sistema de gestión de usuarios. Incluye scripts SQL completos para la creación, mantenimiento y backup de la base de datos.

## Versiones y Herramientas Requeridas

### Motor de Base de Datos
- **MySQL**: Versión 8.0 o superior
- **MariaDB**: Versión 10.4 o superior (compatible)

### Herramientas de Administración
- **MySQL Client**: Para ejecución de scripts desde línea de comandos
- **phpMyAdmin**: Opcional, para administración web
- **MySQL Workbench**: Opcional, para diseño y administración gráfica

## Estructura de Archivos

```
database/
├── 01_creacion_base_datos.sql    # Script principal de creación
├── 02_backup_y_mantenimiento.sql # Scripts de mantenimiento
└── README.md                     # Este archivo
```

## Instalación y Configuración

### 1. Instalar MySQL Server
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install mysql-server

# CentOS/RHEL
sudo yum install mysql-server

# Windows
# Descargar desde https://dev.mysql.com/downloads/mysql/
```

### 2. Configurar MySQL
```bash
# Iniciar servicio MySQL
sudo systemctl start mysql
sudo systemctl enable mysql

# Configurar seguridad (recomendado)
sudo mysql_secure_installation
```

### 3. Crear Base de Datos
```bash
# Ejecutar script de creación
mysql -u root -p < 01_creacion_base_datos.sql

# O desde MySQL console:
mysql -u root -p
source 01_creacion_base_datos.sql;
```

## Esquema de la Base de Datos

### Tabla Principal: `usuarios`

| Columna | Tipo | Nulo | Default | Descripción |
|---------|------|------|---------|-------------|
| `id` | INT AUTO_INCREMENT | No | - | ID único del usuario (PK) |
| `nombre` | VARCHAR(100) | No | - | Nombre completo del usuario |
| `email` | VARCHAR(150) | No | - | Email único del usuario |
| `edad` | INT | Sí | NULL | Edad del usuario (opcional) |
| `fecha_creacion` | TIMESTAMP | No | CURRENT_TIMESTAMP | Fecha de creación |
| `fecha_actualizacion` | TIMESTAMP | No | CURRENT_TIMESTAMP ON UPDATE | Última actualización |
| `estado` | ENUM('activo','inactivo') | No | 'activo' | Estado del usuario |

### Índices
- `PRIMARY KEY` en `id`
- `UNIQUE INDEX` en `email`
- `INDEX` en `nombre`
- `INDEX` en `estado`
- `INDEX` en `fecha_creacion`
- `INDEX COMPUESTO` en `nombre, estado`

### Vistas Disponibles
- `vista_usuarios_activos`: Usuarios con estado 'activo'
- `vista_estadisticas_usuarios`: Estadísticas básicas por fecha

## Comandos Básicos

### Conexión a la Base de Datos
```bash
# Conectar como root
mysql -u root -p

# Conectar a la base de datos específica
mysql -u root -p proyecto_db
```

### Consultas Útiles
```sql
-- Ver todas las tablas
SHOW TABLES;

-- Describir estructura de tabla
DESCRIBE usuarios;

-- Ver todos los usuarios
SELECT * FROM usuarios;

-- Ver usuarios activos
SELECT * FROM vista_usuarios_activos;

-- Ver estadísticas
SELECT * FROM vista_estadisticas_usuarios;
```

## Procedimientos Almacenados

### `sp_obtener_usuario_por_id(id)`
Obtiene información completa de un usuario por su ID.

```sql
CALL sp_obtener_usuario_por_id(1);
```

### `sp_crear_usuario(nombre, email, edad, estado)`
Crea un nuevo usuario y retorna su ID.

```sql
CALL sp_crear_usuario('Nuevo Usuario', 'nuevo@ejemplo.com', 25, 'activo');
```

### `sp_limpiar_usuarios_inactivos(dias)`
Elimina usuarios inactivos con más de X días de antigüedad.

```sql
CALL sp_limpiar_usuarios_inactivos(90);
```

### `sp_actualizar_estadisticas()`
Muestra estadísticas actualizadas del sistema.

```sql
CALL sp_actualizar_estadisticas();
```

## Backup y Restauración

### Backup Completo
```bash
# Backup con fecha
mysqldump -u root -p --single-transaction --routines --triggers proyecto_db > backup_$(date +%Y%m%d_%H%M%S).sql

# Backup comprimido
mysqldump -u root -p --single-transaction --routines --triggers proyecto_db | gzip > backup_$(date +%Y%m%d_%H%M%S).sql.gz
```

### Backup Selectivo
```bash
# Solo datos
mysqldump -u root -p --no-create-info --single-transaction proyecto_db > backup_datos.sql

# Solo estructura
mysqldump -u root -p --no-data --routines --triggers proyecto_db > backup_estructura.sql

# Tabla específica
mysqldump -u root -p --single-transaction proyecto_db usuarios > backup_usuarios.sql
```

### Restauración
```bash
# Restaurar backup completo
mysql -u root -p proyecto_db < backup_20240430_120000.sql

# Restaurar desde archivo comprimido
gunzip < backup_20240430_120000.sql.gz | mysql -u root -p proyecto_db
```

## Mantenimiento

### Optimización Periódica
```sql
-- Optimizar tabla
OPTIMIZE TABLE usuarios;

-- Actualizar estadísticas
ANALYZE TABLE usuarios;

-- Verificar integridad
CHECK TABLE usuarios;
```

### Monitoreo
```sql
-- Ver tamaño de la base de datos
SELECT 
    table_schema as 'Base de Datos',
    ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Tamaño (MB)'
FROM information_schema.tables
WHERE table_schema = 'proyecto_db'
GROUP BY table_schema;

-- Ver conexiones activas
SHOW PROCESSLIST;

-- Ver estado del servidor
SHOW STATUS;
```

## Puertos Requeridos

### Para funcionamiento en contenedor:
- **Puerto 3306**: Puerto estándar de MySQL para conexiones cliente-servidor

### Explicación de puertos:
- **3306**: Es el puerto por defecto donde MySQL escucha conexiones TCP/IP desde clientes externos

## Configuración de Red

### Acceso Remoto
```sql
-- Crear usuario para acceso remoto
CREATE USER 'app_user'@'%' IDENTIFIED BY 'contraseña_segura';
GRANT SELECT, INSERT, UPDATE, DELETE ON proyecto_db.* TO 'app_user'@'%';
FLUSH PRIVILEGES;
```

### Configuración de MySQL Server
En `/etc/mysql/mysql.conf.d/mysqld.cnf` (Linux) o my.ini (Windows):
```ini
[mysqld]
# Permitir conexiones desde cualquier IP
bind-address = 0.0.0.0

# Puerto personalizado (opcional)
port = 3306

# Configuración de caracteres
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
```

## Variables de Entorno para Aplicaciones

| Variable | Descripción | Valor por Defecto |
|----------|-------------|-------------------|
| `DB_HOST` | Host del servidor MySQL | localhost |
| `DB_PORT` | Puerto de MySQL | 3306 |
| `DB_USER` | Usuario de la base de datos | root |
| `DB_PASSWORD` | Contraseña del usuario | (tu contraseña) |
| `DB_NAME` | Nombre de la base de datos | proyecto_db |

## Seguridad

### Buenas Prácticas
1. **No usar root en producción**: Crear usuarios específicos para cada aplicación
2. **Contraseñas seguras**: Usar contraseñas complejas y rotarlas periódicamente
3. **Acceso limitado**: Configurar firewall para permitir solo IPs necesarias
4. **Backups regulares**: Programar backups automáticos diarios
5. **Auditoría**: Habilitar logs de consultas si es necesario

### Configuración SSL (Opcional)
```sql
-- Requerir SSL para conexiones
CREATE USER 'secure_user'@'%' IDENTIFIED BY 'contraseña' REQUIRE SSL;
GRANT SELECT, INSERT, UPDATE, DELETE ON proyecto_db.* TO 'secure_user'@'%';
```

## Troubleshooting

### Problemas Comunes

#### Error de conexión
```bash
# Verificar que MySQL está corriendo
sudo systemctl status mysql

# Verificar puerto
netstat -tlnp | grep 3306

# Revisar logs
sudo tail -f /var/log/mysql/error.log
```

#### Error de permisos
```sql
-- Verificar permisos del usuario
SHOW GRANTS FOR 'app_user'@'%';

-- Otorgar permisos necesarios
GRANT ALL PRIVILEGES ON proyecto_db.* TO 'app_user'@'%';
FLUSH PRIVILEGES;
```

#### Problemas con caracteres
```sql
-- Verificar configuración de caracteres
SHOW VARIABLES LIKE 'character_set%';
SHOW VARIABLES LIKE 'collation%';
```

## Notas Importantes
- Este diseño está optimizado para el proyecto específico de gestión de usuarios
- Los scripts incluyen datos de ejemplo para facilitar las pruebas iniciales
- Se recomienda ejecutar los scripts en orden: primero `01_creacion_base_datos.sql`
- El script `02_backup_y_mantenimiento.sql` es opcional pero recomendado para producción
