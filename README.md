# Please do the following changes
To ensure successful code deployment, follow these steps:
- Add the AppSettings to KeyVault for configuration storage.
- Update the appsettings.json file with KeyVault Vault, ClientId, and ClientSecret configurations. These settings will automatically be fetched from Azure KeyVault, where you have defined the configurations.
- Modify the ApiVersionUsdmVersionMapping setting to the following JSON format:  "{\"SDRVersions\":[{\"apiVersion\":\"v1\",\"usdmVersions\":[\"1.0\"]},{\"apiVersion\":\"v2\",\"usdmVersions\":[\"1.9\"]},{\"apiVersion\":\"v3\",\"usdmVersions\":[\"2.0\"]}]}" .Ensure to use forward slashes, as it won't work correctly without them locally and live we can add {"SDRVersions":[{"apiVersion":"v1","usdmVersions":["1.0"]},{"apiVersion":"v2","usdmVersions":["1.9"]},{"apiVersion":"v3","usdmVersions":["2.0"]}]}.
- Create a MongoDB connection and store the configuration securely in KeyVault.
- Change the Version in OpenApiDocumentName from "V1" to "V3".

# Code setup and debugging
## Pre-requisites

1. Install [Visual Studio 2022](https://visualstudio.microsoft.com/) with default options to run the solution.

2. Create a Mongo DB(or equivalent) and collections with names mentioned below.
```
StudyDefinitions
Groups
ChangeAudit
```
3. A Service Bus Queue must be created for Change Audit functionality. This is optional if user does not intend to capture changes between versions of a study. No action is needed in code setup to disable change audit. You can also skip creation of ChangeAudit collection in previous step.

## How to setup code

1. Clone the repo into a local directory using below git command.

```shell
git clone "repo_url"
```
2. Once repo is cloned, open the solution in Visual Studio 2022 IDE.

## How To Run

**API** 
1. For running the API code locally, take a copy of appsettings.json file and rename the copied file to appsettings.Development.json file in the root folder of TransCelerate.SDR.WebApi project.

2. Edit the appsettings.Development.json and add the values for below mentioned settings.

```
"ConnectionStrings": {
    //Database connection string here
    "ServerName": "mongodb+sre://SDRADMIN:KasdeafsfhttDxaqj@study.cph52.mongodb.net/db", 
    "DatabaseName": "Database Name here"
 },
"StudyHistory": {
	// This parameter will be used to restrict the historical data (last 30/60/90 days) in study history endpoint response, if no date filters are passed in request.
	// Keep this value as "-1" to disable this restriction.
    "DateRange": "30"
 },
 "isGroupFilterEnabled": true  // change value to false to disable user based data filtering,
 "isAuthEnabled": true  // change value to false to disable authorization
 "ApiVersionUsdmVersionMapping":"" // {"SDRVersions":[{"apiVersion":"v1","usdmVersions":["1.0"]},{"apiVersion":"v2","usdmVersions":["1.9"]},{"apiVersion":"v3","usdmVersions":["2.0"]}]}
```
> **Note**  
> **API to USDM Version mapping** - SDR supports 3 major USDM versions at a given point in time along with all their minor versions. API endpoints are up-versioned for breaking changes in USDM (API V1 -> USDM V1.0, API V2 -> USDM 1.9, API V3 -> USDM 2.0).

3. Then, In the Visual Studio IDE, on clicking the IIS Express Icon or on pressing F5, WebApi solution will start running locally.

4. The browser will automatically open the Swagger UI having the SDR API specifications.

**Azure Function App**
1. An Azure Function App is developed to capture Change Audit. An Azure ServiceBus queue message triggers this function app. To run the Azure Function App locally, creation of Azure ServiceBus queue is mandatory. 
2. A local.settings.json file must be created and must be copied to the root folder of TransCelerate.SDR.AzureFunctions project.
3. Edit the local.settings.json and add the values for below mentioned settings.

```

  "Values": {    
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    
    "FUNCTIONS_WORKER_RUNTIME": "dotnet",
    
    "AzureServiceBusConnectionString": "Azure Service Bus Connection String here",
    
    "AzureServiceBusQueueName": "Queue Name here",
    
    //Database connection string here
    "ConnectionStrings:ServerName": "mongodb+sre://SDRADMIN:KasdeafsfhttDxaqj@study.cph52.mongodb.net/db",
    
    "ConnectionStrings:DatabaseName": "Database Name here",
    
    "ApiVersionUsdmVersionMapping": "", //{"SDRVersions":[{"apiVersion":"v1","usdmVersions":["1.0"]},{"apiVersion":"v2","usdmVersions":["1.9"]},{"apiVersion":"v3","usdmVersions":["2.0"]}]}
```

3. Then, In the Visual Studio IDE, select TransCelerate.SDR.AzureFunctions project on the startup project and click Start.

4. The browser will automatically open a console which will start listen on the Azure Service Bus Queue.

# Base solution structure

The solution has the following structure:

```  
  ├── TransCelerate.SDR.sln
      ├── TransCelerate.SDR.AzureFunctions
      │   ├── Properties
      │   ├── DataAccess
      │   ├── MessageProcessor
      │   ├── ChangeAuditFunction.cs
      │   ├── host.json
      │   ├── Startup.cs
      │   └── TransCelerate.SDR.Core.md
      ├── TransCelerate.SDR.Core
      │   ├── AppSettings
      │   ├── DTO
      │   ├── Entities
      │   ├── ErrorModels
      │   ├── Filters
      │   ├── Utilities
      │   └── TransCelerate.SDR.Core.md
      ├── TransCelerate.SDR.DataAccess
      │   ├── Filters
      │   ├── Interfaces
      │   ├── Repositories
      │   └── TransCelerate.SDR.DataAccess.md
      ├── TransCelerate.SDR.RuleEngine
      │   ├── Common
      │   ├── StudyRules
      │   ├── StudyV1Rules
      │   ├── StudyV2Rules
      │   ├── Token
      │   ├── UserGroupMappingRules
      │   ├── ValidationDependencies.cs
      │   ├── ValidationDependenciesCommon.cs
      │   ├── ValidationDependenciesV1.cs 
      │   ├── ValidationDependenciesV2.cs
      │   └── TransCelerate.SDR.RuleEngine.md
      ├── TransCelerate.SDR.Service
      │   ├── Interfaces
      │   ├── Services
      │   └── TransCelerate.SDR.Service.md  
      ├── TransCelerate.SDR.UnitTesting
      │   ├── AzureFunctionsUnitTesting
      │   ├── CommonClassesUnitTesting
      │   ├── ControllerUnitTesting
      │   ├── Data
      │   ├── ServicesUnitTesting
      │   └── TransCelerate.SDR.UnitTesting.md
      └── TransCelerate.SDR.WebApi
          ├── Properties
	  ├── Data
          ├── DependencyInjection
          ├── Controllers
          ├── Mappers
          ├── appsettings.json
          ├── Program.cs
          ├── Startup.cs
          └── TransCelerate.SDR.WebApi.md

```
**[TransCelerate.SDR.AzureFunctions](src/TransCelerate.SDR.AzureFunctions/TransCelerate.SDR.AzureFunctions.md)** - contains Azure function app for change audit.

**[TransCelerate.SDR.Core](src/TransCelerate.SDR.Core/TransCelerate.SDR.Core.md)** - contains entities, DTOs and helper classes.

**[TransCelerate.SDR.DataAccess](src/TransCelerate.SDR.DataAccess/TransCelerate.SDR.DataAccess.md)** - contains code for communicating with MongoDB database.

**[TransCelerate.SDR.RuleEngine](src/TransCelerate.SDR.RuleEngine/TransCelerate.SDR.RuleEngine.md)** - contains code for model validations based on data conformance rules.

**[TransCelerate.SDR.Service](src/TransCelerate.SDR.Service/TransCelerate.SDR.Services.md)** - contains code for service layer which is a bridge between API controller and repository.

**[TransCelerate.SDR.UnitTesting](src/TransCelerate.SDR.UnitTesting/TransCelerate.SDR.UnitTesting.md)** - contains code for unit testing (NUnit Framework).

**[TransCelerate.SDR.WebApi](src/TransCelerate.SDR.WebApi/TransCelerate.SDR.WebApi.md)** - contains API controllers, mappers and the startup for the application.

# SDR API
## List Of Endpoints

The below GET endpoint can be used to GET API Version -> USDM Version mapping.
```
/versions
```

### V1 Endpoints (USDM Version 1.0)

For V1 endpoints, the "usdmVersion" header parameter is mandatory and the header value must be "1.0"

**POST Endpoint**

The below endpoint can be used to create new (or) update existing study definitions.
```
/v1/studydefinitions
```

**GET Endpoints**

The below endpoint can be used to fetch all the elements for a given StudyId.

```
/v1/studydefinitions/{studyId}
```

The below endpoint can be used to fetch the sections of study design for a given StudyId.

```
​/v1​/studydesign​s?study_uuid={studyId}
```

### V2 Endpoints (USDM Version 1.9)

For V2 endpoints, the "usdmVersion" header parameter is mandatory and the header value must be "1.9"

**POST Endpoint**
The below endpoint can be used to create new study definitions.
```
/v2/studydefinitions
```
**PUT Endpoint**
The below endpoint can be used to update existing study definitions (create new version for a study definition).
```
/v2/studydefinitions/{studyId}
```
**GET Endpoints**

The below endpoint can be used to fetch all the elements for a given StudyId.

```
/v2/studydefinitions/{studyId}
```

The below endpoint can be used to fetch the sections of study design for a given StudyId.

```
/v2/studydesigns?studyId={studyId}
```
The below endpoint can be used to export study details mapped to a limited set of CPT Variables grouped by sections within the Common Protocol Template
```
/v2/studydefinitions/{studyId}/studydesigns/ecpt
```
The below endpoint can be used to fetch data from study definitions that help build the Schedule of Activities matrix for a given Schedule Timeline in a Study Design
```
/v2/studydefinitions/{studyId}/studydesigns/soa
```
### V3 Endpoints (USDM Version 2.0)

For V3 endpoints, the "usdmVersion" header parameter is mandatory and the header value must be "2.0"

**POST Endpoint**
The below endpoint can be used to create new study definitions.
```
/v3/studydefinitions
```
The below endpoint can be used to validate the USDM conformance rules for a study definition
```
/v3/studydefinitions/validate-usdm-conformance
```
**PUT Endpoint**
The below endpoint can be used to update existing study definitions (create new version for a study definition).
```
/v3/studydefinitions/{studyId}
```
**GET Endpoints**

The below endpoint can be used to fetch all the elements for a given StudyId.

```
/v3/studydefinitions/{studyId}
```

The below endpoint can be used to fetch the sections of study design for a given StudyId.

```
/v3/studydesigns?studyId={studyId}
```
The below endpoint can be used to get the changes between two SDR Upload Versions of a specific study definition
```
/v3/studydefinitions/{studyId}/version-comparison?sdruploadversionone={sdruploadversionone}&sdruploadversiontwo={sdruploadversiontwo}
```
The below endpoint can be used to export study details mapped to a limited set of CPT Variables grouped by sections within the Common Protocol Template
```
/v3/studydefinitions/{studyId}/studydesigns/ecpt
```
The below endpoint can be used to fetch data from study definitions that help build the Schedule of Activities matrix for a given Schedule Timeline in a Study Design
```
/v3/studydefinitions/{studyId}/studydesigns/soa
```
### Version Neutral Endpoints

The below endpoints can be used to fetch the revision history for a given StudyId.

```
/studydefinitions/{studyId}/revisionhistory
```

The below endpoint can be used to fetch basic details of all study definitions in SDR.

```
/studydefinitions/studyhistory
```
The below endpoint can be used to fetch study definitons in raw JSON string format

```
/studydefinitions/{studyId}/rawdata
```
The below endpoint can be used to fetch the change audit details of a study definiton
```
/studydefinitions/{studyId}/changeaudit
```

### API Spec
To view the API specifications and to run the endpoints locally, the below swagger url can be used.

```
https://localhost:44358/swagger/index.html
```
**Note**: Refer **[DDF SDR API User Guide](documents/sdr-release-v2.0.1/ddf-sdr-ri-api-user-guide-v5.0.pdf)** for detailed information on all the endpoints.

## API Versioning
SDR APIs are defined in such a way that an API version can handle more than one USDM Version. If there are no breaking changes between the USDM Versions, with same API version, more than one USDM Versions can be handled. But, when there is a breaking change in a new USDM Version, a new API version must be created to support the new USDM Version. Below are the list of changes that are required when creating a new API version.
- Configuration for **ApiVersionUsdmVersionMapping** and **ConformanceRules** must be updated to support new API version.
- Create new version for the below listed components
 ```
 TransCelerate.SDR.Core.DTO
 TransCelerate.SDR.Core.Entities
 TransCelerate.SDR.Core.Utilities.Helpers
 TransCelerate.SDR.RuleEngine
 TransCelerate.SDR.DataAccess
 TransCelerate.SDR.Services
 TransCelerate.SDR.WebApi.Controllers
 TransCelerate.SDR.WebApi.Mappers
 ```
- For version neutral endpoint search endpoint, data filters need to be added in below components to support new API version
```
 TransCelerate.SDR.DataAccess
 TransCelerate.SDR.Services
```

# Nuget Packages 

1. **Automapper.Extensions.Microsoft.DependencyInjection** - Used for mapping two different classes.

2. **Microsoft.ApplicationInsights.AspNetCore** - Used for Logging in Azure Application Insights.

3. **Newtonsoft.Json** - Used for Serialization/De-Serialization of JSON Data.
 
4. **Azure.Security.KeyVaults.Secrets** - Used for accessing Azure KeyVault.
 
5. **Azure.Identity** - Used for accessing Azure KeyVault.
 
6. **MongoDB.Driver** - Used for communicating with Mongo DB.
 
7. **NUnit** - Used for Unit Testing.

8. **Moq** - Used for mocking in unit testing.

9. **Autofac.Extras.Moq** - Used for mocking in unit testing.
 
10. **FluentValidation.AspNetCore** - Used for implementing the RuleEngine.

11. **Swashbuckle.AspNetCore** - Used for enabling Swagger for the API's.

12. **Swashbuckle.AspNetCore.Annotations** - Used for adding comments, sample request and response in Swagger.

13. **Microsoft.Extensions.Logging** - Used for logging.

14. **Microsoft.Extensions.Configuration.Abstractions** - Used for Key-Value abstractions.

15. **Microsoft.AspNetCore.Mvc.Core** - Contains common action result types, attribute routing, application model conventions, API explorer, application parts, filters, formatters, model binding, and more.

16. **Microsoft.AspNetCore.Authorization** - Used for API Authorization

17. **Vsxmd** - Used for Converting xml comments into markdown file

18. **ObjectsComparer** - Used for comparing two objects of same type and return the differences

19. **Azure.Messaging.ServiceBus** - Used for sending messages in the service bus queue

20. **Microsoft.NET.Sdk.Functions** - SDK for Azure Funtions

21. **Microsoft.AspNetCore.Mvc.Versioning** - Used for API Versioning

22. **Azure.Extensions.AspNetCore.Configuration.Secrets** - Used to get values from Key vault

23. **JsonSubTypes** - Used to Serialize/Deserialize the inherited classes.

24. **Microsoft.Identity.Web.MicrosoftGraph** - Used to connect with Azure AD and list the users available
