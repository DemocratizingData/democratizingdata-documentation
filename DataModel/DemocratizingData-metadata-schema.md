# Data model for Democratizing Data database
This document describes the full database schema for the metadata about publications and datasets collected by the Democratizing Data project. The core database (currently ShowUStheData_v3 at SciServer) of the project contains all the tables documented here. Some agency-specific databases derived from it will miss a few of these tables, but are otherwise the same.


## agency_run
The table representing *runs* for the different agencies. In a "run" the corpus of publications at Elsevier is searched for the occurrence of references to data sets from an input list provided by the agency.

This table is at the moment only an anchor representing a complete submission from Elsevier. Most other tables in this database have a run_id column referring to this table, indicating for which *run* they represent metadata.
There is no more metadata then the agency's name and a version label produced by Elsevier.

Elsevier may produce multiple submissions for the same agency and collection of input data sets, with different run parameterts. For example adding filtering terms, varying matching acceptance criteria etc.
There is no explicit relation between such submissions here apart from a constancy of the agency name
But also completely different runs with a different collection of data sets to be searched will eventually be contained in this table. These different *types of versions* will be further categorized in future versions of the model where we will add more runtime metadata about the runs.



|column name|description|data type|length|is nullable|
|:---|:---|:---|:---|:---|
|id|unique identifier to the agency_run table|bigint|0|NO|
|agency|name of the agency for which the run was performed|varchar|32|NO|
|version|version of the run for the agency. allows multiple versions on the same data sets, or possibly new runs for the same agency, but with different input data sets.|varchar|32|NO|
|run_date|approximate date the run was performed.|date|0|YES|
|last_updated_date|last time the row was updated. generally the time of creation of the row.|datetime|0|NO|


## asjc
ASJC is the All Science Journal Classification code which defines the research area of a journal and the articles it contains.


|column name|description|data type|length|is nullable|
|:---|:---|:---|:---|:---|
|id|unique identifier for this entry|bigint|0|NO|
|run_id|identifies the agency run for which this entry was determined, foreign key to agency_run.id|bigint|0|NO|
|code|The All Science Journal Classification code which defines the research area of a journal and the articles it contains. There may be more than one ASJC code for each journal / publication. The 334 codes are used here providing a relatively precise definition of research area. |bigint|0|NO|
|label|TBD|nvarchar|-1|YES|
|last_updated_date|last time the row was updated. generally the time of creation of the row.|datetime|0|NO|


## author
table with author information


|column name|description|data type|length|is nullable|
|:---|:---|:---|:---|:---|
|id|unique identifier for this entry|bigint|0|NO|
|run_id|identifies the agency run for which this entry was determined, foreign key to agency_run.id|bigint|0|NO|
|external_id|id assigned by Elsevier to the author. allows different authors to be identified across publications, even if they have different names there.|varchar|128|YES|
|given_name|the unique given name of the author as determined by Elsevier|nvarchar|150|YES|
|family_name|the unique family name of the author as determined by Elsevier|nvarchar|150|YES|
|last_updated_date|last time the row was updated. generally the time of creation of the row.|datetime|0|NO|


## author_affiliation
table linking authors to their affiliations in a publication


|column name|description|data type|length|is nullable|
|:---|:---|:---|:---|:---|
|id|unique identifier for this entry|bigint|0|NO|
|run_id|identifies the agency run for which this entry was determined, foreign key to agency_run.id|bigint|0|NO|
|publication_author_id|identifies the publication_author here linked to a publication affiliation|bigint|0|YES|
|publication_affiliation_id|identifies the publicaiton_affiliation entry here linked to a publication author.|bigint|0|YES|
|last_updated_date|last time the row was updated. generally the time of creation of the row.|date|0|NO|


## dataset_alias
The table with datasets provided by an agency for a particular run and possible aliases


|column name|description|data type|length|is nullable|
|:---|:---|:---|:---|:---|
|id|unique identifier for this entry|bigint|0|NO|
|run_id|identifies the agency run for which this entry was determined, foreign key to agency_run.id|bigint|0|NO|
|alias_id|the alias_id as provided by Elsevier.|bigint|0|NO|
|parent_alias_id|identifies the parent dataset entry in this table, as identified by the alias_id!|bigint|0|YES|
|alias|the name of the data set or the alias depending on whether alias_id==parent_alias_id or not.|varchar|160|YES|
|alias_type|flaf indicating whether the row stores the dataset itself, or an alias.|varchar|50|YES|
|url|URL to information about the dataset|varchar|2048|YES|
|last_updated_date|last time the row was updated. generally the time of creation of the row.|datetime|0|NO|


## dyad
The core table with dyads representing dataset references discovered in publications.


|column name|description|data type|length|is nullable|
|:---|:---|:---|:---|:---|
|id|unique identifier for this entry|bigint|0|NO|
|run_id|identifies the agency run for which this entry was determined, foreign key to agency_run.id|bigint|0|NO|
|publication_id|foreign key the publication table's id column, identifying the publication within which the dataset reference represented by this dyad was identified.|bigint|0|NO|
|dataset_alias_id|foreign key to the dataset_alias table's id column identifying the match made between this dyad and a dataset alias provided by an agency. if no such match was found this column has a NULL|bigint|0|YES|
|alias_id|the intrinsic id assigned by Elsevier to the dataset alias, corresponding to the alias_id column in the dataset_alias table.|bigint|0|YES|
|mention_candidate|the phrase in the publication that was deemed by the algorithm to reflect a reference to a dataset|varchar|1028|NO|
|snippet|snippet of text surrounding the mention_candidate, meant to provide contextual information to reviewers/validators|varchar|-1|YES|
|last_updated_date|last time the row was updated. generally the time of creation of the row.|datetime|0|NO|
|is_fuzzy|column has value 1 if the matching between mention candidate and dataset alias was performed using a fuzuuy algorithm, 0 otherwise.|bit|0|YES|
|fuzzy_score|in case the matching between mention candidate and dataset alias was perfomed using a fuzzy algorithm, this column stores the score indicating how certain the match was deemed to be.|real|0|YES|


## dyad_model
model scores for particular entries in the dyad table


|column name|description|data type|length|is nullable|
|:---|:---|:---|:---|:---|
|id|unique identifier for this entry|bigint|0|NO|
|run_id|identifies the agency run for which this entry was determined, foreign key to agency_run.id|bigint|0|NO|
|dyad_id|identifies the dyad|bigint|0|NO|
|model_id|identifies the model|bigint|0|NO|
|score|the score of this model for the dyad|real|0|YES|
|last_updated_date|last time the row was updated. generally the time of creation of the row.|datetime|0|NO|


## issn
The ISSN / ISBN codes for the journal.


|column name|description|data type|length|is nullable|
|:---|:---|:---|:---|:---|
|id|unique identifier for this entry|bigint|0|NO|
|run_id|identifies the agency run for which this entry was determined, foreign key to agency_run.id|bigint|0|NO|
|journal_id|foreign key to the journal table's id column, identifying the journal for this ISSN|bigint|0|YES|
|ISSN|The ISSN / ISBN codes for the referenced journal/source|varchar|13|YES|
|last_updated_date|last time the row was updated. generally the time of creation of the row.|datetime|0|NO|


## journal
Journal that a publication in the publicaiton table appeared in.


|column name|description|data type|length|is nullable|
|:---|:---|:---|:---|:---|
|id|Unique identifier and primary key for this table.|bigint|0|NO|
|run_id|foreign key, identifier of the agency run for which this entry was determined.|bigint|0|NO|
|publisher_id|foreign key to the publisher table, identifying the publisher for this journal at the time the agency run was executed.|bigint|0|YES|
|external_id|The Scopus ID for the journal / source|varchar|128|YES|
|title|The name of the journal / source that the publication was published in.|varchar|1028|NO|
|cite_score|Citescore is an Elsevier derived metric that measures the relative standing of a journal.|decimal|0|YES|
|last_updated_date|last time the row was updated. generally the time of creation of the row.|datetime|0|NO|


## model
The models that are run. Originally obtained through the Kaggle competition.


|column name|description|data type|length|is nullable|
|:---|:---|:---|:---|:---|
|id|unique identifier for this entry|bigint|0|NO|
|name|the name of the model|varchar|32|NO|
|github_commit_url|the github url where the commit for this model can be found|varchar|1024|YES|
|description|description of the model|nvarchar|-1|YES|
|last_updated_date|last time the row was updated. generally the time of creation of the row.|datetime|0|NO|


## publication
publications discovered in a run


|column name|description|data type|length|is nullable|
|:---|:---|:---|:---|:---|
|id|unique identifier for this entry|bigint|0|NO|
|run_id|identifies the agency run for which this entry was determined, foreign key to agency_run.id|bigint|0|NO|
|journal_id|foreign key to the journal for this publicaiton|bigint|0|YES|
|external_id|the scopus ID (for Elsevier publications) of this publication.|varchar|128|YES|
|title|title of the publication|varchar|400|YES|
|doi|DOI of the publication|varchar|80|YES|
|year|The year that the publication was published as recorded in Scopus|int|0|YES|
|month|The month of publication. May not be available. This will be an integer value i.e. 1 = January etc|int|0|YES|
|pub_type|The type of publication. Types includes - Article, review, book, book chapter, letter.|varchar|30|YES|
|citation_count|The number of times this publication is cited in Scopus|int|0|YES|
|fw_citation_impact|The Field Weighted Citation Impact (FWCI) for the publication. This is a measure for how impactful or important a publication is as measured through normalised citations. The number of times cited divided by the expected number of citations of articles in the same year, subject and publication type. World average across papers is 1.0 for this metric. This metric also changes over time. |float|0|YES|
|last_updated_date|last time the row was updated. generally the time of creation of the row.|datetime|0|NO|
|tested_expressions|a JSON-formatted string indicating whether certain expressions occurred in the publication.|varchar|MAX|YES|

## publication_affiliation
The table with affiliations on a publication.


|column name|description|data type|length|is nullable|
|:---|:---|:---|:---|:---|
|id|unique identifier of the affiliation table|bigint|0|NO|
|run_id|identifies the agency run for which this entry was determined, foreign key to agency_run.id|bigint|0|NO|
|external_id|id assigned by Elsevier to this affiliation|varchar|128|YES|
|institution_name|the name of the institution to which a author was associated.|nvarchar|750|YES|
|address|the address of the author, most likely that of their institution|nvarchar|750|YES|
|country_code|the three letter country code associated to the affiliaiton, most likely of the institution|nvarchar|10|YES|
|last_updated_date|last time the row was updated. generally the time of creation of the row.|datetime|0|NO|
|city|the city for the addres or institute in this affiliation|nvarchar|128|YES|
|state|if appropriate, the state for the addres or institute in this affiliation|nvarchar|128|YES|
|postal_code|if appropriate, the postal code for this affiliation|nvarchar|64|YES|

## publication_affiliation_geo
The table with affiliations on a publication.


|column name|description|data type|length|is nullable|
|:---|:---|:---|:---|:---|
|run_id|identifies the agency run for which this entry was determined, foreign key to agency_run.id|bigint|0|NO|
|publication_affiliation_id|Identifies publication_affiliation for Foreign key to the publication_affiliationid assigned by Elsevier to this affiliation|varchar|128|YES|


## publication_asjc
Associative table linking a publication to ASJC entries.


|column name|description|data type|length|is nullable|
|:---|:---|:---|:---|:---|
|id|unique identifier for this entry|bigint|0|NO|
|run_id|identifies the agency run for which this entry was determined, foreign key to agency_run.id|bigint|0|NO|
|publication_id|foreign key to publication table's id column, identifying the publication in this relation|bigint|0|NO|
|asjc_id|foreign key toi the ASJC|bigint|0|NO|
|last_updated_date|last time the row was updated. generally the time of creation of the row.|datetime|0|NO|


## publication_author
Associative table linking publication and author tables.


|column name|description|data type|length|is nullable|
|:---|:---|:---|:---|:---|
|id|unique identifier for this entry|bigint|0|NO|
|run_id|identifies the agency run for which this entry was determined, foreign key to agency_run.id|bigint|0|NO|
|publication_id|foreign key to the publication for this author|bigint|0|NO|
|author_id|foreigj key to the table with scopus author entries|bigint|0|NO|
|author_position|position of author in the list of authors on the publication|int|0|YES|
|last_updated_date|last time the row was updated. generally the time of creation of the row.|datetime|0|NO|


## publication_topic
identifying the topic assigned to a publication


|column name|description|data type|length|is nullable|
|:---|:---|:---|:---|:---|
|id|unique identifier for this entry|bigint|0|NO|
|run_id|identifies the agency run for which this entry was determined, foreign key to agency_run.id|bigint|0|NO|
|publication_id|foreign key to the topic table's id column identifying the publication in this relation between publications and topics.|bigint|0|NO|
|topic_id|foreign key to the topic table's id column identifying the topic in this relation between publications and topics.|bigint|0|NO|
|score|TBD|real|0|YES|
|last_updated_date|last time the row was updated. generally the time of creation of the row.|datetime|0|NO|


## publisher
Publishers of the journals the publications were published in.


|column name|description|data type|length|is nullable|
|:---|:---|:---|:---|:---|
|id|unique identifier for this entry|bigint|0|NO|
|run_id|identifies the agency run for which this entry was determined, foreign key to agency_run.id|bigint|0|NO|
|external_id|external identifier of this publisher in elseviers scopus repository|nvarchar|128|YES|
|name|name of the publisher|nvarchar|120|YES|
|last_updated_date|last time the row was updated. generally the time of creation of the row.|datetime|0|NO|


## reviewer
Reviewers are susd_user-s assigned to validate dyads in the publication_dataset_alias table.


|column name|description|data type|length|is nullable|
|:---|:---|:---|:---|:---|
|id|unique identifier for this entry|bigint|0|NO|
|susd_user_id|foreign key to the susd_user table's id columns, identifying the user corresponding to this reviewer|bigint|0|NO|
|run_id|identifies the agency run for which this entry was determined, foreign key to agency_run.id|bigint|0|NO|
|last_updated_date|last time the row was updated. generally the time of creation of the row.|datetime|0|NO|
|roles|String formatted as a JSON array with one or more roles that the reviewer has in the validation process. These are <ul><li>REVIEWER: dyads will be assighned to the user to validate</li><li>ADMIN: user can configure validation process for the run, can assigne reviewers and can monitor the progress of the validation process.</li></ul>|varchar|MAX|YES|

## snippet_validation
table storing the validation results for dyads provided by reviewers


|column name|description|data type|length|is nullable|
|:---|:---|:---|:---|:---|
|id|unique identifier for this entry|bigint|0|NO|
|run_id|identifies the agency run for which this entry was determined, foreign key to agency_run.id|bigint|0|NO|
|reviewer_id|foreign key to the reviewer table's id column, identifying the reviewer assigned to validate this snippet|bigint|0|NO|
|dyad_id|foreign key to the dyad table's id column, identifying the dyad that is being validated.|bigint|0|NO|
|is_dataset_reference|if the value in this column is 1 it indcates the dyad indeed has identified a reference to a datasdet, if 0 it is not a dataset reference, if -1 the reviewer was unsure about it.|smallint|0|YES|
|agency_dataset_identified|if the value in this column is 1 it indicates the dyad indeed identified the specific dataset provided by the agency, if 0 it is not a reference to that dataset, if -1 the reviewer was unsure about it.|smallint|0|YES|
|notes|any notes the reviewer attached to the dyad being reviewed|nvarchar|-1|YES|
|last_updated_date|last time the row was updated. generally the time of creation of the row.|datetime|0|NO|


## susd_user
A user of the validation tool.


|column name|description|data type|length|is nullable|
|:---|:---|:---|:---|:---|
|id|unique identifier of this table|bigint|0|NO|
|first_name|First name of the individual.|varchar|100|YES|
|last_name|Surname of the individual.|varchar|100|YES|
|email|email of the user.|varchar|100|YES|
|password|encrypted password of the user.|varchar|100|YES|
|last_updated_date|last time the row was updated. generally the time of creation of the row.|datetime|0|NO|


## topic
topics defined by Elsevier and assigned to publications. consist of three concateneatd keywords


|column name|description|data type|length|is nullable|
|:---|:---|:---|:---|:---|
|id|unique identifier for this entry|bigint|0|NO|
|run_id|identifies the agency run for which this entry was determined, foreign key to agency_run.id|bigint|0|NO|
|keywords|a topic is defined in Elsevier by three keywords. This columns sotres these as a |-separated string.|varchar|1028|YES|
|external_topic_id|external identifier of this topic provided by Elsevier.|varchar|128|YES|
|prominence|TBD|real|0|YES|
|last_updated_date|last time the row was updated. generally the time of creation of the row.|datetime|0|NO|
|last_updated_date|last time the row was updated. generally the time of creation of the row.|datetime|0|NO|

