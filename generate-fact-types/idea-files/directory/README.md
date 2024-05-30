# Directory

## **Overview**

The Directory Fact Type within Generate encompasses critical datasets represented by files c029 and c039. These files play a pivotal role in organizing and managing educational information. Below, you will find detailed information on handling the Directory Fact Type, including toggle questions, CEDS connections, staging validations, and additional considerations for states engaging with these specific files.

## **Files Included**

* [x] **c029**
* [x] **c039**

{% hint style="warning" %}
**Non-IDEA Directory Files - c035, c103, c129, c130, c131, c163, c170, c190, c193, c196, c197, c198, c205, c206 can be found in** [<mark style="color:orange;">**Non-IDEA Files --> Directory**</mark>](../../non-idea-files/directory.md)**.**
{% endhint %}

***

### **ETL Checklist**

Refer to the ETL checklist provided here to guide you through the Extract, Transform, and Load (ETL) process for the Directory Fact Type. Ensure that each step is meticulously followed to guarantee accurate and reliable data outcomes.

[https://ciidta.communities.ed.gov/#communities/pdc/documents/17074](https://ciidta.communities.ed.gov/#communities/pdc/documents/17074)

***

### **Toggle Questions**

<details>

<summary><em>Insert relevant toggle questions for Directory Fact Type.</em></summary>

_Insert relevant toggle answer for Directory Fact Type._

</details>

<details>

<summary><em>Insert relevant toggle questions for Directory Fact Type.</em></summary>

_Insert relevant toggle answer for Directory Fact Type._

</details>

<details>

<summary><em>Insert relevant toggle questions for Directory Fact Type.</em></summary>

_Insert relevant toggle answer for Directory Fact Type._

</details>

***

### **Staging Validations**

Before proceeding with data processing, run the following staging validations to identify and address potential issues:

{% code overflow="wrap" fullWidth="false" %}
```sql
-- Staging Validation SQL for Directory Fact Type

-- Check if required fields are not null
SELECT
COUNT(*)
FROM
StagingDirectory
WHERE
c035 IS NULL
OR c103 IS NULL
OR c129 IS NULL
OR c130 IS NULL
OR c131 IS NULL
OR c163 IS NULL
OR c170 IS NULL
OR c190 IS NULL
OR c193 IS NULL
OR c196 IS NULL
OR c197 IS NULL
OR c198 IS NULL
OR c205 IS NULL
OR c206 IS NULL;

-- Check if numeric values are within a specified range
SELECT
COUNT(*)
FROM
StagingDirectory
WHERE
NOT (
(c035 >= 0 AND c035 <= 100)
AND (c103 >= 0 AND c103 <= 100)
AND (c129 >= 0 AND c129 <= 100)
-- Continue this pattern for other fields
);

-- Check for duplicate records based on a unique identifier (replace 'unique_id' with the actual unique identifier column)
SELECT
unique_id,
COUNT(*)
FROM
StagingDirectory
GROUP BY
unique_id
HAVING
```
{% endcode %}

***

### **Special Requirements for Staging**

Be aware of any unique considerations when loading data into staging for the Directory Fact Type. This may include specific formatting requirements, data integrity checks, or other factors crucial for a successful staging process.

1. **Data Integrity:**
   * Enforce formatting guidelines and validate date formats.
   * Maintain referential integrity and validate identifiers.
2. **Validation and Consistency:**
   * Implement cross-field validation for data consistency.
   * Confirm unique identifiers and handle missing values appropriately.
3. **Security and Controls:**
   * Enable robust error logging and reporting mechanisms.
   * Implement encryption for sensitive data.
   * Establish versioning and rollback procedures for data integrity.

***

### **Additional Information**

* CEDS Connection - The Directory Fact Type is closely aligned with the Common Education Data Standards (CEDS). Ensure that your data adheres to CEDS specifications for seamless integration and standardized reporting.
* GitHub
* TOOLKIT STEP 5 - Make use of the work done in Steps 3 and 4 where elements and their locations were documented in the CEDS Align tool as well as the master integrated data set. The CEDS Align maps can be used to assist in writing the ETL transformation process. The Align tool provides the user the ability to compare one map with another. Comparing the source system map(s) with the destination system map(s) will provide information about the type of transformation required.

\
