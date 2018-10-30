---
layout: post
title: "Unit testing redux containers the better way using jest"
date: 2018-10-19T01:15:00.000Z
categories: front-end
---

## The problem
Testing react components are very easy nowadays, thanks to libraries like jest, enzyme, etc. But testing container components which use redux can be a real pain sometimes and the reason for that is the so called **connect** ([higher order component](https://reactjs.org/docs/higher-order-components.html) from redux).

A usual react container component exports something like this:

`export default connect(mapStateToProps, mapDispatchToProps)(App)`

Since we are exporting a [higher order component](https://reactjs.org/docs/higher-order-components.html), it makes testing the component more complicated than it has to be. 


## Existing solutions

There are a couple of methods which I found online regarding testing redux containers but the problem was either it was too much boilerplate code required or unncessary exports of methods for just testing them.

1. **Wrapping the component in Provider/passing down store/Creating mock store**

    While this method lets us write tests, it adds a lot of boilerplate code for testing each container.

    Example: https://medium.com/@visualskyrim/test-your-redux-container-with-enzyme-a0e10c0574ec

    https://medium.com/@linmic/writing-tests-for-react-redux-containers-5d01083ecfc4

2. **Exporting mapStateToProps, mapDispatchToProps and the component** 

    While this methods has very less boilerplate code and it allows us to properly test each function, it kind of a hack since we are exporting a function just to make is testable which is a code smell. We should export a part of our code only if it is being used somewhere else, not to test it.

    Example: https://stackoverflow.com/questions/51943248/react-redux-testing-mapstatetoprops-and-mapdispatchtoprops-with-enzyme-jest


## The better way of testing redux containers:

What if I told you that you can access mapStateToProps, mapDispatchToProps and the component in your test file without exporting them? Sounds cool right?

Presenting you the react-redux mock, which lets you access all the three functions in the test file.

<script src="https://gist.github.com/rgabs/1afa3f0058baf0d491a24d096f6085c7.js"></script>

### Setting up mock:
We will be using [file mock](https://jestjs.io/docs/en/manual-mocks#mocking-node-modules) from jest to mock react-redux library.

1. Create __mocks__ folder at the same level as node_modules and add a file called react-redux.js.
```
    .
    ├── config
    ├── __mocks__
    │   └── react-redux.js
    ├── node_modules
    └── src
```

2. Add the following code in the newly created mock file.
    ```js
    // This mock will make sure that we are able to access mapStateToProps, mapDispatchToProps and reactComponent in the test file.

    // To use this, just do `jest.mock('react-redux');` in your test.js file.
    const mockDispatch = jest.fn((action) => action);

    module.exports = {
    connect: (mapStateToProps, mapDispatchToProps) => (reactComponent) => ({
        mapStateToProps,
        mapDispatchToProps: (dispatch = mockDispatch, ownProps) => mapDispatchToProps(dispatch, ownProps),
        reactComponent,
        mockDispatch
    }),
    Provider: ({children}) => children
    };
    ```

3. In your test file, tell jest to use mocked version of react-redux instead of using the one from the node_modules. To do this, just add the following lines after your imports:

    `jest.mock('react-redux');`

4. You should now be able to access mapStateToProps, mapDispatchToProps and reactComponent in the test file. 

    ```js
    import App from './App'
    
    App.mapStateToProps() // the mapStateToProps function

    App.mapDispatchToProps() // the mapDispatchToProps function

    <App.reactComponent /> // the react component which is being passed to connect
    
    App.mockDispatch // a jest function, can be used to test if dispatch is being called with the right action in mapDispatchToProps
    
    ``` 

    The app component just exports the connected component as a default export. Nothing changes there. Example:

    `export default connect(mapStateToProps, mapDispatchToProps)(App);`

## Example

I had built a [github repo search](https://github.com/rgabs/github-repo-search) app a while back.

The app contains redux container and test cases for the container component.

URL for redux container file: [ReposContainer.js](https://github.com/rgabs/github-repo-search/blob/master/src/shared/containers/ReposContainer.js)

URL for test case file: [ReposContainer.test.js](https://github.com/rgabs/github-repo-search/blob/master/src/shared/containers/__tests__/ReposContainer.test.js)


## Bonus Topic: What we should/shouldn't test
I strongly believe that we should test only the code that we write, not the library/framework we use. We can always assume that the external dependecies are already tested and mock them to test only the part we added. 

In case of redux containers, all we need to test is:
1. **mapStateToProps**

     Which handles the logic of reading from stores/crunching data to pass to the react component. Its a pure function which gets store state as input and returns an object to be consumed by the react component. Hence we should just test if it is reading the correct keys from store and/or if it is doing the crunching properly.

    While testing this, we should **not** be worried about where the data is coming from or weather its being passed as props to the component(that's the job of react-redux library)

2. **mapDispatchToProps**

    Which handles dispatching actions so that they can be consumed by the reducers. It is also a pure function which gets dispatch as an argument and calls it with the action object. Here all we need to test is if dispatch is being called with the right action. 

    While testing this, we should **not** be worried if click a button on a component dispatches action or dispatching an action changes the store state or not(again. that's the job of react-redux library)

3. **React component** 

    Usually a container component file contains a react component which is passing down props to its children and/or rendering children based on the props. This component reads the props from mapStateToProps and mapDispatchToProps and uses them to render the UI. Here we need to test whateve logic/UI the component has. It should be tested like any other react component using jest/enzyme etc.

    While testing the component. we should **not** be worried if the props are being passed from mapStateToProps or mapDispatchToProps(that's the job of connect and its already tested :)).

Now go ahead and enjoying writing tests without worrying about boilerplate code in redux containers :)

CHEERS!!

<div style="text-align:center">
  <img src="/static/img/cheers.gif" style="width: 70%;display:inline-block;text-align:'left'" vspace="20">
</div>

Don't forget to clap if you like the blog. 

## References
1. https://gist.github.com/rgabs/1afa3f0058baf0d491a24d096f6085c7
2. https://medium.com/@visualskyrim/test-your-redux-container-with-enzyme-a0e10c0574ec 
3. https://jestjs.io/docs/en/manual-mocks#mocking-node-modules
4. https://www.imdb.com/title/tt0386676/