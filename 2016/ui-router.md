##ui-router learning
####[ui-router in deep](https://github.com/angular-ui/ui-router/wiki)

ui-router是Angular's v1的比原来ngRouter更强大的路由系统。它的思想是用状态机来管理路由。
其中`state`可以有自己的:

-  `template`, `templateUrl`, `templateProvider`
-  `controller`, `controllerAs`, `controllerProvider`
- `view`
- `resolve`, `data`
- `OnEnter`,`OnExit`
等可以配置的属性.

state还有自己相关的event:
>在$rootScope中发起的有关state的event：
* `stateChangeStart`
* `stateNotFound`
* `stateChangeSuccess`
* `stateChangeError`

>在$rootScope中broadcasts的view load event:
* `$viewContentLoading`

>$scope view load event:
* `$viewContentLoaded`

####**ui-view**
---------
```
<body>
    <ui-view>
        <i>Some content will load here!</i>
    </ui-view>
</body>
```
####**template**
---------

主要就是template、用url的templateUrl、还有更高级的tempalteProvider
```
$stateProvider.state('contacts', {
  template: '<h1>My Contacts</h1>'
})
--------------------------------
$stateProvider.state('contacts', {
  templateUrl: 'contacts.html'
})
---------------------------------
$stateProvider.state('contacts', {
  templateProvider: function ($timeout, $stateParams) {
    return $timeout(function () {
      return '<h1>' + $stateParams.contactId + '</h1>'
    }, 100);
  }
})
```
####**controller**
--------
可以用函数、已存在的controller的名字、还可以alias、或更复杂的controllerProvider
```
$stateProvider.state('contacts', {
  template: '<h1>{{title}}</h1>',
  controller: function($scope){
    $scope.title = 'My Contacts';
  }
})
---------------------------------
$stateProvider.state('contacts', {
  template: ...,
  controller: 'ContactsCtrl'
})
---------------------------------
$stateProvider.state('contacts', {
  template: '<h1>{{contact.title}}</h1>',
  controller: function(){
    this.title = 'My Contacts';
  },
  controllerAs: 'contact'
})
----------------------------------
$stateProvider.state('contacts', {
  template: ...,
  controller: 'ContactsCtrl as contact'
})
----------------------------------
$stateProvider.state('contacts', {
  template: ...,
  controllerProvider: function($stateParams) {
      var ctrlName = $stateParams.type + "Controller";
      return ctrlName;
  }
})
```
####**resolve**
---
可以配置转入这个state是所需要的资源、可以是简单的同步获取资源，或者是异步获取的资源(*如http资源，promise*)，异步资源要等到全部获取后才会进入controller实例化
```
$stateProvider.state('myState', {
      resolve:{

         // Example using function with simple return value.
         // Since it's not a promise, it resolves immediately.
         simpleObj:  function(){
            return {value: 'simple!'};
         },

         // Example using function with returned promise.
         // This is the typical use case of resolve.
         // You need to inject any services that you are
         // using, e.g. $http in this example
         promiseObj:  function($http){
            // $http returns a promise for the url data
            return $http({method: 'GET', url: '/someUrl'});
         },

         // Another promise example. If you need to do some 
         // processing of the result, use .then, and your 
         // promise is chained in for free. This is another
         // typical use case of resolve.
         promiseObj2:  function($http){
            return $http({method: 'GET', url: '/someUrl'})
               .then (function (data) {
                   return doSomeStuffFirst(data);
               });
         },        

         // Example using a service by name as string.
         // This would look for a 'translations' service
         // within the module and return it.
         // Note: The service could return a promise and
         // it would work just like the example above
         translations: "translations",

         // Example showing injection of service into
         // resolve function. Service then returns a
         // promise. Tip: Inject $stateParams to get
         // access to url parameters.
         translations2: function(translations, $stateParams){
             // Assume that getLang is a service method
             // that uses $http to fetch some translations.
             // Also assume our url was "/:lang/home".
             return translations.getLang($stateParams.lang);
         },

         // Example showing returning of custom made promise
         greeting: function($q, $timeout){
             var deferred = $q.defer();
             $timeout(function() {
                 deferred.resolve('Hello!');
             }, 1000);
             return deferred.promise;
         }
      },

      // The controller waits for every one of the above items to be
      // completely resolved before instantiation. For example, the
      // controller will not instantiate until promiseObj's promise has 
      // been resolved. Then those objects are injected into the controller
      // and available for use.  
      controller: function($scope, simpleObj, promiseObj, promiseObj2, translations, translations2, greeting){
          $scope.simple = simpleObj.value;

          // You can be sure that promiseObj is ready to use!
          $scope.items = promiseObj.data.items;
          $scope.items = promiseObj2.items;

          $scope.title = translations.getLang("english").title;
          $scope.title = translations2.title;

          $scope.greeting = greeting;
      }
   })
```
[Learn more](https://github.com/angular-ui/ui-router/wiki/Nested-States-%26-Nested-Views#inherited-resolved-dependencies) about how resolved dependencies are inherited down to child states.

####**data**
---
```
// Example shows an object-based state and a string-based state
var contacts = { 
    name: 'contacts',
    templateUrl: 'contacts.html',
    data: {
        customData1: 5,
        customData2: "blue"
    }  
}
$stateProvider
  .state(contacts)
  .state('contacts.list', {
    templateUrl: 'contacts.list.html',
    data: {
        customData1: 44,
        customData2: "red"
    } 
  })
---------------------------
function Ctrl($state){
    console.log($state.current.data.customData1) // outputs 5;
    console.log($state.current.data.customData2) // outputs "blue";
}
```

[Learn more](https://github.com/angular-ui/ui-router/wiki/Nested-States-%26-Nested-Views#inherited-custom-data) about how custom data properties are inherited down to child states.

####**onEnter and onExit**
---
当state变得active或则inactive时会用到的hook
```
$stateProvider.state("contacts", {
  template: '<h1>{{title}}</h1>',
  resolve: { title: 'My Contacts' },
  controller: function($scope, title){
    $scope.title = title;
  },
  onEnter: function(title){
    if(title){ ... do something ... }
  },
  onExit: function(title){
    if(title){ ... do something ... }
  }
})
```
####**相关event**
---
> $rootScope:

> - `$stateChangeStart` 
> - `$stateNotFound`
> - `$stateChangeSuccess`
> - `$stateChangeError`
> - `$viewContentLoading`

> $scoop:

> - `$viewContentLoaded`

##Nesting States
---
最简单的dot notation
```
$stateProvider
   .state('contacts', {})
   .state('contacts.list', {})
```
当然还能通过parent属性来制定上层state，还有其它东西到官方github文档上去查

Nesting States 继承 `resolve`， `data`属性
不继承`controller`， `tempalte`等

**$scope属性只有当view 在state中嵌套了，才会从state的继承链上继承**

###Abstract State 
---

继承：

- 把abstrct state的url, prepend到子state中
-  把子state的template插入到abstract state的```<ui-view>```中，还有父(*abstract*)state的$scope会被继承(*通过view 继承*)
- resolve继承
- data继承

例子: [ http://plnkr.co/edit/gmtcE2?p=preview]( http://plnkr.co/edit/gmtcE2?p=preview)

###view Nmae
这里直接给例子
```
<!-- index.html (root unnamed template) -->
<body ng-app>
<div ui-view></div> <!-- contacts.html plugs in here -->
<div ui-view="status"></div>
</body>
```
```
<!-- contacts.html -->
<h1>My Contacts</h1>
<div ui-view></div>
<div ui-view="detail"></div>
```
```
<!-- contacts.detail.html -->
<h1>Contacts Details</h1>
<div ui-view="info"></div>
```
```
$stateProvider
  .state('contacts', {
    // This will get automatically plugged into the unnamed ui-view 
    // of the parent state template. Since this is a top level state, 
    // its parent state template is index.html.
    templateUrl: 'contacts.html'   
  })
  .state('contacts.detail', {
    views: {
        ////////////////////////////////////
        // Relative Targeting             //
        // Targets parent state ui-view's //
        ////////////////////////////////////

        // Relatively targets the 'detail' view in this state's parent state, 'contacts'.
        // <div ui-view='detail'/> within contacts.html
        "detail" : { },            

        // Relatively targets the unnamed view in this state's parent state, 'contacts'.
        // <div ui-view/> within contacts.html
        "" : { }, 

        ///////////////////////////////////////////////////////
        // Absolute Targeting using '@'                      //
        // Targets any view within this state or an ancestor //
        ///////////////////////////////////////////////////////

        // Absolutely targets the 'info' view in this state, 'contacts.detail'.
        // <div ui-view='info'/> within contacts.detail.html
        "info@contacts.detail" : { }

        // Absolutely targets the 'detail' view in the 'contacts' state.
        // <div ui-view='detail'/> within contacts.html
        "detail@contacts" : { }

        // Absolutely targets the unnamed view in parent 'contacts' state.
        // <div ui-view/> within contacts.html
        "@contacts" : { }

        // absolutely targets the 'status' view in root unnamed state.
        // <div ui-view='status'/> within index.html
        "status@" : { }

        // absolutely targets the unnamed view in root unnamed state.
        // <div ui-view/> within index.html
        "@" : { } 
  });
```

##Url Routing
---
####**Basic example**

```
url: "/contacts/:contactId"
url: "/contacts/{contactId}"
<a ui-sref="contacts.detail({contactId: id})">View Contact</a>
// will only match a contactId of one to eight number characters
url: "/contacts/{contactId:[0-9]{1,8}}"
url: "/contacts?myParam"
// will match to url of "/contacts?myParam=value"
url: "/contacts?myParam1&myParam2"
// will match to url of "/contacts?myParam1=value1&myParam2=wowcool"


// If you had a url on your state of:
url: '/users/:id/details/{type}/{repeat:[0-9]+}?from&to'

// Then you navigated your browser to:
'/users/123/details//0'

// Your $stateParams object would be
{ id:'123', type:'', repeat:'0' }

// Then you navigated your browser to:
'/users/123/details/default/0?from=there&to=here'

// Your $stateParams object would be
{ id:'123', type:'default', repeat:'0', from:'there', to:'here' }
```

####**$stateParams**
$stateParams每次被inject的时候只会包含当前state url的param
```
$stateProvider.state('contacts.detail', {
   url: '/contacts/:contactId',   
   controller: function($stateParams){
      $stateParams.contactId  //*** Exists! ***//
   }
}).state('contacts.detail.subitem', {
   url: '/item/:itemId', 
   controller: function($stateParams){
      $stateParams.contactId //*** Watch Out! DOESN'T EXIST!! ***//
      $stateParams.itemId //*** Exists! ***//  
   }
})
```
想访问父state的param话需要借助resolve
```
$stateProvider.state('contacts.detail', {
   url: '/contacts/:contactId',   
   controller: function($stateParams){
      $stateParams.contactId  //*** Exists! ***//
   },
   resolve:{
      contactId: ['$stateParams', function($stateParams){
          return $stateParams.contactId;
      }]
   }
}).state('contacts.detail.subitem', {
   url: '/item/:itemId', 
   controller: function($stateParams, contactId){
      contactId //*** Exists! ***//
      $stateParams.itemId //*** Exists! ***//  
   }
})
```
####**$urlRouterProvider**

- when()
- otherwise()
- rule()

这里只给出常用的otherwise的例子
```
app.config(function($urlRouterProvider){
    // if the path doesn't match any of the urls you configured
    // otherwise will take care of routing the user to the specified url
    $urlRouterProvider.otherwise('/index');

    // Example of using function rule as param
    $urlRouterProvider.otherwise(function($injector, $location){
        ... some advanced code...
    });
})
```

#Last
[ui-router api reference](http://angular-ui.github.io/ui-router/site/#/api/ui.router)
