# Question 3: Student Registration Application on Amazon EC2 and Amazon RDS (MySQL)

**AWS Project Assignment, Set 1 (AWS Only)**

## Problem Statement
Deploy a student registration application using Amazon EC2 and Amazon RDS. Configure the EC2 web server and an RDS MySQL database, establish secure connectivity between them, store student records in the database, and demonstrate successful data insertion and retrieval.

## Project Overview
| Item | Details |
|---|---|
| Web server | Amazon EC2, Ubuntu 24.04, public subnet, Flask app on port 80 |
| Database | Amazon RDS MySQL 8.4, `student-db`, database `studentdb`, **Public access: No** |
| Network | `my-web-vpc` with `web-sg` and `db-sg` |
| Secure link | MySQL 3306 allowed **only from `web-sg`** |

## Architecture
![Architecture](screenshots/00_architecture.png)

The web server sits in the public subnet. The database sits in two private subnets with no route to the Internet Gateway, and only the web server's security group can reach it.

## Implementation Summary
| Item | Configuration |
|---|---|
| Private subnets | 10.0.2.0/24 (1a) and 10.0.3.0/24 (1b), no route to the Internet Gateway |
| `db-sg` | MySQL/Aurora, TCP 3306, source = `web-sg` |
| DB subnet group | `student-db-subnet-group` in `my-web-vpc` |
| RDS | `student-db`, MySQL 8.4, database `studentdb`, Public access: No |
| Credentials | `/etc/student-app.env`, permission 600 |
| Application | Flask + PyMySQL, systemd service `student-app`, port 80 |

## Implementation Steps

### 1. Create the private subnets
Add two subnets in `my-web-vpc`: `Private-Subnet-1` (10.0.2.0/24, zone 1a) and `Private-Subnet-2` (10.0.3.0/24, zone 1b). They are not associated with the public route table, so they have no route to the Internet Gateway.

![Subnets](screenshots/01_subnets_list.png)

### 2. Create the DB subnet group
Create `student-db-subnet-group` in `my-web-vpc` containing both private subnets.

![DB subnet group](screenshots/02_db_subnet_group.png)

### 3. Configure security groups
`db-sg` allows MySQL/Aurora TCP 3306 only from `web-sg`.

![db-sg inbound](screenshots/03_db_sg_inbound_rules.png)

`web-sg` allows HTTP 80 for the web page and SSH for administration.

![web-sg inbound](screenshots/04_web_sg_inbound_rules.png)

![web-sg SSH rule](screenshots/05_web_sg_ssh_rule.png)

### 4. Create the RDS MySQL database
Create `student-db` with engine MySQL 8.4, initial database `studentdb`, the `student-db-subnet-group`, security group `db-sg`, and **Public access: No**.

### 5. Deploy the application on EC2
On the Ubuntu 24.04 instance in the public subnet:
- Install the Flask and PyMySQL application.
- Store the database credentials in `/etc/student-app.env` with permission 600.
- Run the app as the systemd service `student-app` on port 80.

![student-app service](screenshots/06_student_app_service_status.png)

## Testing and Results

**Registration form**

![Registration form](screenshots/07_registration_form.png)

**Insertion and retrieval in the web page**

![Registered students](screenshots/08_registered_students.png)

**Data read directly from RDS on the EC2 prompt** (`SELECT * FROM students`)

![SELECT on EC2](screenshots/09_select_students_ec2.png)

| Test | Result |
|---|---|
| Open EC2 public IP over HTTP | Form and empty table loaded |
| Register 3 students | Each record saved and listed in the table |
| Reload the page | All 3 records returned from RDS |
| `SELECT * FROM students` on EC2 | Same 3 rows read directly from the database |

## Conclusion
The app on EC2 stores and retrieves student records in a private RDS MySQL database that only `web-sg` can reach.

## Cleanup
To avoid charges: stop or terminate the EC2 instance and delete the RDS instance (`student-db`) when finished.
