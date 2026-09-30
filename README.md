

#  Highly Available Web Application using(ALB)

### Objective

Create a highly available web application using an **Application Load
Balancer (ALB)** and multiple **Amazon EC2 instances**.

The project demonstrates:

-   Multiple EC2 web servers
-   Target Group
-   Health checks
-   Application Load Balancer
-   Traffic distribution
-   High availability when one instance becomes unavailable

------------------------------------------------------------------------

## AWS Architecture

``` text
                         Internet
                            |
                            v
              +---------------------------+
              | Application Load Balancer |
              |     Web-Application-LB    |
              |         HTTP : 80         |
              +-------------+-------------+
                            |
                            v
                 +---------------------+
                 |   Web-Target-Group  |
                 |    HTTP Health Check|
                 +----------+----------+
                            |
                 +----------+----------+
                 |                     |
                 v                     v
        +----------------+    +----------------+
        | Web-Server-1   |    | Web-Server-2   |
        | Amazon Linux   |    | Amazon Linux   |
        | HTTP : 80      |    | HTTP : 80      |
        +----------------+    +----------------+
```

------------------------------------------------------------------------

## AWS Resources

  Resource             Configuration
  -------------------- ---------------------------
  EC2 Instance 1       Web-Server-1
  EC2 Instance 2       Web-Server-2
  Target Group         Web-Target-Group
  Load Balancer        Web-Application-LB
  Load Balancer Type   Application Load Balancer
  Scheme               Internet-facing
  Listener             HTTP : 80
  Target Protocol      HTTP
  Target Port          80
  Health Check Path    `/`
  OS                   Amazon Linux 2023

------------------------------------------------------------------------

## Step 1 -- Create Web-Server-1

An EC2 instance named `Web-Server-1` was launched using Amazon Linux
2023.

HTTP traffic on port 80 was allowed through the EC2 security group.

### Install Apache HTTP Server

``` bash
sudo dnf update -y
sudo dnf install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
```

### Create the webpage

``` bash
echo "<h1>Hello from Web Server 1</h1>" | sudo tee /var/www/html/index.html
```

### Check Apache

``` bash
sudo systemctl status httpd
```

Expected result:

``` text
Active: active (running)
```

The webpage was tested using the EC2 public IP address.

------------------------------------------------------------------------

## Step 2 -- Create Web-Server-2

A second EC2 instance named `Web-Server-2` was launched using Amazon
Linux 2023.

### Install Apache

``` bash
sudo dnf update -y
sudo dnf install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
```

### Create the webpage

``` bash
echo "<h1>Hello from Web Server 2</h1>" | sudo tee /var/www/html/index.html
```

### Check Apache

``` bash
sudo systemctl status httpd
```

Expected result:

``` text
Active: active (running)
```

The webpage was tested using the EC2 public IP address.

------------------------------------------------------------------------

## Step 3 -- Create Target Group

A Target Group named:

``` text
Web-Target-Group
```

was created.

### Configuration

-   Target type: **Instances**
-   Protocol: **HTTP**
-   Port: **80**
-   IP address type: **IPv4**
-   Health check protocol: **HTTP**
-   Health check path: `/`
-   Health check interval: **30 seconds**

Both EC2 instances were registered:

``` text
Web-Server-1
Web-Server-2
```

The required Availability Zone containing the EC2 instances was enabled
for the Load Balancer.

------------------------------------------------------------------------

## Step 4 -- Create Application Load Balancer

An Application Load Balancer named:

``` text
Web-Application-LB
```

was created.

### Configuration

-   Type: **Application Load Balancer**
-   Scheme: **Internet-facing**
-   IP address type: **IPv4**
-   Listener: **HTTP : 80**
-   Default action: **Forward to Web-Target-Group**

The Load Balancer became **Active** after creation.

------------------------------------------------------------------------

## Step 5 -- Verify Target Health

After configuring the Load Balancer and Availability Zones, both
registered EC2 instances became healthy.

Expected result:

``` text
Web-Server-1 → Healthy
Web-Server-2 → Healthy
```

The ALB listener forwarded HTTP traffic to:

``` text
Web-Target-Group
```

------------------------------------------------------------------------

## Step 6 -- Test the Load Balancer

The DNS name of `Web-Application-LB` was opened in a browser.

Example:

``` text
http://<LOAD-BALANCER-DNS-NAME>
```

The application was successfully displayed through the Load Balancer.

Example response:

``` text
Hello from Web Server 1
```

------------------------------------------------------------------------

## Step 7 -- High Availability Test

To demonstrate high availability, `Web-Server-1` was stopped from the
EC2 console.

The Target Group then showed:

``` text
Web-Server-1 → Unused / stopped
Web-Server-2 → Healthy
```

The Load Balancer continued serving the application through the healthy
instance.

The browser displayed:

``` text
Hello from Web Server 2
```

This demonstrates that traffic continues to a healthy EC2 instance when
another instance becomes unavailable.

------------------------------------------------------------------------

## Step 8 -- Restore Web-Server-1

`Web-Server-1` was started again.

After the health check completed, the target became healthy again.

Final expected state:

``` text
Web-Server-1 → Healthy
Web-Server-2 → Healthy
```

------------------------------------------------------------------------

## Testing Results

  Test                                    Result
  --------------------------------------- --------
  Web-Server-1 web page                   Passed
  Web-Server-2 web page                   Passed
  Target Group created                    Passed
  Both EC2 instances registered           Passed
  HTTP health checks                      Passed
  Application Load Balancer created       Passed
  ALB HTTP listener                       Passed
  Both targets healthy                    Passed
  Stop Web-Server-1                       Passed
  Web-Server-2 remains healthy            Passed
  ALB serves Web-Server-2 after failure   Passed
  Web-Server-1 restored                   Passed

------------------------------------------------------------------------

## Screenshots


01-web-server-1
![instance](screenshots\instance.png)
02-web-server-1-webpage
![webpage1](screenshots\webpage1.png)
03-web-server-2
![instance](screenshots\instance2.png)
04-web-server-2-webpage.png
![webpage2](screenshots\webpage2.png)
05-target-group-created.png
![target](screenshots\target.png)
06-both-targets-healthy.png
![health](screenshots\health.png)
07-server-1-stopped-server-2-healthy.png
![one-server](screenshots\one-server.png)
08-both-targets-healthy-again.png
![health](screenshots\health.png)

```

## Conclusion

The project successfully demonstrates a highly available web application
using:

-   Amazon EC2
-   Application Load Balancer
-   Target Group
-   HTTP health checks

Two EC2 web servers were registered with the Target Group. When one
server was stopped, the Application Load Balancer continued routing
traffic to the healthy server.

------------------------------------------------------------------------

