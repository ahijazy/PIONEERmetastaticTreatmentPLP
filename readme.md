Patient level prediction package for PIONEER studyathon 2022 (metastatic prostate cancer patients) 
========================================================

<img src="https://img.shields.io/badge/Study%20Status-Started-blue.svg" alt="Study Status: Started">

- Analytics use case(s): **Patient-Level Prediction**
- Study type: **Clinical Application**
- Tags: **Cancer**
- Study lead: **Juan-Gomez Rivas**
- Study lead forums tag: **[bdemeulder](https://forums.ohdsi.org/u/bdemeulder)**
- Study start date: **October 31, 2022**
- Study end date: **-**
- Protocol: **-**
- Publications: **-**
- Results explorer: **-**

The goal of the package is to identify the following: Out of the patients with metastatic hormone sensitive prostate cancer treated with one of the approved treatment plans, who will experience progression and death during an establised follow-up period (1,3,5 years).
 
This study is undertaken by the 2022 prostate cancer studyathon of the [IMI PIONEER](https://prostate-pioneer.eu) project.

Vignette: [Using the package skeleton for patient-level prediction studies](https://raw.githubusercontent.com/OHDSI/StudyProtocolSandbox/master/PIONEERmetastaticTreatmentPLP/inst/doc/UsingSkeletonPackage.pdf)

Suggested Requirements
===================
- R studio (https://rstudio.com)
- Java runtime environment
- Python


Instructions To Build Package
===================

- Open the package project (file ending with '.Rproj') by double clicking it
- Install any missing dependancies for the package by running:
 
```r
library(devtools)
install_github("edwindj/ffbase", subdir="pkg") # installs ffbase
remove.packages(PatientLevelPrediction)        # remove the installed plp package
install_github("OHDSI/PatientLevelPrediction", ref = 'v5.0.5') # ensures that the correct version of plp ins installed
source('./extras/packageDeps.R')
```
- Build the package by clicking the R studio 'Install and Restart' button in the build tab (top right)
- If you do not have ffbase installed you can install it using:
```r
library(devtools)
install_github("edwindj/ffbase", subdir="pkg")
```
Instructions To Run Package
===================
- Execute the study by running the code in (extras/CodeToRun.R) :
```r
  library(PIONEERmetastaticTreatmentPLP)
  # USER INPUTS
#=======================
# The folder where the study intermediate and result files will be written:
outputFolder <- "./PIONEERmetastaticTreatmentPLPResults"

# Specify where the temporary files (used by the ff package) will be created:
options(fftempdir = "location with space to save big data")

# Details for connecting to the server:
dbms <- "you dbms"
user <- 'your username'
pw <- 'your password'
server <- 'your server'
port <- 'your port'

connectionDetails <- DatabaseConnector::createConnectionDetails(dbms = dbms,
                                                                server = server,
                                                                user = user,
                                                                password = pw,
                                                                port = port)

# Add the database containing the OMOP CDM data
cdmDatabaseSchema <- 'cdm database schema'
# Add a sharebale name for the database containing the OMOP CDM data
cdmDatabaseName <- 'a friendly shareable  name for your database'
# Add a database with read/write access as this is where the cohorts will be generated
cohortDatabaseSchema <- 'work database schema'

oracleTempSchema <- NULL

# table name where the cohorts will be generated
cohortTable <- 'PIONEERmetastaticTreatmentPLPCohort'
#=======================

execute(connectionDetails = connectionDetails,
        cdmDatabaseSchema = cdmDatabaseSchema,
		cdmDatabaseName = cdmDatabaseName,
        cohortDatabaseSchema = cohortDatabaseSchema,
        cohortTable = cohortTable,
        oracleTempSchema = oracleTempSchema,
        outputFolder = outputFolder,
        createProtocol = F,
        createCohorts = T,
        runAnalyses = T,
        createResultsDoc = F,
        packageResults = F,
        createValidationPackage = F,
        minCellCount= 5)
```

The 'createCohorts' option will create the target and outcome cohorts into cohortDatabaseSchema.cohortTable if set to T.  The 'runAnalyses' option will create/extract the data for each prediction problem setting (each Analysis), develop a prediction model, internally validate it if set to T.  The results of each Analysis are saved in the 'outputFolder' directory under the subdirectories 'Analysis_1' to 'Analysis_N', where N is the total analyses specified.  After running execute with 'runAnalyses set to T, a 'Validation' subdirectory will be created in the 'outputFolder' directory where you can add the external validation results to make them viewable in the shiny app or journal document that can be automatically generated.


- You can then easily transport all the trained models into a network validation study package by running:
```r
  
  execute(connectionDetails = connectionDetails,
        cdmDatabaseSchema = cdmDatabaseSchema,
		cdmDatabaseName = cdmDatabaseName,
        cohortDatabaseSchema = cohortDatabaseSchema,
        cohortTable = cohortTable,
        outputFolder = outputFolder,
        createValidationPackage = T)
  

```

- To pick specific models (e.g., models from Analysis 1 and 3) to export to a validation study run:
```r
  
  execute(connectionDetails = connectionDetails,
        cdmDatabaseSchema = cdmDatabaseSchema,
		cdmDatabaseName = cdmDatabaseName,
        cohortDatabaseSchema = cohortDatabaseSchema,
        cohortTable = cohortTable,
        outputFolder = outputFolder,
        createValidationPackage = T, 
        analysesToValidate = c(1,3))
```  
This will create a new subdirectory in 'outputFolder' that has the name <yourPredictionStudy>Validation.  For example, if your prediction study package was named 'bestPredictionEver' and you run the execute with 'createValidationPackage' set to T with 'outputFolder'= 'C:/myResults', then you will find a new R package at the directory: 'C:/myResults/bestPredictionEverValidation'.  This package can be executed similarly but will validate the developed model/s rather than develop new model/s.  If you set the validation package outputFolder to the Validation directory of the prediction package results (e.g., 'C:/myResults/Validation'), then the results will be saved in a way that can be viewed by shiny.


- To create the shiny app and view the results, run the following after successfully developing the models:
```r
  
execute(connectionDetails = connectionDetails,
        cdmDatabaseSchema = cdmDatabaseSchema,
		cdmDatabaseName = cdmDatabaseName,
        cohortDatabaseSchema = cohortDatabaseSchema,
        cohortTable = cohortTable,
        outputFolder = outputFolder,
        createShiny = T,
        minCellCount= 5)
PatientLevelPrediction::viewMultiplePlp(outputFolder)

```

If you saved the validation results into the validation folder in the directory you called 'outputFolder' in the structure: '<outputFolder>/Validation/<newDatabaseName>/Analysis_N' then shiny and the journal document creator will automatically include any validation results.  The validation package will automatically save the validation results in this structure if you set the outputFolder for the validation results to: '<outputFolder>/Validation'.

- To create the journal document for the Analysis 1:
```r
  
execute(connectionDetails = connectionDetails,
        cdmDatabaseSchema = cdmDatabaseSchema,
		cdmDatabaseName = cdmDatabaseName,
        cohortDatabaseSchema = cohortDatabaseSchema,
        cohortTable = cohortTable,
        outputFolder = outputFolder,
        createJournalDocument = F,
        analysisIdDocument = 1
        minCellCount= 5)

```


Instructions To Share Package
===================

- Share the package by adding it to the OHDSI/StudyProtocolSandbox github repo and get people to install by:
```r

  # check the package
  PatientLevelPrediction::checkPlpInstallation()
  
  # install the network package
  devtools::install_github("OHDSI/StudyProtocolSandbox/PIONEERmetastaticTreatmentPLP")
```



# Development status
Under development. Do not use
