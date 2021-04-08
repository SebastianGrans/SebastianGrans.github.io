---
layout: post
title: Wikiwand on Safari
---

<center>
<img src="{{ site.baseurl }}/images/20191031-Wikiwand/headerimg.png" alt="Non-breaking space causes headaches" style="display: block;"/>
</center>If you use [AdGuard for Safari](https://apps.apple.com/us/app/adguard-for-safari/id1440147259?mt=12) as your ad blocker, add the filter to the "User filter" to redirect any Wikipedia article to Wikiwand. 


<div markdown="1" style="display:block; background-color:#eff0f1; border-radius: 3px; padding: 5px 20px; margin: 10px;">
**TL;DR** If you use [AdGuard for Safari](https://apps.apple.com/us/app/adguard-for-safari/id1440147259?mt=12) as your ad blocker, add the filter to the "User filter" to redirect any Wikipedia article to Wikiwand. 
</div>

<div markdown="1" style="display:block; background-color:#eff0f1; border-radius: 3px; padding: 5px 20px; margin: 10px;">
**Update:** For some reason, it has stopped working on my MacBook that runs Catalina. It still runs on another MacBook running Mojave. I'll look into it some day.
</div>
<div markdown="1" style="display:block; background-color:#eff0f1; border-radius: 3px; padding: 5px 20px; margin: 10px;">
**Update 2:** I have now updated the rule so that it works again! There's a bug in the AdGuard Base filter. [Link to issue](https://github.com/AdguardTeam/AdguardFilters/issues/79855)
</div>


As of Safari 13, only extensions from the App Store can be used. 
This killed a lot of extensions, including Wikiwand. However, we can get the main functionality back using another approach. 

In order for me to stay sane on the web today, I need to have an ad blocker (I hope for your sanity that you use one too). Since Safari 13 was launched, [AdGuard for Safari](https://apps.apple.com/us/app/adguard-for-safari/id1440147259?mt=12) was the only ad blocker I found that worked well (e.g. it blocked YouTube ads). AdGuard allows you to add user defined filters, and the syntax is explained [here](https://kb.adguard.com/en/general/how-to-create-your-own-ad-filters#javascript-rules). 

> (Please note that AdGuard for Safari ≠ AdGuard for Mac. The latter blocks Ads for any application on your compouter. However, it also costs money. This "tutorial" uses the former and I have no idea if the same would work for the latter.) 

With the filter syntax documentation, and some very basic JavaScript we can create this filter: 

```javascript
@@||wikipedia.org^$generichide,badfilter
wikipedia.org#%#if (window.location.search === "") { var lang = window.location.hostname.split('.')[0]; var article = window.location.pathname.split('/')[2]; window.location.href = "http://www.wikiwand.com/" + lang + "/" + article; } 
```

Add this, as a single line, to your user filters in AdGuard and any time you visit a Wikipedia article, you will be redirected to Wikiwand. 

I hope this helps someone :)  

**Enjoy!**


P.s. if the URL has any kind of query string (e.g. `?query=test`),  you will not be redirected. This is so that you can still read on the original Wikipedia when you press the "Read on Wikipedia"-button in Wikiwand. Because that appends `?oldformat=true`.

P.s.s. Expand the spoiler to see a explanation of the code.
<details markdown="1">
<summary><h3 style="display: inline">Explanation of the code</h3></summary>
Let's assume we visit "[https://en.wikipedia.org/wiki/wikiwand](https://en.wikipedia.org/wiki/wikiwand)"

The first part of the line, `wikipedia.org`, tells AdGuard to trigger the filter. `#%#` tells AdGuard that it should perform a JavaScript injection.
Anything after that should be JavaScript code.

Here is the code in a more readable form:
```javascript
// If we do not have any query string appended (i.e. not ?oldformat=true), 
// then  we should redirect to Wikiwand.
if (window.location.search === "") {  
  
	// This takes the domain: "en.wikipedia.org", splits it at each period into 
	// ["en", "wikipedia", "org"].
	// We then get the zeroth element, i.e. which language we are reading in. 
	var lang = window.location.hostname.split('.')[0];

	// Similarly, we get the path e.g. "/wiki/wikiwand"
	var article = window.location.pathname.split('/')[2]; 
    
 // Then we construct and set the new window location. I.e. the redirect. 
 // which has the form "https://www.wikiwand.com/en/wikiwand"
	window.location.href = "https://www.wikiwand.com/" + lang + "/" + article;
}
```

</details>
