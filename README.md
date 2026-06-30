# 🗄️ Oracle Pluggable Database (PDB) Administration — Assignment II

<div align="center">

![Oracle](https://img.shields.io/badge/Oracle-21c_XE-F80000?style=for-the-badge&logo=oracle&logoColor=white)
![SQL*Plus](https://img.shields.io/badge/SQL*Plus-CLI-4479A1?style=for-the-badge&logo=database&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-Windows-0078D4?style=for-the-badge&logo=windows&logoColor=white)

</div>

---

## 📋 Student Information

| Field | Details |
|-------|---------|
| **Full Name** | Eugene |
| **Student ID** | 32327/2025 |
| **Course** | C11665 – DPR400210: Database Programming |
| **Instructor** | Eric Maniraguha |
| **Institution** | University of Lay Adventists of Kigali (UNILAK) |
| **Submission Date** | June 30, 2026 |

---

## 📌 Assignment Overview

This assignment demonstrates hands-on proficiency in **Oracle Multitenant Architecture** and **Pluggable Database (PDB) Administration** using Oracle Database 21c Express Edition. The work covers the complete lifecycle of a Pluggable Database — from creation and user management to deletion and monitoring via Oracle Enterprise Manager (OEM) Express.

The practical skills acquired here form the foundation for future PL/SQL development, database administration tasks, and Oracle-based laboratory work throughout the course.

---

## 🖥️ Oracle Environment

| Component | Details |
|-----------|---------|
| **Oracle Version** | Oracle Database 21c Express Edition Release 21.0.0.0.0 — Production (21.3.0.0.0) |
| **Operating System** | Windows 11 (64-bit) |
| **Database Instance** | XE (Container Database — CDB) |
| **Listener Host** | 192.168.10.40 — Port 1521 |
| **OEM Express Port** | 5500 (CDB$ROOT) / 5502 (PDB) |
| **Tools Used** | SQL*Plus 21c, Oracle Enterprise Manager Express, Windows Command Prompt (netstat, lsnrctl) |

---

## 📂 Repository Structure

```
oracle_pdb_assignment2_32327_2025_eugene/
│
├── README.md
│
└── screenshots/
    ├── pdb_creation.png              # Task 1 — PDB created successfully
    ├── pdb_open_status.png           # Task 1 — PDB opened and verified
    ├── user_creation.png             # Task 1 — User created inside PDB
    ├── privileges_granted.png        # Task 1 — Privileges assigned
    ├── user_login.png                # Task 1 — Successful login as created user
    ├── temporary_pdb_creation.png    # Task 2 — Temp PDB created
    ├── temporary_pdb_deletion.png    # Task 2 — Temp PDB dropped
    └── oem_dashboard.png             # Task 3 — OEM Express dashboard
```

---

## ⚙️ Task Documentation

### Task 1 — Create and Configure a Personal PDB

**PDB Name:** `eu_pdb_32327_2025`  
**User Created:** `eugene_plsqlauca_32327_2025`

#### Step-by-Step Process

**1. Connected to Oracle as SYSDBA:**
```sql
sqlplus / as sysdba
```

**2. Verified current container and existing PDBs:**
```sql
SHOW CON_NAME;
SELECT name, open_mode FROM v$pdbs;
```

**3. Created the PDB from the PDB seed:**
```sql
CREATE PLUGGABLE DATABASE eu_pdb_32327_2025
  ADMIN USER pdbadmin IDENTIFIED BY ********
  FILE_NAME_CONVERT = (
    '/opt/oracle/oradata/XE/pdbseed/',
    '/opt/oracle/oradata/XE/eu_pdb_32327_2025/'
  );
```

**4. Opened the PDB and saved its state:**
```sql
ALTER PLUGGABLE DATABASE eu_pdb_32327_2025 OPEN;
ALTER PLUGGABLE DATABASE eu_pdb_32327_2025 SAVE STATE;
```
`SAVE STATE` ensures the PDB auto-opens on every database restart without manual intervention.

**5. Switched into the PDB and created the user:**
```sql
ALTER SESSION SET CONTAINER = eu_pdb_32327_2025;

CREATE USER eugene_plsqlauca_32327_2025 IDENTIFIED BY ********
  DEFAULT TABLESPACE users
  QUOTA UNLIMITED ON users;
```

**6. Granted all required privileges:**
```sql
GRANT CREATE SESSION, CREATE TABLE, CREATE VIEW, CREATE SEQUENCE,
      CREATE PROCEDURE, CREATE TRIGGER, RESOURCE, CONNECT
  TO eugene_plsqlauca_32327_2025;
```

**7. Verified login using the new user:**
```sql
sqlplus eugene_plsqlauca_32327_2025/********@//localhost:1521/eu_pdb_32327_2025

SELECT USER FROM dual;
SHOW CON_NAME;
```
Both commands confirmed the session was active inside `eu_pdb_32327_2025` as the correct user.

---

### Task 2 — Create and Delete a Temporary PDB

**Temporary PDB Name:** `eu_to_delete_pdb_32327_2025`

**1. Created the temporary PDB:**
```sql
ALTER SESSION SET CONTAINER = CDB$ROOT;

CREATE PLUGGABLE DATABASE eu_to_delete_pdb_32327_2025
  ADMIN USER tempadmin IDENTIFIED BY ********
  FILE_NAME_CONVERT = (
    '/opt/oracle/oradata/XE/pdbseed/',
    '/opt/oracle/oradata/XE/eu_to_delete_pdb_32327_2025/'
  );
```

**2. Verified it exists:**
```sql
SELECT name, open_mode FROM v$pdbs
  WHERE name = 'EU_TO_DELETE_PDB_32327_2025';
```

**3. Opened the PDB:**
```sql
ALTER PLUGGABLE DATABASE eu_to_delete_pdb_32327_2025 OPEN;
```

**4. Closed and dropped it including all datafiles:**
```sql
ALTER PLUGGABLE DATABASE eu_to_delete_pdb_32327_2025 CLOSE IMMEDIATE;
DROP PLUGGABLE DATABASE eu_to_delete_pdb_32327_2025 INCLUDING DATAFILES;
```
The `INCLUDING DATAFILES` clause is critical — it removes both the Oracle metadata and the physical files from disk, leaving no orphaned data.

**5. Confirmed deletion:**
```sql
SELECT name FROM v$pdbs
  WHERE name = 'EU_TO_DELETE_PDB_32327_2025';
-- 0 rows returned
```

---

### Task 3 — Oracle Enterprise Manager (OEM) Express

**Access URL:** `https://localhost:5500/em` (CDB$ROOT)

OEM Express was configured and accessed as follows:

- The XDB HTTPS listener was confirmed active on port 5500 for `CDB$ROOT` using:
```sql
SELECT DBMS_XDB_CONFIG.GETHTTPSPORT FROM dual;
-- Returns: 5500
```
- A separate HTTPS port (5502) was configured for the PDB `eu_pdb_32327_2025`:
```sql
ALTER SESSION SET CONTAINER = eu_pdb_32327_2025;
EXEC DBMS_XDB_CONFIG.SETHTTPSPORT(5502);
```
- OEM Express was accessed at `https://localhost:5500/em`, showing the Oracle database environment, the CDB instance (`XE`), and the pluggable database `eu_pdb_32327_2025` under Containers.

---

## ⚠️ Challenges and Solutions

| # | Challenge | Solution |
|---|-----------|----------|
| 1 | `ORA-44718: Port conflict in XDB Configuration file` when trying to set port 5500 on the PDB | CDB$ROOT already held port 5500. Assigned a different port (5502) to the PDB to avoid the conflict. |
| 2 | `https://localhost:5500/em` showed `ERR_CONNECTION_REFUSED` in the browser | The XDB HTTP listener thread was not binding after configuration. Restarted the database (`SHUTDOWN IMMEDIATE` / `STARTUP`) and confirmed the port using `netstat -an \| findstr 5500`. |
| 3 | `lsnrctl status` typed inside SQL*Plus returned `SP2-0734: unknown command` | `lsnrctl` is an OS-level command, not SQL. Exited SQL*Plus and ran it from the Windows Command Prompt. |
| 4 | `ORA-01017: invalid username/password` when trying to connect to the PDB | The PDB name was accidentally entered in the username field instead of the user account. Reconnected correctly using `sqlplus / as sysdba`. |
| 5 | PDB-level XDB HTTPS port (5502) not showing in `netstat` after being set | Oracle XE's XDB HTTP listener requires a PDB close/open cycle to bind the new port. Ran `CLOSE IMMEDIATE` → `OPEN` → `SAVE STATE` on the PDB to force re-initialization. |

---

## 📚 Lessons Learned

- **Oracle Multitenant Architecture** separates the Container Database (CDB) from Pluggable Databases (PDBs), each operating as an isolated database sharing the same Oracle instance and SGA.
- The **`SAVE STATE`** option is essential for production PDB management — without it, PDBs default to `MOUNTED` state after a CDB restart and require manual opening.
- **`DROP PLUGGABLE DATABASE ... INCLUDING DATAFILES`** is the correct and complete deletion command — omitting `INCLUDING DATAFILES` leaves orphaned physical files on disk.
- **Port management in multitenant** environments is non-trivial — each container manages its own XDB HTTPS configuration, and port conflicts arise when multiple containers compete for the same port number.
- **OEM Express** is a lightweight web-based monitoring tool embedded inside the Oracle database itself, useful for quickly inspecting instance health, PDB status, and database configuration without installing a separate tool.
- Reading OS-level network output (`netstat`) alongside Oracle SQL diagnostics is a critical DBA habit for verifying that configuration changes have actually taken effect at the system level.

---

## ✅ Submission Checklist

- [x] PDB `eu_pdb_32327_2025` created successfully
- [x] PDB opened and state saved
- [x] User `eugene_plsqlauca_32327_2025` created inside PDB
- [x] All required privileges granted
- [x] Successful login verified with created user
- [x] Temporary PDB `eu_to_delete_pdb_32327_2025` created
- [x] Temporary PDB verified, opened, and completely deleted
- [x] OEM Express accessed and configured
- [x] All screenshots captured and uploaded
- [x] README.md completed
- [x] GitHub repository is PUBLIC
- [x] Google Form submitted before deadline

---

## 🔏 Integrity Statement

> *"I confirm that this assignment represents my own practical work, screenshots, and documentation. All external resources consulted have been properly acknowledged."*

---

<div align="center">

*University of Lay Adventists of Kigali (UNILAK) — Database Programming DPR400210 — 2026*

</div>
