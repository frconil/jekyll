---
title: DevOps starts in your own team.
tags: devops 
layout: post
---
No system is perfect, and errors happen. While it is useful to analyse what went wrong, how, where and when, who's to blame is definitely not one of the key analysis points.

Take a situation where XYZ stopped working as expected after a planned maintenance/upgrade/new deployment. There are roughly two distinct approaches to analyse the situation:

- Finger pointing
	- "You broke XYZ when you upgraded it!"
	- "Why didn't you test your changes after the deployment?"
	- "Do you know how much pain you caused us after that last maintenance?"  
- Stating the facts
	- "XYZ was broken after the upgrade"
	- "Was there anything wrong after the deployment? foo is not working in XYZ anymore"
	- "Users are getting some errors when using XYZ since the last maintenance."
  
An analysis of the vocabulary used in the first case shows how this creates a "us vs them" mentality :

- *YOU* broke it
- *WE* fixed it
- *YOU* caused *US* pain.


That assumes a lack of empathy, a disregard for quality (or even excellence!) and a lack of investment.  
Worst of all, it also send the message that everything that has a chance to break should be left alone	 unless you're 100% sure you can detect and subsequently fix any defect before an outage rears its ugly head (so in essence, *everything should be left alone*) if only because potential outcomes lead to blame, and nobody likes being blamed for anything.  
In the end, instead of taking down team silos, you create additional silos in your own team.  
    
On the other hand, if you calmly assess the facts, as in the second case, you acknowledge that something indeed broke, but foster a climate of collaboration.  
It prepares the team to tackle the issues at hand and focus on the real problems:

- How can we monitor xyz, as it's been shown to be critical?
- what tests should we run next time vs the ones we ran last time?

It's assuming that errors happen and that you should focus on preventing errors you are aware could happen, and decreasing your mean time to recovery. 

The "us vs them" division now becomes "our team vs the problem", and everyone knows nothing unites a team like adversity (please don't break stuff for the sake of team building without warning them first.)

Now I'm not saying it's easy: it's isnt. When you've spent several hours fixing some outage, you feel quite grumpy. You don't quite feel like remembering the time where you were the one who pushed some code without quite testing it, or redeployed a blank template on a prod server.

But that's how you build a great team: by applying the same philosphy that works between dev and ops to individual members of your team.