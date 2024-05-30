# Page 1

Generate Version 11.4 introduces a new and improved Staging Validation process. This new validation process includes an expanded library of validation rules and makes it easier to create and manage custom validation rules.

## Staging Validation Overview

_This section provides a quick overview of the Staging Validation process.  More details are provided in subsequent sections._

Once data has been loaded into the Generate Staging tables, the Staging Validation process can be executed within SQL Server Management Studio (SSMS).  This process scans the data in corresponding staging tables and applies validation rules to determine if any data have issues.

A Staging Validation rule can apply to a particular column in a staging table or can be defined to apply validation logic across multiple columns/conditions.  A rule can be associated to one or more ED_Facts_ reports.  For example, Generate includes a rule that column `StudentIdentifierState` in table `Staging.K12Enrollment` is required.  This rule is associated to all ED_Facts_ reports that produce student counts.

Generate currently includes a library of over fifty pre-defined staging validation rules, located in table `Staging.StagingValidationRules`.  These rules are used in over 1,000 instances across multiple Fact Types and Report Codes.  The full list of rules and relationships can be viewed in `Staging.vwStagingValidationRules`.

Rules can be defined, added, and associated at any time by the Generate developers and State users.  More information for how to add rules is later in this document.

### To run the Staging Validation and review the results, two steps are required:

1. Load data into the Generate Staging Tables, then execute the following procedure in SSMS (with the appropriate parameter values) to run the validation process:

```sql
exec staging.StagingValidation_Execute
@SchoolYear = 2024,
@FactTypeOrReportCode = 'childcount',
@RemoveHistory = 1,
@PrintSQL = 0
```

<table><thead><tr><th width="252">Parameters</th><th>Description</th></tr></thead><tbody><tr><td><strong>@SchoolYear</strong></td><td>The school year that corresponds to the data in staging.</td></tr><tr><td><strong>@FactTypeOrReportCode</strong></td><td>Either an ED<em>Facts</em> Report Code (i.e., C029, C002, C005) or a Generate Fact Type (i.e., Directory, ChildCount, Exiting). If an incorrect value is provided, the query will return a list of valid values.</td></tr><tr><td><strong>@RemoveHistory</strong></td><td>Determines if the validation results will replace historical validation results for the same school year and fact type/report code or will append to the existing results.  This is an optional parameter, and if not supplied will default to 0 (do not remove history).</td></tr><tr><td><strong>@PrintSQL</strong></td><td>A debugging capability that will display the dynamic SQL that will run to perform the staging validation.  This is an optional parameter and if not supplied will default to 0 (do not show the SQL).</td></tr></tbody></table>

2. To view the results of the validation process, execute the following procedure in SSMS (with the appropriate parameter values):

```sql
exec staging.StagingValidation_GetResults
@SchoolYear = 2024,
@FactTypeOrReportCode = 'childcount',
@IncludeHistory = 0
```

&#x20;

| Parameters      | Description                                                                                                                                                                                                                                                                                                                                                                            |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| @IncludeHistory | will include all history of the validation results for the school year and fact type/report code as well as the latest results (assuming the history is still available) based on the @RemoveHistory parameter value used when executing the Staging Validation process.  This is an optional parameter and if not provided will default to 0 (do not include history in the results). |

&#x20;Results of the Staging Validation process will be displayed in SSMS, similar to the example below:

<figure><img src=".gitbook/assets/image (189).png" alt=""><figcaption></figcaption></figure>

The results can be reviewed to determine what, if any, action is required.  If no results are returned, it means the staging data passed all existing validation rules.

The Staging Validation results include the following columns:

Id – this is just an internal Id for the result record

StagingValidationRuleId – this is the Rule Id that was processed to produce the result.  There may be some auto-generated results that do not pertain to a rule:

·       -1 This indicates that a staging table is required for the specific report and cannot be empty

·       -2 This indicates that an option set value in a staging table is not mapped in the Generate Staging.SourceSystemReferenceData table.

·       -9 This indicates that the defined rule has a syntax error and could not be executed.  Any results having a -9 should be reviewed by the rule developer.

SchoolYear – the school year for which the result applies

FactTypeOrReportCode – the fact type or report code for which the result applies

StagingTableName – the staging table name for which the result applies

ColumnName – the column name in the staging table for which the result applies

Severity – the severity level of the result.  Options are:

·       Informational – this result indicates that the data may have an issue that _may_ produce invalid report data.

·       Error – this result indicates that the data will _likely_ produce incorrect report data.

·       Critical – this result indicates that the data may cause the migration to fail and/or the report to not produce.

&#x20;

ValidationMessage -this provides information about the result

RecordCount – the number of records identified as not passing the validation rule

ShowRecordsSQL – a SQL query that will produce the records that failed the validation.  You can copy the contents from this cell into a query window and execute it to see the data.

InsertDate – the date and time the Staging Validation process inserted this result into the table.

&#x20;

ETL developers can review the staging validation results and respond as needed to remediate the finding.  This may include the following actions:

1\.       Changes to the source data

2\.       Changes to the ETL

3\.       Adjustments to mapped values in Staging.SourceSystemReferenceData

4\.       Review by data owners and/or EdFacts coordinators to determine if any action is needed.

After making changes and refreshing staging data if needed, the Staging Validation process can be reran to review the updated results.

## Staging Validation Technical Details

The Staging Validation utility is comprised of the following components:

### Staging Tables

·       StagingValidationResults – contains the results of staging validation executions

·       StagingvalidationRules – contains the staging validation rules

·       StagingValidationRules\_ReportsXREF – contains the cross-reference showing which staging validation rules apply to which EdFacts reports.

### Staging Stored Procedures

·       StagingValidation\_Execute – executes the staging validation process

·       StagingValidation\_GetResults – shows the results of the staging validation process

·       StagingValidation\_InsertRule – used to insert a new rule into the Staging Validation Rules table

·       StagingValidation\_AssignRuleToReports – used to assign an existing rule to reports

### Staging Views

·       vwStagingValidationRules – shows all staging validation rules and their relationship to EdFacts reports

### Viewing Existing Staging Validation Rules

There are several ways to view the Staging Validation rules.

To view rule definitions, query table Staging.StagingValidationRules:

select \* from staging.StagingValidationRules

The following columns exist in this table:

·       StagingValidationRuleId – the internal rule Id

·       StagingTableId – the staging table Id for which this rule applies

·       StagingColumnId – the staging column Id for which this rule applies

·       RuleDscr – a description of the rule

·       Condition – the programmatic condition for this rule

·       ValidationMessage – the message included in the validation result

·       Severity – the severity level for the rule

·       CreatedBy – the rule creator.  All rules created by the CIID development team will show “Generate”.  Custom rules created by States can have whatever value they prefer.

·       CreatedDateTime – the date and time the rule was inserted into the table

Since this table contains Id values, a better way to view the rules is to use the view. To view all existing Staging Validation rules and determine for which reports they are applied, query Staging.vwStagingValidationRules:

select \* from staging.vwStagingValidationRules

The view shows all columns from Staging.StagingValidationRules, but also shows if/where those rules area applied by returning the following additional columns:

·       StagingValidationRuleId – the internal Id for a validation rule.  If this value is NULL, it means that no rule exists that is associated to the report code, table and column shown.

·       StagingValidationRuleId\_XREF – this indicates if the rule is associated to a report code.  If this value is NULL, it means that the rule is not associated to a report code.

#### Understanding StagingValidationRuleId and StagingValidationRuleId\_XREF:

View vwStagingValidationRules shows a left join on all possible combinations of Staging Tables, Staging Columns and EdFacts Reports.  Columns StagingValidationRuleId and StagingValidationRuleId\_XREF shows _if a rule exists_ and _if/where that rule is being applied_.

| StagingValidationRuleId | StagingValidationRuleId\_XREF | Explanation                                                                                          |
| ----------------------- | ----------------------------- | ---------------------------------------------------------------------------------------------------- |
| NULL                    | NULL                          | No rule exists for the Staging Table and Staging Column shown.                                       |
| NOT NULL                | NULL                          | A rule exists for the Staging Table and Staging Column but is not associated with the Report Code.   |
| NOT NULL                | NOT NULL                      | A rule exists for the Staging Table and Staging Column and is associated with the Report Code shown. |
| NULL                    | NOT NULL                      | This combination will not exist.                                                                     |

&#x20;

·       FactTypeCode – the Fact Type for which this rule applies.  Fact Types are groups of EdFacts reports such as “Assessment”, “Directory”, “Discipline”, etc.

·       DimFactTypeId – the internal Id for the Fact Type

·       ReportCode – The EdFacts Report Code for which this rule applies

·       ReportName – the EdFacts Report Name for which this rule applies

·       GenerateReportId – the internal Id for the Report

·       StagingTableName – the name of the Staging table for which this rule applies

·       StagingColumnName – the name of the Staging column for which this rule applies

·       GenerateReportId\_XREF – the internal report Id for which this rule applies

·       Enabled\_XREF – indicates if this rule is enabled.  A rule can be disabled or enabled, and only enabled rules will be executed during the Staging Validation process.

&#x20;

## Creating Staging Validation Rules

The Staging stored procedure named “StagingValidation\_InsertRule” can be used to easily add new rules to the database.  The procedure accepts the following parameters:

·       @FactTypeOrReportCode – The Fact Type or Report Code(s) for which this rule applies.  This parameter can be single Fact Type value, a single Report Code value, a comma-delimited list of Report Code values, or have a value of “All” to indicate this rule applies to all EdFacts reports for which the Staging Table is used.

·       @StagingTableName – the Staging table for which this rule applies

·       @StagingColumnName – the Staging column for which this rule applies

·       @RuleDscr – a description for the rule

·       @Condition – the logic condition this rule applies to determine the rule result.  There are three methods for defining a condition:

“Required” – if the @Condition value is “Required”, this rule indicates that the staging column is required to be populated and must not contain NULL values.

“where ….” – if the @Condition value starts with “where”, the rule looks for all record in the Staging Table and Staging Column where the condition exists and returns all columns from the Staging Table in the results.

“select …” – if the @Condition value starts with “select”, the rule applies the logic defined in the rule, and will return the data elements specified in the condition.

·       @ValidationMessage – the message returned in the Validation Result when the data fails the validation rule.

·       @CreatedBy – indicates who created the rule.  All rules provided in the Generate releases will show “Generate”.  States can populate this with their preferred value for rules they create.

·       @Enabled – indicates if the rule is enabled.  Only enabled rules are executed by the Staging Validation process.

Notice that the “Severity” for a rule is not a parameter for inserting a rule.  Generate will automatically set the severity for a rule.

Also note that Generate automatically applies some validations that are not defined as a rule.  For example, Generate knows which staging tables are required to contain data for certain EdFacts reports.  Generate also knows which staging columns must contain option set values mapped in the SourceSystemReferenceData table.  Therefore, no rules exist in the StagingValidationRules table for these instances.  Instead, Generate automatically applies these rules when executing the Staging Validation process, so rules do not need to manually defined for these conditions.  These automatic results will show a specific Rule Id in the results:

·       -1 This indicates that a staging table is required for the specific report and cannot be empty

·       -2 This indicates that an option set value in a staging table is not mapped in the Generate Staging.SourceSystemReferenceData table.

·       -9 This indicates that the defined rule has a syntax error and could not be executed.  Any results having a -9 should be reviewed by the rule developer.

&#x20;

Below are several examples of using the procedure to insert a rule:

#### Adding a REQUIRED Rule:

This example shows how to add a REQUIRED rule to a specific column in a specific table for a specific EdFacts report code.  The @Condition value = “Required”

exec Staging.StagingValidation\_InsertRule

&#x20;      @FactTypeOrReportCode = 'C141',

&#x20;      @StagingTableName = 'ProgramParticipationSpecialEducation',

&#x20;      @StagingColumnName = 'IdeaIndicator',

&#x20;      @RuleDscr = 'Cannot be NULL',

&#x20;      @Condition = 'Required',

&#x20;      @ValidationMessage = 'Cannot be NULL',

&#x20;      @CreatedBy = 'Generate',

&#x20;      @Enabled = 1

&#x20;

#### Adding a Conditional Rule with Basic Logic:

This example shows how to add a simple conditional rule that applies to a single staging table and column for all reports that are part of Fact Type “Discipline”.  The @Condition value starts with “where”.  The results will search all records in @StagingTableName where the condition exists.

exec Staging.StagingValidation\_InsertRule

&#x20;      @FactTypeOrReportCode = 'Discipline',

&#x20;      @StagingTableName = 'Discipline',

&#x20;      @StagingColumnName = 'DisciplinaryActionStartDate',

&#x20;      @RuleDscr = 'DisciplinaryActionStartDate must be before DisciplinaryActionEndDate',

&#x20;      @Condition = 'where DisciplinaryActionStartDate > isnull(DisciplinaryActionEndDate, getdate())',

&#x20;      @ValidationMessage = 'DisciplinaryActionStartDate must be before DisciplinaryActionEndDate',

&#x20;      @CreatedBy = 'Generate',

&#x20;      @Enabled = 1

&#x20;

#### Adding a Conditional Rule with Expanded Logic:

This example shows how to add a conditional rule with expanded logic that may span multiple tables and columns.  This rule is specific to the list of Reports shown in the @FactTypeOrReportCode parameter.  The @Condition value starts with “select”, so the results will show only the columns defined in the condition.

exec Staging.StagingValidation\_InsertRule

&#x20;      @FactTypeOrReportCode= 'C052, C032, C086, C141',

&#x20;      @StagingTableName = 'K12Enrollment',

&#x20;      @StagingColumnName = 'GradeLevel',

&#x20;      @RuleDscr = 'A student can only be enrolled in a single grade',

&#x20;      @Condition = 'select StudentIdentifierState, count(\*) GradeLevels from (

select distinct StudentIdentifierState, GradeLevel from staging.K12Enrollment) A

group by StudentIdentifierState

having count(\*) > 1',

&#x20;      @ValidationMessage = 'A student can only be enrolled in a single grade',

&#x20;      @CreatedBy = 'Generate',

&#x20;      @Enabled = 1

&#x20;

&#x20;

&#x20;

#### Adding a Conditional Rule For ALL Reports:

This example shows how to add a conditional rule that applies to all EdFacts reports for which the @StagingTableName is used.  The @FactTypeOrReportCode value = “All”.  This rule is saying that the RecordStartDateTime column in Staging.K12Enrollment must meet the condition in all EdFacts reports that use the K12Enrollment table.

&#x20;

exec Staging.StagingValidation\_InsertRule

&#x20;      @FactTypeOrReportCode = 'All',

&#x20;      @StagingTableName = 'K12Enrollment',

&#x20;      @StagingColumnName = 'RecordStartDateTime',

&#x20;      @RuleDscr = 'RecordStartDateTime must be before RecordEndDateTime',

&#x20;      @Condition = 'where RecordStartDateTime > isnull(RecordEndDateTime, getdate())',

&#x20;      @ValidationMessage = 'RecordStartDateTime must be before RecordEndDateTime',

&#x20;      @CreatedBy = 'Generate',

&#x20;      @Enabled = 1

&#x20;

### Assigning Existing Rules to a Fact Type or Report Code(s)

If a rule exists in StagingValidationRules, but is not associated to a specific Fact Type or Report Code, Generate provides the capability to create these relationships with procedure StagingValidation\_AssignRuleToReports.  The procedure accepts the following parameters:

·       @StagingValidationRuleId – The rule Id to be assigned

·       @FactTypeOrReportCode – The Fact Type or Report Code(s) for which this rule applies.  This parameter can be single Fact Type value, a single Report Code value, a comma-delimited list of Report Code values, or have a value of “All” to indicate this rule applies to all EdFacts reports for which the Staging Table is used.

·       @CreatedBy – indicates who created the rule.  All rules provided in the Generate releases will show “Generate”.  States can populate this with their preferred value for rules they create.

·       @Enabled – indicates if the rule is enabled.  Only enabled rules are executed by the Staging Validation process.

Below are several examples of using the procedure to assign a rule:

This example assigns rule 36 to all Assessment reports.

exec Staging.StagingValidation\_AssignRuleToReports

&#x20;      @StagingValidationRuleId = 36,

&#x20;      @FactTypeOrReportCode = 'Assessment',

&#x20;      @CreatedBy = 'Generate',

&#x20;      @Enabled = 1

&#x20;

This example assigns rule 19 to reports C188 and C189.

&#x20;

exec Staging.StagingValidation\_AssignRuleToReports

&#x20;      @StagingValidationRuleId = 19,

&#x20;      @FactTypeOrReportCode = 'C188, C189',

&#x20;      @CreatedBy = 'Generate',

&#x20;      @Enabled = 1

&#x20;

## Additional Information

&#x20;

1\.       When a new version of Generate is released, all Validation Rules provided by the Generate development team will be replaced/updated.  Any Validation Rules created by States will be retained in the new version.

2\.       If new Staging Tables and/or Staging Columns are added to Generate, the Generate Development team will need to add rows to the following tables in the App schema to allow rules to be created for those new tables/columns:

a.       App.GenerateStagingTables

b.       App.GenerateStagingColumns

c.       App.SourceSystemReferenceMapping\_DomainFile\_XREF

&#x20;

3\.       App.vwStagingRelationships shows the relationship between Fact Types, Report Codes, Staging Tables, Staging Columns and SourceSystemReferenceData.
