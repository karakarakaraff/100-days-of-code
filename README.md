# 100 Days of Code Challenge
### Day 1
I started the intermediate mobile web development course through [Udacity's Grow With Google scholarship](https://blog.udacity.com/2017/10/udacity-google-announce-50000-new-scholarships.html). Today's lesson went over the benefits of designing a website for offline first. **Essentially, the goal is to get something on the screen without waiting for the network to succeed or fail.** A few of the key points:
* Getting your code to display on a mobile device follows a journey like this: device > router/cell tower > ISP > proxies > destination server > proxies > ISP > router/cell tower > device
* If any one thing in that route is slow or fails, the entire journey slows down or fails. This can be caused by lots of things, such as a poor signal, busy network, misconfigured proxy, DDOS attack on a server, bug in the server code, even the moon's gravitational pull (though this is debated).
* By designing for offline first, you don't have to depend on all of the pieces working correctly in order to give the user content. Instead, you can deliver whatever is cached on the device first, then attempt to fetch updated content from the network. If the network is fine, fresh data can be sent to the device, and that data will automatically be saved into the cache.

To get started on the offline-first project, I cloned the course's Wittr site and pushed it to my own repo: [karakarakaraff/witter](https://github.com/karakarakaraff/wittr.git). The site is currently built for online-only access, and I will be transitioning it to offline first. I also tested out the different types of connectivity (perfect, slow, lie-fi, offline) via a node server to give me a better idea of how an online-only site behaves under each circumstance.

### Day 2
I got an intro to service workers through the intermediate mobile web development course. From my understanding, a service worker is basically a script that intercepts all of the requests a browser makes. So, for example, instead of the browser going directly to the network, the service worker can redirect the browser to its cache, send the request on to the network, and so on. This is the first key to offline first development since the service worker can determine if the network is up, down, slow, etc. (I'm assuming, I haven't actually gotten to how that part works yet), then direct the browser accordingly.

[Added service worker registration and logging of fetch requests](https://github.com/karakarakaraff/wittr/commit/cb6faa3d17768b07efb0754250405c544999504d): With this commit, I registered the service worker to the website. This includes bypassing the service worker on older browsers that can't handle service workers, plus logging a success or failure for registration on browsers that can handle service workers. Additionally, I set up a bit of code inside the service worker to log each request the browser makes. Then, using Chrome's dev tools, I can see the log.

*Note:* In the dev tools, I can also see if I have a service worker waiting in the background in addition to the active service worker. (This has to do with service worker lifecycles -- refreshing the page won't kill the service worker, but it will set a new waiting service worker. Closing the page or navigating to a different website, then coming back to the site will kill the previously active service worker and activate the service worker in waiting.)


### Day 3
Despite trying to start this challenge *after* finishing my latest Alexa game skill project, Oregon Trail (located in this repo: [karakarakaraff/alexa-skills](https://github.com/karakarakaraff/alexa-skills)), I had to come back to it today. My skill had gone live, and although it passed all of my local tests and Amazon's certification testing, I was getting reports that it was doing really strange stuff. For example, one person told me that all the names of the people in their party had changed half way through the game. Another person told me they tried buying supplies at the general store with $0 to spend (a way to test my handlers, which have a function to stop that), but they were allowed to buy supplies and ended with negative money.

First, I have so many people to thank for being so helpful and awesome in trying out my skill on its first day, from friends and acquaintances to strangers on Twitter. They all gave me excellent feedback, and that's how I was able to diagnose the problem after the skill had been live for only a couple hours.

**The problem:** I had written all of my game variables as global variables inside my AWS Lambda function. Since, in testing, only one person was accessing that Lambda function at a time, this was never a problem. However, in production, multiple people were accessing the Lambda function at the same time, so they were sharing, accessing and overwriting the global variables, thus affecting everyone else who was currently in the game. What a nightmare!

After doing some digging, I found several resources about Alexa's session object. These were the most helpful in giving me a handle on things:
* [Request and Response JSON Reference: Session Object](https://developer.amazon.com/docs/custom-skills/request-and-response-json-reference.html#session-object)
* [Stack Overflow: How to use session-specific variables in Alexa Skills?](https://stackoverflow.com/questions/36652187/how-to-use-session-specific-variables-in-alexa-skills)
* [Raymond Camden: An Example of Sessions with Amazon Alexa Skills](https://www.raymondcamden.com/2017/09/01/an-example-of-sessions-with-amazon-alexa-skills/)

What ended up working for me, at least in testing on my own, is declaring all of the key:value pairs for session.attributes at the beginning of a game, then accessing and updating them by doing a find/replace for all of my global variables. For example, what used to be a global variable, such as `var mainPlayer = "Kara"`, became a session attribute instead, like so: `this.event.session.attributes.mainPlayer = "Kara"`. ([View the commit diff on oregon-trail/lambda/custom/index.js](https://github.com/karakarakaraff/alexa-skills/compare/2137568aebec0c7203c911831c35c0be6f115162...3cff7511478fa639ccdd752ce37af23a487d3b48))

I could ensure it was working by checking the JSON:

```
{
  "session": {
    "sessionId": "amzn1.echo-api.session.[unique-value-here]",
    "attributes": {
      "STATE": "_USERSETUPMODE",
      "mainPlayer": "Kara"
    }
  }
}
```

Overall, it was a very painful learning experience and even cost me a 1-star review on my skill. However, now that the skill is going through certification testing again and will likely be back into production tomorrow, I'm feeling confident that the potential good reviews will outweigh the bad one. I've also walked away from this knowing that I have what it takes to debug and quickly take action. **The biggest takeaway today wasn't necessarily any specific coding knowledge, but instead, it was how to react to a bug in production.** I'm glad today is over!

### Day 4
I got to leave Alexa to the  certification process, so I'm back with my intermediate mobile web development course through [Udacity's Grow With Google scholarship](https://blog.udacity.com/2017/10/udacity-google-announce-50000-new-scholarships.html). I left off on day 2 with an intro to service workers, and I continued that today by learning how to completely handle a browser request with a service worker, all without ever touching the network. It's actually pretty simple! Inside the service worker itself, you can write a chunk of code like this:

```
self.addEventListener('fetch', function(event) {
  event.respondWith(
    new Response('<h1>Hello, world!</h1><p>I am a service worker.</p>', {
      headers: {'content-type': 'text/html'}
    })
  );
});
```

Now, no matter what request the browser makes, and no matter if the network is on, off, slow, etc., it will display the response above. However, it's a very simple response that doesn't take into account the user experience at all, so that part will come tomorrow.

### Day 5
I spent my time today reading about Amazon's DynamoDB service and tinkering around with some code samples. I was curious about how to incorporate it with Alexa because I had an idea for a feature for my Oregon Trail skill. The steps would include:
1. When a user first finishes the game and gets a final score, that score writes to the database.
2. When a user plays the game again and finishes, the skill needs to check the score currently saved in the database.
3. If the user's new score is higher than the score in the database, that new score overwrites the score in the database, and the skill tells the user that they have achieved a new high score.
4. If the user's score is in the top five highest scores across all users, the skill tells them that they have the top/nth highest score.
5. Regardless of how well the user plays, as long as they finish, the skill can ask them if they want to hear the top five scores. If so, the skill will query the database and return the name and score for first place, second place, etc.

I know the step 1 is totally possible and should be fairly simple. Steps 2 and 3 *should* be possible, but it's going to require some code for comparing old and new scores. As for steps 4 and 5, I'm not sure if Alexa allows me to share information between users, so I'll need to read more about it and try some experiments. Anyway, I refactored the gameOver() function into a GAME_OVER state with handlers so I can eventually implement step 5 via the AMAZON.YesIntent, like this:

**Alexa:** Do you want to hear the top five scores?
**User:** Yes.
**Alexa:** _____ is in first place with a score of _____. _____ is in second place with a score of _____. (and so on, and so on.)

Adding these features isn't necessarily needed ASAP, but I would like to do it fairly soon in case the skill becomes popular because, in that case, someone who plays the skill often would probably not be pleased if they got a great score tomorrow, then the feature becomes available the next day.

Here are some of the references I came across:
* [github.com/alexa/alexa-skills-kit-sdk-for-nodejs: Persisting skill attributes through DynamoDB ](https://github.com/alexa/alexa-skills-kit-sdk-for-nodejs#persisting-skill-attributes-through-dynamodb)
* [github.com/alexa/alexa-cookbook: Amazon DynamoDB](https://github.com/alexa/alexa-cookbook/tree/master/aws/Amazon-DynamoDB)
* [NodeJS and DynamoDB Alexa demo](https://github.com/maidoesthings/alexa-demo) (which pairs with [this blog post](https://medium.com/fuzz/hello-alexa-building-your-first-alexa-skill-61764214546))
* [Big Nerd Ranch: Implementing Persistence in an Alexa Skill](https://developer.amazon.com/alexa-skills-kit/big-nerd-ranch/alexa-implementing-persistence)

Doing this today was a bit of a distraction -- I should really be focusing all of my energy on the mobile web development stuff. My goal tomorrow is to put in at least four hours with the course so I can feel like I'm caught up with it. I only have nine more weeks to finish.

### Day 6
Wow, I made a ton of progress with service workers today! Last time, I learned how to intercept a request from the browser by ignoring the network entirely and responding with a simple HTML response. Today, I took that further by:
* Responding to specific requests and returning something that lives on the network (for example, for every .jpg file the browser requests, I can return the same gif, thus turning every image on the website into that gif)
  * [gif-response](https://github.com/karakarakaraff/wittr/blob/gif-response/public/js/sw/index.js)
* Responding depending on the request and the network (for example, if the request returns a 404 error, I can return a specific response; the same can be done if the request fails due to the network being unavailable)
  * [error-handling](https://github.com/karakarakaraff/wittr/blob/error-handling/public/js/sw/index.js)
* Creating a cache and writing to it with the service worker via the install event
  * [task-install](https://github.com/karakarakaraff/wittr/blob/task-install/public/js/sw/index.js)
* Checking the cache for an asset and, if it's there, serving it from the cache; if it's not there, fetching it from the network (assuming the network is up and working)
  * [task-cache-response](https://github.com/karakarakaraff/wittr/blob/task-cache-response/public/js/sw/index.js)
* Setting up the service worker to handle content updates, thus creating new caches
  * [task-handling-updates](https://github.com/karakarakaraff/wittr/blob/task-handling-updates/public/js/sw/index.js)

Seeing exactly how the service worker does its job makes me even more excited to implement offline-first design in future web projects. It feels almost revolutionary to be able to deliver content regardless of the network connection!

### Day 7
Coding today was a challenge because I was sooooo tired, but I powered through and finished the entire service worker section of the mobile web course. Hurray! As promised, there are more user experience upgrades:
* Notify the user when there's an update, which means there's a new service worker waiting
  * [task-update-notify](https://github.com/karakarakaraff/wittr/compare/master...task-update-notify)
* Give the user the option to reload the page, which will push the new service worker into action
  * [task-update-reload](https://github.com/karakarakaraff/wittr/compare/master...task-update-reload)
* Swap out the root page for the skeleton page so, even if the network is down or slow, the user will see our branding and custom messages instead of browser errors
  * [task-page-skeleton](https://github.com/karakarakaraff/wittr/commit/4650723a59914a58371847945cc20df062dcd866)
