# PART 1

## Getting starting with React
Install `create-react-app` to be used globally (-g)
```
sudo npm install -g create-react-app
```
```
npx create-react-app hello-react --use-npm
cd hello-react
npm start
```
`create-react-app` setups a project with a series of utilities like Webpack (light server + dev env that automatically reload as you make changes), Babel (js compiler)

https://codesandbox.io - to start a project quickly

`ReactDom.render` takes in two arguments
1- the element that we want to create (what)
2- where we want to render that element (where)

`React.createElement` takes in two arguments
1- the name of the tag to create.
2- any properties for the element.
3- children. This argument can be nested as in the example below.

```js
ReactDOM.render(
  React.createElement(
    "ul",
    {style: {color: "blue"} }, 
    React.createElement("li", null, "Hot Dogs"),
    React.createElement("li", null, "Hamburg"),
    React.createElement("li", null, "Pizza")
    ),
  document.getElementById('root')
);
```

### JSX Sintax and Javascript
JSX is a syntax extension of JavaScript used to create DOM elements which are rendered in the React DOM. Before JSX can render in a web browser it must first be compiled into regular JavaScript.

### ReactDOM JavaScript library
The JavaScript library ReactDOM renders JSX element to the DOM through taking a JSX expression, creating a corresponding tree of DOM nodes, and adding that tree to the DOM.  The following example shows how to make the JSX expression appear on the screen.

```jsx
import  React  from  'react';
import  ReactDOM  from  'react-dom';

// Copy code here:
const myJSXExpression= (
	<ul style={{color: "blue"}}>
		<li>Hot Dogs</li>
		<li>Hamburgers</li>
		<li>Pizza</li>
	</ul>
	)
ReactDOM.render(myJSXExpression, document.getElementById('app'));
```
Behind the scenes a JavaScript compiler named [Babel](https://babeljs.io/) takes those tags (JSX syntax) and turn them into `createElement` calls using Babel.

Collection of `reactElement`'s are called **components**. *A component is a function that returns some UI*.
```js
function Hello() {
  return (
    <div>
      <h1>Welcome to React!</h1>
      <p>Let's build something cool.</p>
    </div>
  )
}

ReactDOM.render(
  <Hello />,
  document.getElementById('root')
);
```
We can think of our React UI as being reflections of the data we want to display.

Function components:
```js
const lakeList = ["Echo Lake", "Maud Lake", "Cascade Lake"];

// App component is responsible for rendering the lake
function App({ lakes }) {
  return (
    <ul>
      {lakes.map(lake => (
        <li>{lake}</li>
        ))}
    </ul>
  );
}

// we pass the lakeList to our App component
ReactDOM.render(
  <App lakes={lakeList} />,
  document.getElementById('root')
);
```
> when a component is created in React it let's you call it like an HTML tag.

## React state with hooks
### Object and array destructuring
**Object destructuring**, allow us to reference the keys in the objects instead of having to use props and then dot notation each time.

**Array destructuring**, since we can't access the values by key, we can access by index or position. 
Example
```js
const snacks = ["popcorn", "pretzels", "pineapples"];
console.log(snacks[0]);
// output popcorn
```
with array destructuring
```js
const [first] = ["popcorn", "pretzels", "pineapples"];
console.log(first);
// output popcorn
```
```js
const [, , third] = ["popcorn", "pretzels", "pineapples"];
console.log(third);
// output pineapples
```
### How to incorporate State values in our Application
State is what is going on with your application. It refers to an object that has some data. That defines what your application is doing. That correlates to the virtual DOM, feature in react that automatically modify your components when that data or states changes.
>Super cool feature of react: If anything inside the application modifies the state variables react will automatically redraw any part of the application that uses that variable.
>
Status = state value that reflect current status for the application
`{ useState }` in the import is to add the useState hook from the react library.
```js
import React, { useState } from 'react';
```
hook = library/function, i.e., a hook is a function that allows to add functionality to a component. And `useState` is a built-in hook that we can use to handle state changes in our application.
```js
function App() {
  const [status, setStatus] = useState("Open");
  return (
    <div>
      <h1>Status: {status}</h1>
      <button onClick={() => setStatus("Open")}>Open</button>
      <button onClick={() => setStatus("Back in 5'")}>Break</button>
      <button onClick={() => setStatus("Closed")}>Closed</button>
    </div>
  );
```
Other hooks:
- useEffect()
- useReducer()

React will re-render the component tree whenever the state changes. `useEffect` will be called after this renders.

```js
function App() {
  const [val, setVal] = useState("");
  const [val2, setVal2] = useState("");

  useEffect(() => {
    console.log(`field 1: ${val}`);
  }, [val]);

  useEffect(() => {
    console.log(`field 2: ${val2}`);
  }, [val, val2]);

  return (
    <>
      <label>
        Favorite Phrase:
        <input 
          value={val} 
          onChange={e => setVal(e.target.value)}
        />
      </label>
      <br />
      <label>
        Favorite Phrase:
        <input 
          value={val2} 
          onChange={e => setVal2(e.target.value)}
        />
      </label>
    </>
  );
}
```

### Grabing data with React using `useState` and `useEffect`
```js
function GitHubUser({ login }) {
  const [data, setData] = useState(null);

  // grab data from github
  useEffect(() => {
    fetch(`https://api.github.com/users/${login}`)
    .then(res => res.json()) // take response from api and turn into json
    .then(setData) // call the setData with data value from the api
    .catch(console.error);
  }, []);

  if (data) {
    return <div>
      <h1>{data.login}</h1>
      <img src={data.avatar_url} width={100}/>
      </div>
  }
  return null; // if we do not have user then return null
}

function App() {
  return <GitHubUser login="arllanos" />
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

Reducer function, takes in the current state and returns a new state.

```js
function Checkbox() {
  const [checked, toggle] = useReducer(
    checked => !checked,
    false
    );

  return (
    <>
      <input
        type="checkbox" 
        value={checked} 
        onChange={toggle}/>
      {checked ? "checked": "not checked"}
    </>
  );
}

ReactDOM.render(
  <Checkbox />,
  document.getElementById('root')
);
```
### Deploying React app
create a prod build for the project. Take all files, minify them, make them smaller and bundle them up into a pakcage
```
npm run build 
npm install -g serve
serve -s build
```
**serve** is a static server that can run into the computer and will serve the application.

**Netlify**, simple singup/login and drag. Can drag build folder into area at the bottom of netlify. It will give a link to interact with the app.
[https://www.netlify.com/](https://www.netlify.com/)

# PART 2
# [React: Creating and Hosting a Full-Stack Site](https://www.linkedin.com/learning/react-creating-and-hosting-a-full-stack-site/creating-the-app-component)

## Client side
Easiest way to implement navigation in React: **ReactRouter** tool. Make it easy to display certain page/component in a slot depending on the current URL.
```js
npm install --save react-router-dom
```
Imports:
```js
import React from "react";
import { BrowserRouter as Router, Route } from "react-router-dom";
import HomePage from "./pages/HomePage";
import "./App.css";

function App() {
  return (
    <Router>
      <div className="App">
        <Route path="/" component={HomePage} exact />
      </div>
    </Router>
  );
}

export default App;
```

## Backend
 Initialize new folder as a npm package
```
mkdir my-blog-backend
cd my-blog-backend
npm init -y
``` 
Install Express
```
npm install --save express
```
Setup initial folder structure
```
mkdir src
cd src
touch server.js
```
Currently Node.js does no have native support for modern ES6 syntax, so we need to do the following to allow us to write our backend using ES6
First, install a few packages from babel, which is a compiler to allow us to write our codebase in ES6 syntax and still have it working in sustems where ES6 isn't supported natively, such as Node.js.
```
npm install --save-dev @babel/core @babel/node @babel/preset-env
```

### `src/server.js`
```js
import express from 'express';

const app = express();

app.get('/hello', (req, res) => res.send('Hello!'));
app.listen(8000, () => console.log('Listening on port 8080));
```
To run the server
```
npx babel-node src/server.js 
```

To be able to parse body in a POST request install `body-parser` package
```
npm install --save body-parser
```
In order the server automatically restarts when our files changes
```
npm install --save-dev nodemon
```
Then run the server using
```
npx nodemon --exec npx babel-node src/server.js 
```

## MongoDB
### Install with Docker
```
docker pull mongo
sudo mkdir -p /mongodata
docker run --rm --name mongo -v /mongodata:/data/db -p 27017:27017 -d mongo:latest
```
Start MongoDB shell
```
docker exec -it mongo mongo
```

### Usage General
1. To start (not need if using docker container for mongo as mongo is already running when container starts)
```
mongod
```
2. To start the MongoDB shell
```
mongo
```
Or (if using docker)
```
docker exec -it mongo mongo
```
3. Create a new db
```
use my-blog
```
in MongoDB, each **database** is composed of a few or many **collections**. These collections can contain any number of JSON objects and we call these objects **documents**

create a collection called articles. Replace **DATA** for an array of objects.
```
db.articles.insert([DATA])
```
that is 
```
db.articles.insert([{
 name: 'learn-react',
 upvotes: 0,
 comments: [ ],
 }, {
 name: 'learn-node',
 upvotes: 0,
 comments: [ ],
 }, {
 name: 'my-thoughts-on-resumes',
 upvotes: 0,
 comments: [ ],
}
])
```
query the collection
```
db.articles.find({})
```
or
```
db.articles.find({}).pretty()
```
or
```
db.articles.find({ name: 'learn-react' }).pretty()
```
or
```
db.articles.findOne({ name: 'learn-react' })
```

## Backend (contd)
Install the mongodb npm package in our project. This will allow us to connect to and modify our database from inside our Express server, so that we can persist our data.
```
npm install --save mongodb
```

## Strategy for making code more DRY

```js
const withBoilerPlate = (operations) => {
  data = {id: 'id1', text: 'text1'};  
  operations(data)
}

function entryPoint() {
  withBoilerPlate((data) => {
    console.log('entrypoint passed to withBoilerPlate', data)
  });
}

entryPoint();
```
I pass my own function to `withBoilerPlate` function and the latter will perform all the setup and teardown. 
The function being passed is `(data) => {
    console.log('entrypoint passed to withBoilerPlate', data)
 });`
The function takes another function called `operations` as argument and will take care of all of the setup and teardown.

## Fetch API
Simple built-in API (no need to import) to make request from Front-End to a the backend REST API.
Fetch is a simple asynchronous function we can call from the front end.
`fetch('<endpoint_url>, [<options>]')`
e.g.,
```js
fetch('api/articles/learn-node/add-comment', {
	method: 'POST',
	headers: '...',
	body: { ... }
});
```
Although not need to import, in order Fetch works on IE, need to setup a polyfill in index.js:
```
npm install --save whatwg-fetch
```
and edit `index.js` and add:
```js
import 'whatwg-fetch';
```
## State
State give us a place to temporarily store information, for example information I've loaded from the server.

For keeping track of state, starting with React 16.8 (Feb/2019), we can use **React hooks**.
**React hooks** are functions we can call to handle state in a React application.
```js
import React, { useState } from 'react';

const Example = () => {
	const [count, setCount] = useState(0);
	return (
		<div>
			<p>You clicked {count} times</p>
			<button onClick={() => setCount(count+1)}>
				Click me
			</button>
		<div>
	);
}
```
 - `useState`, is a React hook to change the value of `count` using the function `setCount`. The argument passed to `useSate` is the initial state of `count` before loading any data or changing the state
  - `useEffect`, An effect is something that React does not control. React does not know how to make a request to the backend for example. When I mount a component in the DOM, React renders it, and the React hook will make the request to our backend to load data from the server. It helps to perform all the side effect of our components such as, fetching data and setting the sate with the result.

> Advantages of using hooks:
> - hard to share logic related to state between multple components. For example, if need application needs state (e.g., geo coordinates) in different components, would need something complex like Redux or render props. Both add extra wrappers (high order components) making component logic more complex, while the hooks allow to reuse the state in a simple way.
> - componentDidMount, componentWillUnmout or componentWillUpdate may end up having logic not related that better to separate in its own hooks
> avoid confusion inherent to the use of `this`

### Exmple: Handling State in Class Type Component
```js
import React, { Component } from 'react';

export default class SimpleState extends Component {
    state = {
        cuenta: 0,
        cuenta2: 10,
    };

    render() {
        return (
            <div>
                La cuenta es {this.state.cuenta}
                <button onClick={() => this.setState({ cuenta: this.state.cuenta + 1 })}>
                    Aumentar cuenta
                </button>
                <p />
                La cuenta2 es {this.state.cuenta2}
                <button onClick={() => this.setState({ cuenta2: this.state.cuenta2 + 1 })}>
                    Aumentar cuenta2
                </button>
            </div>
        );
    }
}
```
### Exmple: Handling State in Function Type Component
The function components in React cannot maintain state. Hooks introduce the ability of maintaining state in this kind of components.
```js
import React, { useState} from 'react';

export default function SimpleStateFunction() {
    const [cuenta, setCuenta] = useState(0);
    const [cuenta2, setCuenta2] = useState(10);

    return (
        <div>
            La cuenta es {cuenta}
            <button onClick={() => setCuenta(cuenta + 1)}>
                Aumentar cuenta
            </button>
            <p />
            La cuenta2 es {cuenta2}
            <button onClick={() => setCuenta2(cuenta2 + 1)}>
                Aumentar cuenta2
            </button>
        </div>
    );
}
```
