# Github

## Git commands

### Merge Two Repos Into One

Beware this will not create a subfolder for RepoB in RepoA, it will copy everything over to the base die

If you want to merge project-b into project-a:

```bash
cd path/to/project-a
git remote add project-a path/to/project-b
git fetch project-b --tags
git merge --allow-unrelated-histories project-b/master
git remote remove project-b
```

### Enable SSH for repo

I have instructions for how to do this on linux [here](https://rolandw.dev/Notes/Linux/GitonLinux/), i have not tried it on windows so YMMV.

### Untracking a file

In the situation that you originally were tracking/committed a file and no longer want to track it (a common example is .env) you need to tell github to untrack it, as well as add it to the .gitignore file.

Run this command to cause github to "forget" about the file. Specify the filepath from the .git folder.

```none
# Untrack the file in app/.env
git rm --cached app/.env
```

Then add it to the `.gitignore`.

```none
.env
# or
app/.env
# or
*.env
```

## Github webhooks

How to set up and verify webhooks from github on NodeJS/Express

> in soviet russia, server polls you!

A webhook is like an API except in reverse. Instead of polling a server for data, the server polls you!

A GH webhook is set up on a "per project" basis which means that privileges and secrets shared over that channel will only be about that specific repo.

Create a new hook for your repo by going to **settings -> Webhooks** and then entering a the required details.

* **Payload URL**. Your domain name for your project. Github will POST to this when something happens.
* **Secret**. A shared secret to verify the payloads authenticity. Pick something 20 chars long. use `ruby -rsecurerandom -e 'puts SecureRandom.hex(20)'` to pick a strong random secret.
* **Content type**. Select application/json for this example. You can use x-www-forms if you are supporting that on your application.
* **Let me select individual events**. You can either select specific events to trigger the webhook with. Or tick *"send me everything"*.

Make sure you click the update button after making any changes.

![webhook page screenshot](https://i.imgur.com/S0BjdFD.png)

Now lets create some code for our app to receive the webhook. As you can see from the screenshot github will POST to [http://0x4.host:8080/](http://0x4.host:8080/) when something happens, you can also POST to other endpoints like [http://0x4.host:8080/hooks/github](http://0x4.host:8080/hooks/github).

I am using Express for this and need to do 2 things to make it work

1. Define a route to accept the hook from
2. Write some middleware to verify the payload

#### Step 1 - Accepting the hook

Lets define a new route for POST requests on `"/"` that GH can use.

Also we need a way of extracting the body of the request. For this we will use the Body-Parser middleware

```javascript
const bodyParser = require("body-parser");

// Lets decode JSON requests using application/json
// This is if you are using "application/json" on the github hooks page
const jsonParser = bodyParser.json()

// Otherwise if you are using x-www use the following code snippet
const urlencodedParser = bodyParser.urlencoded({ extended: false });

// replace "urlencodedParser" middleware with "jsonParser" for your chosen content type
app.post("/", [urlencodedParser], (req, res) => {
	// print the request body!
	console.log(req.body)

	// reply to github to prevent a timeout
	res.status(200).json({
		success: true,
	});
});
```

You can now test the webhook by sending a test delivery. You will see it show up under "Recent Deliveries"

![https://i.imgur.com/Fa7bNVV.png](https://i.imgur.com/Fa7bNVV.png)

#### Step 2 - Verifying the payload

**But Wait!** How do we know if the data we are receiving is legit?

In short we can use the crypto library and pre established secrets exchanged between our app and github to verify the authenticity of the payload.

Lets use some middleware curtesy of Digital Ocean who wrote an article [here](https://www.digitalocean.com/community/tutorials/how-to-use-node-js-and-github-webhooks-to-keep-remote-projects-in-sync) about gh webhooks.

Make sure to define `process.env.GITHUB_SECRET` in your environment variables folder. Its ideal not to hardcode the value directly as the GH secret is a password to your webhooks.

```javascript
// load in your environment variables
const dotenv = require("dotenv").config();

const secret = process.env.GITHUB_SECRET;
const sigHeaderName = "x-Hub-Signature";

function verifyGithubPayload(req, res, next) {
	const payload = JSON.stringify(req.body);
	let sig =
		"sha1=" +
		crypto
			.createHmac("sha1", secret)
			.update(payload.toString())
			.digest("hex");

	if (req.headers["x-hub-signature"] == sig) {
		console.log("all good");
		// do something
	} else {
		console.log("all bad");
		// tell github something went wrong
		res.status(500).json({ success: false });
	}
	return next();
}
```

Then lets modify the app route to to use this new middleware.

```javascript
// add verifyGithubPayload to the middleware
app.post("/", [verifyGithubPayload, urlencodedParser], (req, res) => {
	// do some stuff
	res.status(200).json({})
}
```

You should now see "all good" or "all bad" printed to the console. If you get an "all bad" try using the ruby SecureRandom function to generate a new secret and make sure you update the github webhook with the new secret and that they match.
