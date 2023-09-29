# Salesforce-Common-Things

# Apex

## Getting List of picklist values for a picklist field of a Object

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

```
// Populate map from SOQL query
Map<ID, Account> m = new Map<ID, Account>([SELECT Id, Name FROM Account LIMIT 10]);
// After populating the map, iterate through the map entries
for (ID idKey : m.keyset()) {
    Account a = m.get(idKey);
    System.debug(a);
}
```

# Lightning Web Components

## Row Selection in Lightning-Datatable Miscellanious Things
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
