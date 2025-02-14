# Real-Time Data Synchronization from MySQL to MongoDB using Kafka & Debezium
## 
This project demonstrates how to **synchronize MySQL data with MongoDB in real time** using **Kafka**. It leverages **Debezium MySQL Connector** to capture changes in MySQL and **MongoDB Kafka Connector** to persist them in MongoDB.


## Technologies & Versions Used

This project was developed on **Windows 11** using the following technologies:

- **MySQL 8.0.34** - Source relational database.
- **MongoDB 8.0.4** - Target NoSQL database.
- **Kafka 3.8.1** - Manages real-time data streaming.
- **Debezium MySQL Connector 2.5.1.Final** - Captures MySQL changes.
- **MongoDB Kafka Connector 1.14.1** - Writes Kafka changes to MongoDB.
- **PowerShell** - Command-line execution.
- **MongoDB Compass** - GUI for MongoDB visualization

## **Project Workflow**
1. **MySQL database (`eticaret`)** is created.
2. Tables (`musteriler`, `siparisler`, `urunler`, `siparis_detaylari`) are set up with **foreign key relationships**.
3. **Kafka & Zookeeper** are started.
4. **Debezium MySQL Connector** captures MySQL changes and streams them to Kafka.
5. **MongoDB Kafka Connector** syncs these changes from Kafka to MongoDB.
6. **MongoDB Compass or mongosh** is used to verify data synchronization.

 ### 
 **The directories where I installed the programs may be different from yours. You can rearrange the commands to suit yourself.** 

## **Installation & Setup Guide**

### **1. Installing & Configuring MySQL**
- https://dev.mysql.com/downloads/workbench/ - I used MySQL 8.0.34

- Modify the `my.ini` configuration file as follows:
  ```ini
  [mysqld]
  server-id=1
  log_bin=ON
  binlog_format=ROW
  default-time-zone='Europe/Istanbul'
  ```
    !!! You can adjust the value of "default-time-zone" according to your own
 
 
- First, I created a sample database and tables in MySQL. 
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

### **2. Installing Kafka**
- Download and extract Kafka. https://kafka.apache.org/downloads 
- C:\kafka
- In `config/server.properties`, add:
  ```
  listeners=PLAINTEXT://localhost:9092
  ```


---

### **3. Configuring Debezium MySQL Connector**
- Add the Debezium plugin path in `connect-distributed.properties`:
  ```
  plugin.path=C:/kafka/connect-plugins
  ```
- Create the **MySQL Connector JSON**:
  ```json
  {
    "name": "mysql-connector",
    "config": {
      "connector.class": "io.debezium.connector.mysql.MySqlConnector",
      "database.hostname": "localhost",
      "database.port": "3306",
      "database.user": "root",
      "database.password": "root",
      "database.server.id": "1",
      "database.include.list": "eticaret",
      "table.include.list": "eticaret.musteriler,eticaret.siparisler,eticaret.urunler,eticaret.siparis_detaylari",
      "database.history.kafka.bootstrap.servers": "localhost:9092",
      "database.history.kafka.topic": "schema-changes.eticaret"
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

### **4. Configuring MongoDB Kafka Connector**
- Define the **MongoDB Sink Connector JSON**:
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

### **Powershell and cmd commands**
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
