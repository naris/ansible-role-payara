# Ansible Role: Payara

This role installs and configures the Payara application server (<http://www.payara.fish/>) with customizable admin credentials, domain configuration, database connections, network listeners, and systemd service integration.

## Requirements

A Java Development Kit is required based on Payara version:

- Payara 4: JDK 8
- Payara 5: JDK 8 or JDK 11
- Payara 6: JDK 11, JDK 17, or JDK 21
- Payara 7: JDK 17 or JDK 21

You can use this ansible role: <https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/java/>

## Role Variables

### Variable Schema Reference

#### `payara_connection_pools` item

| Field | Required | Notes |
| --- | --- | --- |
| `name` | Yes | Connection pool name |
| `classname` | Yes | JDBC datasource class |
| `db.user` | Yes | Database username |
| `db.password` | Yes | Database password |
| `db.url` | Conditional | Use this **or** `db.host` + `db.port` |
| `db.host` | Conditional | Required when `db.url` is not set |
| `db.port` | Conditional | Required when `db.url` is not set |

#### `payara_jdbc_resources` item

| Field | Required | Notes |
| --- | --- | --- |
| `name` | Yes | JNDI name (for example `jdbc/MyDS`) |
| `connection_pool_id` | Yes | Existing pool name |

#### `payara_threadpools` item

| Field | Required | Notes |
| --- | --- | --- |
| `name` | Yes | Thread pool name |
| `maxthreadpoolsize` | No | Default `200` |
| `minthreadpoolsize` | No | Default `2` |
| `maxqueuesize` | No | Default `4096` |

#### `payara_listeners` item

| Field | Required | Notes |
| --- | --- | --- |
| `name` | Yes | Listener name |
| `address` | Yes | Listener address |
| `port` | Yes | Listener port |
| `jkenabled` | No | When `true`, creates a network listener |
| `protocol` | Conditional | Required for JK-enabled listener |
| `default_virtual_server` | No | Default `server` |
| `target` | No | Default `server` |
| `threadpool` | No | Default `http-thread-pool` |
| `transport` | No | Default `tcp` |
| `securityenabled` | No | Protocol option, default `false` |

#### `payara_virtual_servers` item

| Field | Required | Notes |
| --- | --- | --- |
| `name` | Yes | Virtual server name |
| `hosts` | No | Default `localhost` |
| `networklisteners` | No | Default `http-listener-1,http-listener-2` |

#### `payara_deploy_packages` item

| Field | Required | Notes |
| --- | --- | --- |
| `name` | Yes | Application name |
| `package` | Yes | Archive location for `deploy-remote-archive` |
| `contextroot` | No | Default `/` |
| `target` | No | Default `server` |
| `virtual` | No | Default `server` |

#### `payara_managed_executor_services` item

| Field | Required | Notes |
| --- | --- | --- |
| String value | Yes | Service name passed to `create-managed-executor-service` |

### Installation Variables

- **payara_release**: version of the product (Payara), _6.2024.12_ by default.
- **payara_glassfish_version**: major version of the product (Payara), _6_ by default.
- **payara_glassfish_release**: minor version of the product (Payara), if any.
- **payara_multilanguage**: Indicates whether or not to install a Payara Multi-Language (ML) distribution, _false_ by default, ml is not available for micro or minimal distributions.
- **payara_distribution**: Indicates what distribution to install (full, web, micro or minimal), _full_ by default, _minimal_ is only available for payara 4.
- **payara_download_base**: base url to download the product (Payara) from, _<https://repo1.maven.org/maven2/fish/payara>_ by default.
- **payara_url**: url to download the product (Payara) from, built from _payara_download_base_, _payara_distribution_ & _payara_multilanguage_ values.
- **payara_user**: user on the system to own the payara install, _payara_ by default.
- **payara_group**: group on the system to own the payara install, _payara_ by default.
- **payara_base**: base directory to install the product (Payara) into, _/opt/payara_ by default.
- **payara_home**: full directory to install the product (Payara) into, _\{PAYARA_BASE\}/payara\{VERSION\}\{RELEASE\}_ by default.

### Domain Configuration

- **payara_host**: _0.0.0.0_ by default.
- **payara_portbase**: the number with which port assignments should start, _4800_ by default.
- **payara_admin_port**: the payara administration port, _4848_ by default.
- **payara_admin_user**: _admin_ by default.
- **payara_admin_password**: admin password, empty by default.
- **payara_domain_name**: _domain1_ by default.

### Database Configuration

- **payara_jdbc_drivers**: dictionary of JDBC drivers to download and install. Key is the driver name, value is the download URL.
- **payara_connection_pools**: list of JDBC connection pools to create. Each pool requires:
  - `name`: connection pool name
  - `classname`: datasource class name
  - `db.user`: database username
  - `db.password`: database password
  - `db.url`: database URL (optional, if not provided uses ServerName/port)
  - `db.host`: database host (required if url not provided)
  - `db.port`: database port (required if url not provided)
- **payara_jdbc_resources**: list of JDBC resources to create. Each resource requires:
  - `name`: JNDI name
  - `connection_pool_id`: connection pool to use

### Network Configuration

- **payara_threadpools**: list of thread pools to create. Each pool supports:
  - `name`: thread pool name
  - `maxthreadpoolsize`: maximum threads (default: 200)
  - `minthreadpoolsize`: minimum threads (default: 2)
  - `maxqueuesize`: maximum queue size (default: 4096)
- **payara_listeners**: list of network listeners to create. Each listener requires:
  - `name`: listener name
  - `address`: listener address
  - `port`: listener port
  - `target`: Payara target (default: server)
  - `default_virtual_server`: virtual server (default: server, only for non-JK listeners)
  - `protocol`: protocol name (required for JK-enabled listeners, auto-created with JK support)
  - `threadpool`: thread pool name (default: http-thread-pool)
  - `transport`: transport type (default: tcp)
  - `securityenabled`: protocol security flag when creating protocol (default: false)
  - `jkenabled`: set to true to create as network listener with JK support and auto-create protocol
- **payara_virtual_servers**: list of virtual servers to create. Each server supports:
  - `name`: virtual server name
  - `hosts`: hostnames (default: localhost)
  - `networklisteners`: comma-separated listener list (default: http-listener-1,http-listener-2)

### Concurrency Configuration

- **payara_managed_executor_services**: list of managed executor service names to create.

### Deployment Configuration

- **payara_deploy_packages**: list of remote archives to deploy. Each package supports:
  - `name`: application name
  - `package`: path/URL accepted by `deploy-remote-archive`
  - `contextroot`: web context root (default: /)
  - `target`: Payara target (default: server)
  - `virtual`: virtual server name(s) (default: server)

### Service Configuration

- **payara_systemd**: Indicates whether or not to install Payara as a systemd service, _true_ by default.
- **payara_start**: Indicates whether or not to start the Payara systemd service, _true_ by default.

## Dependencies

A Java Development Kit is required based on Payara version (see Requirements section). Check for java galaxy roles like: _geerlingguy.java_

## Example Playbook

### Basic Installation

```yaml
- hosts: all
  roles:
    - role: geerlingguy.java
      java_packages:
        - java-17-openjdk-devel
    - role: naris.ansible-role-payara
```

### Advanced Configuration with Database and Network

```yaml
- hosts: all
  vars:
    payara_admin_password: "secure_password"
    payara_jdbc_drivers:
      postgresql: "https://jdbc.postgresql.org/download/postgresql-42.6.0.jar"
    payara_connection_pools:
      - name: "MyAppPool"
        classname: "org.postgresql.ds.PGSimpleDataSource"
        db:
          user: "dbuser"
          password: "dbpass"
          host: "localhost"
          port: 5432
    payara_jdbc_resources:
      - name: "jdbc/MyAppDS"
        connection_pool_id: "MyAppPool"
    payara_threadpools:
      - name: "custom-thread-pool"
        maxthreadpoolsize: 300
        minthreadpoolsize: 5
    payara_listeners:
      - name: "http-listener"
        address: "0.0.0.0"
        port: 8080
        target: "server"
        default_virtual_server: "server"
      - name: "jk-listener"
        address: "0.0.0.0"
        port: 8009
        target: "server"
        protocol: "jk-listener-protocol"
        threadpool: "custom-thread-pool"
        jkenabled: true
    payara_virtual_servers:
      - name: "apps"
        hosts: "localhost"
        networklisteners: "http-listener,jk-listener"
    payara_managed_executor_services:
      - "concurrent/MyExecutor"
    payara_deploy_packages:
      - name: "myapp"
        package: "https://artifacts.example.com/myapp.war"
        contextroot: "/myapp"
        target: "server"
        virtual: "apps"
  roles:
    - role: geerlingguy.java
      java_packages:
        - java-17-openjdk-devel
    - role: naris.ansible-role-payara
```

### MySQL Database Configuration

```yaml
- hosts: all
  vars:
    payara_jdbc_drivers:
      mysql: "https://repo1.maven.org/maven2/com/mysql/mysql-connector-j/8.2.0/mysql-connector-j-8.2.0.jar"
    payara_connection_pools:
      - name: "MySQLPool"
        classname: "com.mysql.cj.jdbc.MysqlDataSource"
        db:
          user: "myuser"
          password: "mypassword"
          host: "mysql.example.com"
          port: 3306
    payara_jdbc_resources:
      - name: "jdbc/MySQLDS"
        connection_pool_id: "MySQLPool"
  roles:
    - role: geerlingguy.java
      java_packages:
        - java-17-openjdk-devel
    - role: naris.ansible-role-payara
```

### Oracle Database Configuration

```yaml
- hosts: all
  vars:
    payara_jdbc_drivers:
      ojdbc17: "https://repo1.maven.org/maven2/com/oracle/database/jdbc/ojdbc17/23.26.1.0.0/ojdbc17-23.26.1.0.0.jar"
    payara_connection_pools:
      - name: "OraclePool"
        classname: "oracle.jdbc.pool.OracleDataSource"
        db:
          user: "oracleuser"
          password: "oraclepassword"
          url: "jdbc:oracle:thin:@//oracle.example.com:1521/ORCL"
    payara_jdbc_resources:
      - name: "jdbc/OracleDS"
        connection_pool_id: "OraclePool"
  roles:
    - role: geerlingguy.java
      java_packages:
        - java-17-openjdk-devel
    - role: naris.ansible-role-payara
```

## License

MIT

## Author Information

jeqo (<quilcate.jorge@gmail.com>)
naris (<murray.wilson@gmail.com>)
