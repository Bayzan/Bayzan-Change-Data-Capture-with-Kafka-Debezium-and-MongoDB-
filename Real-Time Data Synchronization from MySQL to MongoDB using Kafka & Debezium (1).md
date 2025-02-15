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
- **PowerShell** - Command-line execution.
- **MongoDB Compass** - GUI for MongoDB visualization
- **java 17.0.12** - https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html

### **Project Workflow**
1. **MySQL database (`eticaret`)** is created.
2. Tables (`musteriler`, `siparisler`, `urunler`, `siparis_detaylari`) are set up with **foreign key relationships**.
3. **Kafka & Zookeeper** are started.
4. **Debezium MySQL Connector** captures MySQL changes and streams them to Kafka.
5. **MongoDB Kafka Connector** syncs these changes from Kafka to MongoDB.
6. **MongoDB Compass or mongosh** is used to verify data synchronization.

 ### 
 **The directories where I installed the programs may be different from yours. You can rearrange the commands to suit yourself.** 

# **Installation & Setup Guide**

## **1. Installing & Configuring MySQL**
- https://dev.mysql.com/downloads/workbench/ - I used MySQL 8.0.34
####  **🔍 Finding and Configuring `my.ini` in MySQL**  

#### **1️⃣ What is `my.ini` and Why is it Important?**  
The `my.ini` file is **MySQL's main configuration file** on Windows. It **controls essential settings**, such as:  
- **Storage paths** (`datadir`)  
- **Logging configurations** (`log-error`)  
- **Default time zone** (`default-time-zone`)  
- **Network settings** (ports, bindings, etc.)  

In this project, I needed to **modify `my.ini` to resolve various issues**, including:  
✔ Fixing **time zone errors** (for Kafka-MySQL integration)  
✔ Ensuring **MySQL runs on the correct port**  
✔ Changing **logging settings** to track errors  


####  **2️⃣ How I Found the `my.ini` File**  
By default, MySQL **does not always store `my.ini` in an obvious location**. I had difficulty locating it, so I used the following methods:  

#####  **🛠 Method 1: Using MySQL Command**  
I ran this command in **PowerShell** to check MySQL’s active configuration paths:  
```sh
mysqld --verbose --help | findstr "datadir log_error"
```
🔍 **Output example:**  
```
  -h, --datadir=name  Path to the database root directory
datadir  C:\ProgramData\MySQL\MySQL Server 8.0\Data\
log-error  C:\ProgramData\MySQL\MySQL Server 8.0\Data\LAPTOP-KUITPL27.err
```
From this, I found that **MySQL was using `C:\ProgramData\MySQL\MySQL Server 8.0\` as its directory**.  


#### **🛠 Method 2: Checking Common Directories**  
If `mysqld` doesn’t reveal the path, `my.ini` is typically found in **one of these locations**:  
✔ `C:\ProgramData\MySQL\MySQL Server 8.0\my.ini`  
✔ `C:\Program Files\MySQL\MySQL Server 8.0\my.ini`  
✔ `C:\Windows\my.ini`  

I manually **navigated to these folders** and checked if the file was there.  


####  **3️⃣ Modifying `my.ini` for the Project**  
Once I found the correct `my.ini` file, I **edited it to resolve key issues**:  

#####  **🕒 Fixing the Time Zone Issue**  
Kafka **was failing to connect** because MySQL’s time zone was set to `"Türkiye Standart Saati"` (Turkish Standard Time), which is not recognized by JDBC.  

💡 **Solution:**  
1️⃣ Open `my.ini` in Notepad or any text editor:  
   ```sh
   notepad "C:\ProgramData\MySQL\MySQL Server 8.0\my.ini"
   ```
2️⃣ Add this line under `[mysqld]`:  
   ```
   default-time-zone = 'Europe/Istanbul'
   ```
3️⃣ Save the file and restart MySQL:  
   ```sh
   net stop MySQL80
   net start MySQL80
   ```

#####  **🔄 Ensuring the Correct Data Directory (`datadir`)**  
In some cases, MySQL failed to start due to missing or incorrect `datadir`.  

💡 **Solution:**  
1️⃣ Open `my.ini`  
2️⃣ Locate the **datadir** setting and ensure it matches the correct MySQL installation:  
   ```
   datadir="C:/ProgramData/MySQL/MySQL Server 8.0/Data/"
   ```
3️⃣ Restart MySQL to apply changes.  


#####  **📜 Enabling Error Logging for Debugging**  
To track errors, I added the following lines:  
```
log-error="C:/ProgramData/MySQL/MySQL Server 8.0/Data/mysql-error.log"
```
This helped in identifying **why MySQL failed to start**.  


####  **4️⃣ Final Verification**  
After editing `my.ini`, I ensured that:  
✔ The **time zone issue was fixed**  
✔ MySQL **recognized the correct `datadir`**  
✔ Error logging **was enabled**  

I restarted MySQL and **checked if it was running correctly**:  
```sh
net start MySQL80
mysql -u root -p
SHOW VARIABLES LIKE 'time_zone';
```
If MySQL successfully started, **all fixes were applied correctly**. 🎯  

---


##### **Modify the `my.ini` configuration file as follows:**

 ```ini
  [mysqld]
  server-id=1
  log_bin=ON
  binlog_format=ROW
  default-time-zone='Europe/Istanbul'
 ```
    !!! You can adjust the value of "default-time-zone" according to your own
 
 
###  **Then,I created a sample database and tables in MySQLWorkBench 8.0 CE.** 
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
- **Fixing Time Zone Issues**  
  If MySQL throws a **time zone error (error 400)**, execute:
  ```sh
  mysql_tzinfo_to_sql C:/ProgramData/MySQL/MySQL Server 8.0/Data/zoneinfo.sql | mysql -u root -p mysql
  ```

---

## **2. Installing Kafka**
- Download and extract Kafka. https://kafka.apache.org/downloads 
- C:\kafka
- In `config/server.properties`, add:
  ```
  listeners=PLAINTEXT://localhost:9092
  ```
---

## **3.☕ Java Setup for Kafka and Debezium**  

### **1️⃣ Why Java is Needed in This Project**  
Apache Kafka, Kafka Connect, and Debezium **run on Java**, meaning that a **compatible Java version must be installed** and correctly configured for them to work.  

During the project, **Kafka failed to start due to missing or incompatible Java versions**, and I had to:  
✔ Install the **correct Java version**  
✔ Set up **Java environment variables**  
✔ Verify that **Kafka detects Java correctly**  



### **2️⃣ Installing the Correct Java Version**  
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



#### **3️⃣ Adding Java to Environment Variables**  
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


#### **4️⃣ Configuring Java for Kafka**  
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


#### **5️⃣ Troubleshooting Java Issues in Kafka**  

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

## **4. Configuring Debezium MySQL Connector**
- Add the Debezium plugin path in `connect-distributed.properties`:
  ```
  plugin.path=C:/kafka/connect-plugins
  ```
- Create the **MySQL Connector JSON** 
- "C:\kafka\kafka_2.13-3.8.1\config\mysql-connector.json"

  ```json
  {
  "name": "mysql-connector",
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "tasks.max": "1",
    "database.hostname": "localhost",
    "database.port": "3306",
    "database.user": "root",
    "database.password": "123",
    "database.server.name": "eticaret",
    "database.server.id": "1",
    "database.include.list": "eticaret",
    "table.include.list": "eticaret.musteriler,eticaret.siparisler,eticaret.siparis_detaylari,eticaret.urunler",
    "include.schema.changes": "true",
    "schema.history.internal.kafka.bootstrap.servers": "localhost:9092",
    "schema.history.internal.kafka.topic": "schema-changes.eticaret",
    "topic.prefix": "eticaret"
   }
  }
  ```
- Register the connector:
  ```sh
  Invoke-WebRequest -Uri "http://localhost:8083/connectors" `
    -Method POST `
    -ContentType "application/json" `
    -InFile "C:\kafka\config\mysql-connector.json"
  ```

---

### **5. Configuring MongoDB Kafka Connector**
- Create the MongoDB Connector JSON

- "C:\kafka\kafka_2.13-3.8.1\config\mongodb-sink.json"
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
- Register the connector:
  ```sh
  Invoke-WebRequest -Uri "http://localhost:8083/connectors" `
    -Method POST `
    -ContentType "application/json" `
    -InFile "C:\kafka\config\mongodb-sink.json"
  ```

---

## **6. Powershell and cmd commands**
Write the following steps in the sections written in parentheses. Run PowerShell and CMD as administrator.
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
    
