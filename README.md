---

# 3-Tier Architecture Application

This project demonstrates a **3-Tier Architecture on AWS** with **High Availability (HA)** and **Fault Tolerance**.
The architecture includes **VPC, S3, IAM, RDS, Application Tier, Web Tier, and Load Balancers**.

---

 🏗️ Project Infrastructure Setup

    1. VPC Creation

* Create a **Virtual Private Cloud (VPC)** as the foundation for networking.

### 2. S3 Bucket & IAM Role

* Create an **S3 Bucket** to upload application code.
* Configure an **IAM Role** with necessary permissions and attach it to the **EC2 instances**.

### 3. Database Configuration (RDS)

* Launch and configure an **RDS instance**.
* Update database credentials in:

  ```js
  // application-code/app-tier/DbConfig.js
  module.exports = Object.freeze({
      DB_HOST: 'YOUR-DATABASE-ENDPOINT.ap-south-1.rds.amazonaws.com',
      DB_USER: 'admin',
      DB_PWD: 'your-db-password',
      DB_DATABASE: 'webappdb'
  });
  ```
* Upload the updated `DbConfig.js` file into the S3 bucket under the `app-tier` folder.

### 4. Application Tier Setup

1. **App Server Setup & DB Configuration**

   * Install MySQL client:

     ```bash
     sudo yum install mysql -y
     ```

   * Connect to RDS and configure database:

     ```bash
     mysql -h <DB_ENDPOINT> -u admin -p
     ```

   * Create database and table:

     ```sql
     CREATE DATABASE webappdb;
     USE webappdb;
     CREATE TABLE IF NOT EXISTS transactions(
       id INT NOT NULL AUTO_INCREMENT,
       amount DECIMAL(10,2),
       description VARCHAR(100),
       PRIMARY KEY(id)
     );
     ```

   * Insert sample data:

     ```sql
     INSERT INTO transactions (amount, description) VALUES ('400', 'groceries');
     SELECT * FROM transactions;
     ```

2. **Node.js & PM2 Setup**

   ```bash
   curl -o- https://raw.githubusercontent.com/avizway1/aws_3tier_architecture/main/install.sh | bash
   source ~/.bashrc
   nvm install 16
   nvm use 16
   npm install -g pm2
   ```

3. **Deploy Application Code**

   ```bash
   sudo aws s3 cp s3://<S3BucketName>/application-code/app-tier/ app-tier --recursive
   cd app-tier/
   npm install
   pm2 start index.js
   pm2 save
   ```

   * Verify:

     ```bash
     curl http://localhost:4000/health
     ```

     Expected output:

     ```
     This is the health check.
     ```

4. **Internal Load Balancer Setup**

   * Update `nginx.conf` with **Internal LB DNS**:

     ```nginx
     location /api/ {
         proxy_pass http://<INTERNAL-LB-DNS>:80/;
     }
     ```
   * Upload the updated `nginx.conf` to S3.

### 5. Web Tier Setup

1. **Download Web Tier Code**

   ```bash
   aws s3 cp s3://<S3BucketName>/application-code/web-tier/ web-tier --recursive
   cd web-tier
   npm install
   npm run build
   ```

2. **Nginx Setup**

   ```bash
   sudo amazon-linux-extras install nginx1 -y
   cd /etc/nginx
   sudo rm nginx.conf
   sudo aws s3 cp s3://<S3BucketName>/application-code/nginx.conf .
   sudo service nginx restart
   sudo chkconfig nginx on
   ```

3. **Access Application**

   * Open **port 80 (HTTP)** in Web-Tier Security Group.
   * Visit: `http://<Web-Tier-Public-IP>` in a browser.

---

 🗑️ Cleanup Resources (Teardown Order)

When finished, delete resources in the following order:

1. Auto Scaling Groups (ASG)
2. Load Balancers (LBs)
3. Target Groups (TGs)
4. RDS Database
5. S3 Buckets
6. NAT Gateway
7. Elastic IP
8. VPC


---

✨ That’s it! You now have a **3-Tier AWS Application** deployed end-to-end.
