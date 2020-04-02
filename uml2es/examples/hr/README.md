# Human Resources Example

## Version Note

This is the example presented in the blog post <http://developer.marklogic.com/blog/uml-modeling-marklogic-entity-services-semantics>. It is designed for Marklogic Data Hub Framework 4.1. 

If you're on version 5.1 or higher of Data Hub, please refer to the following examples:

[../hrHub5](../hrHub5)  - This example upgraded for DHF 5.1. It shows the use of a TDE template to generate the semantic triples!
[../hrHub5_mdModelLibs](../hrHub5)  - This example upgraded for DHF 5.1 with a MagicDraw main model that uses a separate library model. Demonstrates parent/child models in MagicDraw.
[../hrHub5_papModelLibs](../hrHub5)  - This example upgraded for DHF 5.1 with a Papyrus main model that uses a separate library model. Demonstrates parent/child models in Papyrus.

## Intro

This example shows the following:
- How to model in UML human resources (employee, department) entities. 
- How to map the UML model to a MarkLogic Entity Services model.
- How to setup a MarkLogic Data Hub to house the human resources data. 
- How to ingest source data into the Data Hub staging database. 
- How to harmonize this source data into the Data Hub final database. Significantly, this harmonized data conforms to the UML model. The harmonization code is generted by the toolkit's *cookie cutter*. The cookie cutter uses the model definition to produce harmonization source code that constructs data according to the model. In this code the cookie cutter includes copious comments to guide the developer. Comments include information from the model (including profile stereotypes), mapping instructions provided by a source data SME, and even discovery hints! More on this below.
- How to link departments and employees using semantics. The model specifies the semantic relationships. The code to create triples is GENERATED when we transform the UML model to Entity Services!!

Here is a blog post about this example: <http://developer.marklogic.com/blog/uml-modeling-marklogic-entity-services-semantics>.

For more on MarkLogic's Data Hub Framework (aka DHF), visit its GitHub page: <https://github.com/marklogic-community/marklogic-data-hub>. 

This example requires DHF 4.1 or higher. If you have an earlier version of DHF and want to run this example, use the UNO release of the UML toolkit at <https://github.com/mhavey/marklogic/releases/tag/0.0.2>. 

Our source data comes from one of the DHF examples. <https://github.com/marklogic-community/marklogic-data-hub/tree/master/examples/hr-hub>

We use the following ontology: <https://www.w3.org/TR/vocab-org/>

## Model
Here is the our UML model:

![DHFEmployeeSample](../umlModels/DHFEmployeeSample.png)

## How We Use Data Hub

A few points about our use of DHF.

First, here is the purpose of the six main databases that constitute DHF:
- Staging: This holds source data:
  - Employee data from ACME Tech
  - Employee data from Global Corp
  - Employee salary data from Global Corp
  - Department data from Global Corp
- Final
  - Harmonized employee documents that conform to the model
  - Harmonized department documents that conform to the model
  - Semantic triples representing: employee reporting structure; employee department membership; acquisition relationship between Global and ACME.
- Modules: This holds server-side modules:
  - The XMI2ES transform and code generator (aka *cookie cutter*). As a transform it translates your UML model to Entity Services. As a code generator, it generates server-side code to harmonize your data according to the model. 
  - The generated DHF harmonization modules.
  - DHF internal modules
- Schemas for Staging: This holds TDE templates and other schematic artifacts for staging. This example does not use schemas.
- Schemas for Final: This holds TDE templates and other schematic artifacts for final. This example does not use schemas.
- Jobs: This database, maintained by DHF, tracks the execution of DHF jobs. These jobs include the HR harmonization flows.

Now, about where the model fits:
- It is the data in FINAL db that conforms to the model.
- Staging DB does not conform to the model.
- Harmonization takes staging data and fashions it into the form expected by the model. Our toolkit is able to generate the harmonization code based on its knowledge of the model.  

Finally, semantics: We keep relationships "soft". Don't link documents to each other via key. Rather, use the following semantic triples:

- An employee reportsTo another employee
- An employee is a memberOf a department
- Global's acquisition of ACME uses an organizational change event.

In the UML model, we use stereotypes to describe the semantic relationships. The transform model, which converts our UML model to Entity Services, generates XQuery code to create triples based on these semantic relationships. In building our harmonization logic, we use this generated code.

## The Cooking Show Approach

Like a cooking show, this example describes how to prepare the souffle but also shows a souffle already prepred for you to consume. 

The "prepared" souffle includes:
- The UML model.
- An ML-gradle project for DHF with tasks added. 
- MLCP gradle tasks to ingest the data to STAGING
- Harmonization plugins to harmonize the data to FINAL. These plugins are generated by the toolkit (*stamped out by the cookie cutter*) but are then tweaked to deal with the fine details. The tweaks are included in the prepared souffle!
- A mapping specification: an Excel document written by a source data SME containing instructions to map source data to the model. 

If you were to start from scratch, you would follow this recipe:
- Devise the UML model in your favorite UML editor. 
- Write the mapping spec in Excel.
- Create a data hub gradle project and embellish it with tasks to load source data via MLCP
- Further embellish the gradle project with tasks to load your UML model, transform it to Entity Services, and generate harmonization code from it. 
- Tweak the generated harmonization code

## How to run:

Our project uses gradle. Before running, view the settings in gradle.properties. Create a file called gradle-local.properties and in this file override any of the properties from gradle.properties. 

### Install the Hub

We will start simple, setting up the basic hub. Run the following:

gradle -PenvironmentName=local -i setup mlDeploy

Confirm:
- No errors in gradle output
- The following app servers were created:
  * xmi2es-examples-hr-FINAL
  * xmi2es-examples-hr-STAGING
  * xmi2es-examples-hr-JOBS
- The following databases were created:
  * xmi2es-examples-hr-FINAL
  * xmi2es-examples-hr-STAGING
  * xmi2es-examples-hr-MODULES
  * xmi2es-examples-hr-final-SCHEMAS
  * xmi2es-examples-hr-final-TRIGGERS
  * xmi2es-examples-hr-staging-SCHEMAS
  * xmi2es-examples-hr-staging-TRIGGERS
- Modules DB (xmi2es-examples-hr-MODULES) contains the following module:
  * /xmi2es/xmi2esTransform.xqy - Main module of the toolkit's transform

### Load and Transform the HR UML Model

Next, move our UML model into ML as an ES model. Run the following:

gradle -b uml2es4dhf.gradle -PenvironmentName=local -i -PmodelName=DHFEmployeeSample uDeployModel 

Confirm:
- Final DB (xmi2es-examples-hr-FINAL) includes the following documents
  * /marklogic.com/entity-services/models/DHFEmployeeSample.json (The deployed ES model)
  * /xmi2es/es/DHFEmployeeSample.json (The ES model descriptor converted to JSON form)
  * /xmi2es/extension/DHFEmployeeSample.ttl (Semantic triples that extend our model)
  * /xmi2es/extension/DHFEmployeeSample.txt (A text summary of our model extension)
  * /xmi2es/gen/DHFEmployeeSample/lib.xqy (Initial generated code from the model. This is a *precursor* to the cookie-cut harmonization. That will come later.)
  * /xmi2es/findings/DHFEmployeeSample.xml (Problems found during transformation)
  * /xmi2es/xmi/DHFEmployeeSample.xml (The original UML model as an XMI document)
- Check /xmi2es/findings/DHFEmployeeSample.xml for issues during transform. It should not indicate any issues.
- In Query Console, open a tab of type SPARQL, point to the final DB, run the following query, and verify you get any results. This means the ES model is in FINAL and its semantic metadata is populated.

select * where {?s ?o ?p}

Among the results, you should see the following:
- <http://com.marklogic.es.uml.hr/HR-0.0.1/Employee> <http://marklogic.com/entity-services#property> http://com.marklogic.es.uml.hr/HR-0.0.1/Employee/emails> from basic ES model
- <http://com.marklogic.es.uml.hr/HR-0.0.1/Employee/memberOf> <http://marklogic.com/xmi2es/xes#relationship>  "association" from the extended ES model

### Create DHF Entities From the HR Model
Now we create our DHF entity plugins. We leverage's the toolkit's ability to cut/generate code. First, ask the toolkit to create the basic plugins (without any flows). It will infer which classes in the model should be plugins. 

gradle -b uml2es4dhf.gradle -PenvironmentName=local -i uCreateDHFEntities -PmodelName=DHFEmployeeSample -PentitySelect=infer 

Confirm:
- In gradle project there are new folders 
  * plugins/entities/Department
  * plugins/entities/Employee

### Create Input Flows For Source Data

For your newly created Employee and Department entities you need input flows for ingestion of source data. Run the following standard DHF gradle commands to create these flows.

gradle -PenvironmentName=local -i hubCreateInputFlow -PentityName=Employee -PflowName=LoadEmployee -PdataFormat=xml -PpluginFormat=xqy -PuseES=false

gradle -PenvironmentName=local -i hubCreateInputFlow -PentityName=Department -PflowName=LoadDepartment -PdataFormat=xml -PpluginFormat=xqy -PuseES=false

gradle -PenvironmentName=local -i mlReloadModules

Confirm:
- In your local gradle project you have newly generated code under plugins/entities/Employee/input and plugins/entities/Department/input
- These new modules are visible in the modules database (xmi2es-examples-hr-MODULES)

### Ingest

Ingest staging data and some triples for FINAL  

Run the following:

gradle -PenvironmentName=local -i loadSummaryOrgTriples runInputMLCP

Confirm:
- In STAGING (xmi2es-examples-hr-STAGING) we now have 2008 or more documents. Of these:
  * 1002 are in Employees collection
  * 1000 are in Salary collection
  * 5 are in Department collection

- In FINAL (xmi2es-examples-hr-FINAL) we have the a document containing triples in the collection http://www.w3.org/ns/org.

### Load Mapping Specs

We have two Excel mapping spec documents in data/mapping folder. One has mapping instructions for Global, the other has mapping instructions for ACME. We'll make use of these when generating the harmonization code. For now, load them into MarkLogic. Run the following:

gradle -b uml2es4dhf.gradle -Pdiscover=true -PspecName=acme-mapping uLoadMappingSpec

gradle -b uml2es4dhf.gradle -Pdiscover=true -PspecName=global-mapping uLoadMappingSpec

Confirm:
- Final DB (xmi2es-examples-hr-FINAL) includes the following documents
  * /xmi2es/excel-mapper/global-mapping.xlsx  (The Global mapping spreadsheet that we composed and loaded into ML)
  * /xmi2es/excel-mapper/global-mapping.json (The Global mapping in JSON form)
  * /xmi2es/excel-mapper/findings/global-mapping.xml  (Problems during the Global load)
  * /xmi2es/discovery/global-mapping.json (Discovery!!! While loading, the toolkit searches the staging database to intelligently map source data to the model. So, even if the author of the mapping spec isn't sure how to map a specific attribute, the tool can help suggest a possible mapping.)
  * /xmi2es/excel-mapper/acme-mapping.xlsx  (The ACME mapping spreadsheet that we composed and loaded into ML)
  * /xmi2es/excel-mapper/acme-mapping.json (The ACME mapping in JSON form)
  * /xmi2es/excel-mapper/findings/acme-mapping.xml  (Problems during the ACME load)
  * /xmi2es/discovery/acme-mapping.json (Discovery!!! While loading, the toolkit searches the staging database to intelligently map source data to the model. So, even if the author of the mapping spec isn't sure how to map a specific attribute, the tool can help suggest a possible mapping.)
- Check /xmi2es/excel-mapper/findings/global-mapping.xml and /xmi2es/excel-mapper/findings/acme-mapping.xml for issues during load. They should not indicate any issues.

### Create Harmonization Flows
We now ask the toolkit for generate harmonization flows. We need three harmonization flows: one to build a Department from Global source data, one to build an Employee from Global source data, and one to build an Employee from ACME source data. We will use the toolkit's DHF cookie cutter to generate harmonizations for each. Run the following:

gradle -b uml2es4dhf.gradle -PenvironmentName=local -i uCreateDHFHarmonizeFlow -PmodelName=DHFEmployeeSample -PflowName=harmonizeES -PentityName=Department -PpluginFormat=xqy -PdataFormat=xml -PcontentMode=es -PmappingSpec=/xmi2es/excel-mapper/global-mapping.json 

gradle -b uml2es4dhf.gradle -PenvironmentName=local -i uCreateDHFHarmonizeFlow -PmodelName=DHFEmployeeSample -PflowName=harmonizeESGlobal -PentityName=Employee -PpluginFormat=xqy -PdataFormat=xml -PcontentMode=es -PmappingSpec=/xmi2es/excel-mapper/global-mapping.json -PmappingSpec=/xmi2es/excel-mapper/global-mapping.json 

gradle -b uml2es4dhf.gradle -PenvironmentName=local -i uCreateDHFHarmonizeFlow -PmodelName=DHFEmployeeSample -PflowName=harmonizeESAcme -PentityName=Employee -PpluginFormat=xqy -PdataFormat=xml -PcontentMode=es -PmappingSpec=/xmi2es/excel-mapper/acme-mapping.json -PmappingSpec=/xmi2es/excel-mapper/acme-mapping.json 

Confirm: 
- In your local gradle project you have newly generated code under plugins/entities/Employee/harmonize/harmonizeESAcme, plugins/entities/Employee/harmonize/harmonizeESGlobal and plugins/entities/Department/harmonize/harmonizeES.
- Open that code in an editor and explore. Notice for each
  * content.xqy, headers.xqy, triples.xqy, and writer.xqy call code generated earlier, doing the model load. This code is in src/main/ml-modules/root/modelgen/DHFEmployeeSample/lib.xqy. 
  * content.xqy has LOTS of comments. These comments describe the entity and each of its attributes. The information in these comments comes from the model's stereotypes, from the Excel mapping spec, and from the discovery process run during the load of the mapping spec. Read through plugins/entities/Employee/harmonize/harmonizeESGlobal and spot the following information in the comments: URI of the full discovery report; Employee model and mapping information; Employee discovery information; firstName mapping and discovery information.
  * content.xqy clearly needs to be tweaked. Although the comments give us most of the mapping advice needed to harmonize the data, it didn't actually write the code to make that mapping. We need to make some edits to the generated code!

### Tweak the Harmonization Flow

We now tweak the content modules of the generation harmonization. We cooked those beforehand; they're in data/tweaks. Let's overwrite the generated code and deploy the changes. (These steps are automated through gradle, but we recommend you open the generated and tweaked code in your favorite diff tool and eyeball the differences.)

gradle -PenvironmentName=local -i tweakHarmonization mlReloadModules

Confirm:
- The code in plugins/entities/Department/harmonization and plugins/entities/Employee/harmonization has the tweaks.

### Harmonize
Run harmonization to move employee and department data to FINAL.

Run the following:

gradle -PenvironmentName=local -i hubRunFlow -PentityName=Department -PflowName=harmonizeES

gradle -PenvironmentName=local -i hubRunFlow -PentityName=Employee -PflowName=harmonizeESAcme

gradle -PenvironmentName=local -i hubRunFlow -PentityName=Employee -PflowName=harmonizeESGlobal

Confirm:
FINAL now contains:  
  - 5 documents in Department collection
  - 1002 documents in Employee collection

### Extra Credit: Compare Cookie-Cutter Harmonizations with DHF ES Harmonizations
Data Hub Framework, like our toolkit, can generate harmonization code that produces content based on an Entity Services model. Let's use DHF's hubCreaeHarmonizeFlow task to create an Employee harmonization. 

gradle -PenvironmentName=local -i hubCreateHarmonizeFlow -PflowName=sampleHarmonizeDHF -PentityName=Employee -PpluginFormat=xqy -PdataFormat=xml -PuseES=true

Confirm:
- Check in local project for new folder plugins/entities/Employee/harmonize/sampleHarmonizeDHF
- Using your favorite diff tool, or by eyeballing, compare this newly generated code with our toolkit's code in plugins/entities/Employee/harmonize/harmonizeESGlobal. Here are the main differences:
  * In sampleHarmonizeDHF, writer.xqy is boilerplate code. In harmonizeESGlobal, writer.xqy uses stereotypes in the model to deteremine how to write the document (i.e., which URI, collections, permissions, and so on). HarmonizeESGlobal calls the previously-generated function xesgen:runWriter_Employee to do this. 
  * In sampleHarmonizeDHF, triples.xqy and headers.xqy is boilerplate code. In harmonizeESGlobal, these modules use stereotypes from the model to determine how to produce the headers and triples sections of the envelope. They call previously-generated functions xesgen:setHeaders_Employee and xesgen:setTriples_Employee respectively. 
  * In sampleHarmonizeDHF content.xqy constructs, on a field-by-field basis, content that conforms to the ES model. HarmonizeESGlobal does this too, but it also embeds in the comments helpful information from the model itself (e.g., stereotypes), from the mapping spec produced by the source system SME, and from the data discovery process. Additionally, HarmonizeESGlobal populates calculated attributes. For example, the call xesgen:doCalculation_Employee_empLabel sets the value of the employee label. This value doesn't appear in the envelope instance, but it is needed for headers and triples generation.

## Explore the Data
In Query Console, import the workspace XMI2ESHR.xml. In each tab, try the query to explore an aspect of the data.
