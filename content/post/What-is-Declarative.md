---
title: "What Is Declarative?"
date: 2018-03-26T00:28:05+08:00
draft: false
---

According to wiki Declarative programming means

> a style of building the structure and elements of computer programs — that expresses the logic of a computation without describing its control flow

Control flow in here means all kind of if-then-else, switch, loops

I suddenly jump out of this question when I come across [react-fns](https://github.com/jaredpalmer/react-fns)’s readme. It say:

> Browser API’s turned into declarative React components and HoC

Are React components must be declarative?

Component like this IS declarative

```js
const Comp = () => (
  <ol>
    <li>Apple</li>
    <li>Boy</li>
  </ol>
)
```

Component like this is NOT very declarative to me

```js
const Comp = ({ list }) => (
  <ol>
    {list.map(item => {
      if (item.type === 'selected') {
        return <li style={{ color: 'blue' }}>{item.name} (selected)</li>
      }
      return <li>{item.name}</li>
    })}
  </ol>
)
```

Even more...

```js
const Comp = ({ data }) => {
  const { fruitlist } = _.get(data, ['xxGraphQuery'])
  const list = letMeDoSomeMagicFunc(fruitlist)
  let evenHasCounting = 0
  return (
    <div>
      <ol>
        {list.filter(item => item.moreComplex).map(item => {
          if (item.type === 'selected') {
            return <li style={{ color: 'blue' }}>{item.name} (selected)</li>
          }
          evenHasCounting++
          return <li>{item.name}</li>
        })}
      </ol>
      {evenHasCounting}
    </div>
  )
}
```

React component (no matter class component with lifecycle or stateless component with only a function) can become very un-declarative.

Any template language, DSL can become very un-declarative. People add a lot of control tags and attributes to angular and vue template. To some extend, you can never understand them in a short-time. Then, how can you say they are declarative?

Example:

```html
<ul class="example-animate-container">
  <li class="animate-repeat" ng-repeat="friend in friends | filter:q as results track by friend.name">
    [{{$index + 1}}] {{friend.name}} who is {{friend.age}} years old.
  </li>
  <li class="animate-repeat" ng-if="results.length === 0">
    <strong>No results found...</strong>
  </li>
</ul>
```

What is going on for `friend in friends | filter:q as results track by friend.name`?

---

Ok. come back to React world. Take a example from [react-fns](https://github.com/jaredpalmer/react-fns)

```js
import { withWindowSize } from 'react-fns'

const Inner = ({ width, height }) => (
  <div>
    Window size: {width}, {height}
  </div>
)

export default withWindowSize(Inner)
```

HoC is never very declarative to name. `withWindowSize` tell me provide window size to `Inner` but never tell me what props it provides. If you use a lot of HoC, your comp's props will be full of stuffs and you don't know your props connect with the HoC.

It become worse in data loading

```js
const Page = ({ dataA, dataB, dataC, d, e, f, g, h, i, j, k, l,... }) => (
  <div>
    <ChildA dataA={dataA} dataB={dataB} ... />
    <ChildA dataB={dataB} dataC={dataC} ... />
    ...
  </div>
)
// move up all children data loading to Page level to prevent each list-item fetch api
export default fetchAllDataForChildrenHoc(Page)
```

How about `renderProp`?

```js
const Example = () => (
  <WindowSize
    render={({ width, height }) => (
      <div>
        Window size: {width}, {height}
      </div>
    )}
  />
)
```

`renderProp` become very popular recently, because new context api is using it.

```js
const ThemeContext = React.createContext('light')
class ThemeProvider extends React.Component {
  state = { theme: 'light' }
  render() {
    return (
      <ThemeContext.Provider value={this.state.theme}>
        {this.props.children}
      </ThemeContext.Provider>
    )
  }
}
class App extends React.Component {
  render() {
    return (
      <ThemeProvider>
        <ThemeContext.Consumer>
          {val => {
            return <div>{val}</div>
          }}
        </ThemeContext.Consumer>
      </ThemeProvider>
    )
  }
}
```

What about?

```js
const App = () => (
  <C1.Consumer>
    {v1 => (
      <C2.Consumer>
        {v2 => (
          <C3.Consumer>
            {v3 => (
              <C4.Consumer>
                {v4 => (
                  <C5.Consumer>
                    {v5 => (
                      <div>
                        {v1} {v2} {v3} {v4} {v5}
                      </div>
                    )}
                  </C5.Consumer>
                )}
              </C4.Consumer>
            )}
          </C3.Consumer>
        )}
      </C2.Consumer>
    )}
  </C1.Consumer>
)
```

Still not bad. What about?

```js
const App = () => (
  <C1.Consumer>
    {v1 => (
      <C2.Consumer>
        {v2 => (
          <C3.Consumer>
            {v3 => (
              <C4.Consumer>
                {v4 =>
                  v2 && (
                    <C5.Consumer>
                      {v5 => (
                        <div>
                          {v1} {v2} {v3} {v4} {v5}
                        </div>
                      )}
                    </C5.Consumer>
                  )
                }
              </C4.Consumer>
            )}
          </C3.Consumer>
        )}
      </C2.Consumer>
    )}
  </C1.Consumer>
)
```

Can you see `v2 &&` which add if-then-else and expect what will happen to the code.

To sum up, any template language can become un-declarative. But nothing wrong with that. No need to be 100% declarative. As long as you can keep 90% declarative, and most part of your code is easily readable. Move up most if-then-else logic to function and use function name to name it. Then your code is better.
