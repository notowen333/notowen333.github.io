---
layout: post
title:  "Road to a PS5 Part 1: Building a Node.js SMS Notification App"
date:   2021-07-04 22:00:00 -0500
categories: code
---

Part 1 in a two part series where I try to buy a PS5 for the list price.


### Background

I've wanted to get the new playstation console since it came out 8 months ago. I figured that the hype would calm down and I'd be able to get my hands on one without refreshing a page for hours. Obviously, this didn't work out and I've been catching myself clicking through the same few pages looking for a restock more than I'm proud of. So, I decided to put my skills to use and automate it.

### Core Libraries

The core functionality of the app uses Google's [puppeteer library](https://www.npmjs.com/package/puppeteer). With puppeteer, you can operate a [headless browser](https://en.wikipedia.org/wiki/Headless_browser#:~:text=A%20headless%20browser%20is%20a,interface%20or%20using%20network%20communication.) to programatically interact with the web.

The SMS notification functionality uses Twilio's SMS API through [Twilio's Node.js wrapper](https://www.npmjs.com/package/twilio). You'll need to make a Twilio account and purchase a fresh phone number to send yourself texts. This will cost you $1, but this is less than the $15 in free credit you'll get with a new account.

### My Idea

1. Load the web pages of PS5 consoles for sale.
2. Search for content that appears when the item is sold out.
3. If this content is missing, send myself a text with the URL of the available PS5.

### Aspects of a Node.js App That I Didn't Understand When I First Saw Them

If you've seen this kind of thing before, go ahead and skip to the next heading. I just wanted to include some key pieces of information that I learned while building this app.

#### Installing dependencies

When you're first starting out run `npm init` to initialze your `package.JSON` file. It will step you through some basic questions in the terminal. You want to populate this file so you can use other `npm` commands.

Then, whenever you add a new library to your project, run `npm install some_library --save`. By adding the `--save` paramater, it will automatically add this dependency (other code the project needs to run) to your `package.JSON`. Now, if someone else clones your code, when they run `npm install`, it will install these libraries onto their machine.

#### Imports vs Require

I've been writing javascript for my work, so I wanted to use the `import { foo, bar } from './file.js` syntax. But for a Node.js app, you want to use `module.exports/require` syntax. So, if you write a function in another file called `foo`, you would say `module.exports = {foo}` in this file. In the file where you use the function, we first say `const fooRef = require('/path/to/file')` and then `fooRef.foo(fooParams)` gives us the function we wrote.

#### Run Script

You can run `node somefile.js` to run a node file from the terminal. When packaging your app, it's best to point an npm command to the node command you want to start the app. So, by including

```JSON
"scripts": {
    "start": "node ./src/index.js"
  },
```

in my `package.JSON` file, `npm start` will launch my app.


### Thoughts about Puppeteer

I didn't do anything too fancy for this app. One important feature was waiting for the page to fully load before going about my search routine. There are a number of ways to do this, but I went with adding the `waitUntil` parameter when navigating to a page:

```javascript
await page.goto(url, { waitUntil: 'networkidle0' });
```

The other feature was to search HTML content. `page.content()` with a regular expression does the trick:

```javascript
let res = (await page.content()).search(regex);
```

Each page gets a regular expression for its particular sold out message—sometimes 'Unaivalable' sometimes 'Sold out', etc.

I was hoping to include Best Buy in search, but their website detects headless browsers and prevents the website from loading. I wasn't willing to put in the work to try to circumvent this security, but others have put a great deal of time into [it](https://github.com/paulirish/headless-cat-n-mouse).

### Automating SMS Messages with Twilio

I was really happy with how easy this turned out to be. I also learned about `.env` in the process.

Here's the full code to send yourself a text message from the app:

```javascript
const accountSid = process.env.TWILIO_ACCOUNT_SID;
const authToken = process.env.TWILIO_AUTH_TOKEN;
const client = require('twilio')(accountSid, authToken);
const message = process.argv[2]; // the message is passed as the third param

client.messages.create({
        body: message,
        from: process.env.TWILIO_PHONE_NUMBER,
        to: process.env.RECEIVING_PHONE_NUMBER, })
    .then(message => console.log(message.status));
```

The only thing I've changed from the example code on Twilio's website is to get the message to send from the command line. I was having trouble passing the message as a function parameter, so I used the built in `child_process` library to run `exec()`, which launches it's own terminal and runs a command. 

So, I just put the code above in its own file and used `exec(node sms.js ${message})` to trigger a variable message.

#### Setting up Twilio

After you make an account, you need to buy a fresh phone number. They give you $15 in free credit and the phone number will cost you $1. After you buy a phone number, I made a `.env` file with the following information:

```
TWILIO_ACCOUNT_SID = xxxx
TWILIO_AUTH_TOKEN = xxxx
TWILIO_PHONE_NUMBER = xxxx
RECEIVING_PHONE_NUMBER = xxxx
```

In this way, if I include `.env` in my `.gitignore`, I can publish my code on the internet without giving away my API code and personal phone number.

Then to load the `.env` information into my app, I just included `require('dotenv').config();` in the root file that gets called on start.

### Source Code and Next Step

You can check out the source code [here](https://github.com/notowen333/ps5check).

Stay tuned for part 2, where I will try to host my app on Google's cloud and run it every couple of minutes. Since the app is taking about 30 seconds to finish, I'm hoping I can stay in the free tier. We will see!
