---
title: Understanding your UI as a Tree 
---

<Intro>

Your React app is taking shape with many components being imported, exported and nested. Now to render your app, but how does React know where to start?

React and many other UI libraries model UI as a tree. Thinking of your React app as a tree is a helpful mental model to understand the relationship between your components and future concepts around performance and state management.

</Intro>

<YouWillLearn>

* How your UI is a tree
* What a React Render tree is and what it is useful for
* What a module dependency tree is and what it is useful for

</YouWillLearn>

## Your UI as a tree {/*your-ui-as-a-tree*/}

Trees are a relationship model between items and UI is often represented using tree structures. For example on browsers, the [DOM](https://developer.mozilla.org/docs/Web/API/Document_Object_Model/Introduction) represents HTML elements while the [CSSOM](https://developer.mozilla.org/docs/Web/API/CSS_Object_Model) does the same for CSS. Similar for mobile, platform views relate to one another in a tree format. There’s even an [Accessibility tree](https://developer.mozilla.org/docs/Glossary/Accessibility_tree)!

<Diagram name="preserving_state_dom_tree" height={193} width={864} alt="Diagram with three sections arranged horizontally. In the first section, there are three rectangles stacked vertically, with labels 'Component A', 'Component B', and 'Component C'. Transitioning to the next pane is an arrow with the React logo on top labeled 'React'. The middle section contains a tree of components, with the root labeled 'A' and two children labeled 'B' and 'C'. The next section is again transitioned using an arrow with the React logo on top labeled 'React'. The third and final section is a wireframe of a browser, containing a tree of 8 nodes, which has only a subset highlighted (indicating the subtree from the middle section).">

From components, React creates a UI tree which React DOM uses to render the DOM
</Diagram>

React also uses tree structures to manage and model the relationship between components in a React app. These trees are useful tools to understand how data flows through a React app and how to optimize rendering and app size.

## The Render Tree {/*the-render-tree*/}

A major feature of components is the ability to compose components of other components. As we [nest components](/learn/your-first-component#nesting-and-organizing-components), we have the concept of parent and child components, where each parent component may itself be a child of another component. 

When we render a React app, we can model this relationship in a tree, known as the render tree.

Here is a React app that renders a random greeting. 

<Sandpack>

```js App.js
import Title from './Title';
import Greeting from './Greeting';
import Copyright from './Copyright';

export default function App() {
  return (
    <>
      <Title text="Generate a random greeting" />
      <Greeting>
        <Copyright year={2004} />
      </Greeting>
    </>
  );
}

```

```js Title.js
export default function Title({fancy, text}) {
    return <h1 className={fancy ? 'fancy' : ''}>{text}</h1>;
}
```

```js Greeting.js
import * as React from 'react';
import greetings from './greetings';
import Title from './Title';

function randIndex(max) {
  return Math.floor(Math.random() * max);
}

export default function Greeting({children}) {
  const [index, setIndex] = React.useState(randIndex(greetings.length));
  const greeting = greetings[index];
  const nextGreeting = () => setIndex(randIndex(greetings.length));
  return (
    <>
      <Title fancy text={greeting} />
      <button onClick={nextGreeting}>🎲</button>
      {children}
    </>
  );
}
```

```js Copyright.js
export default function Copyright({year}) {
  return <p>©️ {year}</p>;
}
```

```js greetings.js
export default ['Howdy!', 'Hello2!', 'Hello3!', 'Hello3!', 'Hello4!'];
```

```css
.fancy {
    font-style: italic;
    color: blue;
}
```

</Sandpack>

<Diagram name="render_tree" height={300} width={509} alt="Tree graph with 5 nodes. The root of the tree is App, with two arrows extending from it to nodes Greeting and Title. The arrows are labelled renders. Greeting node also has two arrows extended pointing down to nodes Title and Copyright.">

React creates a UI tree made of components rendered, known as a render tree.

</Diagram>

From the example app, we can construct the above render tree. Each node in the tree represents a component and the root node is the [root component](/learn/importing-and-exporting-components#the-root-component-file). In this case, the root component is `App` and it is the first component React renders as it works down the children. Every arrow in the tree points from a parent component to a child component.

We often refer to the components near the root of the tree as top-level components. They are ancestors and have a lot of descendent components. Components that have no children are referred to as "leaves". 

<DeepDive>
#### Where are the HTML elements in the render tree? {/*where-are-the-html-elements-in-the-render-tree*/}
TODO
You'll notice that HTML elements, are not part of the tree. 
</DeepDive>

A render tree represents a single render pass of a React application. With [conditional rendering](/learn/conditional-rendering), a parent component may render different children depending on the data passed.

We can update the Greeting app to conditionally render either an image or a text greeting.

<Sandpack>

```js App.js
import { Title } from './GreetingMedia';
import Greeting from './Greeting';
import Copyright from './Copyright';

export default function App() {
  return (
    <>
      <Title text="Generate a random greeting" />
      <Greeting>
        <Copyright year={2004} />
      </Greeting>
    </>
  );
}

```

```js GreetingMedia.js
export function Title({fancy, text}) {
  return <h1 className={fancy ? 'fancy' : ''}>{text}</h1>;
}

export function Image({src}) {
  return <img src={src} />;
}
```

```js Greeting.js
import * as React from 'react';
import greetings from './greetings';
import {Title, Image} from './GreetingMedia';

function randIndex(max) {
  return Math.floor(Math.random() * max);
}

export default function Greeting({children}) {
  const [index, setIndex] = React.useState(randIndex(greetings.length));
  const greeting = greetings[index];
  const nextGreeting = () => setIndex(randIndex(greetings.length));
  return (
    <>
      {greeting.type === 'text' ? (
        <Title fancy text={greeting.value} />
      ) : (
        <Image src={greeting.value} />
      )}
      <button onClick={nextGreeting}>🎲</button>
      {children}
    </>
  );
}

```

```js Copyright.js
export default function Copyright({year}) {
  return <p>©️ {year}</p>;
}
```

```js greetings.js
export default [
    {type: 'text', value: 'Howdy!'},
    {type: 'text', value: 'Hello!'},
    {type: 'image', value: 'https://i.imgur.com/yXOvdOSs.jpg'},
    {type: 'image', value: 'https://i.imgur.com/yXOvdOSs.jpg'},
    {type: 'text', value: 'Howdy!'},
];
```

```css
.fancy {
    font-style: italic;
    color: blue;
}
```
</Sandpack>

<Diagram name="conditional_render_tree" height={300} width={522} alt="TODO.">

With conditional rendering, across different renders, the render tree may render different components.

</Diagram>

In this example, depending on what `greeting.type` is, we may render `<Title>` or `<Image>`. The render tree may be different for each render pass.

Although render trees may differ across render pases, these trees are generally helpful for identifying what the top-level components are in a React app. Top-level components affect the rendering performance of all the components beneath them and often contain the most complexity.

## The Module Dependency Tree {/*the-module-dependency-tree*/}

Another relationship in a React app that can be modeled with a tree are an app's module dependencies. As we [break up our components](/learn/importing-and-exporting-components#exporting-and-importing-a-component) and logic into separate files, we create [JS modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) where we may export components, functions or constants. 

Each node in a module dependency tree is a module and each branch represents an `import` statement in that module.

If we take the previous Greeting app, we can build a module dependency tree, or dependency tree for short. 

<Diagram name="module_dependency_tree" height={400} width={761} alt="TODO.">

The module dependency tree for the Greetings app

</Diagram>

The root node of the tree is the root module, also known as the entrypoint file. It often is the module that contains the root component.

Comparing to the render tree of the same app, there are similar structures but some notable differences:

* Tree nodes are modules vs. components. Often, a module exports a single component but in this case `GreetingMedia.js` exports both `Title` and `Image`.
* Non-component modules are also represented in this tree. The render tree only encapsulates components.
* `Copyright.js` appears under `App.js` but in the render tree, `Copyright`, the component, appears as a child of `Greeting`. This is because `Greeting` accepts JSX as children. 

Dependency trees are useful to determine what modules are necessary to run your React app. When building a React app for production, there is typically a build step that will bundle all the necessary JavaScript to ship to the client. The tool responsible for this is called a bundler, and bundlers will use the dependency tree to determine what modules should be included.

As your app grows, often the bundle size does too. Large bundle sizes are expensive for a client to download and run so getting a sense of your app's dependency tree may help with debugging the issue.

<DeepDive>
#### Conditional Dependencies {/*conditional-dependencies*/}

Similar to conditional rendering, there may be conditional imports.
... TODO
</DeepDive>

<Recap>

* Trees are a common way to represent the relationship between entities. They are often used to model UI.
* Render trees represent the nested relationship between React components across a single render.
* With conditional rendering, the render tree may change across different renders. With different prop values, components may render different children components. 
* Render trees help identify what the top-level components are. Top-level components affect the rendering performance of all components beneath them. Identifying them is useful for debugging slow renders.
* Dependency trees represent the module dependencies in a React app.
* Dependency trees are used by build tools to bundle the necessary code to ship an app.
* Dependency trees are useful for debugging large bundle sizes and opportunities for optimizing what code is bundled.

</Recap>