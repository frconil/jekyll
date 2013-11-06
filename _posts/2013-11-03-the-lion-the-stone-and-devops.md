---
title: The lion, the stone, and DevOps. 
tags: Devops  
layout: post
summary: No, we're not going to have a parody of a well known book by CS Lewis, even if sometimes Software Delivery sounds more like witchcraft than rational science (looking at you, Java error stacks)...
---

No, we're not going to have a parody of a well known book by CS Lewis, even if sometimes Software Delivery sounds more like witchcraft than rational science (looking at you, Java error stacks).

First, joke time:

> A group of tourists are in photo safari in Africa, and soon come to the pinnacle of the trip: the Lion Pride!  
Only problem is that the lions have just been fed, and are feeling more inclined to a nice nap in the shade than prowling around to please a group of tourists.  

> One of them start to complain loudly that it's a shame, and a rip-off, and don't they know who he is? Ten bucks they're just dumb animatronics!  

> He grabs a fair sized rock, aim at one of the males, and hit him square.  
Bad luck for him, the lion was the alpha male AND is not pleased at all. He starts roaring and waking up the whole pride, and they all run towards the fence and slam against it.  

> Obviously all the tourists are getting quite scared and run to the bus, except for one who's just standing there, non plussed.  

> Everyone call for him to stop being stupid and would he come in the bus before something happens, to which he replies:  

> " I don't see the problem, I'm not the one who threw the rock"




I have just been reminded of this after being told "why would you care if they listen to you, this won't be your problem!"  
I think that is the problem at hand. In a traditional model, people just play the blame game, and instead of addressing bugs as early as possible, just pass it on Ops/Devs/Sales/etc with the classic "well I didn't throw the rock!/Didn't commit that bug to trunk/didn't undersize the infrastructure/<insert any real life case here>"  
Once you realise that no matter who f$#%ed up, everyone will pay for it, you're already on the right path to deliver a great product.  
Just because the issue is caused by a bug in the code doesn't mean ops can't give their all to fasttrack tests/changes/monitoring results. 
Just because the issue is caused by the infrastructure not coping with specific parts of the app doesn't mean the devs can't help by optimising those specific parts further, or asisting with dump captures, etc.  

Yes, it does suck when delivery fails along the way, but working towards fixing it instead of palming the responsibility to someone else helps everyone realising what  some individual actions mean for other people (see: on-call roster, sales department, tech support team, devs in charge of fixing bugs or performance issues), and ensure the next time will be better.

