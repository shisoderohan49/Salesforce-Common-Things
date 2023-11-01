# Salesforce-Common-Things

- [Apex](#apex)
- [Lightning Web Components](#lightning-web-components)
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

# Miscellanious
[Back to main](#salesforce-common-things)

<details>
  <summary>List of Contents</summary>
  
  - [Generate Salesforce Authentication Token Using Postman](#generate-salesforce-authentication-token-using-postman)
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

# Javascript Miscellanious
[Back to main](#salesforce-common-things)

<details>
  <summary>List of Contents</summary>
  
  - [Extract only selected keys from array of objects](#extract-only-selected-keys-from-array-of-objects)
  - [Generate Array for sequence of numbers](#generate-array-for-sequence-of-numbers)
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
