# 说明
基于redux、redux-saga封装的状态管理工具


## 使用

#### 安装

- npm

```shell
npm install @lugia/lugiax
```

- yarn

```shell
yarn add @lugia/lugiax
```

#### bind

- 描述

```
将注册的model和组件进行双向绑定。在组件中props中就可以获取到model中的数据。model数据改变后组件也会重新render被绑定组件。也可以通过调用mutation方法去改model中的数据，从而引起被绑定组件render。
```

- 方法简述

```
import { bind } from "@lugia/lugiax";

const newComponent = bind(
  model, // object 
  mapModelState,  // function
  mapModelMutations, // object
  ?option // object
)(ReactComponent);

```

- model

  - 数据类型
    - object
  - 使用lugaix.register() 方法生成的model对象

- mapModelStates

  - 数据类型
    - function
  - 函数方法，使用者通过这个方法定义如何从model中获取的state值，并将获取到的state值存放在对象中返回，最终bind会将返回对象的属性映射到被绑定组件的 `this.props` 中。

- mapModelMutations

  - 数据类型
    - object
  - 自定义对象。使用者通过这个对象定义如何使用model中的mutation方法，最终bind会将这个对象属性映射到被绑定组件的 `this.props` 中。

- option 对象 表示其他扩展属性

  - props  

    - 数据类型
      - object 
    - 传入一些属性值，bind会将这个对象属性映射到被绑定组件的 `this.props` 中。（注：此对象传入的属性优先级最高）

  - eventHandle 

    - 数据类型
      - { [eventName: string] : Function }
    - 传入处理函数，bind会将这个对象属性映射到被绑定组件的 `this.props` 中。

  - withRef

    - 数据类型
      - boolean
    - ？？？？？？？？

  - areStateEqual

    - 数据类型
      - (oldModelState, newModelState)  => { return true or false}
      - oldModelState是model改前的数据， newModelState是model改后的数据

    - 用户自定义方法, 当方法返回true时bind后的组件会被 render，当方法返回值为false时bind后的组件不会被render。

  - areStatePropsEqual

    - 数据类型
      - (oldState, newState)  => { return true or false}
      - oldState组件更新前的state数据，newState更新后的state数据。对应的是model映射过来的值

  - areOwnPropsEqual

    - 数据类型
      - (oldProps, newProps)  => { return true or false}
      - oldProps 是bind后的组件props改变前的数据，oldProps 是bind后的组件props改变后的数据，   对应的是bind后返回组件在调用是传入的属性值
    - 用户自定义方法，让这个方法返回true时bind后的组件会被 render，方法返回值为false时bind后的组件不会被render。

###### 1.使用 `bind` 将简单数据model和组件进行绑定

- 注册model

```javascript
// model.js
import lugaix, { bind } from "@lugia/lugiax";
const titleState = {
  title: "一个测试用例"
};
const modelName = "testTitle";
const titleModel = lugaix.register({
  model: modelName, // 注册的模块名称
  state: titleState, // 初始化值
  mutations: {
    sync: { // 同步mutations
      modifyTitle: (data, inParam) => {  
        // state是immutable类型的数据 inParam用户调用mutation传入的参数
        return data.set("title", inParam); // 注意。必须return修改后的数据才会更新lugiax状态
      }
    }
  }
});
export default titleModel;
```

- model和组件绑定

```javascript
// app.js
import React from "react";
import { bind } from "@lugia/lugiax";
import titleModel from "./model";
class InputTitle extends React.Component<any, any> {
  static displayName = "testInput";
  render() {
    const { props } = this;
    const { title } = this.props;
    return (
      <div>
        <p>{title}</p>
        <div>
          title：
          <input value={title} {...props} />
        </div>
      </div>
    );
  }
}
const App = bind(
  titleModel, // 注册的model
  state => { // 将状态映射到绑定组件的props上
    return { title: state.get("title") };
  },
  {
    onChange(mutations, e) {
      return mutations.modifyTitle(e.target.value);
    }
  }
)(InputTitle);
export default App;

```

###### 2.使用 `bind` 将复杂数据model和组件进行绑定

- 使用bind将model和组件绑定

```javascript
// model.js
import lugaix from "@lugia/lugiax";
const titleState = {
  data: {
    title: "一个测试用例"
  }
};
const modelName = "testTitle";
const titleModel = lugaix.register({
  model: modelName,
  state: titleState,
  mutations: {
    sync: {
      modifyTitle: (state, inParam) => {
        return state.setIn(["data", "title"], inParam);
      }
    }
  }
});
export default titleModel;
```

- 使用bind将model和组件绑定

```javascript
import React from "react";
import { bind } from "@lugia/lugiax";
import titleModel from "./model";
class InputTitle extends React.Component<any, any> {
  static displayName = "testInput";
  render() {
    const { props } = this;
    const { title } = this.props;
    return (
      <div>
        <p>{title}</p>
        <div>
          title：
          <input value={title} {...props} />
        </div>
      </div>
    );
  }
}
const App = bind(
  titleModel,
  state => {
    return { title: state.getIn(["data", "title"]) };
  },
  {
    onChange(mutations, e) {
      return mutations.modifyTitle(e.target.value);
    }
  }
)(InputTitle);
export default App;
```

注意：model中的state是Immutable数据类型。[[更多Immutable数据操作api请参考]](https://facebook.github.io/immutable-js/docs/#/)

###### 3.`bind`方法中 `options` 的使用 

- props  示例 

  ```javascript
  import React from "react";
  import { bind } from "@lugia/lugiax";
  import titleModel from "./model";
  class InputTitle extends React.Component<any, any> {
    static displayName = "testInput";
    render() {
      const { props } = this;
      // 此时title的值 option props传入的值。已经不是注册model下data.title的值
      // props会覆盖 model映射到props中的值
      const { title } = this.props;
      return (
        <div>
          <p>{title}</p>
          <div>
            title：
            <input value={title} {...props} />
          </div>
        </div>
      );
    }
  }
  const App = bind(
    titleModel,
    state => {
      return { title: state.getIn(["data", "title"]) };
    },
    {
      onChange(mutations, e) {
        return mutations.modifyTitle(e.target.value);
      }
    },
    {
      props: { title: "我是Option中传入的数据" }
    }
  )(InputTitle);
  export default App;
  ```
  
- eventHandle 示例   

  ```javascript
  import React from "react";
  import { bind } from "@lugia/lugiax";
  import titleModel from "./model";
  class InputTitle extends React.Component<any, any> {
    static displayName = "testInput";
    render() {
      const { props } = this;
      const { title } = this.props;
      return (
        <div>
          <p>{title}</p>
          <div>
            title：
            <input value={title} {...props} />
          </div>
        </div>
      );
    }
  }
  const App = bind(
    titleModel,
    state => {
      return { title: state.getIn(["data", "title"]) };
    },
    {
      onChange(mutations, e) {
        // 会被调用 如果方法名字相同会依次调用
        console.log("mutations:onChange", e.target.value);
        return mutations.modifyTitle(e.target.value);
      }
    },
    {
      eventHandle: {
        onChange(e) {
          // 会被调用 如果方法名字相同会依次调用
          console.log("eventHandle:onChange", e.target.value);
        }
      }
    }
  )(InputTitle);
  export default App;
  ```


- areStateEqual 示例 

  ```javascript
  import React from "react";
  import { bind } from "@lugia/lugiax";
  import titleModel from "./model";
  class InputTitle extends React.Component<any, any> {
    static displayName = "testInput";
    render() {
      const { props } = this;
      const { title } = this.props;
      return (
        <div>
          <p>{title}</p>
          <div>
            title：
            <input value={title} {...props} />
          </div>
        </div>
      );
    }
  }
  const App = bind(
    titleModel,
    state => {
      return { title: state.getIn(["data", "title"]) };
    },
    {
      onChange(mutations, e) {
        return mutations.modifyTitle(e.target.value);
      }
    },
    {
      // 方法返回true组件才会重新渲染 否则组件不会重新渲染
      areStateEqual(oldModelState, newModelState) {
        // 当输入框输入lugiax时组件不会被重新render
        return newModelState.getIn(["data", "title"]) !== "lugiax";
      }
    }
  )(InputTitle);
  export default App;
  ```

  


- areStatePropsEqual 示例 

  ```javascript
  import React from "react";
  import { bind } from "@lugia/lugiax";
  import titleModel from "./model";
  class InputTitle extends React.Component<any, any> {
    static displayName = "testInput";
    render() {
      const { props } = this;
      const { title } = this.props;
      return (
        <div>
          <p>{title}</p>
          <div>
            title：
            <input value={title} {...props} />
          </div>
        </div>
      );
    }
  }
  const App = bind(
    titleModel,
    state => {
      return { title: state.getIn(["data", "title"]) };
    },
    {
      onChange(mutations, e) {
        return mutations.modifyTitle(e.target.value);
      }
    },
    {
      // 方法返回true组件才会重新渲染 否则组件不会重新渲染
      areStatePropsEqual(oldState, newState) {
        // 当输入框输入lugiax时组件不会被重新render
        return newState.title !== "lugiax";
      }
    }
  )(InputTitle);
  export default App;
  ```

  

- areOwnPropsEqual示例 

  ```javascript
  // app.js
  import React from "react";
  import { bind } from "@lugia/lugiax";
  import titleModel from "./model";
  class InputTitle extends React.Component<any, any> {
    static displayName = "testInput";
    render() {
      const { props } = this;
      const { title } = this.props;
      return (
        <div>
          <p>{title}</p>
          <div>
            title：
            <input value={title} {...props} />
          </div>
        </div>
      );
    }
  }
  const App = bind(
    titleModel,
    state => {
      return { title: state.getIn(["data", "title"]) };
    },
    {
      onChange(mutations, e) {
        return mutations.modifyTitle(e.target.value);
      }
    },
    {
      areOwnPropsEqual(oldProps, newProps) {
        // 当外部props里面title等于 lugiax 时组件不会被重新render
        return newProps.title !== "lugiax";
      }
    }
  )(InputTitle);
  export default App;
  ```

  ```javascript
  import React from "react";
  import App from "./App";
  import { render } from "react-dom";
  export default class Test extends React.Component {
    constructor() {
      super();
      this.state = {
        outTitle: "aaa"
      };
    }
    modifyTitle = () => {
      this.setState({ outTitle: "lugiax" });
    };
    render() {
      const { outTitle: title } = this.state;
      return (
        <div>
          <App title={title} />
          <p>==========================================</p>
          <button onClick={this.modifyTitle}>
            修改父组件的title值从而影响子组件的render
          </button>
        </div>
      );
    }
  }
  ```



#### bindTo

- 描述

  ```
  bindTo是对bind方法的封装。使用者可以省略了自己编写从model取值的代码，只需传入model和组件props的对应关系即可。
  ```

- 方法简述

- ```
  import { bind } from "@lugia/lugiax";
  
  const newComponent = bind(
    model, // object 
    bindConfig,  // object
    eventConfig, // object
    ?option // object
  )(ReactComponent);
  
  ```

 - model

    - 数据类型
      - object
    - 使用lugaix.register() 方法生成的model对象

  - bindConfig

    - 数据类型
      - object | string
    - 当这个值是对象时存储了model和组件Props属性取值的对应关系，bindTo会根据这个对应关系自动生成取值方法并将属性映射到被绑定组件的 `this.props` 中。
    - 当这个值是一个string时,bindTo会将model和组件Props属性取值对应关系会变成{[string]:‘value’}。然后会根据这个对应关系自动生成取值方法并将属性映射到被绑定组件的 `this.props` 中。

  - eventConfig

    - 数据类型
      - object
    - 自定义对象。使用者通过这个对象定义如何使用model中的mutation方法，最终bindTo会将这个对象属性映射到被绑定组件的 `this.props` 中。

  - option对象 表示其他扩展属性

    - props  

      - 数据类型
        - object 
      - 传入一些属性值，bindTo会将这个对象属性映射到被绑定组件的 `this.props` 中。（注：此对象传入的属性优先级最高）

    - eventHandle 

      - 数据类型
        - { [eventName: string] : Function }
      - 传入处理函数，bindTo会将这个对象属性映射到被绑定组件的 `this.props` 中。

    - withRef

      - 数据类型
        - boolean
      - ？？？？？？？？

    - areStateEqual

      - 数据类型
        - (oldModelState, newModelState)  => { return true or false}
        - oldModelState是model改前的数据， newModelState是model改后的数据

      - 用户自定义方法, 当方法返回true时bind后的组件会被 render，当方法返回值为false时bind后的组件不会被render。

    - areStatePropsEqual

      - 数据类型
        - (oldState, newState)  => { return true or false}
        - oldState组件更新前的state数据，newState更新后的state数据。对应的是model映射过来的值

    - areOwnPropsEqual

      - 数据类型
        - (oldProps, newProps)  => { return true or false}
        - oldProps 是bind后的组件props改变前的数据，oldProps 是bind后的组件props改变后的数据，   对应的是bind后返回组件在调用是传入的属性值
      - 用户自定义方法，让这个方法返回true时bind后的组件会被 render，方法返回值为false时bind后的组件不会被render。

#### content
