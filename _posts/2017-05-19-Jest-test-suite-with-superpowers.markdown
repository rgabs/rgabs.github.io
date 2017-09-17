---
layout: post
title: "Jest: Test suite with superpowers 😎"
date: 2017-05-19T01:15:00.000Z
categories: react-native
---

## Intro

With the growing complexities of apps, unit testing is a mandate. Since we are writing code in JS, we can utilize most of the testing frameworks/libraries available out there for `react/web` apps without much changes.

> We recommend using Jest Framework plus some additional utilities like enzyme to make the developer's life easier.

### What's the use of testing UI code?


<div style="text-align:center">
  <img src="/static/img/jest/why.gif" style="width: 70%;display:inline-block;text-align:'left'" vspace="20">
</div>

We all have asked/been asked this question at least once. And agree or not, the term unit testing was not very popular amongst FE developers until recent years. Here are some of the reasons why we should write unit tests.

- It lets you capture bugs before the QA team does:

    We all know that a QA and a dev can never be friends. No dev likes it when QA team finds a bug and tells them to change it. If your code has a good test coverage, there are very less chances to find bugs. Win-win for both sides right?

- Helps other devs understand your code better.

    All the test frameworks have describe block where you define what a method does in a language which any human can understand. Need I say more?

- Refactors your code:

    You will start asking these questions to yourself while coding: *How will I test this code?*, *How do I make sure that each method I wrote can be tested?* If you already ask this questions while writing code, then you are one of the few gems.
- Makes your code modular:

    If your code is tested, there are 99% chances that it is modular, which means that you can easily implement changes in future.

- Makes debugging/implementing changes much, *much* easier :

    There will be cases when you will be required to change a functions logic. *eg, a currency formatter function which return a string.* One way of debugging it would be to go through the UI and check if the desired output is there. Another *smart* way would be to fire your test case, change the test results according to your desired output and __let the test fail__. Now change the logic inside your method to make failed the test pass.

- Does __NOT__ increase the dev time:

    Many of us give this excuse that writing test cases will increase the dev time(Even I used to do this). But believe me it doesn't. During the course of development almost half of the time is spent in debugging/bug fixing .Writing unit tests can decrease the this value from 50% to less than 20%.



## Jest setup

For all who have not heard about jest, have a quick look here: https://facebook.github.io/jest/

>Jest is used by Facebook to test all JavaScript code including React applications. One of Jest's philosophies is to provide an integrated "zero-configuration" experience. We observed that when engineers are provided with ready-to-use tools, they end up writing more tests, which in turn results in more stable and healthy code bases.

We used Jest because of the following reasons:

- Minimal Configuration.
- Watch only changed files.
- Fast
- Snapshot testing(Explained later)
- Coverage out of box


### 4 important test scripts every project should have

```json
"scripts": {
    "test": "jest --verbose --coverage",
    "test:update": "jest --verbose --coverage --updateSnapshot",
    "test:watch": "jest --verbose --watch",
    "coverage": "jest --verbose --coverage && open ./coverage/lcov-report/index.html",
  }
```

- `test`: It will go through all the test files and execute them. This command will also be used in pre-hooks and CI checks.
- `test:watch`: This will watch all the test files. It is very useful while writing tests and quickly see the result.
- `test:update`: This command will update snapshots for all the presentational components. If the snapshot is not there, it will create it for you.
- `coverage`: As the name suggests, this command will generate a coverage report.


### Testing conventions:

It is highly recommended to have conventions for test files as well. Here are the conventions we followed:

- Jest recommends having a `__test__` folder in the same location the file which is to be tested is placed.
- The name convention for test file is `<testFileName>.test.js`.
if you are writing test for `abc.component.js`, then the test filename would be `abc.component.test.js`.
- In each `expect`, we write the function name first which is to be tested.

Here is a small example following the above mentioned conventions:
```js
//counter.util.js
const counter = (a) => a + 1;

//__test__/counter.util.test.js
describe('counter: Should increment the passed value', () => {
  ...
});
```

### JEST configuration:

As we read in the documentation, Jest is indeed very easy to setup. You do not need  a separate config file, the configuration is so simple that it can fit inside `package.json` only.

Here is our jest config:
```json
"jest": {
    "preset": "react-native",
    "cacheDirectory": "./cache",
    "coveragePathIgnorePatterns": [
      "./app/utils/vendor"
    ],
    "coverageThreshold": {
      "global": {
        "statements": 80
      }
    },
    "transformIgnorePatterns": [
      "/node_modules/(?!react-native|react-clone-referenced-element|react-navigation)"
    ]
  }
```

- `preset`: The preset is a node environment that mimics the environment of a React Native app. Because it doesn't load any DOM or browser APIs, it greatly improves Jest's startup time.

- `cacheDirectory`: It helps you greatly improve the test speed. It does so by creating cache of compiled modules so that next time it doesn't have to compile the node_modules while running tests.

- `coveragePathIgnorePatterns`: Define the files which want to skip for coverage reports.

- `coverageThreshold`: Defines the threshold limit for all the tests to pass. If the coverage is less than the defined limit, the tests would fail. This helped us to keep a good amount of coverage at all point of time.

- `transformIgnorePatterns`: We pass all the NPM modules here which needs to be transpiled. These modules are basically ES6/7 modules.

_Note: Make sure to add `cache` and `coverage` in your gitignore file._


## Snapshots

<div style="text-align:center">
  <img src="/static/img/jest/snap.gif" style="width: 20%;display:inline-block;text-align:'left'" vspace="20">
</div>

This features makes testing presentational components a lot easier. With a single line, you can test all your presentational components (their render method only). No need to write test cases for each component returned by render method.



### What are Snapshots:

A snapshot is nothing but a configuration file defining your component style, UI and props. The test case will look somthing like this:

`__tests__/someComponent.component.test.js`

```js
import React from 'react';
import renderer from 'react-test-renderer';
import SomeComponent from '../SomeComponent.component';

describe('Some component', () => {
  it('renders correctly', () => {
    const tree = renderer.create(
      <SomeComponent/>
    ).toJSON();
    expect(tree).toMatchSnapshot();
  });
});
```

Whenever Jest sees this line `expect(tree).toMatchSnapshot();`, it is going to generate a snapshot and compare it with stored snapshot. If the snapshot is not present, Jest will store the generated snap.

The generated snap file will look something like this:

`__tests__/snaphots/someComponent.component.js.snap`
```
exports[`SomeComponent Component: SomeComponent renders correctly 1`] = `
<View
  style={
    Object {
      "flex": 1,
    }
  }
>
  <Text
    accessible={true}
    allowFontScaling={true}
    ellipsizeMode="tail"
    style={
      Object {
        "color": "#000000",
        "fontFamily": "Roboto",
        "fontSize": 24,
        "fontWeight": "500",
        "paddingVertical": 20,
        "textAlign": "center",
      }
    }
  >
    SomeText
  </Text>
</View>
`;
```

As you can see above, the snap contains every possible property of the UI which is being returned by the render method.


### Should I push generated snaps to git?

Yes you should. Snaps should be there in each dev's machine so that if one of the devs changes some other component unknowingly, the snap test for that component will fail and he/she would know before pushing it.
Even the Jest official documentation says this:

>It is expected that all snapshots are part of the code that is run on CI and since new snapshots automatically pass, they should not pass a test run on a CI system. It is recommended to always commit all snapshots and to keep them in version control.

### What to do when snap test fails?

Consider this scenario. You worked on a component, generated a snap and pushed it. Later another dev named John made some change in the component. Now the test of the snap will fail since snap still contains the code which you wrote. John will just need to update the snapshot to make the test pass. No need to update the test case, just one command: `jest --updateSnapshot` and you are done.

*We recommend creating an __npm script__ for updating snaps. As you can see in the package.json of our boiler plate,  it conatains a command called "test:update". This command go through all the test cases and will update the snap whenever it is required.*

__More information can be found here:__ [https://facebook.github.io/jest/docs/en/snapshot-testing.html#content](https://facebook.github.io/jest/docs/en/snapshot-testing.html#content)


## Testing stateful components using Enzyme

We talked about testing presentational components using our beloved feature called Snaphot testing in Jest. But It just tests the UI of the component(just the render method of your component).

_What if your component contains some class methods? What if your component contains state?_

Thats where we use __enzyme__.

### What's enzyme?

Enzyme is a JavaScript Testing utility for React. You will mostly be using shallow utility from enzyme. Shallow utility helps us rendering a component and allows us accessing the class methods/state of the component.

### Integrating Enzyme in your current Jest Framework

The default react-native boilerplate comes with Jest. Integrating enzyme with Jest is just a two step process.

- Install enzyme and jest-enzyme `yarn add enzyme jest-enzyme --dev`

- Add one line in package.json inside jest config:
    `"setupTestFrameworkScriptFile": "./node_modules/jest-enzyme/lib/index.js",`

Thats it. You can start using Enzyme utilities now.

### Using Shallow renderer from enzyme:

- First we need to shallow render our component.

```js
import {shallow} from 'enzyme';
describe('SomeComponent component', () => {
  it('Shallow rendering', () => {
    const wrapper = shallow(<SomeComponent {..props}/>);
  });
});
```

Now Our component is rendered and we can access props/state/methods using `wrapper`. Here is how you access them:


```js
import {shallow} from 'enzyme';
describe('SomeComponent component', () => {
  it('Shallow rendering', () => {
    const wrapper = shallow(<SomeComponent someProp={1}/>);
    const componentInstance = wrapper.instance();
    //Accessing react lifecyle methods
    componentInstance.componentDidMount();
    componentInstance.componentWillMount();
    //Accessing component state
    expect(wrapper.state('someStateKey')).toBe(true);
    //Accessing component props
    expect(wrapper.props.someProp).toEqual(1);
    //Accessing class methods
    expect(componentInstance.counter(1)).toEqual(2);
  });
});
```

As you saw, you can access everything a component possess using shallow utitity. You can also have a look at the example test case in our boilerplate code [here]().


### Example

Lets take an example of a component with state and class method. We will write test case for the methods including snapshot test. The example includes testing class methods, state and props.

```js
import React from 'react';
import renderer from 'react-test-renderer';
import {shallow} from 'enzyme';
import Counter from '../Counter.component';

describe('Counter component', () => {
  it('Counter: renders correctly', () => {
    const tree = renderer.create(<Counter />).toJSON();
    expect(tree).toMatchSnapshot();
  });
  it('componentWillMount: should set the passed initialCountValue to state', () => {
    const wrapper = shallow(<Counter initialCountValue={2}/>);
    expect(wrapper.instance().state.count).toBe(2);
  });
  it('incrementCounter: should increment state.count by 1', () => {
    const wrapper = shallow(<Counter initialCountValue={0}/>);
    const instance = wrapper.instance();
    expect(instance.state.count).toBe(0);
    instance.incrementCounter();
    expect(instance.state.count).toBe(1);
  });
  it('decrementCounter: should decrement state.count by 1', () => {
    const wrapper = shallow(<Counter initialCountValue={1}/>);
    const instance = wrapper.instance();
    expect(instance.state.count).toBe(1);
    instance.decrementCounter();
    expect(instance.state.count).toBe(0);
  });
  it('should call props on increment/decrement', () => {
    const incrementSpy = jest.fn();
    const decrementSpy = jest.fn();
    const wrapper = shallow(<Counter initialCountValue={1} onIncrement={incrementSpy} onDecrement={decrementSpy}/>);
    const instance = wrapper.instance();
    instance.incrementCounter();
    expect(incrementSpy).toBeCalledWith(2);
    instance.decrementCounter();
    expect(decrementSpy).toBeCalledWith(1);
  });
});
```
