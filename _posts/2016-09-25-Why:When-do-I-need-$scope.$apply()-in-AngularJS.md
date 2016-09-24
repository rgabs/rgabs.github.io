---
layout: post
title:  "Why-When do I need $scope.$apply() in AngularJS"
date:  2016-09-25 1:15:00
categories: front-end
---

### Intro
When we start building complex/production applications in AngularJS we all have come across this scenario where we require to manually trigger the [Angular Digest Cycle](https://www.sitepoint.com/understanding-angulars-apply-digest/) using $scope.$apply(). In this blog, I will be sharing my learnings.

### Why do I need $scope.$apply()
AngularJS uses a clever trick of re-rendering the entire view if any of the scope properties is changed. So it is really important for Angular to know when the scope is changed so that it can update the DOM automatically. This is one the reasons why angular has wrappers for native JS methods like `setTimeout`, `setInterval`, etc. By using these methods, Angular internally takes care of updating the DOM internally. But what if the API/Library that you are using is not available as an Angular wrapper (the new native Promise API in my case). Well, in that case, you need to trigger the DOM update yourself using $scope.$apply().

### Example
I've created a fiddle for this example comparing `$http` service by angular with the new ES6 native Promise API.
*Note that you will need to enable CORS in your browser to see the result.*

<iframe width="100%" height="300" src="//jsfiddle.net/rgabs/paxf1Lmn/2/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

```javascript
var myApp = angular.module('myApp',[]);

myApp.factory('myService', function ($http){
	return {
  	getData: ()=>{
	    return $http.get('/')
    },
    getDataFromPromise: ()=>{
    	return new Promise((resolve,reject)=>{
           resolve($http.get('/'))
      })
    }
  }
})
function MyCtrl($scope, myService) {
    $scope.name = 'nananananana';
    myService.getData().then((res)=>{
      $scope.name ='Batman'
      myService.getDataFromPromise().then((res)=>{
        $scope.name ='Bruce Wayne (updated using $apply)' //The view will not update
        $scope.$apply();
      })
    })
}
```

### Important things to take care when using $scope.$apply()
* Make sure that the there is no digest cycle is running when you manually trigger it or else you might get an error saying `Error: $digest already in progress`. Refer to this blog for solving this issue (http://www.bennadel.com/blog/2605-scope-evalasync-vs-timeout-in-angularjs.htm)
* Prefer using Angular service wrappers instead of using native JS functions. (In this case, use $q service by Angular instead of native Promise API)
* Try to use directives whenever possible as $scope.$apply() will just update the view associated with that specific directive.
<!-- <script async src="//jsfiddle.net/rgabs/paxf1Lmn/1/embed/"></script> -->
