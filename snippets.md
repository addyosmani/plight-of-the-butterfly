
## Angular data-binding

```
<!-- automatic persistence, save to disk/server -->
<!-- constraint-based forms, update dependent values -->
<html ng-app>
  <head>
    ...
    <script src="angular.js"></script>
    <script src="controller.js"></script>
  </head>
  <body ng-controller="PhoneListCtrl">
    <ul>
      <li ng-repeat="phone in phones">
        {{phone.name}}
        <p>{{phone.snippet}}</p>
      </li>
    </ul>
  </body>
</html>

<!--
  http://angular.github.io/angular-phonecat/step-2/app/
-->
```

## Angular data-binding and scopes

```
// check with the scoped item length has changed
// Expression executes more quickly

$scope.items = [];

for (var i = 0; i < 1000; i++) {
    $scope.items.push({prop1: 'val1', prop2: 'val2', prop3: 'val3'});
}

$scope.$watch('items.length', function() {

});
```

## Ember data-binding

```
// Container objects
MyApp.president = Ember.Object.create({
  name: "Barak Obama"
});
 
MyApp.country = Ember.Object.create({
  // ending a property with "Binding" tells Ember to
  // create a binding to the presidentName property
  presidentNameBinding: "MyApp.president.name"
});
 
// Later, after Ember has resolved bindings
MyApp.country.get("presidentName");
// "Barack Obama"
 
// Data from the server needs to be converted
// Composes poorly with existing code
```


## Backbone forced data-binding

```
var userModel = Backbone.Model.extend({
    defaults: {
        firstName: "John",
        lastName: "Smith"
    }
});

var userView = Backbone.View.extend({

    events: {
        "change input#firstName": "changeFirstNameHandler"
    },

    initialize: function (options) {
        this.firstNameInput = this.$("#firstName");
        // this.model.on("change:firstName", _.bind(this.onFirstNameChange, this));
        this.listenTo(this.model, 'change:firstName', _.bind(this.onFirstNameChange, this));
    },
    changeFirstNameHandler: function (event) {
        this.model.set("firstName", this.firstNameInput.val());
    },
    onFirstNameChange: function () {
        this.firstNameInput.val(this.model.get("firstName"));
    }

});

// Alternatives: ModelBinder (11KB), Backbone.stickit (3KB)
```

## Object.observe() - observe, mutate, change (3)

```
// A model can be a simple vanilla object
var todoModel = {
  label: 'Default',
  completed: false
};
 
// We then specify a callback for whenever mutations
// are made to the object
function observer(changes){
  changes.forEach(function(change, i){
    console.log(change);
    
    /*
      what property changed? change.name
      how did it change? change.type
      whats the current value? change.object[change.name]
    */
  })
}

// Which we then observe
Object.observe(todoModel, observer);

// Let's play!
todoModel.label = 'Buy some more milk';
 
/*
  label changed
  It was changed by being updated
  Its current value is 'Buy some more milk'
*/
 
todoModel.completeBy = '01/01/2014';
/*
 completeBy changed
 It was changed by being new
 Its current value is '01/01/2014'
*/
 
delete todoModel.completed;
 
/*
 completed changed
 It was changed by being deleted
 Its current value is undefined
*/

// Stop observing changes
Object.unobserve(todoModel, observer);
```

## What can be observed? Notifier.notify (4)

```
function Circle(radius) {

    Object.defineOwnProperty(this, 'radius', {
        get: function () {
            return radius;
        },
        set: function (newRadius) {
            if (radius == newRadius)
                return;

            var notify = Object.getNotifier(this);

            // notify(changeRecord)
            notifier.notify({
                type: 'updated', // deleted, new, reconfigured etc.
                // you can also just custom type (e.g 'foo')
                name: 'radius',
                oldValue: radius
            });

            radius: newRadius;
        }
    });
}

/*
notifier.notify({
    type: 'reconfigured', 
    name: 'radius',
    oldValue: 'circumference'
});
*/
```

## Object.observe() with acceptList (5)

### Thingy.js

```

function Thingy(a, b, c) {
  this.a = a;
  this.b = b;
}

Thingy.MULTIPLY = 'multiply';
Thingy.INCREMENT = 'increment';
Thingy.INCREMENT_AND_MULTIPLY = 'incrementAndMultiply';

var observer, observer2 = {
    records: undefined,
    callbackCount: 0,
    reset: function() {
      this.records = undefined;
      this.callbackCount = 0;
    },
};

observer.callback = function(r) {
    //assertEquals(undefined, this);
    //assertEquals('object', typeof r);
    //assertTrue(r instanceof Array)
    console.log(r);
    observer.records = r;
    observer.callbackCount++;
};


Thingy.prototype = {
  increment: function(amount) {
    var notifier = Object.getNotifier(this);

    // Tell the system that a collection of work
    // compromises a given changeType
    // e.g
    // notifier.performChange('foo', 'performFooChangeFn');
    // notifier.notify('foo', 'fooChangeRecord');
    notifier.performChange(Thingy.INCREMENT, function() {
      this.a += amount;
      this.b += amount;
    }, this);

    notifier.notify({
      object: this,
      type: Thingy.INCREMENT,
      incremented: amount
    });
  },

  multiply: function(amount) {
    var notifier = Object.getNotifier(this);

    notifier.performChange(Thingy.MULTIPLY, function() {
      this.a *= amount;
      this.b *= amount;
    }, this);

    notifier.notify({
      object: this,
      type: Thingy.MULTIPLY,
      multiplied: amount
    });
  },

  incrementAndMultiply: function(incAmount, multAmount) {
    var notifier = Object.getNotifier(this);

    notifier.performChange(Thingy.INCREMENT_AND_MULTIPLY, function() {
      this.increment(incAmount);
      this.multiply(multAmount);
    }, this);

    notifier.notify({
      object: this,
      type: Thingy.INCREMENT_AND_MULTIPLY,
      incremented: incAmount,
      multiplied: multAmount
    });
  }
}



observer2.callback = function(r){
	console.log('Observer 2', r);
}

Thingy.observe = function(thingy, callback) {
  // Object.observe(obj, callback, opt_acceptList)
  Object.observe(thingy, callback, [Thingy.INCREMENT,
                                    Thingy.MULTIPLY,
                                    Thingy.INCREMENT_AND_MULTIPLY,
                                    'updated']);
}

Thingy.unobserve = function(thingy, callback) {
  Object.unobserve(thingy);
}

var thingy = new Thingy(2, 4);

// breaking here..

Object.observe(thingy, observer.callback); // just observer?
Thingy.observe(thingy, observer2.callback);
thingy.increment(3);               // { a: 5, b: 7 }
thingy.b++;                        // { a: 5, b: 8 }
thingy.multiply(2);                // { a: 10, b: 16 }
thingy.a++;                        // { a: 11, b: 16 }
thingy.incrementAndMultiply(2, 2); // { a: 26, b: 36 }
```

### acceptList.js

```
console.clear();

// A model can be a simple vanilla object
var todoModel = {
  label: 'Default',
  completed: false
};
 
// We then specify a callback for whenever mutations
// are made to the object
function observer(changes){
  changes.forEach(function(change, i){
    console.log(change);
  })
};

// Which we then observe
// Note the third argument
Object.observe(todoModel, observer, ['deleted']);
// without this third option, defaults to intrinsic types

todoModel.label = 'Buy some milk'; // note that no changes were reported

// delete todoModel.label; 
// this change was reported
```

## Default to intrinsic change types (6)

```
console.clear();

// A model can be a simple vanilla object
var todoModel = {
  label: 'Default',
  completed: false
};

Object.observe(todoModel, function(changeRecords){
  changeRecords.forEach(function(change){
    console.log({
      changeType:           change.type, 
      affectedObject:       change.object, 
      affectedPropertyName: change.name, 
      valueBeforeChange:    change.oldValue 
    });
  })
});

todoModel.label = 'Buy some bread';
delete todoModel.label;

/*
Object.observe(todoModel, function(changeRecords){
  changeRecords.forEach(function(change){

    console.log(
      
    change.type, // new, updated, deleted or reconfigured
    change.object, // affected JS object
    change.name, // affected property name
    change.oldValue // value of the property before the change

    );
  })
});
*/

## Notifier.notify - computed properties (8)

```
var model = {
      a: {}
    }

var _b = 2;

Object.defineProperty(model.a, 'b', {
  get: function() { return _b; },
  set: function(b) {

    Object.getNotifier(this).notify({
      type: 'updated',
      name: 'b',
      oldValue: _b
    });

    console.log('set', b);

    _b = b;
  }
});

function observer(changes){
  changes.forEach(function(change, i){
    console.log(change);
  })
}

Object.observe(model.a, observer);

model.a.b = 4; // will be observed.
```

## Notifier.performChange (large-scale changes)

```

function Thingy(a, b, c) {
  this.a = a;
  this.b = b;
}

Thingy.MULTIPLY = 'multiply';
Thingy.INCREMENT = 'increment';
Thingy.INCREMENT_AND_MULTIPLY = 'incrementAndMultiply';

var observer, observer2 = {
    records: undefined,
    callbackCount: 0,
    reset: function() {
      this.records = undefined;
      this.callbackCount = 0;
    },
};

observer.callback = function(r) {
    console.log(r);
    observer.records = r;
    observer.callbackCount++;
};


Thingy.prototype = {
  increment: function(amount) {
    var notifier = Object.getNotifier(this);

    // Tell the system that a collection of work
    // compromises a given changeType
    // e.g
    // notifier.performChange('foo', 'performFooChangeFn');
    // notifier.notify('foo', 'fooChangeRecord');
    notifier.performChange(Thingy.INCREMENT, function() {
      this.a += amount;
      this.b += amount;
    }, this);

    notifier.notify({
      object: this,
      type: Thingy.INCREMENT,
      incremented: amount
    });
  },

  multiply: function(amount) {
    var notifier = Object.getNotifier(this);

    notifier.performChange(Thingy.MULTIPLY, function() {
      this.a *= amount;
      this.b *= amount;
    }, this);

    notifier.notify({
      object: this,
      type: Thingy.MULTIPLY,
      multiplied: amount
    });
  },

  incrementAndMultiply: function(incAmount, multAmount) {
    var notifier = Object.getNotifier(this);

    notifier.performChange(Thingy.INCREMENT_AND_MULTIPLY, function() {
      this.increment(incAmount);
      this.multiply(multAmount);
    }, this);

    notifier.notify({
      object: this,
      type: Thingy.INCREMENT_AND_MULTIPLY,
      incremented: incAmount,
      multiplied: multAmount
    });
  }
}


observer2.callback = function(r){
	console.log('Observer 2', r);
}


Thingy.observe = function(thingy, callback) {
  // Object.observe(obj, callback, opt_acceptList)
  Object.observe(thingy, callback, [Thingy.INCREMENT,
                                    Thingy.MULTIPLY,
                                    Thingy.INCREMENT_AND_MULTIPLY,
                                    'updated']);
}

Thingy.unobserve = function(thingy, callback) {
  Object.unobserve(thingy);
}

var thingy = new Thingy(2, 4);

// breaking here..

Object.observe(thingy, observer.callback); 
Thingy.observe(thingy, observer2.callback);
thingy.increment(3);               // { a: 5, b: 7 }
thingy.b++;                        // { a: 5, b: 8 }
thingy.multiply(2);                // { a: 10, b: 16 }
thingy.a++;                        // { a: 11, b: 16 }
thingy.incrementAndMultiply(2, 2); // { a: 26, b: 36 }
```