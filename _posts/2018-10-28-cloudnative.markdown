---
layout: post
title: "How I explained Cloud Native to my daugther"
date: 2018-10-28 11:32:20 +0300
description: In this blog I have tried to explain cloud native in layman's terms with the help of a small parable. # Add post description (optional)
img: cloudnative.png # Add image post (optional)
imgPost: cloudnative.png #Add post header image (optional)
tags: [cloud, cloud native,microservices,cloud enabled]
---
Last weekend was quite hectic as we had the production release. Being the part of production war room I was working from home.
<p>
My elder daugther who is in her 7th grade was working on java program and out of nowhere she asked me
<b><i>"Dad, what is cloud native? Yesterday I saw an article on it in the opensource magazine but not able to understand it. How different it is from the normal Cloud applications. Can you please make me understand in simple language".</i></b>
</p>

Instead of putting in technical terms, I explained to her in the form of a small parable, so that it becomes easy for her to comprehend.

After then I realized to put it in my blog so that it will be useful for others also. I do agree that there can be much better and simpler ways of explaining the concept but this is what I had at that time. This story is just to give a high level understading of what actually cloud native is and how different it is from the normal cloud apps.

... so the story goes like this
<hr>
Long ago there were two powerful kingdoms named <b>"Rashtrahumsa"</b> and <b>"Pratisimha"</b>.They were ruled by the kings namely <b>"Dharma"</b> and <b>"Vasumitra"</b>.They were equally competent in all fields,but differed in their administrative management style.

<div class="alert alert-info" role="alert">
So here the two kingdoms <b>Rashtrahumsa</b> and <b>Pratisimha</b> are nothing but the cloud-enabled and cloud native platforms.
</div>

King Dharma was more <b>domineering in nature and exercised complete authority</b>.He had competent ministers under him, but he kept all the powers in his hands and everyone had to abide by it. The ministers were neither having any judicial power nor had any say in the decision making. King need to be approached for all matters whether big or small <b><i>which delayed decision making and execution</i></b>.

<div class="alert alert-info" role="alert">
Here the King Dharma means a <b>traditional monolith application</b> deployed in the cloud platform.Like King Dharma, for Mononlith application <b>even a small change involves rebuilding and redeploying the entire code base which is a time consuming process</b>.<br><br>Even though the application resides in a cloud platform it is not able to leverage the cloud capabilities fully.Though the monolith is comprised of intelligent modules like ministers they are not independent entities and need to be bide by the monolith app.
</div>

King Vasumitra did <b>power de-centralization and formed a number of administrative ministries</b>. For each ministry he appointed competent ministers  and entrusted them with complete power and authority.<br>They were whole and sole responsible for the growth and development of that ministry.
At any point of time each ministry had a limited number of judegments to be done and were completed on time.Each minister could take care of his ministry without interfering in other ministries.<br>
He also formed the department of administrative affairs that develops steering systems and determines overarching guidelines for effective and coordinated management and operation of the ministry. The department handles administrative matters and ensures that the Ministry’s management is in regulatory compliance.
<div class="alert alert-info" role="alert">
Like King Vasumitra's administration, in <b>Cloud Native each application is comprised of multiple independent applications also knowns as Microservices which abide to single responsibility principle</b>. <br><br>Every service is independent in itself and is not affected by other services lifecycle. Microservices Governance is a methodology or approach that establishes policies, standards, and best practices for the adoption of Microservices to enable an enterprise agile IT environment
</div>
Both the Kings in the quest to expand their kingdoms started conquering foreign lands. For months they were on battlefield, so couldn't spend any time on administrative affairs. King Vasumitra had well established ministry to take care of all administrative matters in his kingdom but for King Dharma it became a major bottleneck, as all his administrative decisions were put on hold in his abscence which made him realize the gigantic mistake he did.He came to know of Vasumitra's administration style and being impressed wanted to adopt the same style.
<div class="alert alert-info" role="alert">
 This section highlights how the scalability is affected when not being cloud native. In monolith it is not possible to scale out a particular module or functionality which is used the most. But in case of microservices it is possible to scale each service independently of the other and auto scaling features of cloud native environment make this more seamlessly
</div>

He then sent few of his ministers to Vasumitra's kingdom with the task of understanding his administration style.The miniters went to Vasumitra's kingdom and settled there. Being immigrants they faced lot of challenges and were deined rights.What was the use of staying in Vasumitra's kingdom and not able to excerise all the rights which the local people were having.They then decided to transform themselves into native beings slowly and steadily. They started learning the local/native language, adopted their art , culture and way of living. Within few years they were not less than the native beings.All the ministers slowly got employed in Vasumitra's administrative ministeries. After getting complete knowledge they returned to their kingdom and explained King Dharma of the Vasumira's administrative style.

<div class="alert alert-info" role="alert">
 This section mentions the approach of transforming the existing monolith application into cloud native using microservices approach. <b>Strangulation is suggested as the recommended approach to transform the monoliths into microservices</b>. Once the monolith is borken down into microservices it becomes fully cloud native
</div>

King Dharma then ordered to transform his administration in the footsteps of his ministers and slowly he started seeing the benefits of the same.Now he could spend more time in strategic decisions and the welfare of his kingdom.

Hope it has helped to throw some light on what is cloud native and how different it is from cloud enabled applications. 
