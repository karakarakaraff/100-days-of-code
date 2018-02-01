# 100 Days of Code Challenge
### Day 1
I started the intermediate mobile web development course through [Udacity's Grow With Google scholarship](https://blog.udacity.com/2017/10/udacity-google-announce-50000-new-scholarships.html). Today's lesson went over the benefits of designing a website for offline first. **Essentially, the goal is to get something on the screen without waiting for the network to succeed or fail.** A few of the key points:
* Getting your code to display on a mobile device follows a journey like this: device > router/cell tower > ISP > proxies > destination server > proxies > ISP > router/cell tower > device
* If any one thing in that route is slow or fails, the entire journey slows down or fails. This can be caused by lots of things, such as a poor signal, busy network, misconfigured proxy, DDOS attack on a server, bug in the server code, even the moon's gravitational pull (though this is debated).
* By designing for offline first, you don't have to depend on all of the pieces working correctly in order to give the user content. Instead, you can deliver whatever is cached on the device first, then attempt to fetch updated content from the network. If the network is fine, fresh data can be sent to the device, and that data will automatically be saved into the cache.

To get started on the offline-first project, I cloned the course's Wittr site and pushed it to my own repo: [karakarakaraff/witter](https://github.com/karakarakaraff/wittr.git). The site is currently built for online-only access, and I will be transitioning it to offline first. I also tested out the different types of connectivity (perfect, slow, lie-fi, offline) via a node server to give me a better idea of how an online-only site behaves under each circumstance.

### Day 2
I got an intro to service workers through the [Google Scholars/Udacity intermediate mobile web development course](https://blog.udacity.com/2017/10/udacity-google-announce-50000-new-scholarships.html). From my understanding, a service worker is basically a script that intercepts all of the requests a browser makes. So, for example, instead of the browser going directly to the network, the service worker can redirect the browser to its cache, send the request on to the network, and so on. This is the first key to offline first development since the service worker can determine if the network is up, down, slow, etc. (I'm assuming, I haven't actually gotten to how that part works yet), then direct the browser accordingly.

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
I got to leave Alexa to the  certification process, so I'm back with my [Google Scholars/Udacity intermediate mobile web development course](https://blog.udacity.com/2017/10/udacity-google-announce-50000-new-scholarships.html). I left off on day 2 with an intro to service workers, and I continued that today by learning how to completely handle a browser request with a service worker, all without ever touching the network. It's actually pretty simple! Inside the service worker itself, you can write a chunk of code like this:

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
Wow, I made a ton of progress with service workers through the [Google Scholars/Udacity intermediate mobile web development course](https://blog.udacity.com/2017/10/udacity-google-announce-50000-new-scholarships.html) today! Last time, I learned how to intercept a request from the browser by ignoring the network entirely and responding with a simple HTML response. Today, I took that further by:
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
Coding today was a challenge because I was sooooo tired, but I powered through and finished the entire service worker section of the [Google Scholars/Udacity intermediate mobile web development course](https://blog.udacity.com/2017/10/udacity-google-announce-50000-new-scholarships.html). Hurray! As promised, there are more user experience upgrades:
* Notify the user when there's an update, which means there's a new service worker waiting
  * [task-update-notify](https://github.com/karakarakaraff/wittr/compare/master...task-update-notify)
* Give the user the option to reload the page, which will push the new service worker into action
  * [task-update-reload](https://github.com/karakarakaraff/wittr/compare/master...task-update-reload)
* Swap out the root page for the skeleton page so, even if the network is down or slow, the user will see our branding and custom messages instead of browser errors
  * [task-page-skeleton](https://github.com/karakarakaraff/wittr/commit/4650723a59914a58371847945cc20df062dcd866)

### Day 8
Today, I moved on to the next section of the [Google Scholars/Udacity intermediate mobile web development course](https://blog.udacity.com/2017/10/udacity-google-announce-50000-new-scholarships.html), which is all about IndexedDB and caching. To get started, there's a little crash course intro to IndexedDB and an accompanying library, [IndexedDB Promised](https://github.com/jakearchibald/idb). The crash course was half theory (for example, why are promises so important that we want to use the library?) and half how-to. I learned the important parts, which are:
* Creating a database with `.open()`
```
var dbPromise = idb.open('test-db', 1, function(upgradeDb) {
  var keyValStore = upgradeDb.createObjectStore('keyval');
  keyValStore.put('world', 'hello');
});
```

* Updating (aka upgrading) a database with changes via switch statements
```
var dbPromise = idb.open('test-db', 2, function(upgradeDb) {
  switch(upgradeDb.oldVersion) {
    case 0:
      var keyValStore = upgradeDb.createObjectStore('keyval');
      keyValStore.put('world', 'hello');
    case 1:
      upgradeDb.createObjectStore('people', { keyPath: 'name' });
  }
});
```

* Reading specific information from a database store based on the key with `.get()`
```
dbPromise.then(function(db){
  var tx = db.transaction('keval');
  var keyValStore = tx.objectStore('keyval');
  return keyValStore.get('hello');
}).then(function(val){
  console.log('The value of "hello" is:', val);
});
```

* Reading everything from a database store wth `.getAll()`
```
dbPromise.then(function(db){
  var tx = db.transaction('people');
  var peopleStore = tx.objectStore('people');
  return peopleStore.getAll();
}).then(function(val){
  console.log('People:', people);
});
```

* Writing a key:value pair to a database store with `.put()`
```
dbPromise.then(function(db){
  var tx = db.transaction('keyval', 'readwrite');
  var keyValStore = tx.objectStore('keyval');
  keyValStore.put('bar', 'foo');
  return tx.complete;
}).then(function(){
  console.log('Added foo:bar to keyval');
});
```

* Writing an object to the database store with `.put()`
```
dbPromise.then(function(db){
  var tx = db.transaction('people', 'readwrite');
  var peopleStore = tx.objectStore('people');
  peopleStore.put({
    name: 'Sam',
    age: 25,
    favoriteAnimal: 'dog'
  });
  return tx.complete;
}).then(function(){
  console.log('Person added to people');
});
```
*Note: I was left with the impression that this code may have looked slightly different if I needed to indicate what the key would be. However, when this particular database store for people got set up (see line 129 with `('people', { keyPath: 'name' })`), the key is set to automatically be the person's name, so you don't have to specify the key in the .put() above.*

* Indexing the database by something other than the key, which involves multiple steps:
```
var dbPromise = idb.open('test-db', 4, function(upgradeDb) {
  switch(upgradeDb.oldVersion) {
  ...
  ...
    case 3:
      // create a new index with these two lines
      var peopleStore = upgradeDb.transaction.objectStore('people');
      peopleStore.createIndex('animal', 'favoriteAnimal');
  }
});

// then set up a transaction to get all the people who like a specific value in that index
dbPromise.then(function(db) {
  var tx = db.transaction('people');
  var peopleStore = tx.objectStore('people');
  var animalIndex = peopleStore.index('animal');

  return animalIndex.getAll('cat');
}).then(function(people) {
  console.log('Cat people:', people);
});

```

Anyway, I have a fascination with databases and querying them and organizing the data, so even though most students in the course are saying this section is the hardest and most boring, I have a feeling it might be one of my favorite (though definitely still hard). Also, I'm realizing the more I go through this course that I still do not have my head wrapped around promises and callbacks. Actually, I think I have at least somewhat of a handle on callbacks, but promises ... what?!? I've been googling and saving a lot of resources, plus bookmarking other resources shared by students in the program, so I'm going to make it a priority to fully learn and understand promises and callbacks as soon as I finish this course. I don't necessarily have to understand them now to make the code work, but I want to make sure I understand them in the future when I'm writing my own code.

### Day 9
Today on the database front of the [Google Scholars/Udacity intermediate mobile web development course](https://blog.udacity.com/2017/10/udacity-google-announce-50000-new-scholarships.html), I learned how to successfully transition the content of a website to offline-first. Basically, here's what you want to happen:

1. The service worker will fetch the page skeleton and assets from the cache
2. The browser will get posts from the database
3. Connect a web socket from the browser to the internet, *completely bypassing the service worker and the cache*, to get updated posts
4. As new posts arrive, add them to the database for next time

To make all of this work, I first had to create a database store to hold the posts. The posts need to be keyed by their ID and indexed by the time they were created.
* [task-idb-store](https://github.com/karakarakaraff/wittr/commit/7bbf637632d996d2b76dd10dfd02d827b6547d46)

Then, I had to tell the browser to get those posts and display them in the correct order.
* [task-show-stored](https://github.com/karakarakaraff/wittr/commit/c7c04cbc0f876b313f4621f6436ac1a58bbc7551)

Lastly, now that the browser is storing all these posts, what do you do when there are hundreds, thousands of posts? I was wondering that myself before the course touched on it, so I'm glad this was part of it! Anyway, the instructions said to keep only the latest 30 posts for offline use, so in other words, you need to write logic for the database to keep itself clean.
* [task-clean-db](https://github.com/karakarakaraff/wittr/commit/b7d0d0c9d22ebadebba8fc35ccab713929d358f9)

Tomorrow, I'll learn the last piece of the puzzle, which is how to store and retrieve images. I'm pretty excited about it!

##### RESOURCE
Udacity encourages students to be active on the forums, so I've been setting aside time each day for that as well. Today, someone had asked about JavaScript promises, and since I said yesterday that I'm still confused by them, I opened the thread to see how other people would explain them. And that's when I found this fantastic resource!

[JavaScript Promises: An Introduction](https://developers.google.com/web/fundamentals/primers/promises)

I've found plenty of other resources about promises, but this one really stands out because it's written by the instructor of the course, so I know it will definitely come in handy as I progress through each session. I'm going to make it required reading for myself at some point this week.

### Day 10
I finished the offline-first part of the [Google Scholars/Udacity intermediate mobile web development course](https://blog.udacity.com/2017/10/udacity-google-announce-50000-new-scholarships.html) today. Hurray! I already cached and stored the skeleton of the page and all of the static content, so that just left dealing with image assets. Dealing with images is a bit different than the rest of the site content because the way databases and caches deal with images is different. Basically, if you have to pull an image from a database, you have to wait until the entire image is ready to send. On the other hand, a cache can send the image bit by bit, so you can see on the screen as it loads. Using caches for images is preferable because of the increased performance and the better used experience.
* [task-cache-photos](https://github.com/karakarakaraff/wittr/commit/cbe409fb24d8853b9df93d7777fd47743ef4be87)

Additionally, you have to keep in mind that images can take up a lot of space, so you don't want cache more than you need. Just like the database store with posts is set to clean itself, the image cache can clean itself as well. With posts, you keep a certain number. With images, though, there's no set number -- you check the posts to see which images are included, then delete anything that isn't included.
* [task-clean-photos](https://github.com/karakarakaraff/wittr/commit/8493cb3fe09a122d11918084d9102880d57f520e)

Avatars are a bit different. Since users can change their avatars, you not only want to cache all of the avatars, but you want to check the network for any avatar updates, then overwrite the avatars as needed.
* [task-cache-avatars](https://github.com/karakarakaraff/wittr/commit/ca10602ad71175fbf125daa75e7fedc473cbb0ff)

Since all of the work for the offline-first project was done in a bunch of separate branches, I wanted to get all of the changes in one place along with comments in the code that explain what each piece is doing. I created a new branch called offline-complete, then merged it with master. And voila! A complete transition from an online-only web app to an offline-first web app.
* [merged offline-complete](https://github.com/karakarakaraff/wittr/commit/5808a8efda310264f1a1f2648017dc956a4db616)

The second half of the course will be dedicated to learning ES6 and expanding my JavaScript skills. I'm equally excited about this part because I know I can tremendously improve the way I write code. I've been writing and using JavaScript for a little more than a year now, and even though I can typically program things to work and get the outcome I want, I know there's probably a better way to get to that outcome. Coding is a craft, so it's just going to take time, practice and challenging myself to get better.

### Day 11
Since I'm at a break point on the [Google Scholars/Udacity intermediate mobile web development course](https://blog.udacity.com/2017/10/udacity-google-announce-50000-new-scholarships.html), I decided to go back to my Oregon Trail Alexa skill yet again. I really wanted to clean up the code, especially the gameOver function and the phrases Alexa uses multiple times. I had kind of started this before, but it was a lot of work that required me fitting in little pieces of time here and there between other obligations. Today, I finally finished and merged all of the changes! They can be seen here: [Merge branch 'improvements'](https://github.com/karakarakaraff/alexa-skills/commit/fd54f8e385e4d1eaaa066dc50278a027a16cc88e)

In addition to that, I also experimented a bit with DynamoDB again, picking up where I left off on [day 5](https://github.com/karakarakaraff/100-days-of-code#day-5). I was correct in assuming a lot of the sources I found before were outdated, and now that I've figured it out, it's almost embarrassing how easy it is to add persistence to an Alexa skill via DynamoDB! (Side note: It looks like this is a new feature as of 16 days ago.) However, after getting the database to save all of my session attributes, I ran into a major problem:

1. Start a game.
2. Quit somewhere halfway through.
3. Open the game again later, expecting to start a new game via the launch request.
4. Instead, Alexa returns to whatever previous game play mode I was in before, but handles the launch request as an unhandled intent.

Once again, I found a bunch of resources online and tons of people having the exact same problem. As it turns out, when Alexa persists the session attributes to the database, she also saves the game state there. To complicate things, what the [documentation](https://github.com/alexa/alexa-skills-kit-sdk-for-nodejs#skill-state-management) says to do doesn't work, though several people have found workarounds ([here](https://github.com/alexa/alexa-skills-kit-sdk-for-nodejs/issues/69) and [here](https://github.com/alexa/alexa-skills-kit-sdk-for-nodejs/issues/159)). Based on Amazon's response to this, it seems it's a known issue and they've put in a feature request to simplify it, but until then ... ``¯\_(ツ)_/¯``

I've got the issue under control in testing and have not pushed it to production (never make the same mistake twice!), so hopefully I can figure this out soon enough. However, there is a long, long delay between me setting my session attributes in a game and the database updating -- I'm talking hours. I don't know what the deal is, but it means I'll have to come back to this issue another time.

### Day 12
Back to the [Google Scholars/Udacity intermediate mobile web development course](https://blog.udacity.com/2017/10/udacity-google-announce-50000-new-scholarships.html)! The rest of the course seems to be less about actual project development and more about learning ES6. Honestly, bundling ES6 with this course was a hugely pleasant surprise for me because I've seen ES6 used in quite a few places but always wondered why it's different than the version of JavaScript I know and love. And actually, if you look at the [Alexa skill samples](), you'll see they're all written in ES6, so they have variables like `const` and strings that include `$s`, and I tried to incorporate as much as I understood in my own [Oregon Trail skill](), but you can see I still have a lot to learn.

As I go through Udacity's content, I'm also going to cross reference with another resource I found: [ES6 Articles by Wes Bos](https://github.com/wesbos/es6-articles). I think it will be good to read from both sources to ensure I get all the concepts and don't get left with any questions. In other words, I've got my bases covered!

Anyway, today I learned:
* The use cases for `const` and `let`, which inevitably leads to the death of `var`.
* How `const`, `let` and `var` act under hoisting, which is why ES6 is killing off `var` (I mean, you can still use it, but there's no reason to)
  * *Note to self:* I'll have to look into how `let` and `const` affect function expressions (such as `var functionNameHere = function() {};`). I prefer writing function expressions as opposed to function declarations (`function functionNameHere() {}`).
  * With just a bit of googling, it looks like this is where arrow functions will come in ... maybe? That will be later in the course.
* How to use template literals and string concatenation -- no more `+` or `\n` needed! Instead, wrap the string in backticks (not single or double quotes), and use `${expression}` for placeholders. Line breaks can be directly inserted in the code, and they will be preserved. Here's an example:

**The old way:**

`var message = student.name + ', \nPlease see ' + teacher.name + ' in ' + teacher.room + ' to pick up your report card.\nThank you!';`

**The new way:**
```
let message = `${student.name},

  Please see ${teacher.name} in ${teacher.room} to pick up your report card.

  Thank you!`;
```

* This kind of simplification of template literals also helps immensely with HTML fragments:
```
function createAnimalTradingCardHTML(animal) {
  const cardHTML = `<div class="card">
    <h3 class="name">${animal.name}</h3>
    <img src="${animal.name}.jpg" alt="${animal.name}" class="picture">
    <div class="description">
      <p class="fact">${animal.fact}</p>
      <ul class="details">
        <li><span class="bold">Scientific Name</span>: ${animal.scientificName}</li>
        <li><span class="bold">Average Lifespan</span>: ${animal.lifespan}</li>
        <li><span class="bold">Average Speed</span>: ${animal.speed}</li>
        <li><span class="bold">Diet</span>: ${animal.diet}</li>
      </ul>
    <p class="brief">${animal.summary}</p>
    </div>
  </div>`;

  return cardHTML;
}
```

It's so beautiful! When I finish this course, my next project is to rebuild my portfolio website as a static site so I can host it on [GitHub Pages](https://pages.github.com/) (it's currently a Wordpress site, which you can see here: [karaflaherty.com](http://karaflaherty.com/)). I absolutely CANNOT WAIT to write my HTML fragments like this!
