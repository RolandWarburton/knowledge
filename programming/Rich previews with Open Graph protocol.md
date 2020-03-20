# Rich previews with Open Graph protocol
Rich previews are a great way to make a website appear more polished and professional by adding display cards to 'social graphs'. 
This protocol is used by facebook, twitter, apple messenger, and other messenger/social platforms.

# Rich previews in different apps
A rich preview looks like this in messenger.

![Rich preview in messenger app](https://i.imgur.com/RKXh8v0.png)

A rich preview looks like this on twitter.

![Rich preview in twitter app](https://i.imgur.com/iEhb3C6.png)

# Enabling rich previews
Simply add meta tags to your head tag.

## og:image
Make sure that the og:image is pointing to a static url on your webpage. 
Dynamic content that needs to be client side rendered will not work

```html
<head>
	<meta charset="utf-8" />
	<meta http-equiv="X-UA-Compatible" content="IE=edge" />
	<meta name="viewport" content="width=device-width, initial-scale=1" />
	<meta name="description" content="Roland's Blog and WWWebsite" />

    <!-- The 4 basic metadata tags -->
    <!-- However if you only want the image then og:image will work on its own -->
	<meta property="og:title" content="Roland's Blog and WWWebsite" />
	<meta property="og:type" content="website" />
	<meta property="og:image" content="https://www.rolandw.dev/media/favicon.png" />
	<meta property="og:url" content="https://www.rolandw.dev/" />

    <!-- Twitter specific tags. not required for other apps -->
	<meta name="twitter:card" content="summary_large_image"/>
	<meta name="twitter:site" content="@RolandIRL" />
	<link rel="icon" type="image/png" href="/media/favicon.ico" />
	<title>My Site</title>
	<link rel="stylesheet" type="text/css" href="/app.css">
</head>
```

# Debugging previews
Facebook will cache your og:image. you can go [here](https://developers.facebook.com/tools/debug/) to ask facebook to crawl your site again.

Twitter previews will only work in tweets and not in direct messages. You can debug to see if your meta tags are working [here](https://cards-dev.twitter.com/validator).