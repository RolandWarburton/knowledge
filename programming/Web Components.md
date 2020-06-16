# Web components

Web components are a modern way of writing components like you would in many front end javascript frameworks. But with the benefit of being baked into vanilla javascript.

Web Components are relatively well supported across the web and while i have not dived too far into using them i definitely plan to in the future for some web projects. They are great for when you need to reuse a lot of HTML but change the content inside.

An important note that i found early on was that **Web Components are NOT HTML imports**. HTML imports are a separate (now deprecated) and similar way of achieving what web components do. except without the use of the three technologies that make up web components, which i have listed below.

## Web components are made up of these things

1. Custom Elements
   1. Think of a component in react. This is what a web component is
2. Shadow Dom
   1. Shadow DOMs help you build components
   2. You can think of shadow DOM as a scoped subtree inside your element.
   3. A shadow DOM is like talking a small DOM and putting it inside of a single element
3. HTML Templates
   1. A template is a skeleton that has content injected into it when it is loaded into the Shadow DOM
   2. A template is a template literal in javascript
   3. You can pass arguments (think of props) to a template. And slots with written content to render in your template

## Using a custom element

* Create a custom HTML tag like ```<header>``` or ```<footer>```
* You can also extend existing HTML tags of make them from scratch

```javascript
class Header extends HTMLElement {...}
window.customElements.define("header", headerClass)

// now we can use <header> element in our HTML
```

## Custom elements life cycle methods

The life cycle works much like the life cycle of a component in react
Elements can take custom attributes (think of props).

* constructor() works like a normal constructor
* connectedCallback() called every time the element is inserted into the DOM
* disconnectedCallback() opposite of connectedCallback
* attributeChangedCallback(attName, oldVal, newVal)
  * called when an attribute is added, removed, updated, or replaced

## Shadow DOM

* Used for self contained components
* encapsulates styles and markup
  * separates its own styles from the global css of the webpage

```html
<!-- Create an "open" element that 
we can interact with using dev tools -->
element.attachShadow({mode: open})
```

## HTML templates

* Used to define encapsulated markup for a web components
* Includes both HTML and CSS in the template
* The template can be dynamic with the use of "slots"

## A simple web component

```html
<!-- Passing in a prop to the component -->
<user-card name="Roland"></user-card>
```

```javascript
// Define the component as a class called UserCard
class UserCard extends HTMLElement {
  constructor() {
    // call the constructor of HTMLElement too with super
    super();
    // do something with the props
    this.innerHTML = `Hello ${this.getAttribute("name")}`
  }
}

// Associate the class UserCard with the element name
// that we want to use in the DOM, ("user-card")
window.customElements.define("user-card", UserCard)
```

![Simple Web Component](/media/SimpleComponent.png)

## Styling web components and the Shadow DOM

The above element is not using the shadow DOM. It will inherit global styles and does not have access to the shadow DOMs life cycle methods. Using the shadow DOM in this example will separate it from the global styles so that the component will not inherit any styles and will have its own subtree in the DOM to manipulate.

Note that i have noticed that the Shadow DOM does not play nicely with the [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) Plugin on VS Code. So if doesn't work there, try experimenting with it on [JSFiddle](https://jsfiddle.net/RolandFiddles/zs2aeLv5/6/).

Also remember to put a semicolon at the end of the template literal! This caused as lot of headaches.

```html
<!-- this style only applies to the vanilla h3 -->
<style>
  h3 {
    color: green;
  }
</style>
<h3>hello world</h3>
<!-- pass in a "name" as a prop to this web component -->
<user-card name="Roland"></user-card>
```

```javascript
// create a new div element in the DOM with its own styles
const template = document.createElement("template");
// the template is just a frame. the h3 tag stays empty
// content in the h3 is injected in the UserCard class
template.innerHTMl = `
<style>
  h3 {
    color: blue;
  }
</style>

<div class="myClass">
    <h3></h3>
</div>
`;

class UserCard extends HTMLElement {
  constructor() {
    // call the constructor of HTMLElement too
    super();

    // Create a shadow DOM
    this.attachShadow({mode: "open"});

    // Attach the template above to the shadow DOM
    this.shadowRoot.appendChild(template.content.cloneNode(true));

    // Inject some content from the props that we passed in
    this.shadowRoot.querySelector("h3")
    .innerText = this.getAttribute("name");
  }
}

window.customElements.define("user-card", UserCard);
```

![Shadow Dom](/media/ShadowDom.png)

## Multi attribute Web Components

You can pass multiple arguments to a component.

```html
<!-- pass in a "name" and an "avatar"  -->
<user-card
name="roland"
avatar="https://randomuser.me/api/portraits/men/1.jpg">
</user-card>
```

```javascript
// create a new div element in the DOM with its own styles
const template = document.createElement("template");

// the template is just a frame. the all its tags are empty
// content will be injected in later
template.innerHTML = `
<style>
h3 {
	color: blue;
}
</style>

<div class="user-card">
  <img />
  <h3></h3>
</div>
`;

class UserCard extends HTMLElement {
  constructor() {
  // Call the constructor of HTMLElement too
  super();

	// Create a shadow DOM
  this.attachShadow({mode: "open"});
  
  // Attach the template above to the shadow DOM
  this.shadowRoot.appendChild(template.content.cloneNode(true));
  
  // Inject some content from the props that we passed in
  this.shadowRoot.querySelector("h3")
  .innerText = this.getAttribute("name");
  
  // Inject some content for an image
  this.shadowRoot.querySelector("img").src = this.getAttribute("avatar");
  }

}

window.customElements.define("user-card", UserCard);

```

## Slots

Slots are content passed to the component inside the body. In this example Hello is the "slot".

Pass a slot into the template using the slot="slotName" HTML below.

```html
<user-card name="roland">
  <!-- This is a slot! -->
  <!-- A slot always begins with slot="" -->
  <!-- You can then use "message" as an ID in the template -->
  <div slot="message">Hello</div>
</user-card>
```

```javascript
const template = document.createElement("template");
template.innerHTML = `
<style>
  h3 {
    color: blue;
  }
</style>

<h1><slot name="message" /></h1>
<h3></h3>
`;


class UserCard extends HTMLElement {
  constructor() {
    super();

    // Create a shadow DOM
    this.attachShadow({mode: "open"});

    // Attach the template above to the shadow DOM
    this.shadowRoot.appendChild(template.content.cloneNode(true));

    // Inject some content from the props that we passed in
    this.shadowRoot.querySelector("h3")
    .innerText = this.getAttribute("name");
  }

}

window.customElements.define("user-card", UserCard);
```

## Event listeners - Do something when an element is mounted

```html
<user-card name="John Doe">
  <div slot="message">Hello</div>
</user-card>
```

```javascript
const template = document.createElement("template");
template.innerHTML = `
<style>
  h3 {
    color: blue;
  }
</style>

<h1 id="targetMe"><slot name="message" /></h1>
<h3></h3>
<button id="toggle">Hide</button>
`;


class UserCard extends HTMLElement {
  constructor() {
    super();

    this.myToggle = false

    // Create a shadow DOM
    this.attachShadow({
      mode: "open"
    });
    // Attach the template above to the shadow DOM
    this.shadowRoot.appendChild(
      template.content.cloneNode(true)
      );
    // Inject some content from the props that we passed in
    this.shadowRoot.querySelector("h3")
    .innerText = this.getAttribute("name");
  }

  myFunction() {
    this.myToggle = !this.myToggle

    const target = this.shadowRoot.querySelector("#targetMe")
    const button = this.shadowRoot.querySelector("#toggle")

    if(this.myToggle) {
    	target.style.display = "none"
      button.innerText = "Show"
    } else {
    	target.style.display = "block"
      button.innerText = "Hide"
    }
  }

  // Do something when mounted. Add an event listener in this case
  connectedCallback() {
    this.shadowRoot.querySelector("#toggle")
    .addEventListener("click", () => this.myFunction())
  }

  // Do something when dismounted. Remove the event listener
  disconnectedCallback() {
    this.shadowRoot.querySelector("#toggle").removeEventListener()
  }

}

window.customElements.define("user-card", UserCard);
```
