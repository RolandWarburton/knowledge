# Fetching private files on NodeJS

How to fetch a file from a private repo on github

1. Create a personal access token to your github account
![Settings](https://i.imgur.com/JLyKjc2.png)
![Tokens list](https://i.imgur.com/OIeSSLs.png)
![Select Scope](https://i.imgur.com/VFujIC0.png)

2. Your token is 40 characters long. To perform a fetch request on nodejs you should fetch the webpage like this
```javascript
fetch('https://TOKEN@raw.githubusercontent.com/USER/REPO/BRANCH/FILENAME.EXT')
	.then(res => res.text())
	.then(body => console.log(body));
```
