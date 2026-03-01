# DB2 z/OS (Mainframe) Training Plan

## Overview

This training plan is designed for DB2 professionals transitioning from **DB2 on Linux, Unix, and Windows (LUW)** to **DB2 on z/OS (mainframe)**. You already know DB2, SQL, Linux, Unix, and Windows — now let's leverage that knowledge while addressing the key differences.

---

## Key Differences: DB2 z/OS vs LUW

### The Good News
Your SQL skills (SELECT, INSERT, UPDATE, DELETE) transfer well! DML is very similar and getting closer with each release.

### Architecture & Environment

| Aspect | LUW | z/OS (Mainframe) |
|--------|-----|------------------|
| **Instance** | Single DB2 instance per server | Multiple subsystems (DB2, CICS, IMS) running as *address spaces* under z/OS |
| **Operating System** | Linux, AIX, Windows | z/OS (proprietary IBM OS) |
| **Storage** | SAN, local disks | DASD (Direct Access Storage Devices), datasets |
| **Object Names** | SCHEMA.TABLE | Three-part: `location.schema.table` or implicit |
| **Tools** | Data Studio, Toad, DBeaver, IBM Data Studio | SPUFI, DSNTIUL, IBM Data Studio, BMC Mainview, CA |
| **Recovery** | Backup/restore | Log-based (archive logs), point-in-time recovery |

### Database Objects

| Aspect | LUW | z/OS |
|--------|-----|------|
| **Catalog** | SYSCAT views (e.g., SYSCAT.TABLES) | SYSIBM.* tables (e.g., SYSIBM.SYSTABLES) |
| **Tablespaces** | SMS-managed | VSAM datasets, managed by z/OS |
| **Partitioning** | Range partitioning, MDC | Partitioned tablespaces (different syntax) |
| **Indexes** | Standard B-tree | Similar but with partition-by-growth options |
| **Schemas** | Created explicitly | Often implicit (authID becomes schema) |

### SQL Differences

| Feature | LUW | z/OS |
|---------|-----|------|
| **NULL handling** | Standard | Slightly different default behavior |
| **VARCHAR** | Standard | Requires additional considerations |
| **Sequences** | Native | Available but syntax differs slightly |
| **Stored Procedures** | Native SQL, external (Java, etc.) | Mostly COBOL/Assembler, plus SQL PL |
| **XML** | Native XML support | Different implementation |
| **Triggers** | Standard | More restrictive syntax |

### Security

| Aspect | LUW | z/OS |
|--------|-----|------|
| **Authentication** | OS + DB | RACF (or ACF2, TopSecret) at mainframe level |
| **Authorization** | DB grants | DB grants + mainframe security (RACF) |
| **Connection** | TCP/IP | CICS, IMS, or DB2 attach |

---

## Training Plan

### Week 1-2: z/OS Fundamentals

**Goal:** Understand the mainframe environment

- [ ] **IBM z/OS Basics** — IBM z/OS is the operating system; DB2 runs as a subsystem
  - Read: [IBM z/OS Overview](https://www.ibm.com/products/zos)
  - Understand: LPARs (Logical Partitions), z/VM, JES (Job Entry Subsystem)
- [ ] **TSO/ISPF Basics** — Mainframe TSO (Time Sharing Option) and ISPF (Interactive System Productivity Facility)
  - This is your "shell" on mainframe
- [ ] **JCL (Job Control Language)** — How jobs are submitted on mainframe
  - You'll need this for running DB2 utilities
- [ ] **Coursera:** [IBM z/OS Mainframe Practitioner Professional Certificate](https://www.coursera.org/professional-certificates/ibm-z-mainframe)

**Resources:**
- [IBM z/OS Documentation](https://www.ibm.com/docs/en/)
- [Wikipedia: z/OS](https://en.wikipedia.org/wiki/Z/OS)

---

### Week 3: DB2 z/OS Architecture

**Goal:** Understand DB2 on z/OS specifically

- [ ] **DB2 Subsystem Concepts**
  - Buffer pools, EDM pool, SORT pool
  - Dataset types: VSAM, linear datasets
  - DB2 address spaces: IRLM, DBM1, DIST
- [ ] **Catalog Structure**
  - SYSIBM.* tables vs SYSCAT views
  - Key tables: SYSIBM.SYSTABLES, SYSIBM.SYSCOLUMNS, SYSIBM.SYSINDEXES
- [ ] **DB2 Objects**
  - Tablespaces, indexspaces, partitions
  - Tables, views, indexes, aliases, synonyms
  - Plans, packages, collections
- [ ] **Tools: SPUFI & DSNTIUL**
  - SPUFI: SQL Processing Using File Input (like your query tool)
  - DSNTIUL: Unload utility

**Resources:**
- [IBM DB2 z/OS Documentation](https://www.ibm.com/docs/en/db2_for_z_os)

---

### Week 4: SQL on z/OS

**Goal:** Apply your SQL skills with z/OS-specific differences

- [ ] **DDL Differences**
  - CREATE TABLE syntax differs (physical storage parameters!)
  - PRIMARY KEY vs UNIQUE constraints
  - Foreign key enforce behavior
- [ ] **DML Practice**
  - SELECT, INSERT, UPDATE, DELETE — similar but test!
  - Predicates, joins, subqueries
- [ ] **NULL Handling**
  - Different default behavior
  - Testing NULL vs empty strings
- [ ] **Dynamic vs Static SQL**
  - z/OS heavily uses static SQL (pre-compiled)
  - Explain plans work differently

**Resources:**
- [SQL differences for DB2 versions (Stack Overflow)](https://stackoverflow.com/questions/27146051/sql-differences-for-db2-versions)
- [Compare DB2 for z/OS and DB2 for LUW (IBM Developer)](https://developer.ibm.com/articles/dm-1108compdb2luwzos/)

---

### Week 5: Storage & Partitioning

**Goal:** Master data organization on mainframe

- [ ] **Tablespace Types**
  - Simple, segmented, partitioned, partition-by-growth
  - When to use each
- [ ] **Partitioning**
  - Partitioned tablespaces (key range)
  - Partition-by-growth (PBG)
  - Different from LUW range partitioning!
- [ ] **Index Design**
  - Clustering indexes
  - Partitioned vs non-partitioned indexes
- [ ] **Data Compression**
  - Dictionary-based compression
  - Different from LUW

**Resources:**
- [DB2 z/OS Table Partitions (Broadcom)](https://techdocs.broadcom.com/us/en/ca-mainframe-software/devops/ca-gen/8-6/developing/designing/creating-db2-z-os-table-partitions.html)

---

### Week 6: Operations & Administration

**Goal:** Day-to-day DBA tasks on z/OS

- [ ] **DB2 Utilities**
  - RUNSTATS — Collect statistics for optimizer
  - REORG — Reorganize tables/indexes
  - CHECK — Check integrity
  - COPY — Backup
  - RECOVER — Restore
- [ ] **Security**
  - RACF basics (or ACF2, TopSecret)
  - GRANT/REVOKE on z/OS
  - TRUSTED CONTEXT
- [ ] **Monitoring**
  - DB2 Explain
  - SQL traces (DB2 Performance Expert)
  - Buffer pool monitoring

**Resources:**
- [DB2 z/OS Utilities Guide](https://www.ibm.com/docs/en/db2_for_z_os)

---

### Week 7-8: Role-Specific Prep

**Goal:** Prepare for your financial services role

- [ ] **Learn Your Employer's Tools**
  - Likely BMC Mainview, CA (Broadcom) tools, or IBM OMEGAMON
  - Understanding their monitoring stack
- [ ] **COBOL + DB2 (if needed)**
  - Many mainframe apps embed SQL in COBOL
  - Understand embedded SQL concepts
- [ ] **Transaction Processing**
  - CICS (Customer Information Control System) — common on mainframe
  - IMS — IBM's other transaction manager
- [ ] **Practice with Sample Data**
  - Use SPUFI to run queries
  - Practice DDL and understand storage implications

---

## Quick Reference Card

### Common SQL Differences

```sql
-- LUW:
CREATE TABLE myschema.mytable (id INT PRIMARY KEY, name VARCHAR(100));

-- z/OS:
CREATE TABLE myschema.mytable (
    id INT NOT NULL,
    name VARCHAR(100)
) IN myts1;
```

### Key Catalog Queries

```sql
-- z/OS: Find tables
SELECT NAME, CREATOR, TYPE
FROM SYSIBM.SYSTABLES
WHERE CREATOR = 'MYAUTH';

-- Equivalent LUW:
SELECT TABNAME, OWNER, TYPE
FROM SYSCAT.TABLES
WHERE OWNER = 'MYSCHEMA';
```

---

## Recommended Resources

### Online Courses
- [IBM z/OS Mainframe Practitioner (Coursera)](https://www.coursera.org/professional-certificates/ibm-z-mainframe)

### Documentation
- [IBM z/OS Documentation](https://www.ibm.com/docs/en/)
- [IBM DB2 for z/OS](https://www.ibm.com/docs/en/db2_for_z_os)
- [IBM Developer: DB2 z/OS vs LUW](https://developer.ibm.com/articles/dm-1108compdb2luwzos/)

### Communities
- [IBM Mainframes Forum](https://ibmmainframes.com/)
- [/r/mainframe (Reddit)](https://www.reddit.com/r/mainframe/)
- [Stack Overflow: DB2](https://stackoverflow.com/questions/tagged/db2)

---

*Generated for Tapio's transition to DB2 z/OS at a Finnish financial services company*
