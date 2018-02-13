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
I got to leave Alexa to the  certification process, so I'm back with my [Google Scholars/Udacity intermediate mobile web development course](https://blog.udacity.com/2017/10/udacity-google-announce-50000-new-scholarships.html). I left off on [day 2](https://github.com/karakarakaraff/100-days-of-code#day-2) with an intro to service workers, and I continued that today by learning how to completely handle a browser request with a service worker, all without ever touching the network. It's actually pretty simple! Inside the service worker itself, you can write a chunk of code like this:

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

### Day 13
I finished learning the rest of the ES6 syntax in the [Google Scholars/Udacity intermediate mobile web development course](https://blog.udacity.com/2017/10/udacity-google-announce-50000-new-scholarships.html). This includes:
* Destructuring arrays
```
const point = [10, 25, -34];
const [x, y, z] = point;
console.log(x, y, z);
>> 10 25 -34
```
* Destructuring objects
```
const gemstone = {
  type: 'quartz',
  color: 'rose',
  karat: 21.29
};

const {type, color, karat} = gemstone;

console.log(type, color, karat);
>> quartz rose 21.29
```
* Object literal shorthand
```
let type = 'quartz';
let color = 'rose';
let carat = 21.29;

const gemstone = { type, color, carat };
```
* Shorthand for methods within objects
```
let gemstone = {
  type,
  color,
  carat,
  calculateWorth() { ... }
};
```
* The for...of loop, which eliminates the need for counting logic, exit conditions and indexes that other for loops require
```
const digits = [0, 1, 2, 3];

for (const digit of digits) {
  console.log(digit);
}
>> 0
>> 1
>> 2
>> 3
```
* The `continue` option that can be used to stop or break a for...of loop at any time
```
const digits = [0, 1, 2, 3, 4, 5];

for (const digit of digits) {
  if (digit % 2 === 0) {
    continue;
  }
  console.log(digit);
}
>> 1
>> 3
>> 5
```
* The spread operator (`...`), which is used to expand, or spread, iterable objects into multiple elements
```
const books = ["Don Quixote", "The Hobbit", "Alice in Wonderland", "Tale of Two Cities"];
console.log(...books);
>> Don Quixote The Hobbit Alice in Wonderland Tale of Two Cities
```
* Combining arrays with the spread operator
```
const fruits = ["apples", "bananas", "pears"];
const vegetables = ["corn", "potatoes", "carrots"];

const produce = [...fruits, ...vegetables];

console.log(produce);
>> ["apples", "bananas", "pears", "corn", "potatoes", "carrots"]
```
* The rest parameter (also `...`), which is used to bundle multiple elements back into an array
```
const order = [20.17, 18.67, 1.50, "cheese", "eggs", "milk", "bread"];
const [total, subtotal, tax, ...items] = order;
console.log(total, subtotal, tax, items);
>> 20.17 18.67 1.5 ["cheese", "eggs", "milk", "bread"]
```
* Using the rest parameter with functions that can take an indefinite number of arguments (aka variadic functions)
```
function sum(...nums) {
  let total = 0;  
  for(const num of nums) {
    total += num;
  }
  return total;
}

sum(1, 2);
>> 3
sum(10, 36, 7, 84, 90, 110);
>> 337
sum(-23, 3000, 575000);
>> 577977
```

My takeaway from ES6 is that it makes everything much more simple and concise. Last year, I learned JavaScript (non-ES6) first, then I learned Ruby, and I was totally blown away by how simple Ruby is! I hadn't considered the idea that JavaScript could be transformed to be easier to write, read and understand. In fact, although I don't know Perl or Python, I read that some of these new updates in ES6 (like destructuring) were modeled after those languages. Over all, it's really interesting to see how programming languages can grow and adapt.

### Day 14
The next section of the [Google Scholars/Udacity intermediate mobile web development course](https://blog.udacity.com/2017/10/udacity-google-announce-50000-new-scholarships.html) is all about functions in ES6. Instead of diving right in, I went over each part of the section and did some of my own reading on the topics that will be covered:
* Arrow functions (I'm especially looking forward to this!)
  * [MDN: arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
  * [freeCodeCamp: JavaScript ES6 Functions: The Good Parts](https://medium.freecodecamp.org/es6-functions-9f61c72b1e86)
  * [Wes Bos: JavaScript Arrow Functions Introduction](http://wesbos.com/arrow-functions/)
  * [Sitepoint: ES6 Arrow Functions: The New Fat & Concise Syntax in JavaScript](https://www.sitepoint.com/es6-arrow-functions-new-fat-concise-syntax-javascript/)
  * [codeburst: JavaScript: Arrow Functions for Beginners](https://codeburst.io/javascript-arrow-functions-for-beginners-926947fc0cdc)
* The addition of default function parameters
  * [MDN: default parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Default_parameters)
  * [Sitepoint: ES6 Default Parameters](https://www.sitepoint.com/es6-default-parameters/)
* The `class` keyword, which can be used to create functions *as* classes
  * [Fun Fun Function: Class keyword in JavaScript](https://medium.com/humans-create-software/class-keyword-in-javascript-f46f2e0b68d6)
* The `super` and `extends` keywords, which connect different classes together
  * [MDN: super](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/super)
  * [MDN: extends](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/extends)

I learned about classes when I first learned JavaScript, and although I understand them academically, I've never actually used them, so I don't know them from a production standpoint. In addition to learning arrow functions, I'm really looking forward to getting a refresher on classes, which will force me to consider how I can use them in a project.

### Day 15
Today's code time is dedicated to meeting with others who are doing #100DaysOfCode! When I interviewed for my current job at the newspaper, I was honest and let them know that I had enrolled in a coding bootcamp and would probably only be there for a year. Lucky for me, my boss hired me anyway! However, he eventually ended up leaving to go to a coding bootcamp as well, and then another coworker also recently left to attend a coding bootcamp. All three of us are taking slightly different routes, but we have the same goal in mind: finding a job that lets us put our coding skills to work everyday and that actually pays a living wage, thus adding a layer of financial freedom and work-life balance where it's currently missing.

Anyway, because we've all talked about coding at work, other coworkers have also expressed their interest in learning to code, and I feel a lot of camaraderie with these people because *I was once there, too.* I had been planning to do #100DaysOfCode as soon as I finished my bootcamp curriculum, and to build upon the momentum, I asked everyone if they'd like to join me -- to my delight, they said yes! We're all at different stages and skill levels, which makes it even more fun because that means we can all learn from each other.

Every Sunday, we're going to meet one hour before work to grab coffee and talk about our progress, our struggles, our questions, etc. Today's the first Sunday we could meet, and we're all on day 15, so I know we'll have lots to cover. Go team!

### Day 16
In the [Google Scholars/Udacity intermediate mobile web development course](https://blog.udacity.com/2017/10/udacity-google-announce-50000-new-scholarships.html), I'm finally on to arrow functions! First and foremost, on [day 12](https://github.com/karakarakaraff/100-days-of-code#day-12), I had a question regarding how ES6 handles function expressions, and I was left with a hunch that arrow functions would have something to do with it, which was confirmed today. As it turns out, **arrow functions are always expressions.** In fact, their full name is "arrow function expressions," so they can only be used where an expression is valid. This includes being:

1. stored in a variable
2. passed as an argument to a function
3. stored in an object's property

Generally speaking, arrow functions are pretty simple. Here's an example of two pieces of code doing the exact same thing:
```
const upperizedNames = ['Farrin', 'Kagure', 'Asser'].map(function(name) {
  return name.toUpperCase();
});

const upperizedNames = ['Farrin', 'Kagure', 'Asser'].map(
  name => name.toUpperCase()
);
```

The first function is a standard JavaScript function, while the second function is an arrow function, which you can see uses an arrow (`=>`). By adding the arrow, you can remove a ton of other pieces from the function, thus making it more concise and easy to read/write. Here are the steps:
1. remove the function keyword
2. remove the parentheses
3. remove the opening and closing curly braces
4. remove the return keyword
5. remove the semicolon
6. add an arrow (`=>`) between the parameter list and the function body

Of course, it's not always that easy, and depending on how the function is expressed, the look of an arrow function might change a bit.

##### ARROW FUNCTION STORED IN A VARIABLE
```
const greet = name => `Hello, ${name}!`;
greet('Kara');
>> Hello, Kara!
```

##### ARROW FUNCTION WITH MORE THAN ONE PARAMETER OR ZERO PARAMETERS
This will require wrapping the parameters (even zero parameters) in parenthesis.

Zero parameters:
```
const greet = () => 'Hello, human!';
greet();
>> Hello, human!
```

Multiple parameters:
```
const greet = (greeting, name) => `${greeting}, ${name}!`;
greet('Hey', 'Kara');
>> Hey, Kara!
```

##### ARROW FUNCTIONS AND BLOCK BODY SYNTAX
You'd need to use block body syntax if your arrow function requires more than one line of code. These require curly braces to wrap the function body, and you need to add a return statement to actually return something from the function.
```
const upperizedNames = ['Kara', 'Morgan', 'Cameron'].map( name => {
  name = name.toUpperCase();
  return `${name} has ${name.length} characters in their name`;
});
>> KARA has 4 characters in their name.
>> MORGAN has 6 characters in their name.
>> CAMERON has 7 characters in their name.
```

The arrow function also had an entire section dedicated to `this`, but my notes are sooooooo long, I'm not going to go through them all here. Instead, here's the key takeaway: **With standard functions, `this` is a dynamic value that’s different depending on how a function is called. But with arrow functions, the value of `this` depends on the where the function is located in the code.**

It can get quite complicated, but here are some resources to help explain `this` in both standard functions and arrow functions:
* [You Don't Know JS: this All Makes Sense Now!](https://github.com/getify/You-Dont-Know-JS/blob/master/this%20%26%20object%20prototypes/ch2.md)
* [freeCodeCamp: Learn ES6 The Dope Way Part II: Arrow functions and the ‘this’ keyword](https://medium.freecodecamp.org/learn-es6-the-dope-way-part-ii-arrow-functions-and-the-this-keyword-381ac7a32881)

### Day 17
The [Google Scholars/Udacity intermediate mobile web development course](https://blog.udacity.com/2017/10/udacity-google-announce-50000-new-scholarships.html) moved from arrow functions and into default function parameters. Defaults are useful because you can call a function without any parameters, and it will still work by giving you the default output. Here's an idea of how default function parameters would look before ES6:

```
function greet(name, greeting) {
  name = (typeof name !== 'undefined') ?  name : 'Student';
  greeting = (typeof greeting !== 'undefined') ?  greeting : 'Welcome';

  return `${greeting}, ${name}!`;
}

>> greet(); // Welcome, Student!
>> greet('James'); // Welcome, James!
>> greet('Richard', 'Howdy'); // Howdy, Richard!
```

As you can see, you'd set those defaults inside the function logic. It works, sure, but it's not the most beautiful or concise way to achieve default parameter functionality. Now, take a look at the same function with ES6:

```
function greet(name = 'Student', greeting = 'Welcome') {
  return `${greeting}, ${name}!`;
}

>> greet(); // Welcome, Student!
>> greet('James'); // Welcome, James!
>> greet('Richard', 'Howdy'); // Howdy, Richard!
```

This takes the default parameters out of the function logic and places them in the actual parameters themselves, *which makes so much sense.* And look how easy it is to read and understand! The other benefit to using default parameters the ES6 way is that you can combine it with destructuring via arrays and objects (see [day 13](https://github.com/karakarakaraff/100-days-of-code#day-17)).

##### DEFAULTS AND DESTRUCTURING ARRAYS
```
function createGrid([width = 5, height = 5] = []) {
  return `Generates a ${width} x ${height} grid`;
}

>> createGrid(); // Generates a 5 x 5 grid
>> createGrid([]); // Generates a 5 x 5 grid
>> createGrid([2]); // Generates a 2 x 5 grid
>> createGrid([2, 3]); // Generates a 2 x 3 grid
>> createGrid([undefined, 3]); // Generates a 5 x 3 grid
```

##### DEFAULTS AND DESTRUCTURING OBJECTS
```
function createSundae({scoops = 1, toppings = ['Hot Fudge']} = {}) {
  const scoopText = scoops === 1 ? 'scoop' : 'scoops';
  return `Your sundae has ${scoops} ${scoopText} with ${toppings.join(' and ')} toppings.`;
}

>> createSundae(); // Your sundae has 1 scoop with Hot Fudge toppings.
>> createSundae({}); // Your sundae has 1 scoop with Hot Fudge toppings.
>> createSundae({scoops: 2}); // Your sundae has 2 scoops with Hot Fudge toppings.
>> createSundae({scoops: 2, toppings: ['Sprinkles']}); // Your sundae has 2 scoops with Sprinkles toppings.
>> createSundae({toppings: ['Cookie Dough', 'Caramel', 'Cherry']}); // Your sundae has 1 scoop with Cookie Dough and Caramel and Cherry toppings.
```

The key to both functions above is also in the parameters, and it's the `= []` and `= {}` pieces. Without those, calling a simple `createGrid()` or `createSundae()` would throw an error because each function expects an array or object, respectively, to be called included in the argument, but since there's no argument at all, the function breaks. However, by including `= []`/`= {}`, it tells the function that if no argument is included, then the function can use an empty array/object as the default.

*NOTE: Since arrays are positionally based, you'd have to pass undefined to "skip" over the first argument (and accept the default) to get to the second argument, like this: `createSundae([undefined, ['Hot Fudge', 'Sprinkles', 'Caramel’]]);`. Unless you've got a strong reason to use array defaults with array destructuring, it is highly recommend go with object defaults and object destructuring instead!*

### Day 18
Today's hour of code was spent putting out yet another Alexa fire, this time with my Stranger Things Trivia skill. Some things to know:
1. I built this skill with Amazon's own official trivia skill sample
2. It has already undergone extensive testing and passed certification
3. It has been live for nearly three months

And yet I got an email from Amazon this morning saying that I need to take immediate action because "your skill endpoint does not consistently return a valid response to user requests." No hint at what the error might be that they found, just that there's at least one error somewhere. In addition to being vague, the message said that if I do not take action within five days, they'll suppress my skill. Nooooooo!

Long story short, I went back through the previous 48 hours of Cloudwatch logs connected to Stranger Things Trivia and took note of any errors that I found. To my relief, the errors were few and far between, but I did notice that it was the same error over and over again, just in two different ways:

`TypeError: Cannot assign to read only property '144' of string 'QUESTIONS.45.In season two, Eleven's mom mumbles several things in a loop. What is <emphasis level="strong">not</emphasis> included in that loop?'`

`TypeError: Cannot assign to read only property '44' of string 'QUESTIONS.4.What is Dr. Brenner's first name?'`

In action, whenever the game would pull one of these two questions, it would play just fine until the user got to this question, then error out and quit completely. Nothing in my code had changed since the skill went live, and those questions had worked just fine in the past, so what went wrong?

This is an error I have never seen before in my life, and as much as I traced it through my code, I had no idea what was causing it or why. A thorough search on the Amazon Developer forums turned up nothing, and a Google search sent me on a bit of a wild goose chase but did not result in finding any answers specific to Node.js-based Alexa skills.

Finally, it occurred to me that I should check the issues on the [official trivia skill sample repo](https://github.com/alexa/skill-sample-nodejs-trivia). I've personally worked with this repo and even submitted a pull request to fix several of the errors I previously ran into, so I knew people were active on there. And sure enough, I was not alone!

Here's issue 39: ["errorMessage": "Cannot assign to read only property '74' of string 'QUESTIONS.1.The 1964 classic Rudolph The Red Nosed Reindeer was filmed in.'"](https://github.com/alexa/skill-sample-nodejs-trivia/issues/39)

And inside that issue is a link to someone else who had the same problem in issue 14: ["errorMessage": "Cannot assign to read only property"](https://github.com/alexa/skill-sample-nodejs-trivia/issues/14)

And, finally, right there in issue 14 is [this answer by mikeybyker](https://github.com/alexa/skill-sample-nodejs-trivia/issues/14#issuecomment-312594753):

> "The problem you are seeing there is down to the translation/i18n library messing up the questions. ... TL;DR change to something like:

`var translatedQuestions = this.t("QUESTIONS", {keySeparator: '#'});`

The fix is in adding `{keySeparator: '#'}`, simple as that. You can see that change in action on my own commit here: [Fixed errors with translation/i18n library](https://github.com/karakarakaraff/alexa-skills/commit/32a9f91ed88362540059ae02a8ce670949641a79)

Now here's the thing: I still don't know what happened to make my skill suddenly have these errors where they didn't exist before. Additionally, I have no idea what the translation/i18n library is or how the keySeparator comes into play. What I *do* know is that this fixed all the problems, and my skill is back to being error-free (for now, maybe?). This was definitely a lesson in being able to hunt down an answer and put a fix into production regardless of fully understanding the ins and outs, which is just as valuable as any other lesson. Now, if only Amazon would fix this in their code and merge [my **approved** pull request](https://github.com/alexa/skill-sample-nodejs-trivia/pulls) (and the pull requests of others), then we'd stop seeing the same issues showing up over and over in Alexa trivia skills.

### Day 19
Today's hour will be dedicated not necessarily to code but instead to a Women In Tech meetup. This particular meetup is held once every quarter, and the last one was my first big networking meetup ever! I connected with some really awesome women who have since offered a lot of advice and encouragement as I embark on my career change into the tech industry, plus I got a much-needed boost of confidence by listening to successful women explain how they got into tech. It made me realize that, because tech is such a relatively new and booming industry, there is no one right way to get your foot in the door.

Here's the description for this quarter's focus:

> The focus of this quarter’s gathering will be on how to land your next job. We will talk through resumes, interviewing skills, networking tips, and ways to build your personal brand. We want to encourage you to be bold, own a confident mindset, and apply to that next role, reach out for a coffee date, and work towards your most ambitious career goals.

The reason I'm including this in my 100 Days of Code challenge is because one of my goals of this challenge is to get a job before the end of the 100 days. With each networking event, each hour spent coding, each Sunday meetup with my fellow 100-dayers, I feel more and more confident that I'll achieve every goal on my list.

Btw, I haven't included that goal list anywhere else in this log, so I'll stick it here:
1. Complete the [Google Scholars/Udacity intermediate mobile web development course](https://blog.udacity.com/2017/10/udacity-google-announce-50000-new-scholarships.html)
2. Rebuild my Wordpress portfolio website as a static website and host it on GitHub Pages
3. Rebuild my BFF's Wordpress photography website as a static website and host it on GitHub Pages
4. Apply, interview, network -- make that career change!
5. If there's extra time, do one or more of the following:
  * Daily JavaScript challenges on CodeWars
  * Rebuild one of my Bloc projects with React (previously built in Angular)
  * Go through the Node.js and Express.js lessons on freeCodeCamp
  * Take the Harvard CS50 Intro to Computer Science course on EdX

### Day 20
Back to the [Google Scholars/Udacity intermediate mobile web development course](https://blog.udacity.com/2017/10/udacity-google-announce-50000-new-scholarships.html) and ES6! Today I learned all about how ES6 handles classes.

*NOTE:* In other languages, **classes** are used to create objects and provide inheritance. However, JavaScript is NOT a class-based language, so it uses **functions** to create objects, and it links objects together (so they can inherit data and functionality) through prototypal inheritance. The addition of keywords like `class`, `super` and `extends` in ES6 does NOT change the underlying functionality of JavaScript.

Here's what a class looks like pre-ES6:
```
// create the class
function Plane(numEngines) {
  this.numEngines = numEngines;
  this.enginesActive = false;
}

// methods "inherited" by all instances
Plane.prototype.startEngines = function () {
  console.log('starting engines...');
  this.enginesActive = true;
};

const bigPlane = new Plane(4);
bigPlane.startEngines();
```

And here's what a class looks like post-ES6:
```
class Plane {
  // create the class
  constructor(numEngines) {
    this.numEngines = numEngines;
    this.enginesActive = false;
  }

  // methods "inherited" by all instances
  startEngines() {
    console.log('starting engines…');
    this.enginesActive = true;
  }
}

const bigPlane = new Plane(4);
bigPlane.startEngines();
```

Generally speaking, the steps for converting a class from ES5 to ES6 goes like this:
1. First, write a function using the `class` keyword.
2. Everything inside the ES5 constructor function is placed inside a method with the name `constructor`, which is placed inside the `class` function. **This constructor method will automatically run when a new object is constructed from this class.** If any data is needed to create the object, it needs to be included as an argument to `constructor()`.
3. All of the methods from the ES5 prototype function are placed inside the `class` as well, keeping everything in one nice, simple package.

In addition to classes, I also learned about static methods, subclasses, and the `super` and `extends` keywords. I have tons of notes on those that I won't copy here, but I will make note of the overall pros and cons to using classes in JavaScript:

**Benefits of classes**
1. Less setup: There's a lot less code that you need to write to create a function
2. Clearly defined constructor function: Inside the class definition, you can clearly specify the constructor function.
3. Everything's contained: All code that's needed for the class is contained in the class declaration. Instead of having the constructor function in one place, then adding methods to the prototype one-by-one, you can do everything all at once!

**Things to look out for when using classes**
1. `class` is not magic: The class keyword brings with it a lot of mental constructs from other, class-based languages. It doesn't magically add this functionality to JavaScript classes.
2. `class` is a mirage over prototypal inheritance: Under the hood, a JavaScript class just uses prototypal inheritance, the same as always.
3. Using classes requires the use of `new`: When creating a new instance of a JavaScript class, **the new keyword must be used.** For example,
```
class Toy {
   ...
}

const myToy1 = new Toy();
```

My thoughts on classes are that I've still never actually used them (I'm pretty sure of that, anyway), but I can totally see how and why they would be used. Also, I did some reading specifically on ES6 static methods within classes, and I found a few discussions linking static methods, and thus classes, with React. This is exciting, especially because building a full web app from scratch using React is on my to-do list, so that means I'll finally have a real reason for using JavaScript classes!

### Day 21
More of the [Google Scholars/Udacity intermediate mobile web development course](https://blog.udacity.com/2017/10/udacity-google-announce-50000-new-scholarships.html) and ES6! It looks like I'll be here for about another week, then I'll have the course finished. Woo hoo!

Anyway, I moved into the lesson on ES6 built-ins. This is going to include symbols, sets, maps, promises, proxies and generators. The two things I'm most excited about are symbols and promises, just because I've come across them in the wild and have had very little understanding of them up to this point. However, I can now say that symbols are in my knowledge base because that's what I studied today.

Some notes on symbols:

Symbols are a **new primitive data** type available to us in JavaScript. A symbol is a unique and immutable data type that is often used to identify object properties.

**Why use symbols?** Image there’s an empty bowl. You put an apple, an orange and a banana in the bowl. Identifying which fruit is the apple, which is the orange, which is the banana is simple. But now, you put a second banana in the bowl. If your program requests a banana from the bowl, how is it supposed to know which banana? You could identify the bananas as `banana1` and `banana2`, but what happens if you add more bananas to the bowl? Is it really a good idea to have `banana10493041984`? With the addition of symbols in ES6, there is a solution for this problem.

To create a symbol, you write `Symbol()` with an optional string as its description.

```
const sym1 = Symbol('apple');
console.log(sym1);
Symbol(apple)
```

This will create a unique symbol and store it in `sym1`. The description `"apple"` is just a way to describe the symbol, but it can’t be used to access the symbol itself.

To drive home how this works, if you compare two symbols with the same description ...

```
const sym2 = Symbol('banana');
const sym3 = Symbol('banana');
console.log(sym2 === sym3);
>> false
```

... then the result is **false** because *the description is only used to describe the symbol.* It’s not used as part of the symbol itself — each time, a new symbol is created regardless of the description.

Going back to the fruit bowl you previously imagined, here’s the code to represent that bowl and the fruit:

```
const bowl = {
  'apple': { color: 'red', weight: 136.078 },
  'banana': { color: 'yellow', weight: 183.15 },
  'orange': { color: 'orange', weight: 170.097 }
};
```

The bowl contains fruit, which are objects that are properties of the bowl. But, we run into a problem when the second banana gets added.

```
const bowl = {
  'apple': { color: 'red', weight: 136.078 },
  'banana': { color: 'yellow', weight: 183.151 },
  'orange': { color: 'orange', weight: 170.097 },
  'banana': { color: 'yellow', weight: 176.845 }
};

console.log(bowl);
>> Object {apple: Object, banana: Object, orange: Object}
```

Instead of adding another banana to the bowl, our previous banana is overwritten by the new banana being added to the bowl. To fix this problem, we can use symbols.

```
const bowl = {
  [Symbol('apple')]: { color: 'red', weight: 136.078 },
  [Symbol('banana')]: { color: 'yellow', weight: 183.15 },
  [Symbol('orange')]: { color: 'orange', weight: 170.097 },
  [Symbol('banana')]: { color: 'yellow', weight: 176.845 }
};

console.log(bowl);
>> Object {Symbol(apple): Object, Symbol(banana): Object, Symbol(orange): Object, Symbol(banana): Object}
```

**By changing the bowl’s properties to use symbols, each property is a unique Symbol and the first banana doesn’t get overwritten by the second banana.**

My thoughts on this are that symbols help make the code more readable to humans and cause it to act more like a real-world situation. When you're working with a bunch of data, you want to be able to access/read/parse/interpret/etc. that data in all different kinds of ways, and adding symbols to the mix gives you one more way to do so.

A couple other things that were packaged in the lesson on symbols were the iterable protocol and the iterator protocol, which use `[Symbol.iterator]` and `.next()`. These explain another use case for symbols, but the subject was kind of glossed over, so I'll have to some more reading on it. Here's an example, though:

```
const digits = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
const arrayIterator = digits[Symbol.iterator]();

console.log(arrayIterator.next());
console.log(arrayIterator.next());
console.log(arrayIterator.next());
>> Object {value: 0, done: false}
>> Object {value: 1, done: false}
>> Object {value: 2, done: false}
```

### Day 22
Today's hour will be dedicated to meeting up with my fellow #100DaysOfCode challengers (same as [day 15](https://github.com/karakarakaraff/100-days-of-code#day-15)). I'm excited because more people can make it this week, plus everyone has made a ton of progress in just the last week, so we can all see and share that progress. One thing I know for sure that we'll be covering is publishing our projects with GitHub Pages. I didn't even know you could publish multiple projects via GitHub Pages until it came up in coding conversation throughout the week -- I just knew about using GitHub Pages for portfolio sites named after your username, which limits you to publishing only one project -- so making it work and seeing it in action will be cool. I only have one static site project in my repository right now, which is being hosted on Netlify, and although Netlify is fantastic, I'd like to publish my project through GitHub just to keep everything together.

Another thing I'm sure we'll talk about today is the career change aspect. We all have the goal of moving out of the newspaper industry and into the tech industry, so I think it would be great to brainstorm how the skills we all have now can translate to other industries, thus setting us apart from other applicants. Of course, not everyone is at the application stage yet, but one way to get there is through a bootcamp, so we'll talk about that, too. Of everyone in the challenge, three of us have gone to three different bootcamps, so we bring a wealth of knowledge and experience about our prospective bootcamps. I hope that's really helpful for the others as they decide if/when they want to commit to a bootcamp.

Overall, I'm just excited to get together with others to talk code and share encouragement. And to drink a fancy chai tea latte!

### Day 23
At the meeting yesterday, I was sharing what I had been learning with ES6 when one of the other #100DaysOfCode challengers also shared his learning experience with ES6. For him, he had found it most useful when building a web app with React, and as he showed us his code, I could see the ES6 syntax throughout, which was pretty cool. On [day 20](), I mentioned in my log that I had found some online forums where developers were discussing use cases for ES6, and React kept coming up over and over. So now, with what I found before and what I saw yesterday, I was more curious than ever today! Here are some resources I read as I researched the topic:
* [BabelJS: React on ES6+](https://babeljs.io/blog/2015/06/07/react-on-es6-plus)
* [Fullstack React: React.createClass vs ES6 class components](https://www.fullstackreact.com/articles/react-create-class-vs-es6-class-components/)
* [Hackernoon: Writing Clean and Concise React Components by Making Full Use of ES6/7 Features and the Container-Component Pattern](https://hackernoon.com/writing-clean-and-concise-react-components-by-making-full-use-of-es6-7-features-and-the-container-4ba0473b7b01)
* [React Docs: Components and props](https://reactjs.org/docs/components-and-props.html)
* [React Docs: React without ES6](https://reactjs.org/docs/react-without-es6.html) -- this especially shows the difference ES6 can make

##### BIG NEWS ALERT
In other news, one of my main goals of this challenge has been completed: I GOT A JOB OFFER! I'm not sure if doing this challenge is one of the things that made me stand out to my future employer, but regardless, I somehow got across the message that I am passionate about coding and continuous learning and profession improvement. I am so excited for this opportunity, I can barely contain myself! The crazy thing is that today was also my official last day with [Bloc](https://www.bloc.io) (even though I finished all of my projects weeks ago, I still kept meeting with my mentor so I could continue to pick his brain). So, **on the same day I graduated, I got a job.** Heck yeah!

If anyone reading this has any questions about Bloc, please don't hesitate to reach out to me via DM on Twitter. (I'm here: [karakarakaraff](https://twitter.com/karakarakaraff)). I'm more than happy to share the pros and cons of the bootcamp, and if you decide to sign up, I can give you a referral code for $500 off tuition. Regardless if you go with Bloc or any bootcamp or no bootcamp at all, I sincerely hope you dedicate yourself to learning how to code (the #100DaysOfCode challenge is a good start, hint hint!) because this new career of mine is going to be truly life changing. Thank you to everyone who supported me and shared advice and encouragement!
