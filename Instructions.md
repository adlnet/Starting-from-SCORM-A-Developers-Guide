
# Starting from SCORM: A Developer's Guide

## Purpose
This session includes an overview of the SCORM to TLA Roadmap as well as a hands-on workshop where an existing SCORM course is updated to track data using xAPI.  In this tutorial you will learn:

   1. the SCORM to xAPI phases and how to determine the best fit for your organization
   2. general information about the xAPI SCORM Profile and its intended use
   3. what the SCORM-to-xAPI wrapper file is, and how to use it to convert SCOs to xAPI-enabled SCOs
   4. examples of new functionality enabled by applying xAPI to a traditional SCORM course
   
This example uses the [xAPI SCORM Profile](https://github.com/adlnet/xAPI-SCORM-Profile/blob/master/xapi-scorm-profile.md) for xAPI vocabulary and some minor behaviors.  If you are following these steps on your own, we strongly recommend that you read the [xAPI SCORM Profile](https://github.com/adlnet/xAPI-SCORM-Profile/blob/master/xapi-scorm-profile.md) (then read it again), before continuing.

## Step by Step Instructions


### Step 1 - Setup
---
In this step, we’ll examine the SCORM course to be used during this workshop and look at several resources that will be used in this demonstration.

Get the required resources:
   
   * [RosesOriginal.zip](https://github.com/adlnet/Starting-from-SCORM-A-Developers-Guide/blob/master/Steps/RosesOriginal.zip?raw=true)
   * [RosesFinal.zip](https://github.com/adlnet/Starting-from-SCORM-A-Developers-Guide/blob/master/Steps/RosesFinal.zip?raw=true) - This is the course with the changes already completed.  

Extract “RosesOriginal.zip” to a local directory on your computer.  This will be our starting place for the hands-on workshop.  

In the "RosesOriginal".zip" directory structure, examine the following files:

  * imsmanifest.xml - to see the complete list of SCOs in the course
  * Assessments/assess_q1.html - to see the typical SCORM calls made for questions in this course
  * Introduction_To_Roses/Introduction.html - to see typical non-question content in this course
  * PostTest/Posttest.html - to see the final assessment SCORM logic
  * [(SCORM 2004) APIWrapper.js](https://raw.githubusercontent.com/adlnet/SCORM-to-xAPI-Wrapper/master/SCORM2004/APIWrapper.js) - The new SCORM Wrapper containing integration points for the conversion code
  * [SCORMToXAPIFunctions.js](https://github.com/adlnet/SCORM-to-xAPI-Wrapper/blob/master/SCORMToXAPIFunctions.js) - Contains all of the code required to map SCORM data model elements to xAPI statements
  * [xapiwrapper.min.js](https://raw.githubusercontent.com/adlnet/xAPIWrapper/master/dist/xapiwrapper.min.js) - Obscures complexities of the xAPI and includes the ADL core verbs
  
Note: If you have access to an LMS and would like to import your course steps into the LMS, please do so.  To import, zip up all course files (so that the imsmanifest.xml is at the root of the zip), and use your LMS import functionality to upload the course.  
*This is not required to complete the workshop but without an LMS, viewing the results of the conversion are not possible.*

*Workshop Demonstration - Show the original SCORM Course running in an LMS and view SCORM results (and the lack of any xAPI statements in a statement viewer)*


### Step 2 - Examining JavaScript Files Used for Communication
---
In this step, we will add some resources and make simple changes to enable the tracking of many SCORM Data Model elements via the xAPI (in addition to the original SCORM tracking).

Verify the following files listed below are in the "/Shared/JavaScript" directory where you extracted "RosesOriginal.zip"

  * xapiwrapper.min.js - This file will be used to abstract the complexity of the xAPI web service components.
  * SCORMToXAPIFunctions.js - This file contains the functionality to convert SCORM data and behaviors to xAPI. 
  * APIWrapper.js - This final implements the SCORM API functionality and is commonly used in SCORM courses.  Note that you will likely be replacing the existing file.  It is important to keep the name "APIWrapper.js".  If you are not using the "Roses" example and your course does not use the APIWrapper.js file, additional details will be included below to assist with a custom integration.

Next, we'll examine the SCORMToXAPIFunction.js to get an overview of some of the complexities abstracted by the file.

Finally, we'll examine the updated APIWrapper.js file to see how the conversion functions are integrated into the exising APIWrapper.js file.


### Step 3 - Update SCOs
---
Next, add the following code in the &lt;head&gt; sections of each SCO in your course. SCO launch files can be identified by looking at the imsmanifest.xml file at the root of the SCORM package. Resource elements with adlcp:scormtype set to "sco" should contain the complete list of SCOs in the course. In this solution, each SCO will be an xAPI 'activity' with associated 'statements'. Paste the following code before the &lt;script&gt; tag that references the existing APIWrapper.js file.  This will include the basic xAPI functionality as well as the SCORM to xAPI conversion functions.

``` javascript
<script type="text/javascript" src="../Shared/JavaScript/xapiwrapper.min.js"></script>
<script type="text/javascript" src="../Shared/JavaScript/SCORMToXAPIFunctions.js"></script>
```

*Be sure that the path in the src attribute above points to the location of the minified xapiwrapper.min.js and SCORMToXAPIFunctions.js file. This location assumes that one directory up from the SCO location, that there is a Shared/JavaScript directory with your JavaScript files.*

The complete list of Roses Course SCO launch files for this step is included below:

   * /Assessments/assess_q1.html
   * /Assessments/assess_q2.html
   * /Assessments/assess_q3.html
   * /Assessments/assess_q4.html
   * /Color_Symbolism/Color_Symbolism.html
   * /Dead_Heading/Dead_Heading.html
   * /Hybrids/Rose_Hybrids.html
   * /Introduction_To_Roses/Introduction.html
   * /PostTest/Posttest.html
   * /Pruning/Pruning.html
   * /Shearing/Shearing.html
   * /Styles_Of_Floristry/Styles_Of_Floristry.html
   * /What_Is_A_Rose/What_Is_A_Rose.html


### Step 4 - Initializing Data
The demonstration requires initialization of xAPI-required configuration data in the Roses Course. For each SCO, the following information is required:

  * LRS endpoint - The LRS location where data should be sent for each SCO 
  * LRS user - The xAPI "authority" to be used when sending statements from the course.
  * LRS password - The password associated with the LRS user authority.
  * Course ID - The IRI of the entire course activity.  This will be used as context to group statements.
  * LMS Home Page - The home page of the LMS hosting the course.  This will be used as part of the actor "account" object.
  * SCORM Version - The conversion wrapper supports both SCORM Version 1.2 and SCORM 2004 but must be configured appropriately when instantiated.
  * Activity ID - The IRI of the SCO activity.  This will be used as the object of most statements.
  * Grouping Context Activity - A context activity, added to all statements, that can be used to identify a synchronous workshop involving this exercise

*Initializing Data Using imsmanifest.xml*

In the imsmanifest.xml file, identify `<item>` tags associated with SCOs that are to be converted.  Create a json object like the example below that includes configuration values of the items in the list above.  Include this object in the `<adlcp:dataFromLMS>` element for the items that reference SCOs.  The `<adlcp:dataFromLMS>` tag value is used to initialize the `cmi.launch_data` SCORM Data Model element and can be used at run-time to set the necessary xAPI configuration values.  The following is an example JSON configuration object:

``` json
    {
      "lrs":{
         "endpoint":"https://lrs.adlnet.gov/xapi/",
         "user":"xapi-workshop",
         "password":"password1234"
      },
      "courseId":"http://adlnet.gov/courses/roses",
      "lmsHomePage":"http://lms.adlnet.gov",
      "isScorm2004":true,
      "activityId":"http://adlnet.gov/courses/roses/posttest",
      "groupingContextActivity":{
         "definition": {
            "name": {
                "en-US": "My Workshop"
            },
            "description": {
                "en-US": "My Workshop happening in Nov"
            }
        },
        "id": "http://adlnet.gov/event/xapiworkshop/myworkshop",
        "objectType": "Activity"
     }
   } 
```

Remember that the `activityId` should change each time this object is used to identify a unique activity.  For the purposes of the Roses course, the following list provides the activity IDs that should be used for this exercise:

   * "http://adlnet.gov/courses/roses/q1"
   * "http://adlnet.gov/courses/roses/q2"
   * "http://adlnet.gov/courses/roses/q3"
   * "http://adlnet.gov/courses/roses/q4"
   * "http://adlnet.gov/courses/roses/symbolism"
   * "http://adlnet.gov/courses/roses/deadheading"
   * "http://adlnet.gov/courses/roses/hybrids"
   * "http://adlnet.gov/courses/roses/introduction"
   * "http://adlnet.gov/courses/roses/posttest"
   * "http://adlnet.gov/courses/roses/pruning"
   * "http://adlnet.gov/courses/roses/shearing"
   * "http://adlnet.gov/courses/roses/styles"
   * "http://adlnet.gov/courses/roses/what"

For information on the imsmanifest.xml file, `<item>` tags and `<adlcp:dataFromLMS>`, see the [SCORM on ADLNet.gov](https://www.adlnet.gov/adl-research/scorm/). 

*If you already use `<adlcp:dataFromLMS>` and the `cmi.launch_data` element, please be sure to modify this approach to handle your existing data AND the JSON object*

In addition, In order to distinguish this course from the original, change the course title in the imsmanifest.xml file at the root of the course.  The code snippet below illustrates the required change:

``` xml
...
<organizations default="ORG-35C6AD226A6FC7DB47BE726A01167EBF">
    <organization identifier="ORG-35C6AD226A6FC7DB47BE726A01167EBF" structure="hierarchical">
      <title>Roses 101 SCORM 2004 - xAPI-Converted Version</title>
      <item identifier="ITEM-5E9AC1DCB6A0F867E6197B8CDB8948C5" isvisible="true">
        <title>Module1</title>
...
```

Now the course can be imported into your LMS and used to track a subset of SCORM data via the xAPI.

*Workshop Demonstration - SCORM Course, with added xAPI tracking, in an LMS*

### Step 5 - Extra Credit
Now that the course is updated to track SCORM data to an LRS, you can access data historically not available to a SCORM SCO.  For example, you can get all of the scores associated with the post test and show the learner's score vs. the average of the class. 

In the SCORM to xAPI functions file (/Shared/JavaScript/SCORMToXAPIFunctions.js), add a function to get ALL statements from the LRS based on search/query parameters.  The complete function is listed below.

``` javascript
// extra credit extension
var GetCompleteStatementListFromLRS = function(search)
{
    var result = ADL.XAPIWrapper.getStatements(search);
    var statements = result.statements;

    while(result.more && result.more !== "")
    {
        var res = ADL.XAPIWrapper.getStatements(null, result.more);
        var stmts = res.statements;

        statements.push.apply(statements, stmts);

        result = res;
    }   

    return statements;
}
```

Then add a function that returns a custom score object that contains data about the score (the average, total number of scores, and total of the scores).   The complete function is listed below.

``` javascript
// extra credit extension
var getScoreData = function()
{
   // Set up object for score data
   var scoreStructure = new Object();
   scoreStructure.totalNumberOfScores = 0;
   scoreStructure.totalScores = 0;
   scoreStructure.average = 0;


   var search = ADL.XAPIWrapper.searchParams();
   search['activity'] = config.activityId;
   search['verb'] = ADL.verbs.scored.id;

   var statements = GetCompleteStatementListFromLRS(search);

   for (var i=0; i < statements.length; i++)
   {
      // figure out the average
      if (statements[i].result != undefined)
      {
         scoreStructure.totalNumberOfScores++;
         scoreStructure.totalScores = scoreStructure.totalScores + statements[i].result.score.scaled;         
      }
   }  

   scoreStructure.average = scoreStructure.totalScores / scoreStructure.totalNumberOfScores;

   return scoreStructure;
}
```

Next update the object return value to include the new public function.  Look for the code below and change as follows:

``` javascript
...
return{
        initializeAttempt: initializeAttempt,
        resumeAttempt: resumeAttempt,
       suspendAttempt: suspendAttempt,
       terminateAttempt: terminateAttempt,
       saveDataValue: saveDataValue,
       setScore: setScore,
       setComplete: setComplete,
       setSuccess: setSuccess,
       configureLRS: configureLRS,
       getScoreData:getScoreData
      }
...
```

Also, in order to distinguish this course from the original, change the course title in the imsmanifest.xml file at the root of the course.  The code snippet below illustrates the required change:

``` xml
...
<organizations default="ORG-35C6AD226A6FC7DB47BE726A01167EBF">
    <organization identifier="ORG-35C6AD226A6FC7DB47BE726A01167EBF" structure="hierarchical">
      <title>Roses 101 SCORM 2004 - xAPI-Converted Version - Extra Credit</title>
      <item identifier="ITEM-5E9AC1DCB6A0F867E6197B8CDB8948C5" isvisible="true">
        <title>Module1</title>
...
```

Finally, report the data by updating the post test (/PostTest/Posttest.html) to call the new getScoreData() function and display the results.  Add the following code above the “res.innerHTML = message;” line.

``` javascript
  // xAPI Extension
  var scoreStructure = xapi.getScoreData();
  message += "<br /><br />";
  message += "<strong>Experience API-Enabled Data:</strong>";
  message += "<br />";
  message += "Average score is: " + scoreStructure.average;
  message += "<br />";
  message += "Total number of scores: " + scoreStructure.totalNumberOfScores;
```

### A Note About Context
The SCORMToXAPIFunction.js file used as part of this example contains additional context activities.  This additional context is used for dashboard & reporting session later today.  


