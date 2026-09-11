# SpeedEx Courier

A Java-based Courier Management System that streamlines the process of booking,
tracking, and managing courier/parcel deliveries through a web interface.

## 🚀 Features

- Courier booking and consignment management
- Parcel tracking
- Customer-facing pages: enquiry, complaint, carrier info, about us
- Admin zone: manage enquiries, complaints, consignments, notifications, job seekers
- MySQL-backed data storage

## 🛠️ Tech Stack

- **Language:** Java (JSP / Servlets, `javax.servlet`)
- **Build Tool:** Apache Ant / NetBeans project
- **Database:** MySQL
- **IDE:** NetBeans
- **Tested with:** Java 8, MySQL 8.0, Jetty 9 (Tomcat 8/9 also works — **not** Tomcat 10+, see note below)

## 📂 Project Structure

```
speedex_courier/
├── build/          # Compiled build output
├── dist/           # ⚠️ Stale pre-built WAR — do not use, see note below
├── nbproject/      # NetBeans project configuration
├── src/java/       # Java source (Servlets: CareerCode, DbManager, SmsSender)
├── web/            # JSP pages, static assets (CSS/JS/images) — current source
├── build.xml       # Ant build script (NetBeans-generated, requires IDE libs)
└── speeddb.sql     # MySQL database schema + seed data
```

## ⚙️ Setup & Installation

### 1. Clone the repository
```bash
git clone https://github.com/imaman4000/speedex_courier.git
cd speedex_courier
```

### 2. Set up the database
```bash
mysql -u root -p -e "CREATE DATABASE speeddb;"
mysql -u root -p speeddb < speeddb.sql
```
This creates 7 tables (`carrier`, `city`, `complain`, `consignment`, `enquiry`,
`login`, `notification`) with seed data, including a default admin login:
`userid: admin`, `password: admin@123`.

### 3. ⚠️ Fix the MySQL charset (required on MySQL 8+)
The app bundles an old MySQL Connector/J driver (`com.mysql.jdbc_5.1.5.jar`,
~2009-era) that doesn't understand MySQL 8's default `utf8mb4` charset. Without
this fix, every DB-backed page throws:
```
SQLException: Unknown initial character set index '255' received from server
```
Fix by setting your MySQL server to a legacy charset, in `my.cnf` /
`mysqld.cnf`:
```ini
[mysqld]
character-set-server=latin1
collation-server=latin1_swedish_ci
```
Then restart MySQL. (Alternative: swap in a modern Connector/J jar and add
`characterEncoding=UTF-8` to the JDBC URL in `src/java/mypack/DbManager.java`
instead of downgrading the server charset.)

### 4. Confirm DB connection settings
`DbManager.java` connects to:
```java
jdbc:mysql://localhost:3306/speeddb  (user: root, password: "")
```
Update this if your local MySQL uses a different user/password.

### 5. Build the WAR
> ⚠️ **Don't use `dist/project.war`** — it's stale and only contains 2 of the
> ~13 JSP pages in the current `web/` source (missing login, tracking, admin
> zone, etc.). Rebuild it fresh instead.

**Option A — NetBeans (recommended):** Open the project in NetBeans and use
Build/Deploy — it will resolve the IDE-specific Ant macros (`libs.CopyLibs`)
automatically, which fail if you run `ant` standalone outside the IDE.

**Option B — Manual build (no IDE):**
```bash
mkdir -p build/classes
javac -source 8 -target 8 -d build/classes \
  -cp "path/to/servlet-api.jar:path/to/jsp-api.jar:path/to/mysql-connector.jar" \
  $(find src/java -name "*.java")

mkdir -p war/WEB-INF/classes war/WEB-INF/lib
cp -r web/. war/
cp -r build/classes/. war/WEB-INF/classes/
cp path/to/mysql-connector.jar war/WEB-INF/lib/

cd war && jar cf ../speedex.war . && cd ..
```

### 6. Deploy
Deploy `speedex.war` to a servlet container. **Use Tomcat 8/9 or Jetty 9** —
this app uses the `javax.servlet` namespace (pre–Jakarta EE), which **Tomcat
10+ does not support** without a migration tool.

```bash
cp speedex.war $CATALINA_HOME/webapps/   # Tomcat
# or
cp speedex.war /var/lib/jetty9/webapps/  # Jetty 9
```

Then visit: `http://localhost:8080/speedex/`

## 📋 Requirements

- JDK 8 (source/target level `1.6`–`1.8`; newer JDKs work as a *runtime* but
  won't compile the `-source 1.6` setting directly)
- Servlet container supporting `javax.servlet` — Tomcat 8/9 or Jetty 9
  (not Tomcat 10+)
- MySQL 5.x/8.x (with the charset fix above if using 8.x)

## ✅ Verified working pages

`index`, `login` (with admin login flow), `aboutus`, `carrier`, `complain`,
`enquiry`, `packettracking`, `searchconsignment`, `masternotice`, and the
admin zone (`adminhome`, `enquires`, `complains`) — all tested end-to-end
after applying the fixes above. `SmsSender.java` compiles cleanly but was not
functionally tested (likely requires external SMS API credentials).

