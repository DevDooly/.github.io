---
layout: post
title:  "AngularJS 사용자 인증 처리 / ui-router"
date:   2016-07-02 11:10:00
categories: blog
---

##### UI-ROUTER 을 이용한 사용자 인증 처리

ui-router의 state에 authenticate 라는 임의 변수를 추가하여 인증 가능한 사용자만 접근 가도록 구현한다.

* signin : 모든 사용자 접근 가능.
* admin : authenticate = true 인 사용자 접근 가능.

route.js
 
```javacript
(function(){
  'use strict';

  angular
    .module('app')
    .config(function ($stateProvider) {
      $stateProvider
        .state('signin', {
          url: '/signin',
          templateUrl: 'signin/signin.html',
          controller: 'SigninCtrl'
        })
        .state('admin', {
          url: '/admin',
          templateUrl: 'app/admin/admin.html',
          controller: 'AdminCtrl',
          authenticate: true
        });
    });

)();
```


##### 인증값에 따른 사용자 페이지 이동

url 변경시 발생하는 $stateChangeStart 이벤트를 이용하여 인증값에 따라 서로 다른 페이지로 이동하거나 추가 이벤트를 발생하도록 한다.

app.js

```javacript
(function(){
	'use strict';

	angular.module('app', [
		'ui.router'
	])
	.run(function ($rootScope, $state) {

		$rootScope.$on('$stateChangeStart', function (event, toState, toParams, fromState, fromParams) {

			// 이동할 페이지에 authenticate 값이 있는지 확인해서 라우팅한다.
			if( toState.authenticate ){
				$state.transitionTo('signin');
				event.preventDefault();
			}
		});
	});
})();
```

##### 참조 사이트

* [miconblog.com](http://miconblog.com/archives/2014/11/anguarjs-ui-router%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%82%AC%EC%9A%A9%EC%9E%90-%EC%9D%B8%EC%A6%9D-%EC%B2%98%EB%A6%AC/)

* [Authentication with AngularJS] (https://medium.com/@mattlanham/authentication-with-angularjs-4e927af3a15f#.vow5h3ko5)