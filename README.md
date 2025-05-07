# Snowpipe

<p><strong>Snowpipe</strong></p>
<p>Snowpipe enables loading data from files as soon as they&rsquo;re available in a stage. This means you can load data from files in micro-batches, making it available to users within minutes, rather than manually executing COPY statements on a schedule to load larger batches.</p>
<p><strong>Process flow</strong></p>
<p><a href="https://azure.microsoft.com/en-us/services/event-grid/">Microsoft Azure Event Grid</a>&nbsp;notifications for an Azure container trigger Snowpipe data loads automatically. The following diagram shows the Snowpipe auto-ingest process flow:</p>

![image](https://github.com/user-attachments/assets/11d3326e-e8b4-4f85-9afc-3cefc0d6b898)

<ol>
<li>Data files are loaded in a stage.</li>
<li>A blob storage event message informs Snowpipe via Event Grid that files are ready to load. Snowpipe copies the files into a queue.</li>
<li>A Snowflake-provided virtual warehouse loads data from the queued files into the target table based on parameters defined in the specified pipe.</li>
</ol>

<p><strong>Note</strong></p>
<p>Only account administrators (users with the ACCOUNTADMIN role) or a role with the global CREATE INTEGRATION privilege can execute this SQL command.</p>

![image](https://github.com/user-attachments/assets/c14e6bc4-ef2b-4d13-8ab8-a344384946bb)


<p><strong>Azure Setup</strong></p>
<p>Snowflake supports the following types of blob storage accounts:</p>
<ul>
<li>Blob storage</li>
<li>Data Lake Storage Gen2</li>
<li>General-purpose v2</li>
</ul>
<p><strong>Step 1: Create a Cloud Storage Integration in Snowflake</strong></p>
<ul>
<li>Create a storage integration using the&nbsp;<a href="https://docs.snowflake.com/en/sql-reference/sql/create-storage-integration">CREATE STORAGE INTEGRATION</a> A storage integration is a Snowflake object that stores a generated service principal for your Azure cloud storage, along with an optional set of allowed or blocked storage locations (i.e. containers).</li>
<li>A single storage integration can support multiple external (i.e. Azure) stages. The URL in the stage definition must align with the Azure containers (and optional paths) specified for the STORAGE_ALLOWED_LOCATIONS parameter.</li>
</ul>

<p><strong>create a database</strong></p>

![image](https://github.com/user-attachments/assets/b33ae04f-92d6-4f98-a02f-2cd4f4a2d2b0)

<p><strong>create a table</strong></p>

![image](https://github.com/user-attachments/assets/2ae81f2e-dd90-436f-aa2e-e754a48e297a)

<p><strong>Creating a Cloud Storage Integration in Snowflake</strong></p>

```sql
CREATE OR REPLACE STORAGE INTEGRATION <storage_integration_name>
TYPE = EXTERNAL_STAGE
STORAGE_PROVIDER = 'AZURE'
ENABLED = True
AZURE_TENANT_ID = '<your_azure_tenant_id>'
STORAGE_ALLOWED_LOCATIONS = ('<storage_account_endpoints>');
```


<p>To find your tenant ID, log into the Azure portal and click&nbsp;<strong>Azure Active Directory</strong>&nbsp;&raquo;&nbsp;<strong>Properties</strong>. The tenant ID is displayed in the&nbsp;<strong>Tenant ID</strong>&nbsp;field.</p>

![image](https://github.com/user-attachments/assets/2ad3e733-96c8-4c63-875e-dcd503dfccdd)

![image](https://github.com/user-attachments/assets/25d19c9a-e661-41bd-8036-b22391b67fdc)


<p>The following example creates an integration that explicitly limits external stages that use the integration to reference either of two containers and paths. In a later step, we will create an external stage that references one of these containers and paths.</p>
<p><strong>&nbsp;</strong></p>
<p><strong>you can always drop the Storage Integration</strong></p>

![image](https://github.com/user-attachments/assets/a1be04f9-227c-492e-9fec-215252afa1ba)

<p><strong>Step 2: Grant Snowflake Access to the Storage Locations</strong></p>
<ol>
<li>Execute the&nbsp;<a href="https://docs.snowflake.com/en/sql-reference/sql/desc-integration">DESCRIBE INTEGRATION</a>command to retrieve the consent URL:</li>
<li><strong>DESC</strong> <strong>STORAGE</strong> <strong>INTEGRATION</strong> <strong>&lt;integration_name&gt;;</strong></li>
</ol>
<p>Note the values in the following columns:</p>
<p><strong>AZURE_CONSENT_URL</strong></p>
<p>URL to the Microsoft permissions request page.</p>
<p><strong>AZURE_MULTI_TENANT_APP_NAME</strong></p>
<p>Name of the Snowflake client application created for your account. In a later step in this section, you will need to grant this application the permissions necessary to obtain an access token on your allowed storage locations.</p>

![image](https://github.com/user-attachments/assets/69145d9d-ccf8-4814-ba86-65cd0b8a13e0)

![image](https://github.com/user-attachments/assets/56ae041d-e8d6-403c-83af-119bc02ae993)






