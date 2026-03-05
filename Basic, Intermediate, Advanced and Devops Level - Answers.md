# Apache Tomcat — Complete Interview & Production Guide

---

# 🟢 LEVEL 1 — Beginner (Basic Understanding)

---

## 1️⃣ What is Apache Tomcat?

**Apache Tomcat** is an open-source **Java Servlet and JSP container**.

It:

* Runs Java web applications
* Executes Servlets
* Processes JSP files
* Converts JSP → Servlet → HTML response

In simple terms:

> Tomcat is a Java application server used to run `.war` files.

---

## 2️⃣ What is the difference between Web Server and Application Server?

| Web Server                                    | Application Server      |
| --------------------------------------------- | ----------------------- |
| Serves static content (HTML, CSS, JS, images) | Runs business logic     |
| Handles HTTP requests                         | Handles dynamic content |
| Example: NGINX                                | Example: Apache Tomcat  |
| Fast for static files                         | Runs Java applications  |

### Simple Example:

```
Browser → Nginx → Tomcat → Database
```

* Nginx serves static content
* Tomcat runs Java code

---

## 3️⃣ What is a WAR file?

WAR = **Web Application Archive**

It is a compressed file like `.zip` that contains:

```
.war
 ├── WEB-INF/
 │    ├── web.xml
 │    ├── classes/
 │    └── lib/
 ├── JSP files
 ├── HTML
 └── Static files
```

When you place a WAR file inside `webapps/`, Tomcat automatically extracts and deploys it.

---

## 4️⃣ Default Tomcat Port?

Default HTTP port: **8080**

Access URL:

```
http://localhost:8080
```

Defined inside:

```
conf/server.xml
```

---

## 5️⃣ What is CATALINA_HOME vs CATALINA_BASE?

### 🔹 CATALINA_HOME

* Location where Tomcat is installed
* Contains original binaries
* Example: `/opt/tomcat`

### 🔹 CATALINA_BASE

* Runtime instance directory
* Contains: `logs/`, `webapps/`, `temp/`, `conf/`

> 👉 You can run multiple Tomcat instances using one `CATALINA_HOME` but different `CATALINA_BASE`.

---

## 6️⃣ What is server.xml used for?

File: `conf/server.xml`

It controls:

* Port numbers
* Connectors (HTTP, AJP)
* SSL configuration
* Thread settings
* Engine configuration

Example:

```xml
<Connector port="8080" protocol="HTTP/1.1"/>
```

> 👉 This file controls how Tomcat listens for requests.

---

## 7️⃣ What is web.xml?

Location: `WEB-INF/web.xml`

It is called the **Deployment Descriptor**. It defines:

* Servlets
* Servlet mappings
* Welcome file
* Session timeout
* Filters

Example:

```xml
<servlet>
   <servlet-name>Hello</servlet-name>
   <servlet-class>com.app.HelloServlet</servlet-class>
</servlet>
```

> 👉 It tells Tomcat how to run your application.

---

## 8️⃣ What is a Context in Tomcat?

A **Context** represents a web application inside Tomcat.

Example — if WAR name is `BusBooking.war`, URL becomes:

```
http://localhost:8080/BusBooking
```

That `/BusBooking` is called the **Context Path**.

Context can be configured in:

* `conf/context.xml`
* `conf/Catalina/localhost/appname.xml`

---

## 9️⃣ What is the purpose of conf/ folder?

| File               | Purpose                       |
| ------------------ | ----------------------------- |
| server.xml         | Port and connector settings   |
| web.xml            | Default servlet configuration |
| context.xml        | Default context settings      |
| tomcat-users.xml   | Manager app users             |
| logging.properties | Log configuration             |

---

## 🔟 How to start and stop Tomcat?

Go to `bin/` directory.

**Start:**

```bash
# Linux
./startup.sh

# Windows
startup.bat
```

**Stop:**

```bash
# Linux
./shutdown.sh

# Windows
shutdown.bat
```

**Check Tomcat status:**

```bash
ps -ef | grep tomcat
```

**Check logs:**

```bash
tail -f logs/catalina.out
```

---

# 🟡 LEVEL 2 — Intermediate

---

## 1️⃣ What is a Connector in Tomcat?

A **Connector** is the component that:

* Listens for incoming requests
* Accepts HTTP/AJP traffic
* Passes requests to Tomcat Engine

Defined in `conf/server.xml`:

```xml
<Connector port="8080" protocol="HTTP/1.1"
           maxThreads="200"
           connectionTimeout="20000" />
```

### Types of Connectors:

1. **HTTP Connector** → Browser → Tomcat
2. **HTTPS Connector** → Secure traffic
3. **AJP Connector** → Used with Apache web server

---

## 2️⃣ Difference Between HTTP and AJP Connector?

| HTTP                      | AJP                                      |
| ------------------------- | ---------------------------------------- |
| Used directly by browsers | Used between web server & Tomcat         |
| Port 8080                 | Port 8009                                |
| Text-based                | Binary protocol                          |
| Common                    | Less used now (security risk if exposed) |

> ⚠️ In production, AJP must be disabled unless required.

---

## 3️⃣ What is JNDI?

JNDI = **Java Naming and Directory Interface**

It allows storing DB configuration in Tomcat, avoiding hardcoded credentials.

❌ Bad practice:

```java
DriverManager.getConnection("jdbc:mysql://...", "user", "pass");
```

✅ Using JNDI:

```java
Context ctx = new InitialContext();
DataSource ds = (DataSource) ctx.lookup("java:comp/env/jdbc/mydb");
```

---

## 4️⃣ How Tomcat Connects to MySQL/MariaDB?

**Steps:**

1. Copy MySQL connector JAR into `lib/`

2. Configure in `conf/context.xml`:

```xml
<Resource name="jdbc/mydb"
          auth="Container"
          type="javax.sql.DataSource"
          maxTotal="100"
          maxIdle="30"
          username="dbuser"
          password="dbpass"
          driverClassName="com.mysql.cj.jdbc.Driver"
          url="jdbc:mysql://localhost:3306/appdb"/>
```

3. Use JNDI in Java code.

---

## 5️⃣ What is a DataSource?

A **DataSource** is a database connection pool.

Instead of opening new DB connections every time, Tomcat creates a pool of reusable connections.

Benefits:

* Faster performance
* Avoids DB overload
* Reuses connections

Important parameters: `maxTotal`, `maxIdle`, `maxWaitMillis`

---

## 6️⃣ How Class Loading Works in Tomcat?

Tomcat uses **Hierarchical ClassLoader**:

1. Bootstrap (JVM classes)
2. System
3. Common (`Tomcat lib/`)
4. Webapp (`WEB-INF/lib`)

> 👉 Each webapp has its own classloader — prevents library conflicts and version clashes.

---

## 7️⃣ What is context.xml?

File: `conf/context.xml`

It defines:

* Global DataSource
* Resource configuration
* Session settings

App-specific context: `conf/Catalina/localhost/appname.xml`

---

## 8️⃣ What is Session Management in Tomcat?

When a user logs in:

* Tomcat creates a session
* Generates Session ID
* Stores it in memory
* Sends `JSESSIONID` cookie

```
Set-Cookie: JSESSIONID=ABC123XYZ
```

Session storage options:

* In memory (default)
* Redis (advanced)
* Database (clustering)

---

## 9️⃣ What is catalina.out?

File: `logs/catalina.out`

Contains:

* Application errors
* Stack traces
* Startup logs
* Console output

```bash
tail -f logs/catalina.out
```

---

## 🔟 How to Enable HTTPS in Tomcat?

**Step 1 — Generate Keystore:**

```bash
keytool -genkey -alias tomcat -keyalg RSA -keystore keystore.jks
```

**Step 2 — Add HTTPS Connector in server.xml:**

```xml
<Connector port="8443"
           protocol="org.apache.coyote.http11.Http11NioProtocol"
           SSLEnabled="true"
           maxThreads="150">
    <SSLHostConfig>
        <Certificate certificateKeystoreFile="conf/keystore.jks"
                     certificateKeystorePassword="password"
                     type="RSA" />
    </SSLHostConfig>
</Connector>
```

Access: `https://localhost:8443`

> 🔥 **Production Tip:** In real production, use Nginx for SSL and keep Tomcat on internal HTTP.
>
> ```
> User → Nginx (SSL) → Tomcat (HTTP)
> ```

---

# 🟠 LEVEL 3 — Advanced (Performance & Architecture)

---

## 1️⃣ Thread Tuning in Apache Tomcat

Tomcat uses a **thread pool** to handle requests (each request = 1 thread).

Configured in `conf/server.xml`:

```xml
<Connector port="8080"
           protocol="HTTP/1.1"
           maxThreads="200"
           minSpareThreads="25"
           acceptCount="100"
           connectionTimeout="20000" />
```

### Important Thread Parameters

| Parameter         | Description                                              |
| ----------------- | -------------------------------------------------------- |
| `maxThreads`      | Maximum request processing threads (e.g., `300`)        |
| `minSpareThreads` | Minimum idle threads always available                    |
| `acceptCount`     | Queue size when all threads are busy                     |
| `connectionTimeout` | Time to wait for client request before closing         |

> If `maxThreads=200` and `acceptCount=100`, max capacity = 300. Beyond that, requests are rejected.

---

## 2️⃣ Performance Tuning (Real Production)

### Enable Compression

```xml
compression="on"
compressionMinSize="1024"
compressableMimeType="text/html,text/xml,text/plain,text/css,text/javascript,application/json"
```

### JVM Memory Tuning

Set in `setenv.sh`:

```bash
export CATALINA_OPTS="-Xms2g -Xmx2g -XX:+UseG1GC"
```

| Flag   | Description   |
| ------ | ------------- |
| `-Xms` | Initial heap  |
| `-Xmx` | Maximum heap  |

### Monitor Threads

```bash
ps -eLf | grep tomcat | wc -l
```

Or use **VisualVM** / **JConsole**.

---

## 3️⃣ Memory Leak Troubleshooting

**Symptoms:** Memory increasing, frequent GC, `OutOfMemoryError`

**Generate heap dump:**

```bash
jmap -dump:live,format=b,file=heap.hprof <PID>
```

**Analyze using:** Eclipse MAT

**Common causes:**

* Static variables
* Unclosed DB connections
* Large session objects

---

## 4️⃣ Clustering in Tomcat

```
User → Load Balancer → Tomcat 1 / Tomcat 2
```

If Tomcat 1 crashes, Tomcat 2 handles the user.

### Session Replication Options

| Method          | Description                              |
| --------------- | ---------------------------------------- |
| `DeltaManager`  | Replicates all sessions to all nodes     |
| `BackupManager` | Replicates to one backup node only       |

```xml
<Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"/>
```

---

## 5️⃣ Sticky vs Non-Sticky Sessions

### Sticky Session (Recommended)

Load balancer always sends user to same Tomcat.

```nginx
ip_hash;
```

Less overhead, better performance.

### Non-Sticky Session

User may hit different Tomcat every request. Requires session replication or Redis.

---

## 6️⃣ High CPU Troubleshooting

```bash
# Check CPU
top

# Find Tomcat PID
ps -ef | grep tomcat

# Thread dump
jstack <PID> > thread.txt
```

Check for: deadlocks, blocked threads, infinite loops.

---

## 7️⃣ DB Connection Pool Exhausted

**Error:**

```
Cannot get connection
Timeout waiting for idle object
```

**Causes:** `maxTotal` too low, slow DB, connections not closed.

**Fix:** Increase pool size, optimize queries, close connections properly.

---

## 8️⃣ Access Log Configuration

```xml
<Valve className="org.apache.catalina.valves.AccessLogValve"
       directory="logs"
       prefix="access"
       suffix=".log"
       pattern="%h %l %u %t &quot;%r&quot; %s %b" />
```

---

## 9️⃣ Running Multiple Tomcat Instances

* Same `CATALINA_HOME`
* Different `CATALINA_BASE`
* Different ports (e.g., `8080` → App1, `9090` → App2)

---

## 🔟 Production Best Practice Architecture

```
Client
   ↓
Nginx (SSL)
   ↓
Tomcat (HTTP)
   ↓
MySQL / MariaDB
   ↓
Redis (Session cache)
```

---

# 🔴 LEVEL 4 — Production Architecture

---

## 1️⃣ Real Production Architecture

```
Client (Browser)
        ↓
Load Balancer / Nginx (SSL Termination)
        ↓
Tomcat-1        Tomcat-2
        ↓
Redis (Session)
        ↓
MySQL Primary → MySQL Replica
```

### Components

| Component        | Role                                                                 |
| ---------------- | -------------------------------------------------------------------- |
| Nginx            | SSL termination, load balancing, rate limiting, static file serving  |
| Apache Tomcat    | Application layer — run as non-root, disable AJP, internal HTTP only |
| Redis            | External session store for horizontal scaling                        |
| MySQL/MariaDB    | Primary (write) + Replica (read) with backups enabled                |

---

## 2️⃣ Load Balancing with Nginx

```nginx
upstream tomcat_cluster {
    server 10.0.0.11:8080;
    server 10.0.0.12:8080;
}

server {
    listen 443 ssl;
    server_name example.com;

    location / {
        proxy_pass http://tomcat_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

**Sticky session:**

```nginx
upstream tomcat_cluster {
    ip_hash;
    server 10.0.0.11:8080;
    server 10.0.0.12:8080;
}
```

---

## 3️⃣ Zero Downtime Deployment 🔥

### Method 1: Rolling Deployment (Recommended)

1. Remove Tomcat-2 from Nginx upstream
2. Deploy new WAR to Tomcat-2
3. Start Tomcat-2
4. Add Tomcat-2 back to Nginx
5. Repeat for Tomcat-1

✔ No downtime | ✔ No session loss (if sticky)

### Method 2: Blue-Green Deployment

1. Deploy to Green environment
2. Test internally
3. Switch Nginx to Green
4. Keep Blue as backup

```bash
nginx -s reload  # instant switch
```

### Method 3: Canary Deployment

* Send 10% traffic to new version
* Monitor logs
* Increase gradually

---

## 4️⃣ Health Checks

```nginx
location /health {
    proxy_pass http://tomcat_cluster/health;
}
```

Application must expose `/health` returning `200 OK`. Load balancer removes unhealthy nodes automatically.

---

## 5️⃣ Production Monitoring

| Tool           | Purpose                          |
| -------------- | -------------------------------- |
| Prometheus     | Metrics collection               |
| Grafana        | Visualization                    |
| JMX Exporter   | Tomcat threads, heap, GC, CPU    |
| Node Exporter  | System-level metrics             |

---

## 6️⃣ Backup Strategy

**Application backup:** WAR files, config files, systemd service file

**Database backup:**

```bash
mysqldump -u root -p appdb > backup.sql
```

Automate with cron.

---

## 7️⃣ Production Security Hardening

| Layer    | Action                                                    |
| -------- | --------------------------------------------------------- |
| Tomcat   | Remove manager app, disable AJP, hide server version, run as `tomcat` user |
| Nginx    | SSL only, strong ciphers, rate limiting, block bots       |
| Firewall | Allow `443` (public), `22` (admin), `8080` internal only  |

---

## 8️⃣ Handling Production Failures

### 🔴 502 Bad Gateway

Check: Is Tomcat running? Port listening? Firewall blocking? MaxThreads exhausted?

### 🔴 High CPU

```bash
top
jstack PID
```

Look for: infinite loops, blocked threads, heavy GC.

### 🔴 OutOfMemoryError

```bash
# Increase heap
-Xmx

# Take heap dump
jmap -dump:live,format=b,file=heap.hprof PID
```

Analyze using **Eclipse MAT**.

---

## 9️⃣ CI/CD Integration

```
Developer → Git → Jenkins/GitHub Actions
        ↓
Build WAR
        ↓
Deploy to Staging
        ↓
Production Rolling Deployment
```

---

## 🔟 Scaling Strategy

| Type                | Method                                       |
| ------------------- | -------------------------------------------- |
| Vertical Scaling    | Increase RAM, CPU, Heap                      |
| Horizontal Scaling  | Add more Tomcat nodes + load balancer        |

> ✅ Best practice: **Horizontal scaling**

---

# 🎯 Skills Summary by Level

| Level    | Key Skills                                                                              |
| -------- | --------------------------------------------------------------------------------------- |
| Level 1  | Deploy WAR, understand config files, start/stop Tomcat                                  |
| Level 2  | Configure DB connection, setup HTTPS, use JNDI, connect Nginx → Tomcat, debug logs     |
| Level 3  | Tune threads, fix high CPU, analyze heap dumps, configure clustering & sticky sessions  |
| Level 4  | Design production architecture, zero-downtime deploy, monitoring, security, scaling     |
