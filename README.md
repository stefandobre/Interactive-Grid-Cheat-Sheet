# Oracle APEX Interactive Grid Cheat Sheet
This Cheat Sheet is a collection of code snippets and blog posts/ forum entries related to the APEX Interactive Grid.

##### JS Docs for 18.2
https://docs.oracle.com/database/apex-18.2/AEXJS/interactiveGrid.html

##### JS Docs for 19.1
https://apex.oracle.com/pls/apex/apex_pm/r/190019/files/static/v160/jsdoc/index.html

### Get Widget
this is the base object for all operations
``` javascript
apex.region('emp')
```
### Methods
calling the methods from the Interactive Grid API
``` javascript
apex.region('emp').call('refresh')
apex.region('emp').call('getCurrentView')
//and so on
```

### Get Record
``` javascript
var grid    = apex.region('emp').call('getViews','grid'); 
var model   = grid.model; 
var record  = model.getRecord(vRecordId);
```

### Get Grid Options
``` javascript
apex.region("emp").call("option","config")
```

### Get Model Options
``` javascript
apex.region("emp").call("getViews").grid.model.getOption("fields");
```
For possible options look at model.js defaultOptions object

### Get Selected Records
From the model:
``` javascript
apex.region("emp").call("getViews","grid").view$.grid("getSelectedRecords")  
```

Selected DOM elements
``` javascript
apex.region("emp").call("getViews","grid").view$.grid("getSelection")  
```

### Check Edit Mode
To check if IG is in edit mode use this:
``` javascript
apex.region("emp").call("getActions").get("edit")
``` 

### Set Column Value
``` javascript
var grid        = apex.region('emp').call('getViews','grid');  
var model       = grid.model; 
var record      = model.getRecord(vRecordId);
model.setValue(record,'ENAME', vEname);
```

### Initial Grid Configuration

``` javascript
function(config) {

    if (!config.toolbar) {
        config.toolbar = {};
    }
    
    config.toolbar.searchField = false; // hide toolbar search field 

    config.toolbar.actionMenu = false; // hide toolbar action menu

    // disable column reorder
    config.views.grid.features.reorderColumns = false;

    // enable selection persistance across different pages
    config.views.grid.features.persistSelection = true;

    // hide grid footer
    config.views.grid.features.footer = false;

    // row selector properties. note that declarative options will override these
    config.views.grid.features.multiple = true;	// multiple selection
    config.views.grid.features.selectAll = true; // selectAll button

    //display rownumbers instead of selector
    config.views.grid.features.rowHeader = 'sequence';

    // turn off auto add row feature
    config.editable.autoAddRow = false;

    // remove certain actions. see list here
    config.initActions = function( actions ) {
        actions.remove("row-duplicate");
    };
    
    // always return the config object
    return config;
}
```

### Initial Grid Column Configuration

``` javascript
function(config) {
    // create 'features' object if it does not exist
    config.features = config.features || {};
    config.features.sort = false;
    config.features.aggregate = false;

    config.defaultGridViewOptions = {
        resizeColumns: false,
        noHeaderActivate: true //The noHeaderActivate option still allows resize, reordering and sorting of columns.
    }

    return config;
}
```


### Refresh Selected Rows (for Editable IG)
``` javascript
apex.region("emp").call("getActions").invoke("selection-refresh")
```
For non-editable IG look at
http://roelhartman.blogspot.hr/2017/07/refresh-selected-rows-in-interactive.html

### Refresh IG on tab activation
1) Add static ID (for example tabs) to the tabs region.
2) Add to page attribute execute when page loads:
``` javascript
$("#tabs").on("atabsactivate", function(event, ui) {
    if (ui.showing) {
        ui.active.panel$.find(".a-GV").grid("refresh");
    }
});
```

### Refresh IG on Region Display Selector activation
``` javascript
$('.apex-rds').data('onRegionChange', function(mode, activeTab) {  
  if (activeTab.href != "#SHOW_ALL"){  
    apex.region(activeTab.href.replace("#","")).refresh();  
  }  
});
```


### Actions
To list actions call:
``` javascript
apex.region("emp").call("getActions").list().forEach(function(a) { console.log("Action Label: " + a.label + ", Name: " + a.name + (a.choice !== undefined ? ", Choice: " + a.choice : "") ); });
```

To call action use:
apex.region("emp").call("getActions").invoke("show-sort-dialog");

### Adding Row Actions
``` javascript
apex.region("emp")
    .call("getViews").grid
    .rowActionMenu$.menu("option")
    .items.push({type:"action", label:"Hi", action: function(){alert("Hi")}});
```

More on [John's blog](http://hardlikesoftware.com/weblog/2017/01/24/how-to-hack-apex-interactive-grid-part-2/)

To add custom row action to specific position in row action menu use this JS code (only change second if statement) and add it to the Function and Global Variable Declaration page property:
``` javascript
$(function() {
  $("#emp").on("interactivegridviewchange", function(event, data) {
    if ( data.view === "grid" && data.created ) {
      var view$ = apex.region("emp").call("getViews", "grid");
      if (view$.rowActionMenu$){
        var menu$ = view$.rowActionMenu$.menu("option").items;          
        for (i = 0; i < menu$.length; i++ ) {
          if (menu$[i].action === 'row-duplicate'){
            menu$.splice(i+1
                       , 0
                       , {
                          type:"action",
                          label:"After Copy Action",
                          icon: "fa fa-user",
                          action: function(menu, element) {
                            var record = view$.getContextRecord( element )[0];
                            alert('After copy action: '+view$.model.getValue(record, "EMPNO"));
                          }
                         })
            break;
          }
        }
      }  
    }        
  });
});  
```

Demo is available [here](https://apex.oracle.com/pls/apex/f?p=100309:41) 

To list all action names see Actions paragraph above.
More on [OTN](https://community.oracle.com/message/14320776#14320776)


### Cancel changes and refresh grid
``` javascript
var view = apex.region("emp").call("getCurrentView");

if ( view.internalIdentifier === "grid" ) { // only grid supports editing
    view.model.clearChanges();
}
apex.region("emp").refresh();
```

### Focus IG
``` javascript
apex.region("regionStaticID").focus();
```


## Interactive Grid Events

* interactivegridviewmodelcreate

  explanation and example here

* interactivegridviewchange
* interactivegridcreate
* interactivegridreportsettingschange
* interactivegridselectionchange
* interactivegridsave

  Fires after the save event of the IG has finished. Similar to the "afterrefresh" event of an Interactive Report. You can use this as a Custom Event in a Dynamic Action.

## Blog Posts

### John Snyders (Oracle APEX dev team)

[Interactive Grid: Under the Hood](http://hardlikesoftware.com/weblog/2016/06/08/interactive-grid-under-the-hood/)

[Interactive Grid column widths](http://hardlikesoftware.com/weblog/2017/01/06/interactive-grid-column-widths/)

[How to hack APEX Interactive Grid Part 1](http://hardlikesoftware.com/weblog/2017/01/18/how-to-hack-apex-interactive-grid-part-1/)

[How to hack APEX Interactive Grid Part 2](http://hardlikesoftware.com/weblog/2017/01/24/how-to-hack-apex-interactive-grid-part-2/)

[How to hack APEX Interactive Grid Part 3](http://hardlikesoftware.com/weblog/2017/02/20/how-to-hack-apex-interactive-grid-part-3/)

[APEX Interactive Grid API Improvements in 5.1.1](http://hardlikesoftware.com/weblog/2017/03/28/apex-interactive-grid-api-improvements-in-5-1-1/)

[How to hack APEX Interactive Grid Part 4](http://hardlikesoftware.com/weblog/2017/03/31/how-to-hack-apex-interactive-grid-part-4/)

[APEX Client-Side Validation](http://hardlikesoftware.com/weblog/2017/05/10/apex-client-side-validation/)

[APEX Interactive Grid Cookbook](http://hardlikesoftware.com/weblog/2017/07/10/apex-interactive-grid-cookbook/)

[Some minor new things in APEX 18.2](http://hardlikesoftware.com/weblog/2018/09/28/some-minor-new-things-in-apex-18-2/)

## Other Links
[Interactive Grid: Download as PDF with jsPDF by MENNO HOOGENDIJK](https://t.co/RtXQgxbQSg)

[Pimp my Grid App - IG Sample App](https://apex.oracle.com/pls/apex/f?p=pimpmygrid)

[How to Fix Blank Toolbar DIV](https://apexbyg.blogspot.com/2017/12/interactive-grid-how-to-fix-blank.html)

## OTN
### Hidden and display columns
[How to set value of a hidden column](https://community.oracle.com/thread/4035826)

[How to set value of a display column](https://community.oracle.com/thread/4047278)

### Processing
[Process Selected Rows](https://community.oracle.com/message/14204014)

### Disabled Cells
[Disable Column Actions](https://community.oracle.com/thread/4030370)
[How to validate disable cell in interactive grid?](https://community.oracle.com/thread/4018344)

### Readonly Cells
[Making Columns Readonly](http://lschilde.blogspot.hr/2017/03/apex-51-interactive-grid-making-columns.html?m=1)

### Disable Column Resize, Sort, Reorder Columns
The noHeaderActivate option still allows resize, reordering and sorting columns. In 5.1.1 you can disable those features with the Advanced JavaScript Code the config options are
config.views.grid.features: resizeColumns, reorderColumns, sort

### Interactive Report vs Interactive Grid
https://community.oracle.com/thread/4049804

### Horizontal Scroll
https://community.oracle.com/thread/4050640
https://community.oracle.com/thread/3812273

### Interactive grid color based on a value
https://community.oracle.com/thread/4047292

### Grid Selection (copy to clipboard)
https://community.oracle.com/thread/4051443

### Highlight Row/Cell
https://community.oracle.com/thread/4036606

### Set Selected Rows After Refresh
https://community.oracle.com/thread/4060416

### More About Selecting Grid Rows
https://community.oracle.com/thread/4030402

### Interactive Grid with more than one table
https://community.oracle.com/message/14473170#14473170

### How to validate IG column values on page load
https://community.oracle.com/thread/4068205

## How To's

- [Pagination How To's](ig_pagination.md)
