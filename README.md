SubSession
==========

Warning - this only works for a subset of use cases.  Do not use this without understanding the compromises you will need to make.

Tabs in browsers have given web users a new mental model which is at odds with how HTTP works.

On some web sites it would be a good thing if tabs could have settings which persisted in that tab, but did not affect other tabs (on the same web site). 


Example:
--------

A browser webmail app may have multiple accounts for a user.  Logging in enables access to all of those accounts (unlike gmail).  

If a user opens two tabs, and switches to a second account in the second tab, they may expect further links (on the second tab) to use their second account, while links in the first tab go to pages relevant to the primary account.  Having a 'compose' page opening in a new tab, yet using the parent-tab's account, would be a good example of this.

Furthermore, if they middle-click a link in their second tab, that 'child' tab should have some relation to the tab from which it was opened (i.e. use the same email account).


Other solutions:
----------------

The example above could be solved by putting the account (or account id) into the URL, i.e. http://webmail.example.com/chris@home.example.com and http://webmail.example.com/chris@work.example.com


### Issues with the URL solution:

1. Scaling - if your have a half-dozen tab-specific settings your URLs will get ugly quickly.
2. RESTfulness - once you have decided to use a URLised setting, it must appear in *all* urls, otherwise a setting will be lost when navigating through pages which do not include a setting in their URL.  
   http://webmail.example.com/myBookmarks will have to become http://webmail.example.com/chris@home.example.com/myBookmarks and http://webmail.example.com/chris@work.example.com/myBookmarks, even though both URLs point to the same resource and the email address has nothing to do with the resource.


A Composite Solution:
---------------------

I am using a composite solution of URLised settings, where appropriate (good for sharing bookmarks) and just using the settings from the SubSession where the page has nothing to do with a particular setting.  e.g.  Your inbox link could be http://www.cmail.com/chris@work.example.com/inbox, even when you are on http://www.cmail.com/myBookmarks, as the current account can be associated with chris@work.example.com in the SubSession, on the server side.


Usage:
------

SubSession is implemented as a jQuery plugin.  Simply including the JavaScript file is enough to make it work.

    <script src="/js/jquery.subsession.js" type="text/javascript"> </script> 

SubSession gives your web application two new cookies, *subsession* and *subsession_breadcrumb*.  SubSession does not include any server-side functionality - you'll have to develop this yourself, for your framework.

The *subsession* cookie contains small integer value which is guaranteed to be unique for that user's current session.  e.g. 7.

The *subsession_breadcrumb* cookie contains the path to the current subsession. e.g. 4/7 shows that the user middle-clicked from subsession 4, creating subsession 7.


Server-side patterns:
---------------------

SubSession storage is typically a hash-map of the SessionID concatenated with the subsession ID.  If a setting cannot be found in HashMap(SessionID+subsession) it is idomatic to walk back up the *subsession_breadcrumb* until a setting is found in a 'parent' tab.


Bugs:
-----

As there is no concept of 'tabs' in HTTP/1.1, this code is a large hack.  It works by setting short lived cookies, using them as a message to the page that is about to be opened.  

They are short lived so that tabs which are opened manually (i.e. not middle-clicked) and have a URL entered do not erroneously get associated with an existing subsession.

If your site often takes more than 20 seconds to load pages, you may want to increase the 'SHORT_DELAY' constant.
