# Real-Time Data Synchronization from MySQL to MongoDB using Kafka & Debezium
## 
This project demonstrates how to **synchronize MySQL data with MongoDB in real time** using **Kafka**. It leverages **Debezium MySQL Connector** to capture changes in MySQL and **MongoDB Kafka Connector** to persist them in MongoDB.


### Technologies & Versions Used

This project was developed on **Windows 11** using the following technologies:

- **MySQL 8.0.34** - Source relational database.
- **MongoDB 8.0.4** - Target NoSQL database.
- **Kafka 3.8.1** - Manages real-time data streaming.
- **Debezium MySQL Connector 2.5.1.Final** - Captures MySQL changes.
- **MongoDB Kafka Connector 1.14.1** - Writes Kafka changes to MongoDB.
- **MongoDB Compass** - GUI for MongoDB visualization
- **java 17.0.12** 

### **Project Workflow**
1. **MySQL database (`eticaret`)** is created.
2. Tables (`musteriler`, `siparisler`, `urunler`, `siparis_detaylari`) are set up with **foreign key relationships**.
3. **Kafka & Zookeeper** are started.
4. **Debezium MySQL Connector** captures MySQL changes and streams them to Kafka.
5. **MongoDB Kafka Connector** syncs these changes from Kafka to MongoDB.
6. **MongoDB Compass or mongosh** is used to verify data synchronization.

 ### 
 **The directories where I installed the programs may be different from yours. You can rearrange the commands to suit yourself.** 
 
---
# **Installation & Setup Guide**

# **1️⃣ Installing & Configuring MySQL**  

## **What is MySQL, and Why Did We Use It?**  
**MySQL** is a widely-used **relational database management system (RDBMS)**. In this project, MySQL acts as the **primary data source**, where all structured data is initially stored.  

We used **Kafka and Debezium** to **capture real-time changes in MySQL** and forward them to **MongoDB**, ensuring **seamless data synchronization** between SQL and NoSQL environments.  

## **MySQL Version Used**  
🔹 **MySQL Version:** `8.0.34`  
🔹 **MySQL Workbench Version:** `8.0 CE`  


## **🔍 Finding and Configuring `my.ini` in MySQL**  

### **What is `my.ini` and Why is it Important?**  
The **`my.ini`** file is **MySQL’s main configuration file** on Windows. It controls essential settings, including:  

✅ **Storage paths (`datadir`)**  
✅ **Logging configurations (`log-error`)**  
✅ **Default time zone (`default-time-zone`)**  
✅ **Network settings (ports, bindings, etc.)**  

I had to modify `my.ini` to **fix various issues**, including:  
✔ **Fixing time zone errors** (for Kafka-MySQL integration)  
✔ **Ensuring MySQL runs on the correct port**  
✔ **Changing logging settings** to track errors  


## **How I Found the `my.ini` File**  

By default, MySQL **does not always store `my.ini` in an obvious location**. Since I had trouble locating it, I used the following methods:  

### **🛠 Method 1: Using MySQL Command**  
I ran this command in **PowerShell** to check **MySQL’s active configuration paths**:  
```powershell
mysqld --verbose --help | findstr "datadir log_error"
```
🔍 **Example Output**:  
```plaintext
  -h, --datadir=name  Path to the database root directory
datadir  C:\ProgramData\MySQL\MySQL Server 8.0\Data\
log-error  C:\ProgramData\MySQL\MySQL Server 8.0\Data\LAPTOP-KUITPL27.err
```
From this, I found that MySQL was using:  
```plaintext
C:\ProgramData\MySQL\MySQL Server 8.0\
```
as its **configuration directory**.  


### **🛠 Method 2: Checking Common Directories**  
If `mysqld` doesn’t reveal the path, **`my.ini` is typically found in one of these locations:**  
✔ `C:\ProgramData\MySQL\MySQL Server 8.0\my.ini`  
✔ `C:\Program Files\MySQL\MySQL Server 8.0\my.ini`  
✔ `C:\Windows\my.ini`  

I **manually navigated** to these folders and checked if `my.ini` existed.  


## **Modifying `my.ini` for the Project**  

### **🕒 Fixing the Time Zone Issue**  

At one point, **Kafka failed to connect** because MySQL’s **time zone setting was incorrect**. The error message indicated:  
```plaintext
Unable to connect: The server time zone value 'Türkiye Standart Saati' is unrecognized.
```
This happened because **JDBC does not recognize non-standard time zone names** like "Türkiye Standart Saati".  

💡 **Solution:**  

1️⃣ Open `my.ini` in Notepad or any text editor:  
```powershell
notepad "C:\ProgramData\MySQL\MySQL Server 8.0\my.ini"
```
2️⃣ Add this line under `[mysqld]`:  
```ini
default-time-zone = 'Europe/Istanbul'
```
3️⃣ **Save the file and restart MySQL:**  
```powershell
net stop MySQL80
net start MySQL80
```
✅ **This ensured MySQL used a time zone recognized by JDBC.**  

## **🌍 Loading Time Zone Data with `zoneinfo.sql`**  

Even after updating `my.ini`, MySQL **could still throw time zone errors**. To fully resolve this, I needed to **import the correct time zone data** using `zoneinfo.sql`.  

### **🔹 What is `zoneinfo.sql`?**  
`zoneinfo.sql` is a SQL script that **contains correct time zone definitions** for MySQL.  
📥 **Download the `zoneinfo.sql` file here:**  
🔗 [zoneinfo.sql](https://github.com/Bayzan/Change-Data-Capture-with-MySQL-MongoDB-Kafka-Debezium/blob/main/zoneinfo.sql)  

### **🛠 Steps to Import `zoneinfo.sql` into MySQL**   
```plaintext
C:\kafka\zoneinfo.sql
```
Then, I ran the following command in **PowerShell** to load the time zone data into MySQL:  
```powershell
mysql_tzinfo_to_sql C:/kafka/zoneinfo.sql | mysql -u root -p mysql
```
✅ **This ensured that MySQL properly recognized and used time zone information.**  


## **🔄 Ensuring the Correct Data Directory (`datadir`)**  

Sometimes, MySQL failed to start because the **`datadir` setting was incorrect**.  

💡 **Solution:**  
1️⃣ Open `my.ini`  
2️⃣ Locate the `datadir` setting and **ensure it matches MySQL’s actual data folder**:  
```ini
datadir="C:/ProgramData/MySQL/MySQL Server 8.0/Data/"
```
3️⃣ Restart MySQL to apply the changes.  


## **📜 Enabling Error Logging for Debugging**  

To **track errors** more easily, I added the following line to `my.ini`:  
```ini
log-error="C:/ProgramData/MySQL/MySQL Server 8.0/Data/mysql-error.log"
```
✅ **This helped in identifying why MySQL failed to start.**  


## **⏳ Final Verification**  

After editing `my.ini`, I ensured that:  
✔ **The time zone issue was fixed**  
✔ **MySQL recognized the correct `datadir`**  
✔ **Error logging was enabled**  

I restarted MySQL and checked if it was running correctly:  
```powershell
net start MySQL80
mysql -u root -p
SHOW VARIABLES LIKE 'time_zone';
```
✅ If MySQL successfully started, **all fixes were applied correctly.** 🎯  


## **🛠 Creating the Database and Tables**  

After MySQL was properly configured, I created a **sample database** and tables in **MySQL Workbench**:  

```sql
CREATE DATABASE eticaret;
USE eticaret;

CREATE TABLE musteriler (
    musteri_id INT AUTO_INCREMENT PRIMARY KEY,
    isim VARCHAR(50),
    soyisim VARCHAR(50),
    email VARCHAR(100)
);

CREATE TABLE siparisler (
    siparis_id INT AUTO_INCREMENT PRIMARY KEY,
    musteri_id INT,
    tarih TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (musteri_id) REFERENCES musteriler(musteri_id)
);

CREATE TABLE urunler (
    urun_id INT AUTO_INCREMENT PRIMARY KEY,
    urun_adi VARCHAR(100),
    fiyat DECIMAL(10,2)
);

CREATE TABLE siparis_detaylari (
    detay_id INT AUTO_INCREMENT PRIMARY KEY,
    siparis_id INT,
    urun_id INT,
    adet INT,
    FOREIGN KEY (siparis_id) REFERENCES siparisler(siparis_id),
    FOREIGN KEY (urun_id) REFERENCES urunler(urun_id)
);
```
✅ **Now, MySQL was fully set up and ready for integration with Kafka and MongoDB!**  


This **updated MySQL section** now includes:  
✅ **Why we modified `my.ini`**  
✅ **What `zoneinfo.sql` is and why we needed it**  
✅ **How to fix time zone issues in MySQL**  
✅ **Detailed steps to configure MySQL correctly**  

Let me know if you need further improvements! 🚀

---


# **2️⃣ Installing & Configuring Apache Kafka**  

## **What is Kafka and Why Did We Use It?**  
**Apache Kafka** is a distributed event streaming platform designed to handle **high-throughput, real-time data streams**. It allows applications to **publish, subscribe, store, and process data streams in a fault-tolerant manner**.  

In this project, **Kafka acts as the bridge between MySQL and MongoDB**, ensuring that **every change in MySQL is captured in real time and forwarded to MongoDB** using **Kafka Connect and Debezium**. Without Kafka, we would need to rely on **batch processing**, which would introduce **delays and inefficiencies**.  

### **Key Reasons for Using Kafka in This Project:**  
✅ **Real-time Data Streaming** – Kafka allows us to instantly capture **INSERT, UPDATE, and DELETE** operations in MySQL.  
✅ **Decoupling Between MySQL and MongoDB** – Instead of writing custom scripts for synchronization, Kafka manages the entire data flow.  
✅ **Fault-Tolerance & Scalability** – If MongoDB goes down, Kafka temporarily holds the data and delivers it once MongoDB is back online.  

## **Kafka Version Used**  
🔹 **Kafka Version:** `Apache Kafka 3.8.1`  
🔹 **Scala Version:** `2.13`  
🔹 **Java Version Used:** `Java 17.0.12`  

## **Installing Kafka**  
1️⃣ **Download and Extract Kafka**  
Download the Kafka binary from the official website:  
🔗 [Apache Kafka Downloads](https://kafka.apache.org/downloads)  

2️⃣ **Extract Kafka to a directory:**  
```plaintext
C:\kafka
```

3️⃣ **Configure Kafka Server**  
Open the Kafka configuration file:  
```plaintext
C:\kafka\config\server.properties
```
Find the following line and modify it to **enable Kafka on localhost**:  
```properties
listeners=PLAINTEXT://localhost:9092
```

4️⃣ **Start Kafka**  
Run the following commands in **PowerShell** to start **Zookeeper** and **Kafka**:  
```powershell
# Start Zookeeper (required for Kafka)
& "C:\kafka\bin\windows\zookeeper-server-start.bat" "C:\kafka\config\zookeeper.properties"

# Start Kafka Broker
& "C:\kafka\bin\windows\kafka-server-start.bat" "C:\kafka\config\server.properties"
```
---

# **3️⃣ ☕ Java Setup for Kafka and Debezium**  

### ** Why Java is Needed in This Project**  
Apache Kafka, Kafka Connect, and Debezium **run on Java**, meaning that a **compatible Java version must be installed** and correctly configured for them to work.  

During the project, **Kafka failed to start due to missing or incompatible Java versions**, and I had to:  
✔ Install the **correct Java version**  
✔ Set up **Java environment variables**  
✔ Verify that **Kafka detects Java correctly**  



### **Installing the Correct Java Version**  
Kafka **requires Java 8 or later**, but some versions have compatibility issues with certain Kafka components.  

💡 **Java version used in this project:**  
```sh
java -version
```
Output:  
```
openjdk version "17.0.9" 2024-01-01
OpenJDK Runtime Environment (AdoptOpenJDK)(build 17.0.9+9)
OpenJDK 64-Bit Server VM (AdoptOpenJDK)(build 17.0.9+9, mixed mode)
```

✔ **Java 17** was used because it is stable and compatible with Kafka 3.8.1 and Debezium.  



#### ** Adding Java to Environment Variables**  
To ensure Java is recognized globally, I added it to the **Windows Environment Variables**.  

**Steps to set up Java environment variables:**  
1. **Find the Java installation directory**  
   - Default path:  
     ```
     C:\Program Files\Java\jdk-17.0.9
     ```
2. **Add `JAVA_HOME` to System Variables**  
   - Open **Start Menu**, search for **"Environment Variables"**  
   - Click on **"Edit the system environment variables"** → **"Environment Variables"**  
   - Under **System Variables**, click **"New"** and enter:  
     ```
     Variable Name: JAVA_HOME
     Variable Value: C:\Program Files\Java\jdk-17.0.9
     ```
3. **Add Java to the `PATH` variable**  
   - Under **System Variables**, find **"Path"**, click **"Edit"**  
   - Click **"New"**, and add:  
     ```
     C:\Program Files\Java\jdk-17.0.9\bin
     ```
4. **Verify Java setup**  
   - Open PowerShell and run:  
     ```sh
     echo $env:JAVA_HOME
     ```
   - Expected output:  
     ```
     C:\Program Files\Java\jdk-17.0.9
     ```


#### **Configuring Java for Kafka**  
After setting up Java, I had to ensure **Kafka recognized it**.  

1. **Verify Java is detected by Kafka:**  
   ```sh
   & "C:\kafka\kafka_2.13-3.8.1\bin\windows\kafka-server-start.bat"
   ```
   If Java is missing, Kafka throws an error like:  
   ```
   ERROR: JAVA_HOME is not set and no 'java' command could be found in your PATH.
   ```
2. **Fixing Java detection issue in Kafka**  
   - If the error occurs, manually set `JAVA_HOME` inside **Kafka startup scripts**:  
     - Open `C:\kafka\kafka_2.13-3.8.1\bin\windows\kafka-run-class.bat`  
     - Find this line:  
       ```
       rem set JAVA_HOME=
       ```
     - Update it to:  
       ```
       set JAVA_HOME=C:\Program Files\Java\jdk-17.0.9
       ```


#### ** Troubleshooting Java Issues in Kafka**  

| Issue | Cause | Solution |
|------|------|------|
| `ERROR: JAVA_HOME is not set` | `JAVA_HOME` is missing from environment variables | Set `JAVA_HOME` and restart system |
| `Kafka does not start, no error message` | Java version is incompatible | Use **Java 17**, not Java 21+ |
| `java command not found` | Java bin path is missing from `PATH` | Add `C:\Program Files\Java\jdk-17.0.9\bin` to `PATH` |
| `Kafka crashes with IllegalArgumentException` | Corrupted Java installation | Reinstall Java and update `JAVA_HOME` |


#### **✅ Final Verification of Java Setup**  
After completing the setup, I ran these commands to confirm everything was working:

```sh
java -version
echo $env:JAVA_HOME
& "C:\kafka\kafka_2.13-3.8.1\bin\windows\kafka-server-start.bat"
```
If **Kafka starts successfully**, Java is correctly configured. 🎯  

---

# **4️⃣ Download debezium and integrate it into kafka**
## **🔗 Understanding Debezium and Its Role in Kafka Connect**  

### ** What is Debezium and Why Did We Use It?**  
Debezium is an **open-source Change Data Capture (CDC) tool** that allows databases to **stream real-time changes** to Kafka.  

💡 **Why is Debezium important?**  
- It **monitors database changes** and sends them to Kafka topics.  
- It enables **event-driven architectures**, where **microservices, data pipelines, and analytics systems** can react instantly.  
- It eliminates the need for **manual database polling**, making data synchronization **more efficient and reliable**.  

For our project, we used **Debezium’s MySQL connector** to capture changes from MySQL and stream them into Kafka.  


### ** How to Download and Install Debezium**  
We needed to install Debezium manually, as it does not come bundled with Kafka by default.  

 **Download Debezium:**  
   - We downloaded Debezium from the official site:  
     👉 [Debezium Connectors](https://debezium.io/releases/)  
   - We specifically used **Debezium MySQL Connector for Kafka** (version **2.5.0**).  

 **Extract the downloaded Debezium files** to integrate them with Kafka.  


### ** Integrating Debezium with Kafka**  
After downloading Debezium, we had to **place it correctly** inside Kafka’s directory structure.  

#### **📌 Steps to Install Debezium in Kafka:**  

✅ **Step 1: Create the `plugins` directory in Kafka**  
   - We navigated to the Kafka installation directory and created a `plugins` folder (if it didn't already exist):  
     ```sh
     C:\kafka\kafka_2.13-3.8.1\plugins
     ```  
   - Inside `plugins`, we created another folder for Debezium:  
     ```sh
     C:\kafka\kafka_2.13-3.8.1\plugins\debezium-connector-mysql
     ```  

✅ **Step 2: Move the Debezium JAR files into the `debezium-connector-mysql` directory**  
   - We **copied the extracted Debezium files** (especially `.jar` files) into:  
     ```sh
     C:\kafka\kafka_2.13-3.8.1\plugins\debezium-connector-mysql
     ```  

✅ **Step 3: Update Kafka Connect Configuration**  
   - We **edited `connect-distributed.properties`** in Kafka to include the new plugin path:  
     ```properties
     plugin.path=C:/kafka/kafka_2.13-3.8.1/plugins
     ```  
   - This ensures that Kafka **recognizes Debezium as a connector**.  

✅ **Step 4: Restart Kafka Connect**  
   - After updating the configuration, we restarted Kafka Connect to apply changes.  

✅ **Step 5: Verify That Debezium Is Installed**  
   - We checked if Kafka recognized Debezium by running:  
     ```powershell
     Invoke-WebRequest -Uri "http://localhost:8083/connector-plugins" -Method GET
     ```
   - If we saw **`io.debezium.connector.mysql.MySqlConnector`** in the list, it meant Debezium was successfully installed! ✅  


### ** How Debezium Works in Our Project**  
Once Debezium was set up, we configured it using the `mysql-connector.json` file.  

💡 **Debezium’s Role in the Project:**  
1. **Listens to MySQL changes** (INSERT, UPDATE, DELETE).  
2. **Writes those changes to Kafka topics** in JSON format.  
3. Kafka topics can then be consumed by **MongoDB, Elasticsearch, or other services**.  


### ** Troubleshooting Debezium Issues**  
🔴 **Error: `Failed to find any class that implements Connector`**  
✔ **Solution:** Ensure that `plugin.path` is correctly set in `connect-distributed.properties`.  

🔴 **Error: `mysql-connector.json` is not recognized**  
✔ **Solution:** Restart Kafka Connect and ensure the connector was added via:  
```powershell
Invoke-WebRequest -Uri "http://localhost:8083/connectors" -Method GET
```

🔴 **Error: `Connector is running, but no changes are appearing in Kafka`**  
✔ **Solution:** Check MySQL binlog settings and ensure `binlog_format=ROW` is set in `my.ini`.  

### **📌 Summary**  
- **Debezium is a Change Data Capture (CDC) tool** that streams **real-time MySQL changes** into Kafka.  
- We downloaded **Debezium MySQL Connector (2.5.0)** and integrated it with Kafka.  
- We **created a `plugins` folder in Kafka**, added a subfolder **`debezium-connector-mysql`**, and placed **Debezium JAR files** inside it.  
- We updated `connect-distributed.properties` to **include the plugin path** and restarted Kafka Connect.  
- **Debezium captures MySQL changes** and sends them to Kafka topics for **real-time processing**.  


This **detailed explanation** ensures that **anyone can integrate Debezium into Kafka successfully**. 🚀 Let me know if you need any refinements! 😊
 
 ---
 
# **5️⃣  Create the 'mysql-connector.json'**
- Create the **MySQL Connector JSON** 
- "C:\kafka\kafka_2.13-3.8.1\config\mysql-connector.json"

#### **🔗 Understanding `mysql-connector.json` in Kafka Connect**  

#### ** What is `mysql-connector.json` and Why is it Needed?**  
The `mysql-connector.json` file **defines the configuration** for Kafka Connect to **capture real-time changes** from a MySQL database and stream them into Kafka topics.  

💡 **Why is this important?**  
- It allows **Change Data Capture (CDC)**, ensuring that **every database update** is mirrored in Kafka.  
- It enables **event-driven architectures**, where **databases, microservices, and other systems** can react to changes instantly.  
- It is essential when integrating **MySQL with streaming platforms**, such as **Apache Kafka, MongoDB, Elasticsearch, or BigQuery**.  


#### ** How to Create `mysql-connector.json`**  
To create `mysql-connector.json`, follow these steps:  

1️⃣ **Open a text editor** (VS Code, Notepad++, etc.).  
2️⃣ **Write the JSON configuration** (explained below).  
3️⃣ **Save it as `mysql-connector.json`** in your Kafka `config` directory.  

📌 **Windows File Extension Issue**  
In Windows, **file extensions may be hidden by default**, meaning you could accidentally name your file as `mysql-connector.json.txt` instead of `mysql-connector.json`.  

💡 **Solution:**  
1. Open **File Explorer**.  
2. Click on the **View** tab.  
3. Check the box for **"File name extensions"**.  
4. Now, rename the file and ensure it is saved as `mysql-connector.json`.  


#### ** Breaking Down the `mysql-connector.json` Configuration**  
Below is an example of a properly configured `mysql-connector.json` file:  

```json
{
  "name": "mysql-connector",
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "tasks.max": "1",
    "database.hostname": "localhost",
    "database.port": "3306",
    "database.user": "root",
    "database.password": "your_password",
    "database.server.id": "1",
    "database.server.name": "eticaret",
    "database.include.list": "eticaret",
    "database.history.kafka.bootstrap.servers": "localhost:9092",
    "database.history.kafka.topic": "schema-changes.eticaret",
    "include.schema.changes": "true",
    "snapshot.mode": "initial",
    "database.time_zone": "Europe/Istanbul",
    "key.converter": "org.apache.kafka.connect.storage.StringConverter",
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "value.converter.schemas.enable": false
  }
}
```

Let’s go through each key-value pair and its meaning:  

##### **📌 Basic Connector Settings**  
| **Key**                         | **Description** |
|---------------------------------|----------------|
| `"name"`                        | The name of the Kafka Connector instance (should be unique). |
| `"connector.class"`             | Specifies the Kafka Connector plugin (`io.debezium.connector.mysql.MySqlConnector`). |
| `"tasks.max"`                    | Number of parallel tasks (for small databases, keep it `1`). |



#### **📌 Database Connection Settings**  
| **Key**                         | **Description** |
|---------------------------------|----------------|
| `"database.hostname"`           | The **IP or hostname** where MySQL is running (`localhost` for local testing). |
| `"database.port"`               | The **MySQL port** (default is `3306`). |
| `"database.user"`               | The MySQL **username** used for connection. |
| `"database.password"`           | The MySQL **password** for authentication. |
| `"database.server.id"`          | A unique **numeric ID** for the database server (used in replication). |



#### **📌 Kafka Integration Settings**  
| **Key**                         | **Description** |
|---------------------------------|----------------|
| `"database.server.name"`        | The **logical name** used for Kafka topics (`eticaret`). |
| `"database.include.list"`       | The **list of databases** to capture changes from (e.g., `"eticaret"`). |
| `"database.history.kafka.bootstrap.servers"` | The Kafka **broker address** (e.g., `"localhost:9092"`). |
| `"database.history.kafka.topic"` | Kafka topic where **schema changes** are stored. |


#### **📌 Change Data Capture (CDC) Options**  
| **Key**                         | **Description** |
|---------------------------------|----------------|
| `"include.schema.changes"`      | If `true`, **schema changes** (ALTER TABLE, etc.) are tracked. |
| `"snapshot.mode"`               | Specifies how the **initial data capture** happens:  
  - `"initial"` → Captures all existing data before streaming changes.  
  - `"when_needed"` → Only captures existing data if needed.  
  - `"schema_only"` → Captures schema only, ignoring existing data. |


#### **📌 Time Zone Configuration**  
| **Key**                         | **Description** |
|---------------------------------|----------------|
| `"database.time_zone"`          | Specifies the **MySQL server's time zone** (`Europe/Istanbul`). |
| `"key.converter"`               | Defines how Kafka keys are stored (`StringConverter`). |
| `"value.converter"`             | Defines how Kafka values are stored (`JsonConverter`). |
| `"value.converter.schemas.enable"` | If `false`, Kafka **does not store schema information** in the message. |


#### ** Deploying the MySQL Connector in Kafka**  
After creating the `mysql-connector.json`, deploy it to Kafka Connect using **PowerShell** or **Command Prompt**:  

```powershell
Invoke-WebRequest -Uri "http://localhost:8083/connectors" `
  -Method POST `
  -ContentType "application/json" `
  -InFile "C:\kafka\config\mysql-connector.json"
```
If successful, Kafka Connect will **start monitoring MySQL changes** and publishing them into Kafka topics.  

#### ** Troubleshooting Common Errors**  
🔴 **Error: `Unable to connect: The server time zone value 'Türkiye Standart Saati' is unrecognized.`**  
✔ **Solution:** Add `default-time-zone='Europe/Istanbul'` in `my.ini` and restart MySQL.  

🔴 **Error: `No status found for connector mysql-connector`**  
✔ **Solution:** Check if Kafka Connect is running:  
```sh
Invoke-WebRequest -Uri "http://localhost:8083/connectors" -Method GET
```
✔ If missing, **recreate the connector** using the POST request above.  

#### ** Final Check - Is It Working?**  
To confirm everything is running:  

**✅ Check MySQL binary logs:**  
```sh
SHOW BINARY LOGS;
```
**✅ Verify Kafka topics:**  
```powershell
kafka-topics.bat --list --bootstrap-server localhost:9092
```
**✅ Read MySQL data from Kafka:**  
```powershell
kafka-console-consumer.bat --bootstrap-server localhost:9092 `
  --topic eticaret.eticaret.musteriler --from-beginning
```
If **changes in MySQL appear in Kafka**, **your MySQL connector is working correctly!** 🚀  


#### **📌 Summary**  
- `mysql-connector.json` **tells Kafka Connect how to listen for MySQL changes** and **send them to Kafka topics**.  
- It contains **database connection details**, **Kafka settings**, and **CDC configurations**.  
- The **MySQL server must be configured correctly** to avoid **timezone or replication errors**.  
- **Deploying the connector** via REST API ensures that **Kafka starts consuming data** from MySQL.  
- **Verifying logs** and **Kafka topics** confirms **successful data flow**.  
- **Make sure file extensions are visible** when creating the `.json` file to prevent naming issues.  


This **detailed explanation** ensures that anyone **setting up MySQL with Kafka** can **follow the steps easily**. 🚀 Let me know if you need any refinements! 😊
- Register the connector:
  ```sh
  Invoke-WebRequest -Uri "http://localhost:8083/connectors" `
    -Method POST `
    -ContentType "application/json" `
    -InFile "C:\kafka\config\mysql-connector.json"
  ```

---

# **6️⃣ Configuring MongoDB Kafka Connector**

# **MongoDB Integration in This Project**  

### **What is MongoDB?**  
MongoDB is a **NoSQL database** that stores data in **JSON-like documents**, making it different from relational databases like MySQL. Instead of storing data in tables with fixed columns, MongoDB uses **collections** where each document can have a flexible structure.  

### **Why Did We Use MongoDB in This Project?**  
In this project, **MongoDB was used as the final destination to store real-time changes happening in MySQL**. Using **Kafka Connect and Debezium**, we were able to capture **INSERT, UPDATE, and DELETE operations** in MySQL and store them in MongoDB for further analysis and reporting.  


## **MongoDB Setup and Tools Used**  

### ** MongoDB Compass: The Graphical Interface**  
We installed **MongoDB Compass**, which is a GUI tool that allows us to:  
✅ View databases and collections visually.  
✅ Perform queries on MongoDB without using the command line.  
✅ Monitor real-time data synchronization from MySQL.  

### ** Mongosh: The Command-Line Interface**  
MongoDB no longer includes the traditional `mongo` shell. Instead, we installed **mongosh**, a new **JavaScript-based command-line shell** for MongoDB, which allows:  
✅ Connecting and managing MongoDB from the terminal.  
✅ Running MongoDB queries interactively.  
✅ Debugging and testing MongoDB commands.  

### ** Creating the MongoDB Sink Connector (`mongodb-sink.json`)**  
To transfer data from Kafka topics to MongoDB, we needed a **MongoDB Sink Connector**. This is a configuration file that tells **Kafka Connect** how to **send data from Kafka topics to MongoDB collections**.  

#### **📌 Here is the `mongodb-sink.json` file we created:**
```json
{
  "name": "mongodb-sink-connector",
  "config": {
    "connector.class": "com.mongodb.kafka.connect.MongoSinkConnector",
    "tasks.max": "1",
    "topics": "eticaret.eticaret.musteriler,eticaret.eticaret.siparisler,eticaret.eticaret.urunler,eticaret.eticaret.siparis_detaylari",
    "connection.uri": "mongodb://localhost:27017",
    "database": "eticaret",
    "collection": "${topic}",
    "key.converter": "org.apache.kafka.connect.storage.StringConverter",
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "value.converter.schemas.enable": false
  }
}
```

### **Explanation of Each Parameter in `mongodb-sink.json`**  

🔹 `"connector.class": "com.mongodb.kafka.connect.MongoSinkConnector"`  
→ Specifies that we are using the **MongoDB Sink Connector** to transfer data from Kafka to MongoDB.  

🔹 `"tasks.max": "1"`  
→ Defines how many tasks Kafka should use to process data. Since this is a small project, we set it to **1**.  

🔹 `"topics": "eticaret.eticaret.musteriler,eticaret.eticaret.siparisler,eticaret.eticaret.urunler,eticaret.eticaret.siparis_detaylari"`  
→ Lists all the **Kafka topics** that should be forwarded to MongoDB. These topics were created when we connected MySQL with Kafka using **Debezium**.  

🔹 `"connection.uri": "mongodb://localhost:27017"`  
→ This is the **MongoDB connection string** that tells Kafka where to find MongoDB.  

🔹 `"database": "eticaret"`  
→ This tells Kafka to store all received data in the **eticaret database** inside MongoDB.  

🔹 `"collection": "${topic}"`  
→ This dynamically maps **Kafka topics to MongoDB collections**.  
For example:  
- Data from **`eticaret.eticaret.musteriler`** will be stored in the **`musteriler`** collection in MongoDB.  
- Data from **`eticaret.eticaret.urunler`** will be stored in the **`urunler`** collection in MongoDB.  

🔹 `"key.converter": "org.apache.kafka.connect.storage.StringConverter"`  
→ Converts Kafka record keys to **strings**.  

🔹 `"value.converter": "org.apache.kafka.connect.json.JsonConverter"`  
→ Converts Kafka record values to **JSON format** before storing them in MongoDB.  

🔹 `"value.converter.schemas.enable": false`  
→ Disables schema validation for JSON data.  


## ** Registering the MongoDB Sink Connector in Kafka**  
After creating the `mongodb-sink.json` file, we registered it with Kafka Connect by running this **PowerShell command**:  
```powershell
Invoke-WebRequest -Uri "http://localhost:8083/connectors" `
  -Method POST `
  -ContentType "application/json" `
  -InFile "C:\kafka\config\mongodb-sink.json"
```
This command **informs Kafka Connect** that we are adding a new **MongoDB Sink Connector**, which will start pulling data from Kafka topics and inserting it into MongoDB collections.  


## **Verifying That MongoDB is Receiving Data**  

After registering the connector, we verified that **data from MySQL was being transferred to MongoDB** using:  

### ✅ **Check data inside MongoDB using mongosh**
```powershell
mongosh
use eticaret
db.musteriler.find().pretty()
```
This command **retrieves all documents** inside the `musteriler` collection to check if data from MySQL was successfully transferred.  


## **Troubleshooting: Issues Faced & How We Fixed Them**  

### ❌ **1. Data was not appearing in MongoDB**  
✅ **Solution:**  
- We restarted Kafka Connect and MongoDB.  
- We verified the Kafka topics using:  
  ```powershell
  & "C:\kafka\kafka_2.13-3.8.1\bin\windows\kafka-console-consumer.bat" `
  --bootstrap-server localhost:9092 `
  --topic eticaret.eticaret.musteriler
  ```
- If Kafka showed data but MongoDB did not, we **re-registered the MongoDB Sink Connector**:  
  ```powershell
  Invoke-WebRequest -Uri "http://localhost:8083/connectors/mongodb-sink-connector" -Method DELETE
  Invoke-WebRequest -Uri "http://localhost:8083/connectors" -Method POST -ContentType "application/json" -InFile "C:\kafka\config\mongodb-sink.json"
  ```

### ❌ **2. MongoDB Sink Connector was not recognized in Kafka**  
✅ **Solution:**  
- We checked that the **MongoDB Kafka Connector JAR file** was in the **correct folder** inside Kafka.  
- We restarted Kafka Connect after adding the plugin.  


## **Final Thoughts**  
MongoDB played a crucial role in our project by **storing real-time data changes** from MySQL. Using **Kafka Connect and Debezium**, we successfully **captured MySQL operations** and stored them in MongoDB for further analysis.  

With **MongoDB Compass**, we were able to **visually inspect the database**, and with **mongosh**, we could run **queries and debug issues easily**.  

### **📌 Key Achievements of This Integration:**  
✅ Successfully **captured MySQL changes** and stored them in MongoDB.  
✅ Used **Kafka Connect & Debezium** for real-time data streaming.  
✅ Ensured **MongoDB Compass & mongosh** were set up for data monitoring.  
✅ Troubleshot various **connector and data flow issues** to ensure smooth synchronization.  

---

# **7️⃣ Powershell and cmd commands**
### - Commands that should be written from the very beginning.
### - Write the following steps in the sections written in parentheses. Run PowerShell and CMD as administrator.
- **1. Start Zookeeper (PowerShell)**
  ```sh
  & "C:\kafka\kafka_2.13-3.8.1\bin\windows\zookeeper-server-start.bat" "C:\kafka\kafka_2.13-3.8.1\config\zookeeper.properties"
  ```
- **2. Start Kafka Broker (PowerShell)**
  ```sh
  & "C:\kafka\kafka_2.13-3.8.1\bin\windows\kafka-server-start.bat" "C:\kafka\kafka_2.13-3.8.1\config\server.properties"
  ```
  
 - **3. Start Kafka Connect (PowerShell)**
   ```sh
   & "C:\kafka\kafka_2.13-3.8.1\bin\windows\connect-distributed.bat" "C:\kafka\kafka_2.13-3.8.1\config\connect-distributed.properties"
   ``` 
  
- **4. Check Kafka Connect Status (PowerShell)**
 If there is no connection, run step 5, otherwise you don't need to do step 5
   ```sh
   Invoke-WebRequest -Uri "http://localhost:8083/connectors/mysql-connector/status" -Method GET
   ```   
   
- **5. Uploading JSON File to Kafka Connect (PowerShell)**
   ```sh
   Invoke-WebRequest -Uri "http://localhost:8083/connectors" `
  -Method POST `
  -ContentType "application/json" `
  -InFile "C:\kafka\kafka_2.13-3.8.1\config\mysql-connector.json"
   ``` 
   
 - **6. Listing Current Topicst (PowerShell)**
  If topics do not appear, create them
   ```sh
    cd C:\kafka\kafka_2.13-3.8.1\bin\windows
    .\kafka-topics.bat --list --bootstrap-server localhost:9092
   ``` 
   
 - **7. Create Consumer (PowerShell)**
 Shows changes to the customers table since the beginning
   ```sh
   & "C:\kafka\kafka_2.13-3.8.1\bin\windows\kafka-console-consumer.bat" `
   --bootstrap-server localhost:9092 `
   --topic eticaret.eticaret.musteriler --from-beginning
   ```  
   
   Shows changes in the "eticaret" database since the beginning
    ```sh
   & "C:\kafka\kafka_2.13-3.8.1\bin\windows\kafka-console-consumer.bat" `
   --bootstrap-server localhost:9092 `
   --topic eticaret --from-beginning
   ```
   
   You can run this command to see the changes after running the command.
    ```sh
   & "C:\kafka\kafka_2.13-3.8.1\bin\windows\kafka-console-consumer.bat" `
   --bootstrap-server localhost:9092 `
   --topic eticaret.eticaret.musteriler
   ``` 
   
 - **8. Upload Sink Connector to Kafka Connect (cmd)**

   ```sh
   curl -X POST -H "Content-Type: application/json" --data @C:/kafka/kafka_2.13-3.8.1/config/mongodb-sink.json http://localhost:8083/connectors
   ```   
   
 - **9. To make sure it has been installed successfully, use the following command (cmd)**

   ```sh
   curl -X GET http://localhost:8083/connectors/mongodb-sink-connector/statutors
   ```   
   
 - **10. Make a change to the "customers" table in MySQL and observe the changes it makes both in the PowerShell window created in the "consumer" and in MongoDBCompass.**
    
