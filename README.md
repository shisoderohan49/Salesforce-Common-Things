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
