# angular-360-no-scope

## What?
### `$watch` your controllerAs controller's data, without injecting $scope.

## How do I use it?

- `[npm|bower] install angular-360-no-scope`
- Include angular-360-no-scope.js in your app
- Add a dependency on `angular-360-no-scope` to your app module.
- Write your controller as usual, but avoid `$scope`
- Utilize `this.$watch()` as needed
 
## Live Demo

http://plnkr.co/edit/WzJCeB4CiEpkFyF81Zez?p=preview

## Why?

### Preface
When using angular's controllerAs, a controller is given a name and a reference to the controller is placed on the $scope.  When writing the logic for the controller, data is typically stored directly on the controller itself, not on the $scope.  When referencing the data from a template, the data is namespaced by the controllerAs name.  

This provides various benefits, which help us write cleaner, more maintainable code.  See the style guides by [Todd Motto](https://github.com/toddmotto/angularjs-styleguide#controllers) and [John Papa](https://github.com/johnpapa/angularjs-styleguide#controllers) for more details.

### Still need $scope capabilities
One oddity of writing ControllerAs code is that we no longer tend to have the `$scope` reference handy.  Besides being a tempting dumping ground for data, `$scope` also provides some important funtionality that we occasionally need such as `$watch` (and `$on`, `$broadcast`, and `$emit`).  

A simple mechanism to provide those functions is to inject `$scope` into the controller function.  Then, in order to watch your controller data, you may do something like so: 
```javascript
$scope.$watch(function() { return ctrl.someData }, callback)
```
This is a little clunkier than watching scope data, i.e., `$scope.$watch("some.scope.variable", callback)`.  Additionally, if you want to watch nested attributes whose parents may or may not be initialized, we might need to add yet another dependency on `$parse` so we must do something like:
```javascript
$scope.$watch(function() { return $parse("some.controller.variable")(ctrl); }, callback);
```

## 360-no-scope makes this easier

With **360-no-scope**, your controllers are decorated, and augmented with a `$watch` function.  The `$watch` function is bound to the controller instance.  This allows you to write `ctrl.$watch("some.controller.variable", callback)`, much like the simple `$scope.$watch` you are already familiar with.

### Sample Controller

```javascript
app.controller("MyController", function () { // HERE, no $scope is necessary
  var ctrl = this;
  ctrl.watchCount = 0;
  ctrl.foo = {};
  
  // Here is the "scope-less" watch registration.  Watching "ctrl.foo.bar.baz"
  ctrl.$watch("foo.bar.baz", callback);  // &lt;-- HERE, $watch something on the controller
  function callback (newVal, oldVal) { console.log("WatchCount: " + ctrl.watchCount++, newVal, oldVal); }
}
```

## How does it work?

This lib decorates `$controllerProvider.register` and the `$controller` service.  When a controller is registered with `$controllerProvider`, or when a controller is instantiated with the `$controller()` service, the controller fn passed in is augmented with a `$watch` function (as well as with `$on`, `$broadcast`, and `$emit`).

**360-no-scope** augments the controller fn by wrapping it in a surrogate controller which is executed instead.  The surrogate is annotated with the same injectable dependencies as the real controller fn.  Then, $scope is added to the dependency list.  When angular instantiates the controller surrogate, the surrogate always gets `$scope`.  It then builds the `$scope` passthrough functions and adds them to the real controller's prototype.  Finally, it instantiates and returns the real controller.


