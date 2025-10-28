
### Guía Completa: Docker Ubuntu con MySQL, PostgreSQL, MongoDB y SQL Server

En el documento se explica paso a paso cómo tomar una imagen Ubuntu y convertirla en un servidor con **4 motores de bases de datos instalados y accesibles desde el exterior**.

---

### Descargar la imagen Ubuntu

```bash
docker pull ubuntu:22.04
```

Descarga la versión más reciente de Ubuntu 22.04 desde Docker Hub.

---

### Crear y ejecutar el contenedor con puertos expuestos

```bash
docker run -it --name ubuntu -p 14395:1433 -p 3395:3306 -p 5495:5432 -p 27095:27017 ubuntu:22.04 bash
```

| Puerto | Base de Datos |
|--------|----------------|
| 14395  | SQL Server     |
| 3395   | MySQL          |
| 5495   | PostgreSQL     |
| 27095  | MongoDB        |

 `-it`: Terminal interactiva  
 `--name ubuntu`: Nombre del contenedor

---

### Entrar al contenedor

```bash
docker exec -it ubuntu bash
```

---

### Preparar el sistema

```bash
apt update
apt-get update
apt install curl gnupg2 -y
apt update && apt install -y systemd systemd-sysv
apt-get install -y curl wget gnupg lsb-release software-properties-common apt-transport-https ca-certificates vim net-tools iproute2 gnupg2
apt-get update && apt-get install -y nano
```

Instala herramientas necesarias para gestionar repositorios, puertos y servicios.

---

### Instalar y configurar MySQL

```bash
apt-get install -y mysql-server
```

Editar configuración para permitir conexiones externas:

```bash
nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

 Cambiar `bind-address` a:

```
0.0.0.0
```

Reiniciar MySQL:

```bash
service mysql restart
```

Configurar usuarios:

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'juan123';
CREATE USER 'camilo'@'%' IDENTIFIED BY 'juan123';
GRANT ALL PRIVILEGES ON *.* TO 'camilo'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```

Verificar MySQL:

```bash
ps aux | grep mysqld
mysql -u camilo -p
```

---

### Instalar y configurar PostgreSQL

```bash
install -d -m 0755 /etc/apt/keyrings
curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor -o /etc/apt/keyrings/pgdg.gpg
echo "deb [signed-by=/etc/apt/keyrings/pgdg.gpg] http://apt.postgresql.org/pub/repos/apt jammy-pgdg main" > /etc/apt/sources.list.d/pgdg.list
apt-get update && apt-get install -y postgresql postgresql-contrib
```

Configurar para acceso externo:

```bash
nano /etc/postgresql/18/main/postgresql.conf
```

Cambiar:

```
listen_addresses = '*'
```

Modificar autenticación:

```bash
nano /etc/postgresql/18/main/pg_hba.conf
```

Agregar:

```
host    all    all    0.0.0.0/0    md5
```

Reiniciar:

```bash
service postgresql restart
```

Crear usuario admin:

```bash
su - postgres
psql
CREATE USER juan WITH PASSWORD 'juan123';
ALTER ROLE juan WITH SUPERUSER;
\q
```

---

### Instalar y configurar MongoDB

```bash
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | gpg --dearmor -o /usr/share/keyrings/mongodb-server-7.gpg
echo "deb [signed-by=/usr/share/keyrings/mongodb-server-7.gpg] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | tee /etc/apt/sources.list.d/mongodb-org-7.0.list
apt-get update && apt-get install -y mongodb-org
```

Configurar rutas y permisos:

```bash
mkdir -p /var/lib/mongodb /var/log/mongodb
chown -R mongodb:mongodb /var/lib/mongodb /var/log/mongodb
```

Iniciar con acceso externo:

```bash
mongod --dbpath /var/lib/mongodb --bind_ip 0.0.0.0 --logpath /var/log/mongodb/mongod.log --fork
```

Crear usuario admin:

```js
use admin
db.createUser({
  user: "juan",
  pwd: "juan123",
  roles: [{ role: "root", db: "admin" }]
})
```

Conexión:

```
mongodb://juan:juan123@localhost:27095/admin?authSource=admin
```

---

### Instalar y configurar SQL Server

```bash
apt-get update
apt-get install -y curl gnupg2 ca-certificates apt-transport-https software-properties-common
curl -sSL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor -o /etc/apt/keyrings/microsoft.gpg
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/microsoft.gpg] https://packages.microsoft.com/ubuntu/22.04/prod jammy main" > /etc/apt/sources.list.d/mssql-server.list
apt-get update
apt-get install -y mssql-server
```

Inicializar SQL Server:

```bash
/opt/mssql/bin/mssql-conf setup
```
Seleccionar:  
Licencia  2  
Password  `juan123`

Instalar cliente `sqlcmd`:

```bash
apt-get install -y mssql-tools18 unixodbc-dev
ln -s /opt/mssql-tools18/bin/sqlcmd /usr/local/bin/sqlcmd
```

Conectar desde nueva terminal:

```bash
sqlcmd -S localhost,1433 -U SA -P "juan123" -C
```

 Desde SQL Server Management Studio (SSMS)

| Campo | Valor |
|-------|------|
| Server | localhost,14395 |
| User | SA |
| Password | juan123 |
| Trust | Enabled  |

---

### Todo funcionando!!!

Tu servidor Ubuntu ahora tiene **4 bases de datos** all ready-to-go:

 MySQL  
 PostgreSQL  
 MongoDB  
 SQL Server  

---

### Conexiones externas (desde cualquier cliente)

| DB | Conexión |
|----|----------|
| MySQL | `localhost:3395` |
| PostgreSQL | `localhost:5495` |
| MongoDB | `localhost:27095` |
| SQL Server | `localhost:14395` |

---


