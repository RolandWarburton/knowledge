# Resources for inspiration

* [eti desktops collection](https://eti.tf/desktops/)

## Firefox

### Firefox user styles

#### Resources

* [userchrome](https://www.userchrome.org/how-create-userchrome-css.html)
* [floopymoose](http://www.floppymoose.com/)

#### Instructions

1. First navigate to `about:support`
2. Click *open user directory*
3. Create a chrome dir `mkdir chrome`
4. Create file inside chrome directory `touch chrome/userChrome.css` The permissions for the file should be `-rw-r--r--` or `644`

Heres an example to check that its working (will make your tabs red)

```css
/* userChrome.css examples*/

/* Change the tabs to red*/
.tabbrowser-tab {
	background-color: red;
}

/* Make tabs smaller */
#tabbrowser-arrowscrollbox {
	--tab-min-width: 20px;
}
```

Heres an example that i use a lot to remove firefox tab scrolling and fix tab overflow issues.

```css
/* Prevent scrolling */
/* Tab width */
.tabbrowser-tab[fadein]:not([pinned]) {
	min-width: 10px !important;
}

.tab-close-button {
	display: none;
}

* .tab-content {
	overflow: hidden !important;
	padding: 0px 3px !important;
}

tab-icon-image:not([pinned]) {
	margin-inline-end: 0px !important;
}

tab-icon-image {
	margin-inline: 6px !important;
	/* margin: 0px !important; */
}

.tab-icon-image:not([pinned]) {
	margin-inline-end: 6px !important;
}
```

5. Navigate to `about:config`
6. Set `toolkit.legacyUserProfileCustomizations.stylesheets = true`
7. restart firefox.
8. Profit!
