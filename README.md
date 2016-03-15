# Sharing Directive Controllers

## Overview

Directives become even more powerful because we can communicate between their controllers. For instance, if we wanted to create a tabs component for navigation. We'd want to have our main `tabs` directive, and then directives for every tab. We can then link these together in order to create our tab picker.

## Objectives

- Describe the `require` property
- Create another Directive
- Create another Controller
- Bind Controller to Directive
- Import another Controller's instance

## require

We can require the parent controller for a certain directive by using the `require` property in our directive. We pass in a string, with the directives name that we want.

Following on from our example above, we might have the following setup:

```html
<tabs>
  <tab label="Tab 1">
    Tab 1 contents!
   </tab>
   <tab label="Tab 2">
    Tab 2 contents!
   </tab>
   <tab label="Tab 3">
    Tab 3 contents!
   </tab>
</tabs>
```

If we ran this now, our `tabs` component wouldn't know about the child tabs, and there is no list of tabs to choose from when we want to change what tab is active. This is where require comes in! Require allows us to notify the parent that it exists, so we can have a list of tabs at the top to change the active tab.

Imagine our `tabs` directive looks like this:

```js
function tabs() {
  return {
    restrict: 'E',
    scope: {},
    transclude: true,
    controller: function () {
      this.tabs = [];
    },
    controllerAs: 'tabs',
    template: [
      '<div class="tabs">',
        '<ul class="tabs__list"></ul>',
        '<div class="tabs__content" ng-transclude></div>',
      '</div>'
    ].join('')
  };
}

angular
  .module('app', [])
  .directive('tabs', tabs);
```

You might notice a property we haven't mentioned before - `transclude`. Don't worry about this yet, we're going to learn this very shortly!

In each tab, we've got a label for the tab. We're going to want to put this inside our `tabs__list` list. However, inside the `tabs` component we don't actually know what tabs we have inside the element.

One easy way to populate this list is if each of our `tab` directives notice our `tabs` directive that they exist. In our example, we've got three `tab` elements, so each of these will notify the parent controller.

Imagine our `tab` directive looked like this:

```js
function tab() {
  return {
    restrict: 'E',
    scope: {
      label: '@'
    },
    require: '^tabs',
    transclude: true,
    template: `
      <div class="tabs__content" ng-if="tab.selected">
        <div ng-transclude></div>
      </div>
    `,
    link: function ($scope, $element, $attrs) {

    }
  };
}

angular
  .module('app', [])
  .directive('tab', tab)
  .directive('tabs', tabs);
```

You'll notice we've added `require` with the value `^tabs` - this is telling Angular to require the parent controller from our `tabs` component.

Here, we have a simple sort-of "transparent" directive (our content is passed into our directive, not replaced). What this does (and again, more on the whole tranclusion concept later) is wrap our contents in a new `<div />`. For instance:

```html
<tab label="Tab 3">
	Tab 3 contents!
</tab>
```

Will become:

```html
<tab label="Tab 3">
	<div class="tabs__content" ng-if="tab.selected">
		<div ng-transclude>
			Tab 3 contents!
		</div>
	</div>
</tab>
```

You'll notice we've also got a property named `link`. We're going to learn more about this later, but for now, assume that it is magic and we're going to be using it in this example.

Normally, our link function has three parameters - scope (`$scope`), element (the mounted DOM element) and attrs (the attributes passed through to the directive). However, when we use `require`, we get a fourth - ctrl. This will be equal to the parents controller, allowing us to access everything to do with it.

Let's add an `addTab` method to our `tabs` directives controller. This will add a tab to the list so we can repeat and display the tabs labels at the top of the directive, so the user can click on them to change the active tab.

```js
function tabs() {
  return {
    restrict: 'E',
    scope: {},
    transclude: true,
    controller: function () {
      this.tabs = [];

      this.addTab = function (tab) {
        this.tabs.push(tab);
      };
    },
    controllerAs: 'tabs',
    template: [
      '<div class="tabs">',
        '<ul class="tabs__list"></ul>',
        '<div class="tabs__content" ng-transclude></div>',
      '</div>'
    ].join('')
  };
}

angular
  .module('app', [])
  .directive('tabs', tabs);
```

Now, if we access the fourth argument in our link function inside our `tab` directive:

```js
function tab() {
  return {
    restrict: 'E',
    scope: {
      label: '@'
    },
    require: '^tabs',
    transclude: true,
    template: [
      '<div class="tabs__content" ng-if="tab.selected">',
        '<div ng-transclude></div>',
      '</div>'
    ].join(''),
    link: function ($scope, $element, $attrs, $ctrl) {
        // $ctrl = { tabs: [], addTab: func(){} }
    }
  };
}

angular
  .module('app', [])
  .directive('tab', tab)
  .directive('tabs', tabs);
```

You'll see that we can now access the parent's controller and methods! From here, we can add the tab information in to our parent's tab array.
