---
layout: post
title: Read URL GET params with javascript  
tags: web javascript
---

While working on this site, I had a requirement to read URL GET params. After quick searching on Google, Found [this old code](http://snipplr.com/view/19838/get-url-parameters/) . This javascript snippet parse the GET params and store them into a hash map which will be easy to retrieve. Exactly what I need for my reuirement and the code works out of the box.

{% highlight javascript %}
function getUrlVars() {
       	var vars = {};
	var parts = window.location.href.replace(/[?&]+([^=&]+)=([^&]*)/gi, function(m,key,value) {
	    vars[key] = value;
	});
	return vars;
}
{% endhighlight %}

	http://somedomain.com/index.html?id=1&group=2&item=xyz

To read variables from above URL, Javascript code should be something like this:

{% highlight javascript %}
<script type="text/javascript">
    var vars = {};
    vars = getUrlVars();
    alert(vars['id']);
    alert(vars['group']);
    alert(vars['item']);
</script>
{% endhighlight %}

There are some limitations of this code. It can't handle multiple values of same keys. To overcome these limitations, You can use `getUrlVar()` function from [this gist](https://gist.github.com/1771618).
