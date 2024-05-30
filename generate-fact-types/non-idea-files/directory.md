# Directory

The Directory Fact Type within Generate encompasses critical datasets represented by files c029 and c039. These files play a pivotal role in organizing and managing educational information. Below, you will find detailed information on handling the Non-IDEA Directory Fact Type, specifically files c035, c103, c129, c130, c131, c163, c170, c190, c193, c196, c197, c198, c205, and c206, which can be found in Non-IDEA Files --> Directory.

**Files Included:**

* [x] c029
* [x] c039
* [x] c035
* [x] c103
* [x] c129
* [x] c130
* [x] c131
* [x] c163
* [x] c170
* [x] c190
* [x] c193
* [x] c196
* [x] c197
* [x] c198
* [x] c205
* [x] c206

***

### <img src="../../.gitbook/assets/CEDS logo.png" alt="" data-size="line"> **CEDS Connection:**&#x20;

The Non-IDEA Directory Fact Type is closely aligned with the Common Education Data Standards (CEDS). Ensure that your data adheres to CEDS specifications for seamless integration and standardized reporting.

{% embed url="https://github.com/CEDStandards/CEDS-IDS" %}

***

### <img src="../../.gitbook/assets/Toolkit Logo.png" alt="" data-size="line"> **Toolkit Step 5 - CEDS Align**

Make use of the work done in Steps 3 and 4 where elements and their locations were documented in the CEDS Align tool. The CEDS Align maps can assist in writing the ETL transformation process, providing information about the type of transformation required.

### **ETL Checklist:**&#x20;

Refer to the [ETL checklist](https://ciidta.communities.ed.gov/#communities/pdc/documents/17074) provided to guide you through the Extract, Transform, and Load (ETL) process for the Non-IDEA Directory Fact Type. Ensure meticulous adherence to each step for accurate and reliable data outcomes.

***

### <img src="../../.gitbook/assets/Generate_logo_900px (1).png" alt="" data-size="line"> **Toggle Questions:**&#x20;

_Insert relevant toggle questions for the Non-IDEA Directory Fact Type._

***

### <img src="../../.gitbook/assets/Generate_logo_900px (2).png" alt="" data-size="line"> **Staging Validations:**&#x20;

Before proceeding with data processing, run the following staging validations to identify and address potential issues:

```sql
sqlCopy code-- Staging Validation SQL for Non-IDEA Directory Fact Type

-- Check if required fields are not null
SELECT
    COUNT(*)
FROM
    StagingNonIDEADirectory
WHERE
    c035 IS NULL
    OR c103 IS NULL
    -- Continue for other fields;

-- Check if numeric values are within a specified range
SELECT
    COUNT(*)
FROM
    StagingNonIDEADirectory
WHERE
    NOT (
        (c035 >= 0 AND c035 <= 100)
        AND (c103 >= 0 AND c103 <= 100)
        -- Continue for other fields
    );

-- Check for duplicate records based on a unique identifier (replace 'unique_id' with the actual unique identifier column)
SELECT
    unique_id,
    COUNT(*)
FROM
    StagingNonIDEADirectory
GROUP BY
    unique_id
HAVING
    COUNT(*) > 1;
```

### <img src="../../.gitbook/assets/Generate_logo_900px (2).png" alt="" data-size="line"> **Special Requirements for Staging:**&#x20;

Be aware of any unique considerations when loading data into staging for the Non-IDEA Directory Fact Type. This may include specific formatting requirements, data integrity checks, or other factors crucial for a successful staging process.

* **Data Integrity:**
  * Enforce formatting guidelines and validate date formats.
  * Maintain referential integrity and validate identifiers.
* **Validation and Consistency:**
  * Implement cross-field validation for data consistency.
  * Confirm unique identifiers and handle missing values appropriately.
* **Security and Controls:**
  * Enable robust error logging and reporting mechanisms.
  * Implement encryption for sensitive data.
  * Establish versioning and rollback procedures for data integrity.

### **Additional Information:**

* _Include any additional information or notes specific to the Non-IDEA Directory Fact Type._
* _Link to relevant external resources or documentation._
