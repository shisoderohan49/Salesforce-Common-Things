# Salesforce-Common-Things

- [Apex](#apex)
- [Lightning Web Components](#lightning-web-components)
- [Aura Web Components](#aura-web-components)
- [Useful SOQL Queries](#useful-soql-queries)
- [Miscellanious](#miscellanious)
- [Javascript Miscellanious](#javascript-miscellanious)

# Apex
[Back to main](#salesforce-common-things)
<details>
  <summary>List of Contents</summary>
  
  - [Getting List of picklist values for a picklist field of a Object](#getting-list-of-picklist-values-for-a-picklist-field-of-a-object)
  - [Getting List of all possible picklist values for a non-picklist field of a Object (AggregateResult)](#getting-list-of-all-possible-picklist-values-for-a-non-picklist-field-of-a-object-aggregateresult)
  - [Auto-Populating Map Entries from a SOQL Query](#auto-populating-map-entries-from-a-soql-query)
  - [Dynamic SOQL with Database.query and Database.queryWithBinds](#dynamic-soql-with-databasequery-and-databasequerywithbinds)
  - [Track status of fired Platform Events and implement subsequent business logic for failure and success events using Callbacks](#track-status-of-fired-platform-events-and-implement-subsequent-business-logic-for-failure-and-success-events-using-callbacks)
  - [Convert JSON Data to XML Data in Apex](#convert-json-data-to-xml-data-in-apex)
  - [Parsing JSON String in Apex with JSONParser class](#parsing-json-string-in-apex-with-jsonparser-class)
  - [Enable Debug Mode for a User using Apex in Execute Anonymous Window](#enable-debug-mode-for-a-user-using-apex-in-execute-anonymous-window)
  - [Reset password for a user easily using Apex in Execute Anonymous Window](#reset-password-for-a-user-easily-using-apex-in-execute-anonymous-window)
  - [Create/Update Custom Metadata Using Apex](#createupdate-custom-metadata-using-apex)
  - [Get Session Id in Execute Anonymous Window](#get-session-id-in-execute-anonymous-window)
</details>

## Getting List of picklist values for a picklist field of a Object
[Back to List of Contents](#apex)

```
Schema.DescribeFieldResult F = Object_Name.Field_API_Name.getDescribe();
List<Schema.PicklistEntry> P = F.getPicklistValues();
```
Example: 
```
Schema.DescribeFieldResult F = Account.Industry.getDescribe();
List<Schema.PicklistEntry> P = F.getPicklistValues();
```

The following are methods for PicklistEntry. All are instance methods.

| Method | Function |
|:-------:|:----------|
| getLabel() | Returns the display name of this item in the picklist. |
| getValue() | Returns the value of this item in the picklist. |
| isActive() | Returns true if this item must be displayed in the drop-down list for the picklist field in the user interface, false otherwise. |
| isDefaultValue() | Returns true if this item is the default value for the picklist, false otherwise. Only one item in a picklist can be designated as the default. |

```
Schema.DescribeFieldResult F = Account.Industry.getDescribe();
List<Schema.PicklistEntry> P = F.getPicklistValues();

List<Schema.PicklistEntry> activePicklistValues = new List<Schema.PicklistEntry>();
for(Schema.PicklistEntry entry: P){
  if(entry.isActive()){
      activePicklistValues.add(entry);
  }
}
return activePicklistValues;
```

## Getting List of all possible picklist values for a non-picklist field of a Object (AggregateResult)
[Back to List of Contents](#apex)

```
List<SelectOptionCustom> options = new List<SelectOptionCustom>{new SelectOptionCustom('--None--','')};
for(AggregateResult region: [
    SELECT FieldAPIName
    FROM ObjectName
    WHERE FieldAPIName != null
    GROUP BY FieldAPIName
    ORDER BY FieldAPIName ASC
]){
  options.add(new SelectOptionCustom((String)region.get('FieldAPIName'),(String)region.get('FieldAPIName')));
}
return options;
```

Example: 
```
List<SelectOptionCustom> options = new List<SelectOptionCustom>{new SelectOptionCustom('--None--','')};
for(AggregateResult region: [
    SELECT Country__c
    FROM Account
    WHERE Country__c != null
    GROUP BY Country__c
    ORDER BY Country__c ASC
]){
  options.add(new SelectOptionCustom((String)region.get('Country__c'),(String)region.get('Country__c')));
}
return options;
```

## Auto-Populating Map Entries from a SOQL Query
[Back to List of Contents](#apex)

```
// Populate map from SOQL query
Map<ID, Account> m = new Map<ID, Account>([SELECT Id, Name FROM Account LIMIT 10]);
// After populating the map, iterate through the map entries
for (ID idKey : m.keyset()) {
    Account a = m.get(idKey);
    System.debug(a);
}
```

## Dynamic SOQL with Database.query and Database.queryWithBinds
[Back to List of Contents](#apex)

Dynamic SOQL refers to the creation of a SOQL string at run time with Apex code. 

**query(queryString,accessLevel)**<br/>
You can use simple bind variables in dynamic SOQL query strings when using Database.query. The following is allowed:
```
String myTestString = 'TestName';
List<sObject> sobjList = Database.query('SELECT Id FROM MyCustomObject__c WHERE Name = :myTestString');
```

However, unlike inline SOQL, you can’t use bind variable fields in the query string with Database.query. The following
example isn’t supported and results in a **Variable does not exist** error.
```
MyCustomObject__c myVariable = new MyCustomObject__c(field1__c ='TestField');
List<sObject> sobjList = Database.query('SELECT Id FROM MyCustomObject__c WHERE field1__c = :myVariable.field1__c');
```

You can instead resolve the variable field into a string and use the string in your dynamic SOQL query:
```
String resolvedField1 = myVariable.field1__c;
List<sObject> sobjList = Database.query('SELECT Id FROM MyCustomObject__c WHERE field1__c =  :resolvedField1');
```
---
**queryWithBinds(queryString, bindMap, accessLevel)**<br/>
Creates a dynamic SOQL query at runtime. Bind variables in the query are resolved from the bindMap Map parameter directly with the key, rather than from Apex code variables.<br/>
`public static List<SObject> queryWithBinds(String queryString, Map<String, Object> bindMap, System.AccessLevel accessLevel)`

- `queryString`<br/>
SOQL query that includes Apex bind variables or expressions preceded by a colon. All bind variables must have a key in the bindMap Map.
- `bindMap`<br/>
A map that contains keys for each bind variable specified in the SOQL queryString and its value. The keys can’t be null or duplicates, and the values can’t be null or empty strings.
- `accessLevel`<br/>
The accessLevel parameter specifies whether the method runs in system mode (AccessLevel.SYSTEM_MODE) or user mode
(AccessLevel.USER_MODE).<br/>
In system mode, the object and field-level permissions of the current user are ignored, and the record sharing rules are
controlled by the class sharing keywords.<br/>
In user mode, the object permissions, field-level security, and sharing rules of the current user are enforced.
```
public static List<Account> simpleBindingSoqlQuery(Map<String, Object> bindParams) {
    String queryString =
        'SELECT Id, Name ' +
        'FROM Account ' +
        'WHERE name = :name';
    return Database.queryWithBinds(
        queryString,
        bindParams,
        AccessLevel.USER_MODE
    );
}

String accountName = 'Acme Inc.';
Map<String, Object> nameBind = new Map<String, Object>{'name' => accountName};
List<Account> accounts = simpleBindingSoqlQuery(nameBind);
System.debug(accounts);
```

## Track status of fired Platform Events and implement subsequent business logic for failure and success events using Callbacks
[Back to List of Contents](#apex)

```
public class SampleEventStatusCallback implements EventBus.EventPublishFailureCallback,EventBus.EventPublishSuccessCallback{
  public void onFailure(EventBus.FailureResult result){
    // Get Event UUIDs from the result
    List<String> eventUuids = result.getEventUuids();
    //.. code ..
  }

  public void onSuccess(EventBus.SuccessResult result){
    //Get Event UUIDs from the result
    List<String> eventUuids = result.getEventUuids();
    //.. code ..
  }
}
```

```
//apex code to fire the event
List<Sample_Event__e> events = new List<Sample_Event__e>();
events.add(new Sample_Event__e());
EventBus.publish(events,SampleEventStatusCallback);
```

## Convert JSON Data to XML Data in Apex
[Back to List of Contents](#apex)

```
Map<String, Object> jsonData = (Map<String, Object>) JSON.deserializeUntyped(jsonBody);
Dom.Document xmlDoc = new Dom.Document();
Dom.XmlNode rootNode = xmlDoc.createRootElement('Records', null, null);
// Extract the "records" array from the JSON
List<Object> records = (List<Object>) jsonData.get('records');
// Iterate through the records in the JSON array
for (Object record : records) {
    Map<String, Object> recordMap = (Map<String, Object>) record;
    Dom.XmlNode recordNode = rootNode.addChildElement('Record', null, null);
    // Iterate through the fields in each record
    for (String field : recordMap.keySet()) {
        Object value = recordMap.get(field);
        Dom.XmlNode fieldNode = recordNode.addChildElement(field, null, null);
        // If the value is a map, we need to handle nested elements
        if (value instanceof Map<String, Object>) {
            for (String subField : ((Map<String, Object>) value).keySet()) {
                Dom.XmlNode subFieldNode = fieldNode.addChildElement(subField, null, null);
                subFieldNode.addTextNode(String.valueOf(((Map<String, Object>) value).get(subField)));
            }
        } else {
            fieldNode.addTextNode(String.valueOf(value));
        }
    }
}
String xmlString = xmlDoc.toXmlString();
System.debug(xmlString);
```

## Parsing JSON String in Apex with JSONParser class
[Back to List of Contents](#apex)

```
public class JSONParser {
    public static void parseJSONString(String jsonString){
        System.JSONParser parser = JSON.createParser(jsonString);
        List<Invoice> invoiceList = new List<Invoice>();
        while(parser.nextToken() != null){
            if(parser.getText() == 'lineItems'){
                if(parser.nextToken() == JSONToken.START_ARRAY){
                    List<LineItem> lineItems = (List<LineItem>)parser.readValueAs(List<LineItem>.class);
                    System.debug('lineItems : ' + lineItems);
                    parser.skipChildren();
                    Invoice inv = new Invoice();
                    if(parser.nextToken() == JSONToken.FIELD_NAME && parser.getText() == 'invoiceNumber'){
                        parser.nextToken();
                        inv.invoiceNumber = parser.getLongValue();
                        inv.lineItems = lineItems;
                    }
                    System.debug('Invoice Debug : ' + inv);
                    invoiceList.add(inv);
                }
            }
        }
        return;
    }    
    
    public class Invoice{
        public Double totalPrice;
        public DateTime statementDate;
        public Long invoiceNumber;
        List<LineItem> lineItems;
        
        public Invoice(){
            
        }
        
        public Invoice(Double price,DateTime statementDate,Long invoiceNumber,List<LineItem> liList){
            this.totalPrice = price;
            this.statementDate = statementDate;
            this.invoiceNumber = invoiceNumber;
            this.lineItems = liList;
        }
    }
    public class LineItem{
        public Double unitPrice;
        public Double quantity;
        public String productName;
    }
}
```
Execute Anonymous Window Code Execution
```
String jsonStr = 
    '{"invoiceList":[' +
    '{"totalPrice":5.5,"statementDate":"2011-10-04T16:58:54.858Z","lineItems":[' +
    '{"UnitPrice":1.0,"Quantity":5.0,"ProductName":"Pencil"},' +
    '{"UnitPrice":0.5,"Quantity":1.0,"ProductName":"Eraser"}],' +
    '"invoiceNumber":1},' +
    '{"totalPrice":11.5,"statementDate":"2011-10-04T16:58:54.858Z","lineItems":[' +
    '{"UnitPrice":6.0,"Quantity":1.0,"ProductName":"Notebook"},' +
    '{"UnitPrice":2.5,"Quantity":1.0,"ProductName":"Ruler"},' +
    '{"UnitPrice":1.5,"Quantity":2.0,"ProductName":"Pen"}],"invoiceNumber":2}' +
    ']}';

JSONParser.parseJSONString(jsonStr);
```

[JSONParser Class Documentation](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_class_System_JsonParser.htm#apex_System_JsonParser_readValueAs)
[JSONToken Enum](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_enum_System_JsonToken.htm)

## Enable Debug Mode for a User using Apex in Execute Anonymous Window
[Back to List of Contents](#apex)

```
User rohan = [SELECT Id,UserPreferencesUserDebugModePref FROM User WHERE Name = 'Rohan Shisode'];
rohan.UserPreferencesUserDebugModePref = true;
update rohan;
System.debug('rohan details : ' + rohan);
```

## Reset password for a user easily using Apex in Execute Anonymous Window
[Back to List of Contents](#apex)

```
User rohan = [SELECT Id FROM User WHERE Name = 'Rohan Shisode'];
System.resetPassword(rohan.Id,true);
```

[System.resetPassword Reference](https://developer.salesforce.com/docs/atlas.en-us.apexref.meta/apexref/apex_methods_system_system.htm#apex_System_System_resetPassword)<br/>
When the user logs in with the new password, they are prompted to enter a new password, and to select a security question and answer if they haven't already.<br/>
If you specify true for sendUserEmail, the user is sent an email notifying them that their password was reset. A link to sign onto Salesforce using the new password is included in the email.<br/> 
Use `setPassword(userId, password)` if you don't want the user to be prompted to enter a new password when they log in.

**Ensure that the email address that the user has is a valid email address**

## Create/Update Custom Metadata Using Apex
[Back to List of Contents](#apex)

[Original Article From Panter Schools](https://www.pantherschools.com/create-update-custom-metadata-using-apex/)

Steps - 
1. **Create Custom Metadata**<br/>

Navigate to Setup > Develop > Custom Metadata Types > New Custom Metadata Type > Create the Custom Metadata Type

2. **Create a class and implement the Metadata.DeployCallback interface**<br/>

```
public class CreateUpdateMetadataUtils implements Metadata.DeployCallback {
    
}
```

3. **Implement handleResult method in the class**
```
public class CreateUpdateMetadataUtils implements Metadata.DeployCallback {
    /* Below method recieves the 2 parameters 
       1 - Metadata.DeployResult => Object of MetadataContainer class where this method has been implemented.
       2 - Metadata.DeployCallbackContext => Object of the class where this method has been implemented
   */
    public void handleResult(Metadata.DeployResult result,
                             Metadata.DeployCallbackContext context) {
        if (result.status == Metadata.DeployStatus.Succeeded) {
            // Deployment was successful
        } else {
            // Deployment was not successful
        }
    }
}
```

4. **Prepare the Custom Metadata**<br/>

Create the Object of `Metadata.CustomMetadata` as follows
```
Metadata.CustomMetadata customMetadata = new Metadata.CustomMetadata();
customMetadata.fullName = 'Profile_Setting.Admin'
customMetadata.label = 'Admin';
```

Value for `fullName` we need to provide the Qualified Name of the Custom MetadataType<br/>
If Object Name of Metadata is 'Profile_Setting' and you want to create a record named Admin, then <br/>
fullName will be `Profile_Setting.Admin` and <br/>
label will be `Admin`.

5. **Add field values to the Metadata Record**<br/>
```
/* Create the Object of CustomMetadataValue */
Metadata.CustomMetadataValue customField = new Metadata.CustomMetadataValue();
/* Provide the API Name of the Field like Admin__c */
customField.field = 'Profile_Full_Name__c';
/* Provide the value for the field */
customField.value = 'Admin';
/* Add this field to the Metadata That we created */ 
customMetadata.values.add(customField);

/* If you wanted to add more than one field you need to repeat the above step OR You can use Map<String, Object> where Key is the Field API Name and value is the value for the field */

/* In my example, I will create a separate method which is responsible for deploying the Custom Metadata Record */
```

6. **Create a Deploy Container**<br/>

Let's create a deployed container that will be responsible for deploying the Custom Metadata Record.<br/>
Create the DeployContainer class object<br/>
`Metadata.DeployContainer mdContainer = new Metadata.DeployContainer()`<br/>
Add the Custom Metadata in the DeployContainer using `addMetadata` method<br/>
`mdContainer.addMetadata(customMetadata)`

7. **Deploy the Custom Metadata**<br/>

Create the object of our class<br/>
`CreateUpdateMetadataUtils callback = new CreateUpdateMetadataUtils();`
Deploy the Metadata using this code<br/>
`Id jobId = Metadata.Operations.enqueueDeployment(mdContainer,callback)`

The Final and complete class - 
```
public class CreateUpdateMetadataUtils implements Metadata.DeployCallback {
    
    public void handleResult(Metadata.DeployResult result, Metadata.DeployCallbackContext context) {
        if (result.status == Metadata.DeployStatus.Succeeded) {
            System.debug(' success : '+ result);
        } else {
            System.debug(' fail : '+ result);
        }
    }
    
    public static void createUpdateMetadata(String fullName, String label, Map<String, Object> fieldWithValuesMap){
        Metadata.CustomMetadata customMetadata =  new Metadata.CustomMetadata();
        customMetadata.fullName = fullName;
        customMetadata.label = label;
        
        for(String key : fieldWithValuesMap.keySet()){
            Metadata.CustomMetadataValue customField = new Metadata.CustomMetadataValue();
            customField.field = key;
            customField.value = fieldWithValuesMap.get(key); 
            customMetadata.values.add(customField);
        }
        
        Metadata.DeployContainer mdContainer = new Metadata.DeployContainer();
        mdContainer.addMetadata(customMetadata);
        CreateUpdateMetadataUtils callback = new CreateUpdateMetadataUtils();
        Id jobId = Metadata.Operations.enqueueDeployment(mdContainer, callback);
    }
}
```

## Get Session Id in Execute Anonymous Window
[Back to List of Contents](#apex)

```
System.debug('Session id '+UserInfo.getOrganizationId() + UserInfo.getSessionId().substring(15));
```

# Lightning Web Components
[Back to main](#salesforce-common-things)
<details>
  <summary>List of Contents</summary>
  
  - [Row Selection in Lightning-Datatable Miscellanious Things](#row-selection-in-lightning-datatable-miscellanious-things)
  - [Show Toast Function](#show-toast-function)
  - [Hyperlink Record in lightning-datatable](#hyperlink-record-in-lightning-datatable)
  - [Applying Styles to Lightning Datatable](#applying-styles-to-lightning-datatable)
  - [Dynamically Instantiate Components](#dynamically-instantiate-components)
  - [lwc:spread - Spread Properties on Child Components](#lwcspread---spread-properties-on-child-components)
  - [lwc:if|elseif={expression} and lwc:else Conditional Rendering](#lwcifelseifexpression-and-lwcelse-conditional-rendering)
  - [for:each directive](#foreach-directive)
  - [Iterating with iterator lwc directive](#iterating-with-iterator-lwc-directive)
  - [Picklist Values of Object Field In LWC](#picklist-values-of-object-field-in-lwc)
  - [Access Record, Object Context and Component Width-Aware When Component used on a Lightning Record Page](#access-record-object-context-and-component-width-aware-when-component-used-on-a-lightning-record-page)
  - [Access Record, Object Context When Component Used in Experience Builder Sites](#access-record-object-context-when-component-used-in-experience-builder-sites)
  - [Picklist Custom Datatype for Lightning Datatable](#picklist-custom-datatype-for-lightning-datatable)
  - [Cancel Changes of a single row in LightningDatatable Inline Edit](#cancel-changes-of-a-single-row-in-lightningdatatable-inline-edit)
  - [Implement Infinite Table Loading in a normal HTML Table in LWC](#implement-infinite-table-loading-in-a-normal-html-table-in-lwc)
  - [Implement Picklist Dependencies between two field comboboxes in LWC](#implement-picklist-dependencies-between-two-field-comboboxes-in-lwc)
  - [Execute functions of Child LWC Components from Parent LWC Component](#execute-functions-of-child-lwc-components-from-parent-lwc-component)
  - [Implement Debouncing in Lightning Web Component](#implement-debouncing-in-lightning-web-component)
  - [Implement toast notifications in Flows](#implement-toast-notifications-in-flows)
  - [Implement Throttling](#implement-throttling)
  - [Implement toast notifications in Flows](#implement-toast-notifications-in-flows)
  - [Display Toast Message in multiple lines](#display-toast-message-in-multiple-lines)
</details>

## Row Selection in Lightning-Datatable Miscellanious Things
[Back to List of Contents](#lightning-web-components)

HTML Element
```
<div class="slds-var-p-around_small" style="height: 300px">
  <lightning-datatable
        key-field="Id"
        data={tableData}
        columns={columns}
        selected-rows={selectedRows}
        onrowselection={handleRowSelection}
        max-row-selection="2"
  ></lightning-datatable>
</div>
```

in JS Element, handling row selection
```
selectedRows = [];

handleRowSelection(event){
  var eventDetail = JSON.parse(JSON.stringify(event.detail));

  //Get array of selected rows - those elements in tableData that have been selected
  var selectedRow = eventDetail.selectedRows;
  //Get the action that was done by the user
  //userActions - rowSelect,rowDeselect,selectAllRows
  var userAction = eventDetail.config.action;
  //Get the row number that was selected (starts with 1)
  var targetRow = eventDetail.config.value;

  //get row that user acted on from tableData
  let tRow = JSON.parse(JSON.stringify(this.tableData[targetRow - 1]));

  //Write logic when the first element is selected
  if( userAction == "rowSelect" && selectedRow.length === 1 ){
     //logic ...
  }

  //write logic when more than one element has been selected
  if( userAction == "rowSelect" && selectedRow.length > 1 ){
    //logic ...
  }

  //write logic when a element has been deselected and now there are still selected rows left
  if( userAction == "rowDeselect" && selectedRow.length >= 1 ){
    //logic ...
  }

  //when the only remaining element has been de-selected by the user
  if( userAction == "rowDeselect" && selectedRow.length === 0 ){
    //logic ...
  }

  //write logic for when user selects all rows (clicks on the checkbox in the header)
  if( userAction == 'selectAllRows' ){
    //logic ...
  }
}
```

## Show Toast Function
[Back to List of Contents](#lightning-web-components)

Import Statement
```
import { ShowToastEvent } from "lightning/platformShowToastEvent"
```

Function
```
showToast(title, message, variant) {
    const event = new ShowToastEvent({
      title: title,
      message: message,
      variant: variant
    });
    this.dispatchEvent(event);
}
```

## Hyperlink Record in lightning-datatable
[Back to List of Contents](#lightning-web-components)

```
columns = [
  {
    label: 'ColumnHeaderName',
    fieldName: 'fieldNameURLLink',
    type: 'url',
    typeAttributes: {
      label: {
        fieldName: 'fieldNameAppearsInRow'
      },
      target: '_blank'
    },
    wrapText: true
  }
]
```

Record ID Link
```
tableData[i].fieldNameURLLink = '/' + recordId;
```

|target="..."|Function|
|:-----------:|:----------|
| _blank |Opens the linked document in a new window or tab|
| _self |	Opens the linked document in the same frame as it was clicked (this is default)|

## Applying Styles to Lightning Datatable
[List of Contents](#lightning-web-components)

```
const columns = [
    {
        label: 'Account Name',
        fieldName: 'Name',
        cellAttributes: {
            class: 'slds-text-color_success slds-text-title_caps'
        }
    }
];
```

Custom Styling - 

1. Upload your custom CSS File as a static resource
2. Import the static resource and load it using loadStyle

```
import { loadStyle } from 'lightning/platformResourceLoader';
import lwcDatatableStyle from '@salesforce/resourceUrl/lwcDatatableStyle';

export default class LWCDatatableCSS extends LightningElement {
    isCSSLoaded = false;
    /* Rest of the LWC Component Logic
      ...............
    */  

    renderedCallback(){
        if(this.isCSSLoaded) {
            return;
        }

        this.isCSSLoaded = true;

        loadStyle(this,lwcDatatableStyle).then(() => {
              console.log("Loaded Successfully");
        }).catch(error => {
            console.log(error);
        });
    }
}
```

- Align text in cellAttributes<br>
`alignment` can be `left`,`right` or `center`
```
{
  label: 'Account Name',
  fieldName: 'Name',
  cellAttributes: {
    alignment: 'left'
  }
}
```

## Dynamically Instantiate Components
[Back to List of Contents](#lightning-web-components)

- To instantiate a dynamic component, a component's configuration file must include the lightning__dynamicComponent
capability.

```
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
  <apiVersion>59.0</apiVersion>
  <capabilities>
    <capability>lightning__dynamicComponent</capability>
  </capabilities>
</LightningComponentBundle>
```
To use this capability, you must set the apiVersion property to 55.0 or later.

- HTML Template Syntax
```
<template>
    <div class="container">
        <lwc:component lwc:is={componentConstructor}></lwc:component>
    </div>
</template>
```

- `<lwc:component>` serves as a placeholder in the DOM that renders the specified dynamic component. You must use `<lwc:component>` with the lwc:is directive.
- The `lwc:is` directive provides an imported constructor at runtime to the `<lwc:component>` managed element. `lwc:is` accepts an expression that resolves to a LightningElement constructor at runtime.
- If the constructor is false, the `<lwc:component>` tag along with all of its children aren't rendered.
- If the expression value is defined but not a LightningElement constructor, an error is thrown.
- In the component's JavaScript file, import the custom element using the import() dynamic import syntax.

```
import { LightningElement } from "lwc";
export default class extends LightningElement {
  componentConstructor;
  // Use connectedCallback() on the dynamic component
  // to signal when it's attached to the DOM
  connectedCallback() {
    import("c/concreteComponent")
      .then(({ default: ctor }) => (this.componentConstructor = ctor))
      .catch((err) => console.log("Error importing component"));
  }
}
```

- Similar to a regular component, the dynamic component is instantiated and attached to the DOM. If the dynamic component's constructor changes, the existing element is removed from the DOM.

Alternatively using async and await keywords to return the component constructor

```
import { LightningElement } from "lwc";
export default class App extends LightningElement {
  componentConstructor;
  async connectedCallback() {
    const { default: ctor } = await import("c/myComponent");
    this.componentConstructor = ctor;
  }
}
```

**Make the dynamic imports statically analyzable**<br/>
While not currently implemented, future framework optimizations can optimize code where dynamic imports are statically analyzable. Pass a JavaScript string literal to the `import()` function:
```
import("c/analyzable"); // 👍: Statically analyzable
import("c/" + "analyzable"); // 👎: Not statically analyzable
import("c/" + componentName); // 👎: Not statically analyzable
```

**Example of using dynamic imports**

```
// 👍 USE THIS
// Example with statically analyzable dynamic import:
import { LightningElement, api } from "lwc";

const KNOWN_TYPE = new Set(["bar", "pie", "line"]);
const CHART_MAPPING = {
  bar: () => import("c/barChart"),
  pie: () => import("c/pieChart"),
  line: () => import("c/lineChart"),
};

export default class App extends LightningElement {
  chartCtor;

  _type = "line";

  @api
  get type() {
    return this._type;
  }
  set type(val) {
    if (!KNOWN_TYPE.has(val)) {
      console.warn(`Unknown chart type: ${val}`);
    }

    this._type = val;
    CHART_MAPPING[val]().then(({ default: ChartCtor }) => {
      this.chartCtor = ChartCtor;
    });
  }
}
```

## lwc:spread - Spread Properties on Child Components
[Back to List of Contents](#lightning-web-components)

The `lwc:spread` directive accepts one object.<br>
Use an object with key-value pairs, where the keys are property names.

App Component
```
<!-- app.html -->
<template>
  <c-child lwc:spread={childProps}></c-child>
</template>
```
```
// app.js
import { LightningElement } from "lwc";

export default class extends LightningElement {
  childProps = { name: "James Smith", country: "USA" };
}
```

Child Component
```
<!-- child.html -->
<template>
  <p>Name: {name}</p>
  <p>Country : {country}</p>
</template>
```
```
// child.js
import { LightningElement, api } from "lwc";

export default class Child extends LightningElement {
  @api name;
  @api country;
}
```

**Reflect HTML Attributes on a Child Component**<br>
- Most of the HTML attributes are reflected as properties.
- For example, class attribute is reflected as className property.

```
<!-- app.html -->
<template>
  <c-child lwc:spread={spanProps}></c-child>
</template>
```
```
// app.js
import { LightningElement } from "lwc";

export default class extends LightningElement {
  spanProps = { className: "spanclass", id: "myspan" };
}
```


## lwc:if|elseif={expression} and lwc:else Conditional Rendering
[Back to List of Contents](#lightning-web-components)
```
<template lwc:if={expression}></template>
<template lwc:elseif={expression_elseif1}></template>
<template lwc:elseif={expression_elseif2}></template>
<template lwc:else></template>
```
- Conditionally render DOM elements in a template. `lwc:if`, `lwc:elseif`, and `lwc:else` supersede the `if:true` and `if:false` directives.
- Use the conditional directives on nested \<template> tags, \<div> tags or other HTML elements, and on your custom
components tags like <c-custom-cmp>.
- Both `lwc:elseif` and `lwc:else` must be immediately preceded by a sibling `lwc:if` or `lwc:elseif`.
- The expression passed in to `lwc:if` and `lwc:elseif` supports simple dot notation. Complex expressions like `!condition`, `object?.property?.condition` or `sum % 2 === 1` aren't supported. To compute such expressions, use a getter in the JavaScript class.
- You can't precede `lwc:elseif` or `lwc:else` with text or another element. Whitespace is ignored between the tags when the whitespace is a sibling of the conditional directive. 

## for:each directive
[Back to List of Contents](#lightning-web-components)

- When using the `for:each` directive, use `for:item="currentItem"` to access the current item. This example doesn’t use it, but to access the current item’s index, use `for:index="index"`.
- To assign a key to the first element in the nested template, use the `key={uniqueId}` directive.

```
<!-- helloForEach.html -->
<template>
  <lightning-card title="HelloForEach" icon-name="custom:custom14">
    <ul class="slds-m-around_medium">
      <template for:each={contacts} for:item="contact">
        <li key={contact.Id}>{contact.Name}, {contact.Title}</li>
      </template>
    </ul>
  </lightning-card>
</template>
```
```
// helloForEach.js
import { LightningElement } from "lwc";
export default class HelloForEach extends LightningElement {
  contacts = [
    {
      Id: 1,
      Name: "Amy Taylor",
      Title: "VP of Engineering",
    },
    {
      Id: 2,
      Name: "Michael Jones",
      Title: "VP of Sales",
    },
    {
      Id: 3,
      Name: "Jennifer Wu",
      Title: "CEO",
    },
  ];
}
```
## Iterating with iterator lwc directive
[Back to List of Contents](#lightning-web-components)

**To apply a special behavior to the first or last item in a list, use the iterator directive, `iterator:iteratorName={array}`. Use the iterator directive on a template tag.**
<br/>
Use iteratorName to access these properties:

- `value` — The value of the item in the list. Use this property to access the properties of the array. For example, `{iteratorName}.value.{propertyName}`.
- `index` — The index of the item in the list.
- `first`— A boolean value indicating whether this item is the first item in the list.
- `last` — A boolean value indicating whether this item is the last item in the list.

To apply special rendering to the first and last items in the list, the code uses the first and last properties with the `lwc:if` directive.
- If the item is first in the list, the \<div> tag renders with the styling defined in the CSS list-first class.
- If the item is last in the list, the \<div> tag renders with the styling defined in the CSS list-last class.

```
<template>
  <lightning-card title="HelloIterator" icon-name="custom:custom14">
    <ul class="slds-m-around_medium">
      <template iterator:it={contacts}>
        <li key={it.value.Id}>
          <div lwc:if={it.first} class="list-first"></div>
          {it.value.Name}, {it.value.Title}
          <div lwc:if={it.last} class="list-last"></div>
        </li>
      </template>
    </ul>
  </lightning-card>
</template>
```
```
.list-first {
  border-top: 1px solid black;
  padding-top: 5px;
}
.list-last {
  border-bottom: 1px solid black;
  padding-bottom: 5px;
}
```

## Picklist Values of Object Field in LWC
[Back to List of Contents](#lightning-web-components)

Using `getPicklistValues` and `getObjectInfo` from `'lightning/uiObjectInfoApi'`

```
import {LightningElement,wire} from 'lwc';
import {getPicklistValues,getObjectInfo} from 'lightning/uiObjectInfoApi';
import OBJECT_INFO from '@salesforce/schema/ObjectApiName';
import FIELD_INFO from '@salesforce/schema/ObjectApiName.FieldApiName';
export default class someLWCComponent extends LightningElement{
  @track picklistOptions;

  @wire(getObjectInfo,{
    objectApiName: OBJECT_INFO
  })
  objectInfo;

  @wire(getPicklistValues,{
    recordTypeId: '$objectInfo.data.defaultRecordTypeId',
    fieldApiName: FIELD_INFO
  })
  wirePicklist({error,data}){
    if(data){
      this.picklistOptions = data.values;
    }else if(error){
      console.log(error);
    }
  }
}
```

---
**2nd Method**

Apex Class
```
public class AccountDataController{
  @AuraEnabled
  public static List<SelectOptionCustom> getTypePicklistValues(){
    Schema.DescribeFieldResult typeFieldResult = Account.Type.getDescribe();
    List<Schema.PicklistEntry> picklistEntries = typeFieldResult.getPicklistValues();

    List<SelectOptionCustom> activePicklistValues = new List<SelectOptionCustom>();
    for(Schema.PicklistEntry entry: picklistEntries){
      if(entry.isActive()){
        activePicklistValues.add(new SelectOptionCustom(entry.getLabel(),entry.getValue()));
      }
    }

    return activePicklistValues;
  }

  public class SelectOptionCustom{
    @AuraEnabled
    public String label;

    @AuraEnabled
    public String value;

    public SelectOptionCustom(String label,String value){
      this.label = label;
      this.value = value;
    }
  }
}
```

In LWC
```
import getTypePicklistValues from '@salesforce/apex/AccountDataController.getTypePicklistValues';
//imports ....
export default class CustomLWCComponent extends LightningElement{
  //code ...
  picklistOptions = [];
  initializeData(){
    getTypePicklistValues()
    .then(data => {
      this.picklistOptions = data;
      console.log('this.picklistOptions',this.picklistOptions);
    })
    .catch(error => {
      console.log(error);
    });
}
```
## Access Record, Object Context and Component Width-Aware When Component used on a Lightning Record Page
[Back to List of Contents](#lightning-web-components)

- If a lightning web component with a recordId property is used on a Lightning record page, then the page sets recordId to the 18-character ID of the record. Create a public recordId property by declaring it using @api decorator.
```
@api recordId;
```

- If a lightning web component with an objectApiName property is used on a Lightning record page, then the page sets the API name of the object associated with the record being viewed. Create a public objectApiName property by declaring it using the @api decorator.
```
@api objectApiName;
```

- If a lightning web component with a flexipageRegionWidth property is used on a Lightning record page, then the page pass the region’s width to the component. Create a public flexipageRegionWidth property by declaring it using @api decorator.
```
@api flexipageRegionWidth;
```

## Access Record, Object Context When Component Used in Experience Builder Sites
[Back to List of Contents](#lightning-web-components)

Unlike lightning record pages Experience Builder Sites do not automatically bind the recordId and objectApiName to the component’s template.

To access the record Id of the current record you have to add recordId and objectApiName in an expression in the component’s *.js-meta.xml file. 
```
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>55.0</apiVersion>
    <isExposed>true</isExposed>
    <targets>
        <target>lightningCommunity__Default</target>
        <target>lightning__RecordPage</target>
    </targets>
    <targetConfigs>
        <targetConfig targets="lightningCommunity__Default">
            <property
                name="recordId"
                type="String"
                label="Record Id"
                description="Pass the page's record id to the component variable"
                default="{!recordId}" />
            <property
                name="objectApiName"
                type="String"
                label="Object Name"
                description="Pass the page's object name to the component variable"
                default="{!objectApiName}" />
        </targetConfig>
    </targetConfigs>
</LightningComponentBundle>
```

## Picklist Custom Datatype for Lightning Datatable
[Back to List of Contents](#lightning-web-components)

Originally taken from [TechDicer](https://techdicer.com/picklist-in-lwc-datatable-inline-edit/)

Template<br/>
`picklistColumn.html`
```
<template>
    <span class="slds-truncate" title={value}>
        {value}
    </span>
</template>
```

Edit Template<br/>
`picklistStatic.html`
```
<template>
  <lightning-combobox
    name="picklist"
    data-inputable="true"
    label={typeAttributes.label}
    value={editedValue}
    placeholder={typeAttributes.placeholder}
    options={typeAttributes.options}
    variant="label-hidden"
  ></lightning-combobox>
</template>
```

HTML FILE<br/>
`lWCCustomDatatableType.html`
```
<template>
</template>
```

JS File<br/>
`lWCCustomDatatableType.js`
```
import LightningDatatable from 'lightning/datatable';
import picklistColumn from './picklistColumn.html';
import picklistStatic from './picklistStatic.html';

export default class LWCCustomDatatableType extends LightningDatatable {
    static customTypes = {
        picklistColumn: {
            template: picklistStatic,
            editTemplate: picklistColumn,
            standardCellLayout: true,
            typeAttributes: ['label','placeholder','options','value','context','variant','name']
        }
    }
}
```

## Cancel Changes of a single row in LightningDatatable Inline Edit
[Back to List of Contents](#lightning-web-components)

![image](https://github.com/shisoderohan49/Salesforce-Common-Things/assets/90911451/252cd4b7-5e6c-4b4d-95f3-ea0f5f5d20a8)

The LightningDatatable or the extended LightningDatatable in HTML
```
<c-l-w-c-custom-datatable-type
          key-field="Id"
          data={data}
          columns={columns}
          onvalueselect={handleSelection}
          draft-values={draftValues}
          oncellchange={handleCellChange}
          onrowaction={callRowAction}
          onsave={handleSave}
          oncancel={handelCancel}
          hide-checkbox-column
></c-l-w-c-custom-datatable-type>
```

Let the apex function for updating the records be imported in the JS file
```
@AuraEnabled
public static List<Account> updateAccountRecords(String wrapperText){
    List<Account> accountList = (List<Account>)JSON.deserialize(wrapperText, List<Account>.class);
    System.debug('wrapperText : ' + wrapperText);
    update accountList;
    return accountList;
}
```
`import updateAccountRecords from '@salesforce/apex/AccountDataController.updateAccountRecords';`

Columns in JS File
```
columns = [
  //...
  {
    type: 'button',
    fixedWidth: 50,
    label: '',
    typeAttributes: {
      name: "removeFromDraftValues",
      disabled: false,
      iconName: 'utility:delete',
      variant: 'base'
    },
    cellAttributes: {
      alignment: 'center'
    }
]
```

Make all draftValues array empty when user cancels inline edit
```
handleCancel(event){
  this.draftValues = [];
}
```

on save event 
- update the records in Salesforce with Imperative Apex 
- update the respective row in data array with the new field values
- make draft values array empty
```
handleSave(event){
  Promise.resolve()
  .then(() => {
      console.log('handleSave event',JSON.parse(JSON.stringify(event.detail)));
      let newAccountValues = event.detail.draftValues;
      return updateAccountRecords({wrapperText: JSON.stringify(newAccountValues)});
  })
  .then((result) => {
      console.log('updateAccountValues result : ',result);
      result.forEach(element => {
        let index = this.data.findIndex(ele => ele.Id === element.Id);
        for(let property in element){
          (this.data[index])[property] = element[property];
        }
      });
      this.template.querySelector('c-l-w-c-custom-datatable-type').draftValues = [];
  })
  .catch((error) => {
      console.log('error : ',error);
  })
}
```

onrowaction event handler
```
callRowAction(event){
  let actionName = event.detail.action.name;
  let eventDetail = JSON.parse(JSON.stringify(event.detail));
  console.log('event.detail',eventDetail);
  switch(actionName){
    case "removeFromDraftValues":
      this.removeFromDraftValuesAction(eventDetail);
  }
}
```

onrowaction helper function - removeFromDraftValuesAction
```
removeFromDraftValues(eventDetail){
    let draftValues = this.template.querySelector('c-l-w-c-custom-datatable-type').draftValues;
    if(draftValues.some(e => e.Id === eventDetail.row.Id)){
      var newValues = [...draftValues].filter(e => e.Id !== eventDetail.row.Id);
      this.template.querySelector('c-l-w-c-custom-datatable-type').draftValues = newValues;
    }
}
```


## Implement Infinite Table Loading in a normal HTML Table in LWC
[Back to List of Contents](#lightning-web-components)

```
//Infinite loading for Table
renderedCallback(){
    const tableDiv = this.template.querySelector('.tableFixHead');
    tableDiv.addEventListener(
        'scroll',
        () => {
            if(Math.abs(tableDiv.scrollHeight - tableDiv.clientHeight - tableDiv.scrollTop) < 1){
                this.loadMoreChowRecords();
            }
        },
        {
            passive: true,
        }
    )
}

loadMoreChowRecords(){
    if(this.displayedRecords.length < this.totalRecords.length){
        this.displayedRecords = [...(this.totalRecords.slice(0,this.totalRecords.length + 100))]
    }
}
```

## Implement Picklist Dependencies between two field comboboxes in LWC
[Back to List of Contents](#lightning-web-components)

In HTML

```
<lightning-combobox
  name="grantType"
  label="Grant Type"
  value={grantTypeValue}
  placeholder="Select a Grant Type from picklist"
  options={grantTypeValues}
  onchange={handleGrantTypeChange}
  required>
</lightning-combobox>
<lightning-combobox
  name="requestType"
  label="Reason For Request"
  value={requestForReasonValue}
  placeholder="Select Reason For Request"
  options={requestForReasonValues}
  onchange={handleReasonChange} required>
</lightning-combobox>

```

In LWC, import getObjectInfo and getPicklistValuesByRecordType
```
import OBJECT_REFERENCE from '@salesforce/schema/OBJECT_API_NAME';
import { getObjectInfo, getPicklistValuesByRecordType } from 'lightning/uiObjectInfoApi';
```
In the defined Lightning Class `export default class ...`.
```
controllingPicklistValue;
dependentPicklistValue;

controllingPicklistOptions;
dependentPicklistOptions;

objectPicklistValues;

//.... code ....
@wire(getObjectInfo,{
    objectApiName: OBJECT_REFERENCE
})
objectInfo;

@wire(getPicklistValuesByRecordType,{
    objectApiName: OBJECT_REFERENCE,
    recordTypeId: '$objectInfo.data.defaultRecordTypeId'
})
wiredPicklistValues({error,data}){
    if(data){
        console.log('data : ',JSON.parse(JSON.stringify(data)));
        this.objectPicklistValues = data.picklistFieldValues;
        let controllingPicklistList = data.picklistFieldValues.CONTROLLING_FIELD_API_NAME.values;
        let controllingPicklistValues = [];
        for(let i = 0; i < controllingPicklistList.length; i++){
            controllingPicklistValues.push({
                label: controllingPicklistList[i].label,
                value: controllingPicklistList[i].value
            });
        }
        console.log('controllingPicklistValues: ',JSON.parse(JSON.stringify(controllingPicklistValues)));
        this.controllingPicklistOptions = controllingPicklistValues;
        console.log('this.controllingPicklistOptions: ',JSON.parse(JSON.stringify(this.controllingPicklistOptions)));
    }else if(error){
        console.log('error: ',JSON.parse(JSON.stringify(error)));
    }
}

handleControllingFieldChange(event){
    this.controllingPicklistValue = event.detail.value;
    this.dependentPicklistValue = null;

    console.log('this.controllingPicklistValue after change: ',JSON.parse(JSON.stringify(this.controllingPicklistValue)));
    console.log('this.dependentPicklistValue: ',this.dependentPicklistValue);

    if(this.controllingPicklistValue){
        let totalDependentField = this.objectPicklistValues.DEPENDENT_FIELD_API_NAME;
        console.log('totalDependentField: ',JSON.parse(JSON.stringify(totalDependentField)));
        let controllerValueIndex = totalDependentField.controllerValues[this.controllingPicklistValue];
        let totalDependentFieldValues = totalDependentField.values;
        let dependentPicklistValues = [];
        totalDependentFieldValues.forEach(key => {
            for(let i = 0;i < key.validFor.length;i++){
                if(controllerValueIndex == key.validFor[i]){
                    dependentPicklistValues.push({
                        label: key.label,
                        value: key.value
                    });
                }
            }
        });
        console.log('dependentPicklistValues: ',JSON.parse(JSON.stringify(dependentPicklistValues)));
        if(dependentPicklistValues && dependentPicklistValues.length > 0){
            this.dependentPicklistOptions = dependentPicklistValues;
        }
    }
}
```

## Execute functions of Child LWC Components from Parent LWC Component
[Back to List of Contents](#lightning-web-components)

Child LWC Component
```
export default class childLWCComponent extends LightningElement{
    //... code ...
    @api
    executeSampleFunction(detail){
        //code execution
    }
}
```

In Parent LWC Component, use this code wherever required.<br/>
```
let detailObj = {
    //....
}
this.template.querySelector('c-child-l-w-c-component').executeFunction(detailObj);
```

## Implement Debouncing in Lightning Web Component
[Back to List of Contents](#lightning-web-components)

Debouncing - Programming Practice used to ensure that time-consuming tasks do not fire so often, that it stalls the performance of the web page. In other words, it limits the rate at which a function is invoked.<br/>
Debouncing is removing unwanted input noise from buttons, switches or other user input. Debouncing prevents extra activations or slow functions from triggering too often.

HTML Component - input element
```
<input type="text" value={searchKey} onkeyup={handleKeyChange} onchange={handleKeyChange}/> 
```

JS Component
```
export default class LookupComponent extends LightningElement {
    /* set up variables */
    timer;

    /* code */
    handleKeyChange(event){
        window.clearTimeout(this.timer);
        const searchKey = event.target.value;
        this.searchKey - searchKey;
        this.timer = setTimeout(() => {
            //processing code
        },500);
        //set up the time - 500ms according to your response needs
    }
}
```

## Implement Throttling 
[Back to List of Contents](#lightning-web-components)

Throttling is a technique that limits how often a function can be called in a given period of time.
It is useful for improving the performance and responsiveness of web pages that have event listeners that trigger heavy or expensive operations.

![image](https://github.com/shisoderohan49/Salesforce-Common-Things/assets/90911451/fa72d3cc-2f00-453b-9749-7ffd4b545d78)

```
function throttle(mainFunction,delay){
    let timerFlag = null;// Variable to keep track of the timer
    
    // Returning a throttled version
    return (...args) => {
        if(timerFlag === null){ // If there is no timer currently running
            mainFunction(...args); //Execute main function
            timerFlag = setTimeout(() => { // Set a timer to clear the timerFlag after the specified delay
                timerFlag = null; // Clear the timerFlag to allow the main function to be executed again
            },delay);
        }
    }
}
```

Use Case: 
```
//Define a function that fetches some data from an API
function fetchData(){
    console.log("Fetching data...");
    //Simulate an API call with a random delay
    setTimeout(() => {
        console.log("Data fetched!");
    },Math.random() * 1000);
}

//Throttle the fetchData function with a delay of 5000ms
const throttledFetchData = throttle(fetchData,5000);

//Add an event listener to the window scroll event that calls the throttledFetchData function
window.addEventListener("scroll",throttledFetchData);
```

## Implement toast notifications in Flows
[Back to List of Contents](#lightning-web-components)

[Reference](https://developer.salesforce.com/docs/component-library/bundle/lightning-flow-support/documentation)
[Inspiration](https://hugolemos.medium.com/display-toast-notifications-in-flows-9a5c12c473ed)

Deploy following lightning web component in org and then use it in flow.
Set the Navigation Event Type according to requirement.

`showToastFlow.html`
```
<template>
</template>
```

`showToastFlow.js`
```
import { LightningElement,api} from 'lwc';
import {FlowNavigationNextEvent,FlowNavigationFinishEvent} from 'lightning/flowSupport';
import { ShowToastEvent } from "lightning/platformShowToastEvent";

export default class ShowToastFlow extends LightningElement {

    hasRendered = false;

    @api title;
    @api variant;
    @api message;
    //NEXT or FINISH
    @api navigationEventType;

    renderedCallback(){
        this.showToastMessage(this.title,this.message,this.variant);
        if(this.navigationEventType === 'NEXT'){
            const navigateNextEvent = new FlowNavigationNextEvent();
            this.dispatchEvent(navigateNextEvent);
        }else if(this.navigationEventType === 'FINISH'){
            const navigateFinishEvent = new FlowNavigationFinishEvent();
            this.dispatchEvent(navigateFinishEvent);
        }
    }

    async showToastMessage(title,message,variant){
        const event = new ShowToastEvent({
            title: title,
            message: message,
            variant: variant,
        });
        this.dispatchEvent(event);
    }
}
```

`showToastFlow.js-meta.xml`
```
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>59.0</apiVersion>
    <isExposed>true</isExposed>
    <targets>
        <target>lightning__FlowScreen</target>
    </targets>
    <targetConfigs>
        <targetConfig targets="lightning__FlowScreen">
            <property name="title" label="Toast Title" type="string" role="inputOnly" />
            <property name="message" label="Toast Message" type="string" role="inputOnly" />
            <property name="variant" label="Toast Variant" type="string" role="inputOnly" />
            <property name="navigationEventType" label="Navigation Event Type (NEXT or FINISH)" type="string" role="inputOnly" />
        </targetConfig>
    </targetConfigs>
</LightningComponentBundle>
```

![image](https://github.com/shisoderohan49/Salesforce-Common-Things/assets/90911451/37381a1d-60f2-498b-a13c-34f7ab236c86)

## Display Toast Message in multiple lines 
[Back to List of Contents](#lightning-web-components)

[Link](https://salesforce.stackexchange.com/questions/407814/how-to-display-toast-message-in-multiple-lines-using-lwc)

Need to override the CSS to show the toast message in separate lines.
- Create one Static Resource file for the CSS.
`toastMessage.css`
```
.toastMessage {
        white-space: break-spaces !important;
}
```
Use this static resource in your LWC
```
import { LightningElement, track } from 'lwc';
import { ShowToastEvent } from 'lightning/platformShowToastEvent';
import { loadStyle } from 'lightning/platformResourceLoader';
import customCSS from '@salesforce/resourceUrl/toastCSS';
export default class DataBinding extends LightningElement {
    
    @track isCSSLoaded = false;

    renderedCallback() {

        if (this.isCSSLoaded) return;
        this.isCSSLoaded = true;
        loadStyle(this, customCSS).then(() => {
            console.log('css loaded successfully');
        }).catch(error => {
            console.log('error to load css');
        });
    }


    handleToastMsg() {
        var msg = 'First Message\nSecond Message\nThird Message';

        const evt = new ShowToastEvent({
            title: 'Toast Success',
            message: msg,
            variant: 'success',
            mode: 'dismissable'
        });
        this.dispatchEvent(evt);
    }
}
```

# Aura Web Components
[Back to main](#salesforce-common-things)

<details>
  <summary>List of Contents</summary>

  - [lightning:overlayLibrary to use for modals in Aura Components](#lightningoverlaylibrary-to-use-for-modals-in-aura-components)
</details>

## lightning:overlayLibrary to use for modals in Aura Components

The lightning:overlayLib component provides access to methods you can use in your components to open and close modals and popovers.

Include one <lightning:overlayLibrary aura:id="overlayLib"/> tag in the component that triggers the modal or popover, where aura:id is a unique local ID. Only one tag is needed in a component to open multiple messages.
Include the tag also in a component that closes a modal or popover, even if it doesn't open them.

an example of a component c:modalOpener that uses a button to open a modal. When clicked, the button displays a modal with a custom body that's defined in another component.<br/>
**modalOpener.cmp**
```
<aura:component>
    <lightning:overlayLibrary aura:id="overlayLib"/>
    <lightning:button name="modal" label="Show Modal" onclick="{!c.handleShowModal}"/>
</aura:component>
```

The modalOpenerController.js client-side controller displays the modal. To create and display a modal, pass in the modal parameters using component.find('overlayLib').showCustomModal(), where overlayLib matches the aura:id specified in the <lightning:overlayLibrary> tag in the component markup. This example dynamically creates a custom body using $A.createComponent().<br/>
**modalOpenerController.js**
```
({
    handleShowModal: function(component, evt, helper) {
        var modalBody;
        $A.createComponent("c:modalContent", {},
           function(content, status) {
               if (status === "SUCCESS") {
                   modalBody = content;
                   component.find('overlayLib').showCustomModal({
                       header: "Application Confirmation",
                       body: modalBody,
                       showCloseButton: true,
                       cssClass: "mymodal",
                       closeCallback: function() {
                           alert('You closed the alert!');
                       }
                   })
               }
           });
    }
})
```

c:modalContent is a custom component that displays an icon and message as shown in this markup.<br/>
**modalContent.cmp**
```
<aura:component>
    <lightning:icon size="medium" iconName="action:approval" alternativeText="Approved" />
    Your application has been approved.
</aura:component>
```
- The showCloseButton parameter when set to true causes the X close button to be displayed. The cssClass parameter specifies a class to apply to the modal.
- Any custom CSS class you add with the cssClass parameter must be accompanied by the cMyCmp class, where c is your namespace and MyCmp is the name of the component that creates the modal. For this example, the class is cModalOpener. Adding this class ensures that the custom styling is properly scoped.
- The closeCallback parameter causes a function to be called when the modal is closed. Firing an event is a typical use case.

**Creating a Modal Footer**<br/>
Pass in a footer for the modal by using the footer parameter. You can pass a string or a component to include buttons, for example. Here's a client-side controller that creates a custom body and custom footer using $A.createComponents().
<br/>
**modalOpenerController.js**
```
({
    handleShowModalFooter : function (component, event, helper) {
    var modalBody;
    var modalFooter;
    $A.createComponents([
        ["c:modalContent",{}],
        ["c:modalFooter",{}]
    ],
    function(components, status){
        if (status === "SUCCESS") {
            modalBody = components[0];
            modalFooter = components[1];
            component.find('overlayLib').showCustomModal({
               header: "Application Confirmation",
               body: modalBody,
               footer: modalFooter,
               showCloseButton: true,
               cssClass: "my-modal,my-custom-class,my-other-class,cModalOpener",
               closeCallback: function() {
                   alert('You closed the alert!');
               }
           })
        }
    }
   );
  }
})
```
c:modalFooter is a custom component that displays Cancel and OK buttons as shown in this markup. The footer component includes the <lightning:overlayLibrary aura:id="overlayLib"/> tag so it can call the notifyClose() method in the client-side controller.
<br/>
**modalFooter.cmp**
```
<aura:component>
    <lightning:overlayLibrary aura:id="overlayLib"/>
    <lightning:button name="cancel" label="Cancel" onclick="{!c.handleCancel}"/>
    <lightning:button name="ok" label="OK" variant="brand" onclick="{!c.handleOK}"/>
</aura:component>
```
modalFooterController.js defines what happens when you click the footer buttons. The handleCancel function uses the notifyClose() method from the overlay library.
<br/>
**modalFooterController.js**
```
({
    handleCancel : function(component, event, helper) {
        //closes the modal or popover from the component
        component.find("overlayLib").notifyClose();
    },
    handleOK : function(component, event, helper) {
        //do something
    }
})
```
showCustomModal() returns a promise, which is useful if you want to get a reference to the modal when it’s displayed. 

# Useful SOQL Queries
[Back to main](#salesforce-common-things)

<details>
  <summary>List of Contents</summary>

  - [Get All Permission Set and Permission Set Group Assignments to a User](#get-all-permission-set-and-permission-set-group-assignments-to-a-user)
  - [Get All Users that have been assigned a certain Permission Set](#get-all-users-that-have-been-assigned-a-certain-permission-set)
  - [Get All Permission Sets contained in a Permission Set Group](#get-all-permission-sets-contained-in-a-permission-set-group)
  - [Retrieve all Apex classes granted access within a Permission Set](#retrieve-all-apex-classes-granted-access-within-a-permission-set)
  - [Retrieve all Permission Sets that grant access to an Apex Class](#retrieve-all-permission-sets-that-grant-access-to-an-apex-class)
  - [Get All Asynchronous Jobs filtered by the job's Status, Created By User,Apex Class Name, Job Type](#get-all-asynchronous-jobs-filtered-by-the-jobs-status-created-by-userapex-class-name-job-type)
  - [Get Information about Flows From their Developer Names](#get-information-about-flows-from-their-developer-names)
  - [Get All Profiles which have Edit Field Level Security for a Field](#get-all-profiles-which-have-edit-field-level-security-for-a-field)
  - [Retrieve All Permission Sets that provide Edit Access to a Field](#retrieve-all-permission-sets-that-provide-edit-access-to-a-field)
  - [Retrieve Read and Edit Permissions for all fields of a Specific Object for a given Profile](#retrieve-read-and-edit-permissions-for-all-fields-of-a-specific-object-for-a-given-profile)
  - [Retrieve all components that reference or utilize an Apex Class - Tooling API](#retrieve-all-components-that-reference-or-utilize-an-apex-class-tooling-api)
</details>

## Get All Permission Set and Permission Set Group Assignments to a User
[Back to List of Contents](#useful-soql-queries)

```
SELECT Id,Assignee.Name,PermissionSetGroup.DeveloperName,PermissionSet.Name FROM PermissionSetAssignment WHERE Assignee.Name = 'Rohan Shisode'
```

## Get All Users that have been assigned a certain Permission Set
[Back to List of Contents](#useful-soql-queries)

```
SELECT Id,AssigneeId,Assignee.Name,Assignee.Profile.Name,PermissionSet.Name FROM PermissionSetAssignment WHERE PermissionSet.Name = 'PERMISSION_SET_NAME'
```

## Get All Permission Sets contained in a Permission Set Group
[Back to List of Contents](#useful-soql-queries)

```
SELECT PermissionSetGroup.DeveloperName,PermissionSet.Name FROM PermissionSetGroupComponent WHERE PermissionSetGroup.DeveloperName = 'PermissionSetGroupName'
```

## Retrieve all Apex classes granted access within a Permission Set
[Back to List of Contents](#useful-soql-queries)

```
SELECT SetupEntityType,SetupEntity.Name FROM SetupEntityAccess 
WHERE ParentId IN (SELECT Id FROM PermissionSet WHERE Name = 'PERMISSION_SET_NAME') 
AND SetupEntityType = 'ApexClass' 
ORDER BY SetupEntity.Name
```

## Retrieve all Permission Sets that grant access to an Apex Class
[Back to List of Contents](#useful-soql-queries)

```
SELECT Name,(SELECT Id,Parent.Name FROM SetupEntityAccessItems) FROM ApexClass 
WHERE Name = 'APEX_CLASS_NAME'
```

## Get All Asynchronous Jobs filtered by the job's Status, Created By User,Apex Class Name, Job Type
[Back to List of Contents](#useful-soql-queries)

```
SELECT Id,ApexClass.Name,Status,JobType,CreatedBy.Name FROM AsyncApexJob WHERE JobType IN ('ScheduledApex','BatchApex') AND CreatedBy.Name = 'USERNAME' AND ApexClass.Name = 'APEXCLASSNAME'
```

## Get Information about Flows From their Developer Names
[Back to List of Contents](#useful-soql-queries)

```
SELECT Id,MasterLabel,DeveloperName, LastModifiedDate, LastModifiedBy.Name,Description FROM FlowDefinition WHERE DeveloperName IN ('FLOWDEVELOPERNAME1','FLOWDEVELOPERNAME2',FLOWDEVELOPERNAME3')
```

## Get All Profiles which have Edit Field Level Security for a Field
[Back to List of Contents](#useful-soql-queries)

```
SELECT Id,Field,SObjectType,PermissionsRead,PermissionsEdit,Parent.Profile.Name FROM FieldPermissions 
WHERE SObjectType = 'OBJECT_API_NAME' AND Field = 'OBJECT_API_NAME.FIELD_API_NAME' AND PermissionsEdit = true AND Parent.ProfileId != null 
ORDER BY Parent.Profile.Name
```

## Retrieve all Permission Sets that provide Edit Access to a Field
[Back to List of Contents](#useful-soql-queries)

```
SELECT Parent.Name from FieldPermissions WHERE Field = 'SOBJECT_API_NAME.FIELD_API_NAME' and PermissionsEdit = true and Parent.IsOwnedByProfile = false
```

## Retrieve Read and Edit Permissions for all fields of a Specific Object for a given Profile
[Back to List of Contents](#useful-soql-queries)

```
SELECT Field,PermissionsRead,PermissionsEdit FROM FieldPermissions 
WHERE SObjectType = 'OBJECT_API_NAME' AND Parent.Profile.Name = 'PROFILE_NAME'
ORDER BY Field
```

## Retrieve all components that reference or utilize an Apex Class [Tooling API]
[Back to List of Contents](#useful-soql-queries)

Retrieve the Id of the Apex Class 
```
SELECT Id FROM ApexClass WHERE Name = 'APEX_CLASS_NAME'
```
Then Run this Query 
```
SELECT MetadataComponentName, MetadataComponentType, RefMetadataComponentName, RefMetadataComponentType FROM MetadataComponentDependency WHERE RefMetadataComponentId = 'APEX_CLASS_ID'
```

# Miscellanious
[Back to main](#salesforce-common-things)

<details>
  <summary>List of Contents</summary>
  
  - [Generate Salesforce Authentication Token Using Postman](#generate-salesforce-authentication-token-using-postman)
  - [Profile Error Due to Access Granted to Salesforce Licensed Custom Objects (LCO)](#profile-error-due-to-access-granted-to-salesforce-licensed-custom-objects-lco)
  - [Using JSForce npm package for automating Tooling/Bulk API Requests](#using-jsforce-npm-package-for-automating-toolingbulk-api-requests)
</details>

## Generate Salesforce Authentication Token Using Postman

* Set up a connected app in Salesforce

* Get Consumer details from Connected App (consumer key and consumer secret)
![image](https://github.com/shisoderohan49/Salesforce-Common-Things/assets/90911451/df6084b2-cbfd-4305-886e-02670d183b43)
![image](https://github.com/shisoderohan49/Salesforce-Common-Things/assets/90911451/670264bb-00ff-4c3e-b844-448488df59e5)

* Get Security Token from Personal User Settings
![image](https://github.com/shisoderohan49/Salesforce-Common-Things/assets/90911451/5df17d45-f666-4014-9c0c-3fa9a07d3b4b)

* Create POSTMAN Request

POST Request

Authentication URL: https://login.salesforce.com/services/oauth2/token
(https://test.salesforce.com/services/oauth2/token for sandbox org)

In Body Tab for the POST Request

|Field|Value|
|:------:|:------|
|grant_type|password|
|client_id|CONSUMER_KEY|
|client_secret|CONSUMER_SECRET|
|username|YOUR_SALESFORCE_USERNAME|
|password|YOUR_SALESFORCE_PASSWORD + YOUR_SALESFORCE_SECURITY_TOKEN|

![image](https://github.com/shisoderohan49/Salesforce-Common-Things/assets/90911451/d201ab3d-1378-4865-9a98-4315820a6931)


* Receive the Access Token in the Response Body

![image](https://github.com/shisoderohan49/Salesforce-Common-Things/assets/90911451/65d2736f-27fa-411c-8a49-efa3ab2f6bad)

## Profile Error Due to Access Granted to Salesforce Licensed Custom Objects (LCO)

Beginning in the Winter '21 releases for CPQ and Advanced Approvals (AA), profiles are no longer able to grant access to any Salesforce Licensed Custom Objects (LCO). To interact with CPQ or AA related objects, users are required to have Permission Set Licenses (PSL) assigned. Validation error messages will appear when editing permissions in profiles or permission sets in 228 if any user belonging to the group does not have an appropriate license.<br/>

`ERROR:  [PERMISSION NAME] on [OBJECT NAME] can't be granted. Grant the permission using a permission set with the required license or use a permission set not associated with a specific license.`

<br/>
Salesforce Help Article to tackle the problem if the issue exists for multiple profiles 
- [Anonymous Apex to remove CPQ Licensed Custom Object access from Profiles](https://help.salesforce.com/s/articleView?id=000390380&type=1/)

If numerous profiles are in need of updates to be compliant post-upgrade to CPQ 228 (Winter '21), admins can use the query below to assist in removing CPQ related object permissions from custom profiles. Follow the steps below to execute the script in the Developer Console:
1. Open the Developer Console
2. Debug > Open Execute Anonymous Window (CTRL+E)
3. Copy and paste code in Anonymous Window
4. Check: Open Log
5. Execute
6. Check the 'Debug Only' filter box

`**REMOVES ALL CPQ LCO + DEPENDENT OBJECT PERMISSIONS:`
```
// 1. Query for Permission set owned by custom profiles, exclude System Admin (By Customize Application perm)
List<PermissionSet> profilePermSetIds = [select ProfileId from PermissionSet where IsOwnedByProfile = true and Profile.PermissionsModifyAllData=false and IsCustom=true];
Set<Id> permSetIds = (new Map<Id,SObject>(profilePermSetIds)).keySet();
System.debug('Profiles found: '+permSetIds.size());

// 2. Define CPQ+AA objects (LCO + dependents) by hierarchy order for deletion
String[] customLCOAndDependentObjects = new String[] {
'SBQQ__QuoteLinePricingGuidance__c','SBQQ__QuoteDocument__c','SBQQ__QuoteLineConsumptionRate__c','SBQQ__QuoteLineConsumptionSchedule__c','SBQQ__QuoteLine__c','SBQQ__QuoteLineGroup__c','SBQQ__Quote__c',
'SBQQ__PricingGuidanceTier__c','SBQQ__PricingGuidance__c',
'SBQQ__ErrorCondition__c','SBQQ__ConfigurationRule__c','SBQQ__ProductAction__c','SBQQ__ProductRule__c',
'SBQQ__LookupQuery__c','SBQQ__PriceCondition__c','SBQQ__PriceAction__c','SBQQ__PriceRule__c',
'SBQQ__LineColumn__c','SBQQ__TemplateSection__c','SBQQ__QuoteTemplate__c',
'SBQQ__SubscriptionConsumptionRate__c','SBQQ__SubscriptionConsumptionSchedule__c','SBQQ__SubscribedAsset__c','SBQQ__Subscription__c',
'sbaa__ApprovalSnapshot__c','sbaa__Approval__c',
'sbaa__ApprovalCondition__c','sbaa__ApprovalRule__c'
};

ObjectPermissions[] profileObjPerms;
// 3. Iterate and delete, so not all done in 1 batch
for(String objName : customLCOAndDependentObjects) {
// Query for object permissions by object
profileObjPerms = [SELECT Id FROM ObjectPermissions
where Parent.Id in :permSetIds and SobjectType = :objName];
System.debug('Object perms for object:'+objName+':'+profileObjPerms.size());
try{
// Uncomment below line before executing
// delete profileObjPerms;
} catch(Exception e){
System.debug('Obj perms update failed due to the follow reason: '+ e + ',' + e.getStackTraceString());
}
}

// 4. Query to ensure all the relevant object perms are cleared
ObjectPermissions[] objPerms = [SELECT Id FROM ObjectPermissions
where Parent.Id in :permSetIds and SobjectType = :customLCOAndDependentObjects];
System.debug('Rerun Object Perm Records for CPQ & AA: '+ objPerms.size());
```

## Using JSForce npm package for automating Tooling/Bulk API Requests
[Back to Miscellanious](#miscellanious)

- Install the `JSforce` npm package and import it in your JS script  
`npm install jsforce` OR `npm --force install jsforce` if required  
`const jsforce = require('jsforce')`  

- Retrieve the Session Id from Developer Console with this command<br/>
  `System.debug('Session id '+UserInfo.getOrganizationId() + UserInfo.getSessionId().substring(15));`

    
Javascript function for creating a connection object (sid is your Session ID)
```
async function getConnectionObj(sid){
    var conn = new jsforce.Connection({
        instanceUrl: 'YOUR_INSTANCE_URL',
        serverUrl: 'YOUR_SERVER_URL',
        sessionId: sid
    });
    return conn;
}
```

Code for retrieving results of Tooling API Query  
```
const conn = await getConnectionObj('YOUR_SESSION_ID');
const result = await conn.tooling.query('YOUR_TOOLING_API_QUERY');
var records = result.records;
```

Code for creating Bulk API Query Job
```
var requestBody = {
    operation: 'query',
    query: /services/data/v62.0/tooling/jobs/query
}
var recordStream = await conn.request({
    method: 'POST',
    url: 'REQUIRED_URL',
    body: JSON.stringify(requestBody),
    headers: {
        'Content-Type': 'application/json'
    }
})
```
Output will be of the format 
```

HTTP/1.1 200 OK
{
    "id" : "750R0000000$H8AAU",
    "operation" : "query",
    "object" : "MetadataComponentDependency",
    "createdById" : "005xx000001X9JAAS",
    "createdDate" : "2020-04-09T17:51:02.000+0000",
    "systemModstamp" : "2020-04-09T17:51:02.000+0000",
    "state" : "UploadComplete",
    "concurrencyMode" : "Parallel",
    "contentType" : "CSV",
    "apiVersion" : 49.0,
    "lineEnding" : "LF",
    "columnDelimiter" : "COMMA"
}
```
After the Bulk API Job is Complete.<br/>
Get the query job's results by running this request using the earlier job's ID
```
var resultURL = `/services/data/v62.0/tooling/jobs/query/${jobId}/results`;
var jobResults = await conn.request({
    method: 'GET',
    url: resultURL,
    headers: {
        'Content-Type': 'application/json'
    }
});
```

# Javascript Miscellanious
[Back to main](#salesforce-common-things)

<details>
  <summary>List of Contents</summary>
  
  - [Extract only selected keys from array of objects](#extract-only-selected-keys-from-array-of-objects)
  - [Generate Array for sequence of numbers](#generate-array-for-sequence-of-numbers)
  - [Get all permutations of an array](#get-all-permutations-of-an-array)
  - [Group elements in an Array](#group-elements-in-an-array)
  - [Divide the array into chunks of a given size](#divide-the-array-into-chunks-of-a-given-size)
  - [Remove duplicates from an array of objects](#remove-duplicates-from-an-array-of-objects)
  - [Dependent Async Tasks ](#dependent-async-tasks)
</details>

## Extract only selected keys from array of objects
[Back to List of Contents](#javascript-miscellanious)

```
function extractKeys(arrayToBeMapped,keys){
    let result = arrayToBeMapped.map(element => {
        var newObject = {};
        keys.forEach(key => {
            if(typeof(element[key]) != undefined && element[key]!= null){
                newObject[key] = element[key];
            }
        });
        return newObject;
    });
    return Array.from(result);
}
```

## Generate Array for sequence of numbers 
[Back to List of Contents](#javascript-miscellanious)

```
function generateSequence(num){
  return Array.from(new Array(num + 1).keys()).slice(1);
}
```

## Get all permutations of an array 
[Back to List of Contents](#javascript-miscellanious)

```
function permutations(inputArr){
    let result = [];
    const permute = (arr,m = []) => {
      if(arr.length === 0){
        result.push(m);
      } else {
        for(let i = 0;i < arr.length; i++){
          let curr = arr.slice();
          let next = curr.splice(i,1);
          permute(curr.slice(),m.concat(next));
        }
      }
    }

    permute(inputArr);
    return result;
}
```
## Group elements in an Array
[Back to List of Contents](#javascript-miscellanious)

1. Using the Object.groupBy(iterable,callbackFn) method
2. Using the reduce function

**Using the Object.groupBy method**

- Just using a field as the grouping parameter

```
const inventory = [
  { name: "asparagus", type: "vegetables", quantity: 5 },
  { name: "bananas", type: "fruit", quantity: 0 },
  { name: "goat", type: "meat", quantity: 23 },
  { name: "cherries", type: "fruit", quantity: 5 },
  { name: "fish", type: "meat", quantity: 22 },
];

const result = Object.groupBy(inventory,({type}) => type);
```
Result : 
```
{
  vegetables: [
    { name: 'asparagus', type: 'vegetables', quantity: 5 },
  ],
  fruit: [
    { name: "bananas", type: "fruit", quantity: 0 },
    { name: "cherries", type: "fruit", quantity: 5 }
  ],
  meat: [
    { name: "goat", type: "meat", quantity: 23 },
    { name: "fish", type: "meat", quantity: 22 }
  ]
}
```

- Using a callback function as the grouping parameter

```
const inventory = [
  { name: "asparagus", type: "vegetables", quantity: 5 },
  { name: "bananas", type: "fruit", quantity: 0 },
  { name: "goat", type: "meat", quantity: 23 },
  { name: "cherries", type: "fruit", quantity: 5 },
  { name: "fish", type: "meat", quantity: 22 },
];

function myCallback({quantity}){
    return quantity > 5 ? 'ok' : 'restock';
}

const result2 = Object.groupBy(inventory,myCallback);
```
Result : 
```
{
  restock: [
    { name: "asparagus", type: "vegetables", quantity: 5 },
    { name: "bananas", type: "fruit", quantity: 0 },
    { name: "cherries", type: "fruit", quantity: 5 }
  ],
  ok: [
    { name: "goat", type: "meat", quantity: 23 },
    { name: "fish", type: "meat", quantity: 22 }
  ]
}
```

**Using the reduce method**

```
const inventory = [
  { name: "asparagus", type: "vegetables", quantity: 5 },
  { name: "bananas", type: "fruit", quantity: 0 },
  { name: "goat", type: "meat", quantity: 23 },
  { name: "cherries", type: "fruit", quantity: 5 },
  { name: "fish", type: "meat", quantity: 22 },
];

const result = inventory.reduce((acc,curr) => {
    acc[curr['type']] = acc[curr['type']] || [];
    acc[curr['type']].push(curr);
    return acc;
},Object.create(null));
```
Result : 
```
{
  vegetables: [
    { name: 'asparagus', type: 'vegetables', quantity: 5 },
  ],
  fruit: [
    { name: "bananas", type: "fruit", quantity: 0 },
    { name: "cherries", type: "fruit", quantity: 5 }
  ],
  meat: [
    { name: "goat", type: "meat", quantity: 23 },
    { name: "fish", type: "meat", quantity: 22 }
  ]
}
```
Using some other criteria than a field to group elements of an array
```
const inventory = [
  { name: "asparagus", type: "vegetables", quantity: 5 },
  { name: "bananas", type: "fruit", quantity: 0 },
  { name: "goat", type: "meat", quantity: 23 },
  { name: "cherries", type: "fruit", quantity: 5 },
  { name: "fish", type: "meat", quantity: 22 },
];

let initialValue = {
    ok: [],
    restock: []
};

const result = inventory.reduce((acc,curr) => {
    if(curr['quantity'] > 5) acc['ok'].push(curr);
    else acc['restock'].push(curr);
    return acc;
},initialValue);
```
Result: 
```
{
  restock: [
    { name: "asparagus", type: "vegetables", quantity: 5 },
    { name: "bananas", type: "fruit", quantity: 0 },
    { name: "cherries", type: "fruit", quantity: 5 }
  ],
  ok: [
    { name: "goat", type: "meat", quantity: 23 },
    { name: "fish", type: "meat", quantity: 22 }
  ]
}
```

## Divide the array into chunks of a given size
[Back to List of Contents](#javascript-miscellanious)

```
var returnChunkArr = function(zipcodeArr,chunkSize){
    var result = [];
    var i = 0;
    while(i < zipcodeArr.length){
        result.push(zipcodeArr.slice(i,i + chunkSize));
        i += chunkSize;
    }
    return result;
}
```

## Remove duplicates from an array of objects
[Back to List of Contents](#javascript-miscellanious)

```
function removeDuplicates(arr){
    var properties = Object.keys(arr);
    return arr.filter((value,index,self) => {
        return index === self.findIndex(element => {
            var result = true;
            properties.forEach(prop => {
                result = result && (element[prop] === value[prop]);
            });
            return result;
        });
    })
}
```

## Dependent Async Tasks 
[Back to List of Contents](#javascript-miscellanious)

To solve problem of dependent async tasks, we create a class that will take the dependencies and a callback function as input.

In the dependencies list, we will check if all the tasks in the list are completed or not, if it is completed then filter them out as we no longer need to run them.

If there are not items in dependencies, invoke the callback directly.

Otherwise, if there are dependencies pending, push them into the list, and execute them one by one. Once all are completed, invoke the callback.

Use a flag to determine the state of the task, i.e completed or not.

```
class Task {
    // accept the dependencies list
    // and the callback
    constructor(dependencies = [],job){
        //filter the dependencies that are not yet completed
        this.dependencies = dependencies ? dependencies.filter(dependency => dependency instanceof Task && !dependency.isCompleted) : [];
        this.currentDependencyCount = this.dependencies.length;
        
        // the callback
        this.job = job;
        
        // if current task is done
        this.isCompleted = false;
        
        // store the dependencies list callback
        // to execute is sequence;
        this.subscribedList = [];
        
        //start the job
        this.processJob();
    }
    
    processJob() {
        // if there is dependency
        // subscribe to each of them
        if (this.dependencies && this.dependencies.length) {
            for (let dependency of this.dependencies){
                dependency.subscribe(this.trackDependency.bind(this));
            }
        }
        // else invoke the callback directly
        else {
            this.job(this.done.bind(this));
        }
    }
    
    // if all the dependencies are executed 
    // invoke the callback
    trackDependency(){
        this.currentDependencyCount--;
        if (this.currentDependencyCount === 0){
            this.job(this.done.bind(this));
        }
    }
    
    // push the callback to the list
    subscribe(cb) {
        this.subscribedList.push(cb);
    }
    
    // if the current task is done
    // mark it as complete
    // invoke all the dependency callbacks
    // to print it in sequence
    done() {
        this.isCompleted = true;
        for (const callback of this.subscribedList) {
            callback();
        }
    }
}
```
