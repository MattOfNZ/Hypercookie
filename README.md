# Information Disclosure using iOS 9 Content Blockers
## A working example of a cross-domain "hypercookie"

In this repository, you will find a working example of a cross-domain "hypercookie", implemented by leveraging the Content Blockers introduced in iOS 9.  This "hypercookie" is able to be used to uniquely identify a user on any domain, and in all browsing modes (private or otherwise).  It also forms a proof of concept for an outbound communications channel formed in content blocking rules which could be used for other forms of information disclosure. 

[See a demonstration video here](http://www.youtube.com/watch?v=aTqvmDLUwVM)

__Note:__ The code that is currently available in this repository is the first functional version of the code, released in a very early stage.  Consider it a working prototype, a proof of concept.  In reviewing the technique, there are far more efficient means that could be used to transmit data using a content blocker, but this one is easy to understand and explain so I've stuck with it.  I'll leave making a more efficient version up to you.

### iOS 9 Content Blockers
As part of iOS 9,  Apple has introduced the a Content Blockimg feature for Safari.  It distinguishes itself from Content Blockers on other platforms by isolating the rule processing and content access through the use of rules.  When a content blocker is created for iOS, it passes a list of blocking rules to be enforced by the Safari engine.

### Not Inherently Trustworthy
Part of the idea of separating the implementation of the rules from the enforement was to prevent content blocking apps from interacting with the page the user is viewing.  From a privacy and security benefit this is obviously an improvement over some existing implementations on other platforms, where you effectively place full trust in the developer to access your content on all websites.  However allowing the removal of specific content can become a communications channel in itself.

### Hypercookie Implementation Requirements
To demonstrate that content blocking can be used as a communications channel, I set out to create a cross-domain cookie that could uniquely identify a user with minimal impact on the overall page load time.  This cookie needed to be present in both normal and "Private" browsing sessions, and persist without the use of any technology beyond the content blocker itself.  A unkillable "hypercookie".  For demonstration purposes, the cookie should also disappear when the page is loaded without a content blocker - to quickly demonstrate that the content blocker is clearly the source of the cookie.

### The Technique
This implementation uses the "css-display-none" option to remove unwanted content from a set of elements added to a page.  The simplest way to imagine this is as a matrix.  An example of the elements that would be rendered for a simple string:

| Position 1 | Position 2 | Position 3 |
|:----------:|:----------:|:----------:|
0|0|0
1|1|1
2|2|2

This matrix allows us to communicate any three digit combination of the numbers 0, 1 and 2.

To communicate the combination "012", we would simply remove the elements that aren't part of our message:

| Position 1 | Position 2 | Position 3 |
|:----------:|:----------:|:----------:|
**0**|~~0~~|~~0~~
~~1~~|**1**|~~1~~
~~2~~|~~2~~|**2**

We acheive this by implementing 6 rules to hide unwanted content.  We can then extract what remains, and rebuild the original combination.

Since we have used the css-display-none option to remove content, extracting the final combination is as simple as grabbing the text inside of our hypercookie element.

This approach was scaled out to support a 32 hex digit GUID.  512 elements are rendered to the page, each element presenting a positon in the string and a digit.  This is presented as follows:

```html
<div class="hypercookie">
  <div class="p0">
    <div class="c0">0</div>
    <div class="c1">1</div>
    <div class="c2">2</div>
    ...
  </div>
  ...
</div>
```

Then, 480 rules are added to a content blocker to remove the elements we want to hide, leaving only 32 elements behind.  The rules are in JSON format, and are compatible with the [1Blocker](https://itunes.apple.com/us/app/1blocker-block-ads-tracking/id1025729002?mt=8) packages so that you can play along at home.

These rules are presented as follows: 

```javascript
{  
   "id":"08ce2d93-c996-4aef-962b-789e31347d87",
   "name":"Hide GUID (Pos 0, Char \"0\")",
   "content":{  
      "trigger":{  
         "url-filter":".*",
         "load-type":[]
      },
      "action":{  
         "type":"css-display-none",
         "selector":".hypercookie .p0 .c0"
      }
   }
}
```

This allows us to extract the original GUID, as Safari has removed every digit that isn't required for us.

### The Code
In this repository, you will find two files:

__create_package.html__ This will allow you to create your own package with your own random GUID for use with the iOS 9 Content Blocker.  It includes the JSON for your review, and the ability to download this file as a [1Blocker](https://itunes.apple.com/us/app/1blocker-block-ads-tracking/id1025729002?mt=8) Package - allowing you to deploy these rules to your own device for testing.  You can deploy multiple GUIDs to your device, but this prototype will fail to detect the cookie if you run more than one package at a time.  (Please don't leave this rule "on" for your device once you've seen it in action.  I'm trying to make you aware of tracking, not get you tracked.)

__read_hypercookie.html__ This shows a working implementation of reading the hypercookie, and displaying it inline in the page.  It also includes a basic detection routine, and will provide some messages to tell you what has occured.  Everything below the horizontal line on this page is the hypercookie itself, allowing you to 'see it in action'.  

### Is This a Real Risk?
Just because this is technically possible, does not mean that it's definitely a risk in a real world scenario.  For me, there are three major mitigating points here.

__Apple's Review Process:__ An app released to the app store goes through a review process, and I hope Apple thought of this kind of usage of their new feature.  Detecting a developer doing this within their app should be reasonably straightforward:  Any time that a deveoloper generates a rule based on some identifying factor on the device, or from some value stored on the device, that technique should be treated as suspect.  If Apple weren't checking for this before, there's a good chance they will be soon (if this page gains any traction.)  Apple's review process should be our first line of defence.

__Third-Party Packages:__ If some person on GitHub offers you an awesome third-party package with content blocking rules, maybe you should consider declining that offer (unless you're willing to read and review the rules, or trust them not to track you.)  The ball is in your court with this one.  Don't install random stuff on your device.

__Requires Massive Collusion To Execute:__ While it's true that some advertisers could distribute this kind of code through their networks, we have other mitigations (resource blocking etc) to prevent this from ever executing.  In addition, developers of content blocking apps would have to be in on the act.  In the current state of play, this doesn't seem practical to me.   However, with the suddenly accessible content blockers on mobile devices challenging the status quo, this could change.  

### Potential Improvements
The technique presented here is pretty unsophisticated;  and there are many potential improvements.  Here are my top few:

* Reduce the ruleset to 32 rules - remove the actual data and identify what's missing clientside
* Implement in a way that doesn't rely on visibility of elements
* Create some more robust detection and processing logic

Cheers,
[@MattOfNZ](https://twitter.com/MattOfNZ)

