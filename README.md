# Lab 5 — Assignment 1: RDS PostgreSQL Deployment & Connection

## Overview

This assignment demonstrates the migration of the Mealie application's database from a local PostgreSQL installation running on an Amazon EC2 instance to a managed Amazon RDS PostgreSQL database.

The existing Mealie application was retained on the EC2 instance, while the PostgreSQL database was migrated to Amazon RDS. After migration, the application was reconfigured to connect to RDS, and CRUD operations were tested successfully.

---

## Architecture

```text
Client
  |
  | SSH Tunnel / HTTP
  v
Amazon EC2 Instance
  |
  | Docker Container
  | Mealie Application
  | 127.0.0.1:9000
  |
  | PostgreSQL Connection
  | TCP Port 5432
  v
Amazon RDS PostgreSQL
  |
  v
Mealie Database
```

The EC2 instance continues to host the Mealie application using Docker. The database layer was changed from local PostgreSQL to Amazon RDS PostgreSQL.

---

## EC2 Configuration

| Setting | Value |
|---|---|
| Instance Name | `mealie-lab4` |
| Operating System | Ubuntu Linux |
| Application | Mealie |
| Container Runtime | Docker |
| Mealie Version | `v3.24.0` |
| Application Port | `9000` |
| Application Binding | `127.0.0.1:9000` |
| AWS Region | `eu-north-1` |

The Mealie application was originally deployed on the EC2 instance during Lab 4.

---

## RDS Configuration

| Setting | Value |
|---|---|
| DB Instance Identifier | `mealie-lab5-rds` |
| Database Engine | PostgreSQL |
| Database Name | `mealie` |
| Master Username | `mealie_admin` |
| Port | `5432` |
| Public Access | Disabled |
| Endpoint | `mealie-lab5-rds.cxkmac80k9i1.eu-north-1.rds.amazonaws.com` |

> Database passwords and other sensitive credentials have been redacted and are not included in this repository.

---

## Security Group Configuration

A security group was configured for the RDS instance to allow PostgreSQL traffic only from the EC2 instance's security group.

| Type | Protocol | Port | Source |
|---|---|---|---|
| PostgreSQL | TCP | 5432 | EC2 Security Group |

The RDS security group does not allow access from `0.0.0.0/0`.

This ensures that the database is not directly accessible from the public internet and can only be accessed by the authorized EC2 instance.

---

## Database Migration

The local PostgreSQL database used by Mealie was migrated to Amazon RDS PostgreSQL.

### Step 1: Export the Local Database

The local PostgreSQL database was exported using `pg_dump`.

```bash
pg_dump \
  -U mealie_user \
  -h 127.0.0.1 \
  -d mealie_db \
  > mealie-migration.sql
```

### Step 2: Restore the Database into RDS

The exported SQL file was restored into the RDS PostgreSQL database.

```bash
psql \
  -h mealie-lab5-rds.cxkmac80k9i1.eu-north-1.rds.amazonaws.com \
  -U mealie_admin \
  -d mealie \
  -f mealie-migration.sql
```

The RDS database was checked after restoration to verify that the migrated tables and data were present.

The restored database contained the Mealie application tables, confirming that the migration was completed successfully.

---

## Application Reconnection

After the database migration, the Mealie environment configuration was updated to use the RDS PostgreSQL endpoint.

Example configuration:

```env
DB_ENGINE=postgres
POSTGRES_SERVER=mealie-lab5-rds.cxkmac80k9i1.eu-north-1.rds.amazonaws.com
POSTGRES_PORT=5432
POSTGRES_DB=mealie
POSTGRES_USER=mealie_admin
POSTGRES_PASSWORD=<redacted>
```

The Mealie Docker container was restarted after updating the configuration.

```bash
docker restart mealie
```

The application was then checked using:

```bash
curl -s http://127.0.0.1:9000/api/app/about
```

A successful response confirmed that the Mealie application was running after reconnecting to the RDS database.

---

## Accessing the Mealie Application

Since the application was bound to `127.0.0.1:9000` on the EC2 instance, an SSH tunnel was used to access it from the local computer.

```bash
ssh -i <PRIVATE_KEY.pem> \
  -L 9000:127.0.0.1:9000 \
  ubuntu@<EC2_PUBLIC_IP>
```

The application was then accessed in a browser using:

```text
http://localhost:9000
```

---

## CRUD Demonstration

CRUD operations were performed through the running Mealie application after connecting it to Amazon RDS PostgreSQL.

| Operation | Description | Result |
|---|---|---|
| Create | Added a new test recipe | Successful |
| Read | Viewed an existing recipe | Successful |
| Update | Edited recipe details | Successful |
| Delete | Deleted the test recipe | Successful |

These operations confirmed that Mealie was successfully reading from and writing to the RDS PostgreSQL database.

---

## Verification

The following checks were completed:

- Mealie Docker container was running.
- RDS PostgreSQL instance was available.
- RDS endpoint was configured in Mealie.
- RDS connectivity was verified.
- Database tables were present after migration.
- Existing application data was retained.
- Create operation was tested.
- Read operation was tested.
- Update operation was tested.
- Delete operation was tested.
- RDS was not publicly accessible.

---

## Final Architecture

```text
                         User Browser
                              |
                              |
                       SSH Port Forwarding
                              |
                              v
                    Amazon EC2 Instance
                    -------------------
                    Ubuntu Linux
                    Docker
                    Mealie Application
                    Port 9000
                              |
                              | TCP 5432
                              | Private VPC Connection
                              v
                    Amazon RDS PostgreSQL
                    --------------------
                    Database: mealie
                    Port: 5432
```

---

## Security Considerations

- RDS public access was disabled.
- Port `5432` was restricted to the EC2 security group.
- No `0.0.0.0/0` rule was configured for PostgreSQL.
- Database credentials were not committed to GitHub.
- The EC2 private key was not uploaded.
- Sensitive screenshots were excluded or redacted.
- Environment variables were used for database configuration.

---

## Repository Structure

```text
mealie-lab5/
│
├── README.md
│
├── evidence/
│   ├── ec2-instance.png
│   ├── rds-instance.png
│   ├── rds-security-group.png
│   ├── rds-connectivity.png
│   ├── database-tables.png
│   ├── mealie-running.png
│   ├── crud-create.png
│   ├── crud-read.png
│   ├── crud-update.png
│   └── crud-delete.png
│
└── report/
    └── Cloud_Computing_Lab5_Report.pdf
```

---

## Conclusion

The Mealie application's database was successfully migrated from a local PostgreSQL installation on Amazon EC2 to Amazon RDS PostgreSQL.

The Mealie application continued running on EC2 and successfully connected to the RDS database. The migrated data was verified, and all four CRUD operations were performed successfully.

This lab demonstrates the use of Amazon RDS as a managed, secure, and reliable database service for a cloud-hosted application.

---

## Author

**Name:** Armaan Syed  
**Roll Number:** 10703  
**Branch:** Computer Engineering  
**College:** Fr. Conceicao Rodrigues College of Engineering