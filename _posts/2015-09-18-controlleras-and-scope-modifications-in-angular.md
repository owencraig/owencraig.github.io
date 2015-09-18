---
layout: post
title: "controllerAs and scope modifications in AngularJs"
comments: true
---

I was having a conversation with a client recently about how to get dirty checking for a form in angular. this then turned into a conversation about binding from views and how it's affected by controllerAs.

## The issue

Due to the way that [ng-form](https://docs.angularjs.org/api/ng/directive/ngForm) works, we're able to check on the form for ``` form.$dirty ``` and ``` form.$invalid ```. After getting it set up they were able to control the state of some controls (a save button) like so:

``` html
<form name="theFormInQuestion">
    <button class="save" ng-disabled="theFormInQuestion.$invalid || !theFormInQuestion.$dirty" ng-click="save()">Save</button>
    <!-- form inputs -->
</form>
```

Now since we're using the controllerAs syntax, we've got the controller named as vm. The issue they were asking me was that after their save executed, they weren't able to get a handle of the form object to run [``` $setPristine() ```](https://docs.angularjs.org/api/ng/type/form.FormController#$setPristine) so that the save button was once again disabled.

``` js
    angular.module('someApp').controller('someController', ['someService',function(someService){
        var vm = this;
        vm.save = save;

        function save(){
            someService.saveSomeData(vm.someBoundObject)
                .then(resetForm);
        }

        function resetForm(valueData){
            // do some setup/rollback with the data
            vm.theFormInQuestion.$setPristine();
        }
    }]);
```

The issue was that ``` vm.theFormInQuestion ``` was undefined, so trying to call ``` vm.theFormInQuestion.$setPristine() ``` was throwing a TypeError. The way to fix this is to simply change our bindings in our view to escape the form with the name from the controllerAs declaration (vm).

``` html
<form name="vm.theFormInQuestion">
    <button class="save" ng-disabled="vm.theFormInQuestion.$invalid || !vm.theFormInQuestion.$dirty" ng-click="vm.save()">Save</button>
    <!-- form inputs -->
</form>
```
After doing this we've got our button disabling as we expected, but we're now also able to reference the form from within our controller, excellent! Job done was can all go home.

## Oooh! Magic!

I'm not overly a fan of things just magically working, so maybe we can't go home straight away. The reason behind the a) buttons being disabled correctly in the first place, b) us not being able to aeccss the form in our controller originally and c) why prefixing the name made everything magically work is based on how controllerAs interacts with the scope we get passed into our views.

### Before controllerAs, injecting $scope

Prior to Angular 1.2 (or after following one of the plethora of online tutorials that use it) the only way to provide data to our view was to inject the $scope object, then use those bindings in our view:

``` js
    angular.module('someApp').controller('scopeController', ['$scope', '$window',function($scope, $window){
        var _nameOutputPrompt = 'Hi there, ';

        $scope.prompt = 'Hello, please enter your name';

        $scope.alertName = function(){
            $window.alert(_nameOutputPrompt + $scope.name);
        }
    }]);
```

``` html
    <div ng-controller="scopeController">
        <label for="nameInput">{{prompt}}</label><input type="text" ng-model="name" />

        <button ng-click="alertName()">Show it!</button>
    </div>
```
(plunkr [here](http://plnkr.co/edit/nSkknyibKF8xJfACKamJ))

What happens for this to work is:

* Angular creates a new scope object that inherits from its parent scope
* In our controller we inject this scope object
* We bind any fields we need onto the scope object
* That scope object is passed to the view for rendering

Doing this binds the property ``` prompt ``` and the function ``` alertName ``` on to the the scope data object that the view sees. What we also get is the ``` name ``` property being bound back on to the same object so we can access it from the controller. This gives us the scope object (ignoring any angular properties and just looking at what we've bound on):

``` js
{
    // a bunch of angular stuff
    "prompt": "Hello, please enter your name",
    "name": undefined, // or whatever we've entered in our view
    "alertName": // Function Expression
}
```

### controllerAs, no $scope in sight

With the controllerAs syntax, our controller and view rom above would become :

``` js
    angular.module('someApp').controller('controllerAsController', ['$window', function($window){
        var self = this;
        var _nameOutputPrompt = 'Hi there, ';

        self.prompt = 'Hello, please enter your name';
        self.alertName = alertName;
      
        function alertName(){
            $window.alert(_nameOutputPrompt + self.name);
        }
    }]);
```

``` html
    <div ng-controller="controllerAsController as vm">
        <label for="nameInput">{{vm.prompt}}</label><input type="text" ng-model="vm.name" />

        <button ng-click="vm.alertName()">Show it!</button>
    </div>
```
(plunkr [here](http://plnkr.co/edit/CWaaCznS5LicfZKgzlzP))

What this looks like we're doing is creating a new object and passing that in as the ``` vm ``` in this case so we don't have the same child scope as we did previously. What is in fact happening is a similar process to the one described above, but with some extra sugar:

* Angular creates a new scope object that inherits from its parent scope
* Our controller is called as a constructor function, binding anything required onto itself
* The object returned by instatiating the controller gets bound onto the scope on the vm parameter (N.B. The parameter name here is driven by what we put in the controller as)
* The scope object is passed to the view for rendering

So although it looks like we're dealing only with our own object, we're adding on to the scope:

``` js
{
    // a bunch of angular stuff
    vm: {
        "prompt": "Hello, please enter your name",
        "name": undefined, // or whatever we've entered in our view
        "alertName": // Function Expression
    }
}
```

### Okay, so what...

Although this looks like a mostly intellectual difference, it has some pretty big impacts..

Because scopes within the view heirarchy are created with their parents as their prototype if we use the standard $scope way of doing things, then each time we bind something onto our current scope _all_ children will inherit that property. This leads to some instances where you're expecting to bind vertically back to the parent but instead of setting its property, you've got a new shadowed version of it on the child.

The opposite of this is that with using controllerAs you avoid binding instance properties on the scope to get inherited  within any controllers backing any children. Since you're writing each controller as a constructor function they only have access to their prototype, which will be separate of the prototype we were talking about earlier. 

If however you did need to let some properties inherit down the chain, you're able to mix and match between the two styles (not that I would recommend it).

## How this affects our problem

In our initial example the binding of the form was on to ``` theFormInQuestion ```, which gets resolved as binding on to the scope directly. Because it was bound on here and the view ng-disabled etc were also bound, they were able to reference each other.

Since we're using controller as though, within the controller the context we have is the vm property on the scope so we're not able to reference the form directly. By changing all references to point to ``` vm.theFormInQuestion ``` we were acting on the correct object across the board.