## class

[![Build Status](https://travis-ci.org/bloodyowl/class.svg)](https://travis-ci.org/bloodyowl/class)

[![browser support](https://ci.testling.com/bloodyowl/class.png)](https://ci.testling.com/bloodyowl/class)

### Install

```
$ npm install bloody-class
```

### Require

```javascript
var klass = require("bloody-class")
```

### Definition

Classes are objects that contain inherits for instances.
You can extend and create instances of an existing class.
Classes are based on prototypal inheritance, that way, you can easily update
all subclasses and instances from one of their parent class.

### Methods

#### `klass.extend([object])` -> `newClass`

Creates a new class that inherits from `klass`. Optionaly takes an `object`
arguments that extends the `newClass` as owned properties.

##### `object`

- `object.mixins` : array

list of classes the given class should compose with.

#### `klass.create([args …])` -> `newInstance`

Creates a new instance that inherits from `klass`. Its arguments are passed to
`klass.constructor` which is called if `klass` owns a `constructor` method.

#### `instance.destroy([args …])`

Removes all the internal references to `instance`, as in `parent.instances` for
instance. Its arguments are passed to `klass.destructor` which is called if
`klass` owns a `destructor` method.

#### `instance.accessor(name) > function`

Returns a function that calls `instance[name]` with `instance` as `thisValue`
and passes its arguments to the method.

### Example

```javascript
var klass = require("bloody-class")
var $ = require("jquery")

var view = klass.extend({
  constructor : function(){
    this.element = $(this.element || document.createElement("div"))
    if(typeof this.name != "string") {
      throw new TypeError("name must be a string")
    }
    if(typeof this.initialise == "function") {
      this.initialise.apply(this, arguments)
    }
    this.initEvent()
  },
  $ : function(selector){
    return this.element.find(selector)
  },
  destructor : function(){
    this.element.off("." + this.name)
    if(typeof this.release == "function") {
      this.release.apply(this, arguments)
    }
  },
  initEvent : function(){
    if(!this.events) return
    var index = -1
    var length = this.events.length
    var thisValue = this
    while(++index < length) {
      ;(function(object){
        this.element.on(
          object.type + ("." + this.name),
          object.selector,
          function() {
            return thisValue[object.listener]
              .apply(thisValue, arguments)
          }
        )
      }).call(this, this.events[index])
    }
  }
})


var myView = view.create({
  name : "elementView",
  element : "#element",
  mixins : [events],
  events : [
    {
      type : "click",
      selector : ".js-anchor"
      listener : "navigateTo"
    }
  ],
  navigateTo : function(eventObject){
    window.scrollTo(0, $(eventObject.currentTarget).offset().top)
  }
})
```
