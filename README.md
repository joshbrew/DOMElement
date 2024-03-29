## DOMElement.js
//By Joshua Brewster (AGPL v3.0)

![fragelement-status](https://img.shields.io/npm/v/fragelement.svg) 
![fragelement-downloads](https://img.shields.io/npm/dt/fragelement.svg)
![fragelem-size](https://img.shields.io/github/size/brainsatplay/DOMElement/DOMElement.js)
![fragelem-l](https://img.shields.io/npm/l/fragelement)

`npm i fragelement`

[fragelement-based webcomponent app example](https://github.com/moothyknight/esbuild_base_webcomponents)

This is a simple wrapper for the native web components with template fragments in javascript.

### DOMElement 
This class extends the HTMLElement class and implements a template fragment rendering method:


Extend it like:
```js
class CustomElement extends DOMElement { 
  props={defaultprop:1}:
  useShadow=false; //shadow DOM root? Allows scoped stylesheets, uses 'open' mode so it's further programmable from script.
  styles=undefined //you can include a stylesheet template string here to trigger the shadow dom for scoped style sheets automatically, prepended to the template or inserted into head if its in the template. Triggers an html updated if already rendered.

  //The template can be an imported html file when building in node.js for a better experience
  template=(self,props)=>{return `<div>New Element: ${JSON.stringify(props)}</div>`} 
         
  oncreate=undefined, //(self,props)=>{} when the node is created e.g. setting up buttons (props) => {}
  ondelete=undefined, //(self,props)=>{} when the node is deleted, e.g. cleaning up events (props) => {}
  onresize=undefined, //(self,props => {} run on window.onresize event 
  onchanged=undefined, //if props change, e.g. re-render? (self,props) => {}. Using past tense to not conflict with built in onchange event in most elements
  renderonchanged=false //(self,props) => {}  //true or a function fired after rerendering, will auto trigger rerenders when props changed
  
}

CustomElement.addElement('custom-element'); //adds the custom class to the window's built-in customElementRegistry before instantiating the new element
//if you don't provide a tag, the element will be registered as the classname plus a dash like 'customelement-'
```
 
where all that needs to be set is the template variable.

Then this *should* work in html:

```html
<customelement- props='{"a":"1","b":"2","c":"3"}'><customelement- /> 
```
Can define props, onresize, onchanged, oncreate, ondelete, and even template just like other stock html functions. The prop onrenderchange if included will trigger the render function, and you can supply a function to fire after the fact.

```js
let elm = document.querySelector('customelement-');

elm.addEventListener('resized',(e) => {
  console.log(e.target.props);
});

elm.addEventListener('changed',(e) => {
  console.log(e.target.props);
  //e.g. elm.render() //re-render the element
});

elm.addEventListener('deleted',(e) => {
  console.log(e.target.props);
});

elm.addEventListener('rendered',(e) => {
  console.log(e.target.props);
});

```

Custom elements have to have a '-' in the names for whatever reason, they are auto added on the end of the class name if none specified in addElement


### Styles:

Set the `styles` property on the element to a template string of your style sheet contents and it will be prepended with the shadow root. You can set it on attributes on page init or in js and it will work. Otherwise if you want a shadow DOM root to use scoped stylesheets, set `elm.useShadow = true`.

### Even more fun:

```html

<body>
    <script>
        function foo(x=123){ console.log(x); return 1; }
    </script>


    <customelement- 
      props='{"a":"1"}' 
      teststr="abc" 
      testvalue="123" 
      eval_foo="foo(456)" 
      eval_boo="(inp)=>{console.log('this is probably a dumb feature', inp); return 2; }">
    </customelement->

    <script>
        let elem = document.getElementsByTagName('customelement-')[0];
        console.log(Array.from(elem.attributes));
        console.log(elem.boo('but hello world'));
        console.log(elem.props);
        console.log(elem.testvalue);
    </script>
</body>
```

### All methods in DOMElement:

```js

let elm = new DOMElement()

document.body.appendChild(elm) //add to body

elm

   .addElement(tag=this.tag, cls=this, extend=undefined) //static method to add this class to the dom, or some other class, input a tag or by default it uses the class name with a '-' on the end
   
   .render(props=this.props) //render the element with the props
   
   .delete() //delete the element and call the ondelete callback
   
   .state
   
          .setState(updateObj={}) //set state and trigger subscriptions for key:value pairs
          
          .subscribeTrigger(key,onchanged=(res)=>{}) //subscribe a function when a key:value pair is updated by setState. returns an int
          
          .unsubscribeTrigger(key,sub) //unsubscribe using the int returned by subscribeTrigger
          
          .subscribeTriggerOnce(key,onchanged=(res)=>{}) //subscribe a function to run once when a key:value pair is changed by setState({})
          
          .data //data object in state
          
   //internal functions
   
   .attributeChangedCallback(name, old, val) //when an observed attribute is updated run this function. Arbitrary attributes can be defined in the html tag, 
   //if 'eval_' is attached you can even add custom functions that are available on the element. E.g. elm.custom() (e.g. eval_custom='console.log('hello world')').
   //Any arbitrary attributes can be get/set at any time as well, they will set the same key:value pair on props for conditional rendering purposes in the template.
   
   .connectedCallback() //runs when the element is connected to the DOM. 
          

```
