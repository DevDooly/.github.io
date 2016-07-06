---
layout: post
title:  "AngularJS JSON Key값 받아오기"
date:   2016-07-01 00:00:00
categories: blog
---

~~~
<div ng-repeat="(key, data) in dataSets">
	{{key}}
</div>
~~~

주의사항 : (key, data) 형태를 벗어나면 값을 가져오지 않는다.