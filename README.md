# styled-components-themer - JSON to CSS

This utility function allows you to quickly write styles in JavaScript object notation (JSON) which are compiled into CSS. It greatly improves on styled-components' theming capabilities by escaping the hassle of template literal syntax. It allows for greater code reuse and modular styling. It also contains a few shortcuts to make your job even easier and is designed to be modified to your specific needs.

Themer essentially outputs a string that contains the style properties to be inserted into a styled-component's string literal.

This is not a dependency package. It's just a file with some code.

Note: This was written to work with styled-components, but the output of the function can be compiled into CSS by most pre-processors.

## Installation
Simply copy themer.js anywhere into your project and import it where you see fit.

## Basic Usage
When you create a component with styled-components, call the themer function within the template literal and pass the theme prop to it:

```jsx
import styled   from 'styled-components'
import {themer} from '../utils/themer'

const Div = styled.div`${props => themer(props.theme)}`

export default Div
```

Then, write your styles as if they were a JavaScript object:

```js
const styleObject = {
    display: flex,
    flexDirection: column,
    alignItems: center,
    margin: '0 auto',
    height: '100%',
    width: '100%',
    position: absolute,
    top: 0,
    left: 0,
    boxSizing: borderBox,
}
```

Finally, pass your style object to the component's theme prop:

```jsx
<Div theme={styleObject} {...props} />
```

## Writing Styles in JSON
In the basic usage example, you'll see a plain JavaScript object. The object's properties should be familiar to you, as this is how inline styles are written for standard React components. Themer takes that concept and runs with it.

### Shortcuts
Here are a few ways to make writing styles in JSON even easier.

#### Numbers to Pixel Values
Just like when writing inline styles for React components, you can use a raw number value where a single pixel value is needed. Themer will automatically add the 'px' at the end. If the style needs multiple values, you should wrap them in quotes.

```js
const exampleStyle = {
    height: 100,
    margin: '0 auto'
}

// height: 100px;
// margin: 0 auto;
```

#### Mixins & the Spread Operator
With JavaScript objects, every style is potentially a mixin. Just use the ES6 spread operator to insert the properties of one object into another. Then, you can overwrite the properties as you see fit.

```js
const exampleMixin = {
    height: 50,
    backgroundColor: 'blue'
}

const exampleStyle = {
    ...exampleMixin,
    height: 100
}

// background-color: blue;
// height: 100px;
```

#### Common String Values as Variables
You may notice that in some of the examples above, some string values do not have quotes around them. That's because themer.js exports a set of variables that are direct translations from the variable name to a CSS value. These are provided for the most common string values, provided that their names don't clash with any other variable. If your IDE has some auto-import feature for used variables, you can focus on writing just the CSS without those pesky single quotes slowing you down.

#### Input Field Placeholders
Adjusting the appearance of placeholder text within a field is pretty simple:

```js
const exampleStyle = {
    placeholder: {
        color: black
    }
}
```

### CSS Pseudo-classes and Pseudo-objects
Hover states are easy - just write the hover property as an object that contains its own styles. Other pseudo-classes are just as easy. For pseudo-classes that require a parameter, just add the 'param' property to the object.

```js
const exampleStyle = {
    color: white,
    margin: '0 auto',
    hover: {
        color: black,
    },
    lastChild: {
        borderBottom: '1px solid #ccc'
    },
    nthChild: {
        param: '3n+3',
        backgroundColor: 'blue'
    },
    before: {
        content: '"\\2013\\0020"',
        height: 25,
    },
}
```
### Media Queries
To provide styling for a particular media query, you can define that media query string within the 'media' object near the bottom of the themer.js file.

Default configuration: The predefined breakpoints are: mobile, tablet, small (small desktop), large (large desktop), and print. This arrangement assumes that styles are written mobile-first. This means that styles written outside of the media query notation will be applied to mobile layouts and will be inherited all the way up to larger breakpoints. If you want to write styles that ONLY apply to the mobile breakpoint without having to cancel them out in other scopes, use the 'mobile' breakpoint (it's essentially "mobile-only"). Additionally, the print layout only inherits the tablet breakpoint (I've found this extremely convenient when styling for printers).

Example:

```js
const exampleStyle = {
    display: flex,
    fontSize: 18,
    mobile: {
        flexDirection: column
    },
    tablet: {
        fontSize: 16
    },
    small: {
        fontSize: 18
    },
    large: {
        fontSize: 20
    },
}
```

You can nest media queries inside of styles, or you can nest styles inside of media queries. Write it however you desire.

### Modifier Classes
With React and styled-components, you don't often need modifier classes, but when you do, themer has got you covered. If the component you're styling gets a className applied to it that is supposed to change its appearance, you can use the following notation:

```js
const exampleStyle = {
    class: {
        name: 'active',
        backgroundColor: 'yellow'
    }
}

// .component-class.active { background-color: yellow; }
```

If you need multiple modifier classes, just make the class property an array of multiple objects.

### Child Selectors
If you need to style a child element, just do this:

```js
const exampleStyle = {
    child: {
        selector: '> a',
        color: 'blue'
    }
}

// .component-class > a { color: blue; }
```

Just like with modifier classes above, if you need to define styles for multiple child selectors, make the child property an array.

## Usage Tips

### Extreme Component Reuse
With themer, you don't need to declare a bunch of different components via styled-components for every unique element in your design. For example, you only need to declare a single styled.div component. From there, just pass the desired (and appropriately-named) style object as the component's theme prop.

```js
const Div = styled.div`${props => themer(props.theme)}`
```

Now you can just use the Div component anywhere you need a styled div.

```js
<Div theme={exampleStyle} {...props} />
```

### Default and Custom Themes

With the handy use of the ES6 spread operator, you can set a default style for a component while allowing another style object passed to the theme prop:

```js
const ExampleComponent = ({theme}) =>
    <Div theme={{...defaultStyle, ...theme}} {...props} />
```

You can also do this within the styled-component's declaration if you want to allow theming for a less reusable component:

```js
const Div = styled.div`${props => themer({...defaultStyle, ...props.theme})}`
```

### Complex Styles / Atomic Design
Things may get a little tedious if you have to declare a single style object variable for each and every element that needs to be styled. You can get around this by packaging all of the styles for a single "[molecule](http://bradfrost.com/blog/post/atomic-web-design/#molecules)" together - as long as the names you give to these nested styles do not conflict with any CSS property name or other property that themer uses to function.

```js
const defaultStyle = {
    padding: 25,
    backgroundColor: 'pink',
    inner: {
        backgroundColor: black
    }
}

const ExampleComponent = ({theme}) =>
    <Div theme={{...defaultStyle, ...theme}} {...props}>
        <Div theme={{...defaultStyle.inner, ...theme.inner}}>Inner Content</Div>
    </Div>
```