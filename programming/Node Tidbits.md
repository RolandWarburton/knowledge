# Node tidbits

> A small collection of node tricks that i have learnt.

##### Cleaning up old node servers
```
! List all servers running on port :8080
lsof -i :8080

! Kill all servers running on port :8080
kill -9 $(lsof -t -i:8080)
```

```
! Sample of a server running on node with express
COMMAND    PID   USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
node    132177 roland   19u  IPv6 4009851      0t0  TCP *:http-alt (LISTEN)
```

##### Using ENV
The NPM package [dotenv](https://www.npmjs.com/package/dotenv) allows for you to set static variables before you run your program inside a file. For example you could store a secret password or a path to the root directory to refer to from anywhere in your application.

**Make sure to but .env in .gitignore. Or better yet create a rule for all dotfiles as a general rule.**

```
! An example file structure
.
├── package.json
├── package-lock.json
├── index.js
└── .env
```

```
! .env

MYVAR=abc123
```

```
<!-- index.js -->

<!-- You can now require dotenv from anywhere and have access to its variables -->
require('dotenv').config()

module.exports = () => {
console.log(process.env.MYVAR) 
}
```