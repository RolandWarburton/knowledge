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
4. Create file inside chrome directory `touch userChrome.css`

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

5. Navigate to `about:config`
6. Set `toolkit.legacyUserProfileCustomizations.stylesheets = true`
7. restart firefox.
8. Profit!
