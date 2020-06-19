Relative Risk of Cervical Neoplasms Associated with Copper and Levonorgestrel Secreting Intrauterine Devices: Real World Evidence from the OHDSI Network
==============================

<img src="https://img.shields.io/badge/Study%20Status-Design%20Finalized-brightgreen.svg" alt="Study Status: Design Finalized"> 

- Analytics use case(s): **Characterization** and **Population-Level Estimation**
- Study type: **Clinical Application**
- Tags: **iud**
- Study lead: **Matthew Spotnitz** and **Karthik Natarajan**
- Study lead forums tag: **[mattspotnitz](https://forums.ohdsi.org/u/mattspotnitz)**
- Study start date: **September 23, 2019**
- Study end date: **-**
- Protocol: **[Word file](https://github.com/ohdsi-studies/IUDEHREstimationStudy/blob/master/documents/IUD%20Cervical%20Neoplasms%20Estimation%20Protocol.docx)**
- Publications: **-**
- Results explorer: **-**

This study extends the [prior single-site study](https://journals.lww.com/greenjournal/fulltext/2020/02000/relative_risk_of_cervical_neoplasms_among_copper.11.aspx) to the OHDSI network.


Requirements
============

- A database in [Common Data Model version 5](https://github.com/OHDSI/CommonDataModel) in one of these platforms: SQL Server, Oracle, PostgreSQL, IBM Netezza, Apache Impala, Amazon RedShift, Google BigQuery, or Microsoft APS.
- R version 3.5.0 or newer
- On Windows: [RTools](http://cran.r-project.org/bin/windows/Rtools/)
- [Java](http://java.com)
- 25 GB of free disk space

See [these instructions](https://ohdsi.github.io/MethodsLibrary/rSetup.html) on how to set up the R environment on Windows.

If you have access to a claims data set please also run this study on it, which is described in the "Run Study on Claims Data" section below

Run Study 
=========
1. In `R`, use the following code to install the dependencies:

	```r
	install.packages("devtools")
	library(devtools)
	install_github("ohdsi/SqlRender", ref = "v1.6.0")
	install_github("ohdsi/DatabaseConnector", ref = "v2.3.0")
	install_github("ohdsi/OhdsiSharing", ref = "v0.1.3")
	install_github("ohdsi/FeatureExtraction", ref = "v2.2.3")
	install_github("ohdsi/CohortMethod", ref = "v3.0.2")
	install_github("ohdsi/EmpiricalCalibration", ref = "v2.0.0")
	install_github("ohdsi/MethodEvaluation", ref = "v1.0.2")
	```

	If you experience problems on Windows where rJava can't find Java, one solution may be to add `"--no-multiarch"` to each `install_github` call, for example these are two ways to ignore the i386 architecture:
	
	```r
	install_github("ohdsi/SqlRender", args = "--no-multiarch")
	install_github("ohdsi/SqlRender", INSTALL_opts=c("--no-multiarch"))
	```
	
	OR for all installs, one can try:
	
	```r
	options(devtools.install.args = "--no-multiarch")
	```
	
	Alternatively, ensure that you have installed both 32-bit and 64-bit JDK versions, as mentioned in the [video tutorial](https://youtu.be/K9_0s2Rchbo).
	
2. In `R`, use the following `devtools` command to install the IUDCLW package:

	```r
	# install the network package
    devtools::install_github("https://github.com/ohdsi-studies/IUDEHREstimationStudy")
	```
	
3. Once installed, you can execute the study by modifying and using the code below. For your convenience, this code is also provided under `extras/CodeToRun.R`:

	```r
	library(IUDEHRStudy)
	
	# Optional: specify where the temporary files (used by the ff package) will be created:
	options(fftempdir = "c:/FFtemp")
	
	# Maximum number of cores to be used:
	maxCores <- parallel::detectCores()
	
	# Minimum cell count when exporting data:
	minCellCount <- 10
	
	# The folder where the study intermediate and result files will be written:
	outputFolder <- "c:/IUDEHRStudy"
	
	# Details for connecting to the server:
	# See ?DatabaseConnector::createConnectionDetails for help
	connectionDetails <- DatabaseConnector::createConnectionDetails(dbms = "postgresql",
									server = "some.server.com/ohdsi",
									user = "",
									password = "")
	
	# The name of the database schema where the CDM data can be found:
	cdmDatabaseSchema <- "cdm_synpuf"
	
	# The name of the database schema and table where the study-specific cohorts will be instantiated:
	cohortDatabaseSchema <- "scratch.dbo" #You mush have rights to create tables in this schema
	cohortTable <- "iud_study_ehr"
	
	# Some meta-information that will be used by the export function:
	databaseId <- ""          #SiteName
	databaseName <- ""        #SiteName_DatabaseName
	databaseDescription <- "" #Description of site's database
	
	# For Oracle: define a schema that can be used to emulate temp tables:
	oracleTempSchema <- NULL
	
	execute(connectionDetails = connectionDetails,
            cdmDatabaseSchema = cdmDatabaseSchema,
            cohortDatabaseSchema = cohortDatabaseSchema,
            cohortTable = cohortTable,
            oracleTempSchema = oracleTempSchema,
            outputFolder = outputFolder,
            databaseId = databaseId,
            databaseName = databaseName,
            databaseDescription = databaseDescription,
            createCohorts = TRUE,
            synthesizePositiveControls = TRUE,
            runAnalyses = TRUE,
            runDiagnostics = TRUE,
            packageResults = TRUE,
            maxCores = maxCores)
	```

4. To view the results, use the Shiny app:

	```r
	prepareForEvidenceExplorer(paste0("Results",databaseId,".zip"), "/shinyData")
	launchEvidenceExplorer("/shinyData", blind = TRUE)
	```
	
	Note that you can save plots from within the Shiny app. It is possible to view results from more than one database by applying `prepareForEvidenceExplorer` to the Results file from each database, and using the same data folder. Set `blind = FALSE` if you wish to be unblinded to the final results.
  
5. Please contact both Matt Spotnitz (mes2165 at cumc dot columbia dot edu) and Karthik Natarajan (kn2174 at cumc dot columbia dot edu) for an account and key in order to upload the results. Once the account information is provided, the file ```export/Results<DatabaseId>.zip``` in the export folder can be uploaded to the study coordinator. Below is the R code to upload the files:

	```r
    # one time R package install
    install_github("ohdsi/OhdsiSharing")
 
    # upload local file to sftp server study folder using the '/tmp/privateKeyFileName' private key
    privateKeyFileName <- ""                        #full path to the private key file that was provided by the study coordinator
    userName <- ""                                  #username provided by study coordinator
    fileName <- paste0("Results",databaseId,".zip") #results zip file
    submitResults(outputFolder, fileName, userName, privateKeyFileName)
    ```


Run for Claims Data
===================

As mentioned above, if you have access to a claims data follow the below instructions to run an additional analysis.

```r
devtools::install_github("https://github.com/ohdsi-studies/IUDEHREstimationStudy/additionalEstimationPackage/IUDClaimsEstimation")
library(IUDClaimsStudy)

# Optional: specify where the temporary files (used by the ff package) will be created:
options(fftempdir = "c:/FFtemp")

# Maximum number of cores to be used:
maxCores <- parallel::detectCores()

# Minimum cell count when exporting data:
minCellCount <- 10

# The folder where the study intermediate and result files will be written:
outputFolder <- paste0(outputFolder,"/IUDClaimsStudy") #If running this analysis in isolation (i.e. without EHR analysis) please enter the file directory here (i.e. "C:/IUDClaimsStudy")

# Details for connecting to the server:
# See ?DatabaseConnector::createConnectionDetails for help
connectionDetails <- DatabaseConnector::createConnectionDetails(dbms = "postgresql",
                                server = "some.server.com/ohdsi",
                                user = "",
                                password = "")

# The name of the database schema where the CDM data can be found:
cdmDatabaseSchema <- "cdm_synpuf"

# The name of the database schema and table where the study-specific cohorts will be instantiated:
cohortDatabaseSchema <- "scratch.dbo" #You mush have rights to create tables in this schema
cohortTable <- "iud_study_claims"

# Some meta-information that will be used by the export function:
databaseId <- ""          #SiteName
databaseName <- ""        #SiteName_DatabaseName
databaseDescription <- "" #Description of site's database

# For Oracle: define a schema that can be used to emulate temp tables:
oracleTempSchema <- NULL

IUDClaimsEstimation::execute(connectionDetails = connectionDetails,
        cdmDatabaseSchema = cdmDatabaseSchema,
        cohortDatabaseSchema = cohortDatabaseSchema,
        cohortTable = cohortTable,
        oracleTempSchema = oracleTempSchema,
        outputFolder = outputFolder,
        databaseId = databaseId,
        databaseName = databaseName,
        databaseDescription = databaseDescription,
        createCohorts = TRUE,
        synthesizePositiveControls = TRUE,
        runAnalyses = TRUE,
        runDiagnostics = TRUE,
        packageResults = TRUE,
        maxCores = maxCores)
```


Development
===========
IUDEHRStudy was developed in ATLAS and R Studio. The package was modified to include additional analyses from the initial Atlas package. All additional analyses and code are located in the _AdditionalAnalysis.R_ file. The following are the additional analyses and modifications:
1. Calculates counts to additional cohorts for sensitivity analysis
2. Calculates the cumulative incidence of the cohorts
3. Calculates the yearly distribution of all cohorts
4. Creates KM graphs for the cohorts of interest
5. Copies all diagnostic graphs in the diagnostic folder to the export folder 
6. All cohort counts and distributions are filtered based on minimum cell count

License
=======
The IUDEHRStudy package is licensed under Apache License 2.0
