---
layout:    template
title:     Todo List
---
<div id="post" class="round_shadow">
{% capture m %}
## TODO List
Not in specific order.

* Add rss feed to this site  
<st><sn>Build online portfolio;</sn></st> <done>Dt: 03-06-2013</done>
* Submit atleast one patch to linux kernel
* Visit to leh-ladakh
{% endcapture %}
{{ m | markdownify }}
</div>
