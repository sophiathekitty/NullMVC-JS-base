# NullMVC-JS
 
a library i'm making to handle the web app interface for [NullHub](http://github.com/sophiathekitty/NullHub). the model handles loading data from the server and stores a copy in local storage. it also handles saving local changes in local storage as well as pushing those changes back to the server. views hand building the interface from the templates and displaying the data from it's model. controllers handle interaction events for their view. it also uses a php script to combine all the files into one so it's easier for me to work on and lets me include all of them with a single tag in the html files.

## Model

handles loading data from the server and stores a copy in local storage. can also store local changes and then push local changes to the server.

mainly just need to setup the model's name and api url (and save url)

```js
class ClockModel extends Model {
    constructor(){
        super("clock","/api/clock","/api/clock");
    }
}
```

## Collection

adds list functions to Model.

```js
class ExtensionsCollection extends Collection {
    constructor(){
        super("extensions","extension","/api/info/extensions/","/api/info/extensions/")
    }
}
```

## Template

a Model that loads in an html template to be used with view

## View

builds the interface elements using it's template and item_template. displays data from it's model or collection.

```js
class ClockView extends View {
    constructor(debug = false){
        if(debug) console.log("ClockView::Constructor");
        super(new ClockModel(),new Template("clock","/templates/stamps/clock.html"),null,60000,debug);
    }
    build(){
        console.log("ClockView::Build");
        if(this.template){
            this.template.getData(html=>{
                if(this.debug) console.log("ClockView::Build",html);
                $("#clock_stamp").html("");
                $(html).appendTo("#clock_stamp");
                $(html).appendTo(".app main");
                $(".clock").attr("show","sunrise");
                this.display();
                if(this.interval) clearInterval(this.interval);
                this.interval = setInterval(this.refresh.bind(this),1000);
            });
        }
    }
    display(){
        if(this.debug) console.log("ClockView::Display");
        if(this.model){
            this.model.getData(json=>{
                //console.log("ClockView::Display",json);
                // apply the values
                $(".clock").attr("moon_visible",json.moon_out);
                $(".clock [var=sunrise]").html(this.Time24to12(json.sunrise));
                $(".clock [var=sunrise]").attr("unit",this.Time24toAM(json.sunrise));
                // etc...
            });
        }
    }
    refresh(){
        if(this.debug) console.log("ClockView::Refresh");
        this.display();
    }
}
```

## Controller

handles interaction events for it's view.

```js
class ClockController extends Controller {
    constructor(debug = false){
        if(debug) console.log("ClockController::Constructor");
        super(new ClockView(),debug);
        this.first_ready = true;
    }
    ready(){
        if(this.first_ready){
            if(this.debug) console.log("ClockController::Ready");
            this.view.build();
            if(this.interval) clearTimeout(this.interval);
            this.interval = setTimeout(this.refresh.bind(this),60000);
            this.click(".clock",e=>{
                // do slideshow stuff
                if(this.interval) clearTimeout(this.interval);
                this.interval = setTimeout(this.refresh.bind(this),240000);    
            });
            this.first_ready = false;
        }
    }
    refresh(){
        // do refresh stuff
        if(this.interval) clearTimeout(this.interval);
        this.interval = setTimeout(this.refresh.bind(this),120000);
    }
}
```
