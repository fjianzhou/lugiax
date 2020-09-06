# Lugiax 是什么

lugiax 是一种解决 react 项目路由和数据状态管理的技术方案。

lugiax 是 lugia 生态中不可缺少的一部分，已直服务了 lugia-mega、lugia-admin 等多个项目。

# Lugiax 设计理念

1. 简化开发难度。开箱即用。lugiax 是为了配合 lugia 生态单独设计开发的。让前端开发者在不需要了解 redux 和 react-router 技术的情况下，也能开发业务。上开发者更多的精力投入到业务开发中。
2. 数据状态使用了不可变数据类型 Immutable。避免了数据篡改。
3. 修改数据状态的唯一方式是通过 mutation。

# Lugiax 目录结构

```
|-- lugiax
| |-- packages
| | |-- lugiax  // model和组件绑定
| | | |-- src
| | | | |-- bind.js
| | | | |-- bindTo.js
| | | | |-- connect.js
| | | | |-- index.js
| | | | |-- utils.js
| | |-- lugiax-common // 发布订阅
| | | |-- src
| | | | |-- index.js
| | | | |-- subscribe.js
| | |-- lugiax-core // 数据存以及更新
| | | |-- src
| | | | |-- combineReducers.js
| | | | |-- index.js
| | | | |-- render.js
| | | | |-- stack.js
| | |-- lugiax-router 路由封装
| | | |-- src
| | | | |-- go.js
| | | | |-- index.js
| | | | |-- Link.js
| | | | |-- Loading.js
| | | | |-- router.js
```

# Lugiax 如何工作

![图片](https://uploader.shimo.im/f/njpLgK9r2k8QGpAY.jpg!thumbnail)

# lugiax 快速上手

# 环境准备

安装[node](https://nodejs.org/en/).js。

也可以选择本机安装 nvm 进行 node.js 的安装管理。

# 安装依赖

## npm

```
npm install @lugia/lugiax
```

## yarn

> 推荐使用 yarn 管理 npm 依赖

```
yarn add @lugia/lugiax
```

# 模型注册

## register 方法

### 描述

> 使用 register 方法注册 model 到 lugiax，通过 bind、bindTo、connect 方法。将注册的 model 和组件进行绑定。

### 方法使用简述

```javascript
import lugaix from '@lugia/lugiax';
lugaix.register({ model: string,state: Object,mutations?: Mutations});
```

### 参数描述

- model
  - 注册 model 名称
  - 数据类型
    - string
- state
  - 注册 model 的初始值，在 lugaix 内部会将 state 对象会被转换成 Immutable 类型
  - 数据类型
    - object
- mutations
  - 存放用户自定义的修改 state 同步或者异步的方法。
  - 数据类型
    - object
  - 语法结构

```
{
  async:{
    async fn1(state,inputParameter,{mutations,getState}){}
    ...
    async fn10(state,inputParameter,{mutations,getState}){}
  },
  sync:{
    fn1(state,inputParameter,{mutations,getState}){}
    ...
    fn10(state,inputParameter,{mutations,getState}){}
  }
}
```

> 备注：async 中的方法。在 register 后会被修改方法名称。eg：fn1 => asyncFn1 ,调用是需要注意。

### 函数返回值

- model
  - model 名称
  - 数据类型
    - string
- mutations
  - 更新 model 中 state 状态的方法
  - 数据类型
    - object
- getState
  - 获取当前 model 最新的 state 数据
  - 数据类型
    - function
- addMutation
  - 向已注册的 model，添加同步更新 state 数据的方法
  - 数据类型
    - function
- addAsyncMutation
  - 向已注册的 model，添加异步修改 state 数据的方法
  - 数据类型
    - function
- destroy
  - 销毁注册的模块
  - 数据类型
    - function

## 示例代码

### 在 lugiax 中注册 model

```
// model.js
import lugaix from '@lugia/lugiax';
import { fromJS } from 'immutable';
const userModelName = 'userModel';const userState = {
  peoples: [],
};
  
export const userModel = lugaix.register({
  model: userModelName,
  state: userState,
  mutations: {
    // 同步修改state的方法
    sync: {
      removePeoplesById(state, inpar, { getState }) {
        const newList = state.get('peoples').filter(item => {
          return item.get('id') !== inpar;
        });
        return state.set('peoples', newList);
      },
    },
    async: {
      // 这里用setTimeout代表异步ajax请求
      async getPeoples(state, inpar, { mutations, getState }) {
        let data = await new Promise(reslove => {
          const peoples = [
            { id: '1', name: '小名', age: 18, hobby: '睡觉' },
            { id: '2', name: '小王', age: 18, hobby: '打游戏' },
            { id: '3', name: '小张', age: 18, hobby: '看电视' },
            { id: '4', name: '小李', age: 18, hobby: '看小说' },
          ];
          setTimeout(() => {
            reslove({ result: 'ok', data: peoples });
          }, 1000);
        });
        if (data.result === 'ok') {
          // 更新state是一定要获取最新的state
          state = getState();
          return state.set('peoples', fromJS(data.data));
        }
      },
    },
  },
});
```

- 返回值

![图片](https://uploader.shimo.im/f/9oaqgh0qSNsDXUjc.jpg!thumbnail)

# 组件绑定

## bind

### 描述

- 将注册的 model 和自定义组件进行双向绑定。在组件中 this.props 中就可以获取到 model 中的 state 数据和用于更新 state 的 mutation 方法
- model 数据改变后组件也会重新 render 被绑定组件。也可以通过调用 mutation 方法去改 model 中的数据，从而引起被绑定组件 render。
  > 备注：一个自定义组件只能 bind 绑定一个 model

### 使用简述

```
import { bind } from "@lugia/lugiax";
constnewComponent = bind(
model,// object
mapModelState, // function
mapModelMutations, // object
?option // object
)(ReactComponent);
```

### 参数描述

- model

  - 使用 lugaix.register() 方法返回的对象
  - 数据类型
    - object

- mapModelState
  - 函数方法，使用者通过这个方法定义如何从 model 中获取的 state 值，并将获取到的 state 值存放在对象中返回，最终 bind 会将返回对象的属性映射到被绑定组件的 this.props 中。
  - 数据类型
    - function
- mapModelMutations
  - 使用者通过这个对象定义如何使用 model 中的 mutation 方法，最终 bind 会将这个对象属性映射到被绑定组件的 this.props 中
  - 数据类型
    - function
- option 其他扩展参数
  _ props
  _ 传入一些属性值，bind 会将这些属性映射到被绑定组件的 this.props 中。
  _ 数据类型
  _ object
  _ eventHandle
  _ 传入处理函数，bind 会将这个对象属性映射到被绑定组件的 this.props 中，当调用组件在 props 中的函数、mapModelMutations 映射的函数、eventHandle 传入的函数名字相同，会将三个函数合并为一个新函数，并在传给被 bind 的组件，这个新函数会依次执行以上三个函数。
  _ 数据类型
  _ { [eventName: string] : Function }
  _ withRef
  _ withRef 是 true 时，可在 bind()后返回的组件的 ref 中拿到被 bind 组件的实例(返回值的 target 属性)。可以调用被 bind 组件上的方法或者属性。如果 withRef 是 false 时，拿到被 bind 组件的实例
  _ 数据类型
  _ boolean
  _ areStateEqual
  _ 用户自定义方法, 当方法返回 true 时 bind 后的组件会被 render，当方法返回值为 false 时 bind 后的组件不会被 render。
  _ 数据类型
  _ (oldModelState, newModelState) => { return true or false}
  _ oldModelState 是 model 改前的数据
  _ newModelState 是 model 改后的数据
  _ areStatePropsEqual
  _ 用户自定义方法，当这个方法返回 true 时 bind 后的组件会被 render，方法返回值为 false 时 bind 后的组件不会被 render。
  _ 数据类型
  _ (oldState, newState) => { return true or false}
  _ oldState 组件更新前的 state 数据，对应的是 model 上 state 映射到 this.props 的值
  _ newState 更新后的 state 数据，对应的是 model 上 state 映射 this.props 的值
  _ areOwnPropsEqual
  _ 用户自定义方法，当这个方法返回 true 时 bind 后的组件会被 render，方法返回值为 false 时 bind 后的组件不会被 render。
  _ 数据类型
  _ (oldProps, newProps) => { return true or false}
  _ oldProps 是 bind 后的组件 props 改变前的数据，对应的是 bind 后返回组件在调用是传入的 props 属性值
  _ oldProps 是 bind 后的组件 props 改变后的数据， 对应的是 bind 后返回组件在调用是传入的 props 属性值
  > 备注：areStateEqual 、areStatePropsEqual 、areOwnPropsEqual 阻止刷新的优先级顺序
  > areStateEqual > areStatePropsEqual > areOwnPropsEqual

## 示例代码

### **使用 bind 将简单数据 model 和组件进行绑定**

- 在 lugaix 中注册 model

```
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

- 使用 bind 将 model 和组件绑定

```
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
  state => { // 将状态映射到被绑定组件的props上
    return { title: state.get("title") };
  },
  {
    onChange(mutations, e) {
      return mutations.modifyTitle(e.target.value);
    }
  }
)(InputTitle);
```

### export default App;**使用 bind 将复杂数据 model 和组件进行绑定**

- 在 lugaix 中注册 model

```
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

- 使用 bind 将 model 和组件绑定

```
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

### **bind 方法中 options 的使用**

### props 示例

在 lugaix 中注册 model

```
import lugaix from '@lugia/lugiax';
const titleState = {
  data: {
    title: '一个测试用例',
  },
};
const modelName = 'testTitle';
const titleModel = lugaix.register({
  model: modelName,
  state: titleState,
  mutations: {
    sync: {
      modifyTitle: (state, inParam) => {
        console.log(inParam);
        return state.setIn(['data', 'title'], inParam);
      },
    },
  },
});
export default titleModel;
```

使用 bind 将 model 和组件绑定

```
import React from "react";
import { bind } from "@lugia/lugiax";
import titleModel from "./model";
class InputTitle extends React.Component<any, any> {
  static displayName = "testInput";
  render() {
    const { props } = this;
    // 此时title的值 option props传入的值。已经不是注册model下data.title的值
    // options.props会覆盖 model映射到props中的值
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

运行结果
![图片](https://uploader.shimo.im/f/45KS2dYIBgIyl5E3.jpg!thumbnail)

> 备注：options.props 属性会覆盖 model 中 state 映射到 props 中的值

### eventHandle 示例

在 lugaix 中注册 model

```
import lugaix from '@lugia/lugiax';
const titleState = {
  data: {
    title: '一个测试用例',
  },
};
const modelName = 'testTitle';
const titleModel = lugaix.register({
  model: modelName,
  state: titleState,
  mutations: {
    sync: {
      modifyTitle: (state, inParam) => {
        return state.setIn(['data', 'title'], inParam);
      },
    },
  },
});
export default titleModel;
```

使用 bind 将 model 和组件绑定

```
import React from 'react';
import { bind } from '@lugia/lugiax';
import titleModel from './model';
class InputTitle extends React.Component<any, any> {
  static displayName = 'testInput';
  render() {
    const { props } = this;
    const { title } = this.props;
    return (
      <div>
        <p>{title}</p>
        <div>
          title：
          <input value={title} {...props} />
        </div>
      </div>
    );
  }
}
const App = bind(
  titleModel,
  state => {
    return { title: state.getIn(['data', 'title']) };
  },
  {
    onChange(mutations, e) {
      // 会被调用 如果方法名字相同会依次调用
      console.log('mutations:onChange', e.target.value);
      return mutations.modifyTitle(e.target.value);
    },
  },
  {
    eventHandle: {
      onChange(e) {
        // 会被调用 如果方法名字相同会依次调用
        console.log('eventHandle:onChange', e.target.value);
      },
    },
  }
)(InputTitle);
const outAppOnChange = e => {
  console.log('outAppOnChange:onChange', e.target.value);
};
const OutApp = () => {
  return <App onChange={outAppOnChange}></App>;
};
export default OutApp;
```

运行结果
![图片](https://uploader.shimo.im/f/qWwTGKvls2wKedFL.jpg!thumbnail)

> 备注：调用组件在 props 中的函数、mapModelMutations 映射的函数、eventHandle 传入的函数名字相同，会将三个函数合并为一个新函数，并在传给被 bind 的组件，这个新函数会依次执行以上三个函数。

### areStateEqual 示例

在 lugaix 中注册 model

```
import lugaix from '@lugia/lugiax';
const titleState = {
  data: {
    title: '一个测试用例',
  },
};
const modelName = 'testTitle';
const titleModel = lugaix.register({
  model: modelName,
  state: titleState,
  mutations: {
    sync: {
      modifyTitle: (state, inParam) => {
        return state.setIn(['data', 'title'], inParam);
      },
    },
  },
});
export default titleModel;
```

使用 bind 将 model 和组件绑定

```
import React from 'react';
import { bind } from '@lugia/lugiax';
import titleModel from './model';
class InputTitle extends React.Component<any, any> {
  static displayName = 'testInput';
  render() {
    const { props } = this;
    const { title } = this.props;
    return (
      <div>
        <p>{title}</p>
        <div>
          title：
          <input value={title} {...props} />
        </div>
      </div>
    );
  }
}
const App = bind(
  titleModel,
  state => {
    return { title: state.getIn(['data', 'title']) };
  },
  {
    onChange(mutations, e) {
      return mutations.modifyTitle(e.target.value);
    },
  },
  {
    // 方法返回true组件才会重新渲染 否则组件不会重新渲染
    areStateEqual(oldModelState, newModelState) {
      // 当输入框输入lugiax时组件不会被重新render
      const flag = newModelState.getIn(['data', 'title']) !== 'lugiax';
      if (!flag) {
        console.log('输入的值是lugiax,视图不能被更新');
      }
      return flag;
    },
  }
)(InputTitle);
export default App;
```

运行结果
![图片](https://uploader.shimo.im/f/n1XtSrEWU2AVbCEh.png!thumbnail)

### areStatePropsEqual 示例

在 lugaix 中注册 model

```
import lugaix from '@lugia/lugiax';
const titleState = {
  data: {
    title: '一个测试用例',
  },
};
const modelName = 'testTitle';
const titleModel = lugaix.register({
  model: modelName,
  state: titleState,
  mutations: {
    sync: {
      modifyTitle: (state, inParam) => {
        return state.setIn(['data', 'title'], inParam);
      },
    },
  },
});
export default titleModel;
```

使用 bind 将 model 和组件绑定

```
import React from 'react';
import { bind } from '@lugia/lugiax';
import titleModel from './model';
class InputTitle extends React.Component<any, any> {
  static displayName = 'testInput';
  render() {
    const { props } = this;
    const { title } = this.props;
    return (
      <div>
        <p>{title}</p>
        <div>
          title：
          <input value={title} {...props} />
        </div>
      </div>
    );
  }
}
const App = bind(
  titleModel,
  state => {
    return { title: state.getIn(['data', 'title']) };
  },
  {
    onChange(mutations, e) {
      return mutations.modifyTitle(e.target.value);
    },
  },
  {
    // 方法返回true组件才会重新渲染 否则组件不会重新渲染
    areStatePropsEqual(oldState, newState) {
      // 当输入框输入lugiax时组件不会被重新render
      const flag = newState.title !== 'lugiax';
      if (!flag) {
        console.log('输入的值是lugiax,视图不能被更新');
      }
      return flag;
    },
  }
)(InputTitle);
export default App;
```

运行结果
![图片](https://uploader.shimo.im/f/BlQ0N5ndR3ElmolP.png!thumbnail)

### areOwnPropsEqual 示例

在 lugaix 中注册 model

```
import lugaix from '@lugia/lugiax';
const titleState = {
  data: {
    title: '一个测试用例',
  },
};
const modelName = 'testTitle';
const titleModel = lugaix.register({
  model: modelName,
  state: titleState,
  mutations: {
    sync: {
      modifyTitle: (state, inParam) => {
        return state.setIn(['data', 'title'], inParam);
      },
    },
  },
});
export default titleModel;
```

使用 bind 将 model 和组件绑定

```
// app.js
import React from 'react';
import { bind } from '@lugia/lugiax';
import titleModel from './model';
class InputTitle extends React.Component<any, any> {
  static displayName = 'testInput';
  render() {
    const { props } = this;
    const { title } = this.props;
    return (
      <div>
        <p>{title}</p>
        <div>
          title：
          <input value={title} {...props} />
        </div>
      </div>
    );
  }
  }
  const App = bind(
    titleModel,
    state => {
      return { title: state.getIn(['data', 'title']) };
    },
    {
    onChange(mutations, e) {
      return mutations.modifyTitle(e.target.value);
    },
  },
  {
    areOwnPropsEqual(oldProps, newProps) {
      // 当外部props里面title等于 lugiax 时组件不会被重新render
      const flag = newProps.title !== 'lugiax';
      if (!flag) {
        console.log('输入的值是lugiax,视图不能被更新');
      }
      return flag;
    },
  }
)(InputTitle);export default class Test extends React.Component {
  constructor() {
    super();
    this.state = {
      outTitle: 'aaa',
    };
  }
  modifyTitle = inputParameter => {
    this.setState(inputParameter);
  };
  render() {
    const { outTitle: title } = this.state;
    return (
      <div>
        <App title={title} />
        <p>==========================================</p>
        <button
          onClick={() => {
            console.log('改变title的值为lugiax,阻止组件刷新');
            this.modifyTitle({ outTitle: 'lugiax' });
          }}
        >
          改变title的值为lugiax,阻止组件刷新
        </button>
        <br></br>
        <button
          onClick={() => {
            console.log('改变title的值为其他值,组件正常渲染');
            this.modifyTitle({ outTitle: '其他值' });
          }}
        >
          改变title的值为其他值,组件正常渲染
        </button>
      </div>
    );
  }
}
```

运行结果
![图片](https://uploader.shimo.im/f/yso8xkJRzmwOpDxq.png!thumbnail)

### withRef 示例

在 lugaix 中注册 mode

```
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

使用 bind 将 model 和组件绑定

```
// App.js
import React from 'react';
import { bind } from '@lugia/lugiax';
import titleModel from './model';
class InputTitle extends React.Component<any, any> {
  static displayName = 'testInput';
  content = '我是被bind的组件中的值';
  getComponentData() {
    return this.content;
  }
  render() {
    const { props } = this;
    const { title } = this.props;
    return (
      <div style={{ border: '1px solid red', padding: '20px' }}>
        <p style={{ padding: 0, margin: 0 }}>被bind的组件</p>
        <div>
          title：
          <input value={title} {...props} />
        </div>
      </div>
    );
  }
}
const App = bind(
  titleModel,
  state => {
    return { title: state.getIn(['data', 'title']) };
  },
  {},
  {
    withRef: true,
  }
)(InputTitle);export default class Test extends React.Component {
  constructor() {
    super();
    this.state = {
      outTitle: 'aaa',
    };
  }
  showComponentData = () => {
    // 拿到被bind组件的实例。并调用组件中的方法（注：target是固定的）
    console.log(this.myInput.target.getComponentData());
  };
  render() {
    const { outTitle: title } = this.state;
    return (
      <div>
        <App title={title} ref={cmp => (this.myInput = cmp)} />
        <p>==========================================</p>
        <button onClick={this.showComponentData}>调用被bind组件中的方法</button>
      </div>
    );
  }
}
```

运行结果
![图片](https://uploader.shimo.im/f/nCkyWvFlN80PqqMA.jpg!thumbnail)

## bindTo

### 描述

- bindTo 是对 bind 方法的封装。使用者可以省略了自己编写从 model 取值的代码，只需传入 model 中 state 属性和组件 props 属性的对应关系即可。
  > 备注：一个自定义组件只能 bind 绑定一个 model

### 使用简述

```
import { bind } from "@lugia/lugiax";
const newComponent = bind(
model,// object
bindConfig,// object || string
eventConfig,// object
?option// object
)(ReactComponent);
```

### 参数描述

- model
  - 使用 lugaix.register() 方法返回的 model 对象
  - 数据类型
    - object
- bindConfig
  - model 中 state 属性映射到组件 props 属性的对应关系，对象 value 值表示 props 的属性对象的 key 值表示 model 中 state 属性。props[value] ===model.state[key]
  - 数据类型
    - object | string
      - 当这个值是对象时存储了 model 中 state 和组件 Props 属性取值的对应关系，bindTo 会根据这个对应关系自动生成取值方法并将属性映射到被绑定组件的 this.props 中。
      - 当这个值是一个 string 时,bindTo 会将 model 中 state 和组件 Props 属性取值对应关系会变成{[string]:‘value’}。然后会根据这个对应关系自动生成取值方法并将属性映射到被绑定组件的 this.props 中。
- eventConfig
  - 使用者通过这个对象定义如何使用 model 中的 mutation 方法，最终 bindTo 会将这个对象属性映射到被绑定组件的 this.props 中。
  - 数据类型
    - object
- option 其他扩展参数
  - props
    - 传入一些属性值，bindTo 会将这个对象属性映射到被绑定组件的 this.props 中。
    - 数据类型
      - object
  - eventHandle
    - 传入处理函数，bindTo 会将这个对象属性映射到被绑定组件的 this.props 中。调用组件在 props 中的函数、mapModelMutations 映射的函数、eventHandle 传入的函数名字相同，会将三个函数合并为一个新函数，并在传给被 bind 的组件，这个新函数会依次执行以上三个函数。
    - 数据类型
      - { [eventName: string] : Function }
  - withRef
    - withRef 是 true 时，可在 bind()后返回的组件的 ref 中拿到被 bind 组件的实例。可以调用被 bind 组件上的方法或者属性(返回值的 target 属性)。如果 withRef 是 false 时，拿到被 bind 组件的实例
    - 数据类型
      - boolean
  - areStateEqual
    - 用户自定义阻止组件 render 函数, 当方法返回 true 时 bind 后的组件会被 render，当方法返回值为 false 时 bind 后的组件不会被 render。
    - 数据类型
      - (oldModelState, newModelState) => { return true or false}
        - oldModelState 是 model 改前的数据
        - newModelState 是 model 改后的数据
  - areStatePropsEqual
    - 用户自定义方法，当这个方法返回 true 时 bind 后的组件会被 render，方法返回值为 false 时 bind 后的组件不会被 render。
    - 数据类型
      - (oldState, newState) => { return true or false}
        - oldState 组件更新前的 state 数据，对应的是 model 上 state 映射到 this.props 的值
        - newState 更新后的 state 数据，对应的是 model 上 state 映射 this.props 的值
    -     * areOwnPropsEqual
    - 用户自定义方法，当这个方法返回 true 时 bind 后的组件会被 render，方法返回值为 false 时 bind 后的组件不会被 render。
    - 数据类型
      - (oldProps, newProps) => { return true or false}
        - oldProps 是 bind 后的组件 props 改变前的数据，对应的是 bind 后返回组件在调用是传入的 props 属性值
        - oldProps 是 bind 后的组件 props 改变后的数据， 对应的是 bind 后返回组件在调用是传入的 props 属性值

## 示例代码

### **使用 bindTo 将简单数据 model 和组件进行绑定**

在 lugaix 中注册 mode

```
// model.js
import lugaix from "@lugia/lugiax";
const titleState = {
  title: "一个测试用例"
};
const modelName = "testTitle";
const titleModel = lugaix.register({
  model: modelName,
  state: titleState,
  //bindTo会自动添加onChange对应的mutations方法。
  mutations: {
  }
});
export default titleModel;
```

使用 bindTo  将 model 和组件绑定

```
// app.js
import React from 'react';
import { bindTo } from '@lugia/lugiax';
import titleModel from './model';
class InputTitle extends React.Component<any, any> {
  static displayName = 'testInput';
  render() {
    const { props } = this;
    const { value } = this.props;
    return (
      <div>
        <p>{value}</p>
        <div>
          title：
          <input {...this.props} />
        </div>
      </div>
    );
  }
}
//bindTo会自动将原生的onChange对应的mutations方法进行关联
const App = bindTo(
  titleModel,
  {
    title: 'value',
  }
  //如果想更精细的控制onChange后的值可以使用如下操作
  // ,{
  //   onChange: {
  //     title(e) {
  //       // Todo
  //       console.log(`onChange值：${e.target.value}`);
  //       return e.target.value;
  //     },
  //   },
  // }
)(InputTitle);
export default App;
```

### **使用 bindTo 将复杂数据 model 和组件进行绑定**

在 lugaix 中注册 mode

```
// model.js
import lugaix from '@lugia/lugiax';
const titleState = {
form: {
title: '嵌套数据',
},
};
const modelName = 'testTitle';
const titleModel = lugaix.register({
model: modelName,
state: titleState,
mutations: {},
});
export default titleModel;
```

使用 bindTo  将 model 和组件绑定

```
// app.js
import React from 'react';
import { bindTo } from '@lugia/lugiax';
import titleModel from './model';
class InputTitle extends React.Component<any, any> {
  static displayName = 'testInput';
  render() {
    const { props } = this;
    const { value } = this.props;
    return (
      <div>
        <p>{value}</p>
        <div>
          title：
          <input {...this.props} />
        </div>
      </div>
    );
  }
}
//bindTo会自动将原生的onChange对应的mutations方法进行关联
const App = bindTo(
  titleModel,
  {
    'form.title': 'value',
  },
  //如果想更精细的控制onChange后的值可以使用如下操作
  {
    onChange: {
      'form.title': e => {
        console.log('2222', e.target.value);
        return e.target.value;
      },
    },
  }
)(InputTitle);
export default App;
```

## connect

### 描述

- 将注册的一个或者多个 model 和组件进行绑定。在组件中 props 中就可以获取到 model 中 state 的数据。可以通过调用 mutation 方法去改 model 中的数据，从而引起被绑定组件 render。

### 使用简述

```
import { connect } from "@lugia/lugiax";
const newComponent = connect(
  model,// object || Arrray<Object>
mapModelStateToProps,// function
mapModelToMutationsToProps, // function
?option// object
)(ReactComponent);
```

### 参数描述

- model
  - 使用 lugaix.register() 方法返回的 model 对象
  - 数据类型
    - object | Array<object>
- mapModelStateToProps
  - connect 会根据这个方法返回的对象。将 model 里的 state 属性映射到 this.props 属性上
  - (modelState) => {return {}}
    - 当 connect 单个 model 时。modelState 是当前 model 里的 state 对象
    - 当 connect 多个 model 时。modelState 是 model 里的 state 对象的数组。并且这个数组的顺序和 connect 时传入的 model 数组的顺序是一致的。
- mapModelToMutationsToProps
  - connect 会根据这个方法返回的对象。将 model 里的 mutations 映射到 this.props 中.
  - 数据类型
    - (modelMutations) => { return {} }
      - 当 connect 单个 model 是。modelMutations 是当前 model 里的 mutations 对象
      - 当 connect 多个 model 时。modelMutations 是 model 里的 mutations 对象的数组。并且这个数组的顺序 connect 时传入的 model 数组的顺序是一致的。
- option 其他扩展参数
  - props
    - 传入一些属性值，connect 会将这个对象属性映射到被绑定组件的 this.props 中。
    - 数据类型
      - object
  - eventHandle
    - 传入处理函数，connect 会将这个对象属性映射到被绑定组件的 this.props 中。调用组件在 props 中的函数、mapModelMutations 映射的函数、eventHandle 传入的函数名字相同，会将三个函数合并为一个新函数，并在传给被 bind 的组件，这个新函数会依次执行以上三个函数。
    - 数据类型
      - { [eventName: string] : Function }
  - areStateEqual
    - 用户自定义方法, 当方法返回 true 时 connect 后的组件会被 render，当方法返回值为 false 时 connect 后的组件不会被 render。
    - 数据类型
      - (oldModelState, newModelState) => { return true or false}
        - oldModelState 是 model 改前的数据
        - newModelState 是 model 改后的数据
  - areStatePropsEqual
    - 用户自定义方法，当这个方法返回 true 时 bind 后的组件会被 render，方法返回值为 false 时 bind 后的组件不会被 render。
    - 数据类型
      - (oldState, newState) => { return true or false}
        - oldState 组件更新前的 state 数据，对应的是 model 上 state 映射到 this.props 的值
        - newState 更新后的 state 数据，对应的是 model 上 state 映射到 this.props 的值
  - areOwnPropsEqual
    - 用户自定义方法，让这个方法返回 true 时 connect 后的组件会被 render，方法返回值为 false 时 connect 后的组件不会被 render。
    - 数据类型
      - (oldProps, newProps) => { return true or false}
        - oldProps 是 connect 后的组件 props 改变前的数据，对应的是 bind 后返回组件在调用是传入的 props 属性值
        - oldProps 是 bind 后的组件 props 改变后的数据，对应的是 bind 后返回组件在调用是传入的 props 属性值
  - withRef
    - withRef 是 true 时，可在 bind()后返回的组件的 ref 中拿到被 bind 组件的实例。可以调用被 bind 组件上的方法或者属性(返回值的 target 属性)。如果 withRef 是 false 时，拿到被 bind 组件的实例
    - 数据类型
      - boolean

## 示例代码

### **使用 content 将单个 model 和组件进行绑定，并调用 mutations 操作数据**

在 lugaix 中注册 mode

```
// model.js
import lugaix from '@lugia/lugiax';
const userState = {
  peoples: [
    { id: '1', name: '小名', age: 18, hobby: '睡觉' },
    { id: '2', name: '小王', age: 18, hobby: '打游戏' },
    { id: '3', name: '小张', age: 18, hobby: '看电视' },
    { id: '4', name: '小李', age: 18, hobby: '看小说' },
  ],
};
const modelName = 'userInfo';
export const userModel = lugaix.register({
  model: modelName,
  state: userState,
  mutations: {
    sync: {
      // 同步删除方法
      removePeopleById(state, inpar, { mutations }) {
        // 获取的数据类型为 Immutable
        const list = state.get('peoples');
     	// Immutable数据的操作方法请参考官网
        const newList = list.filter(item => {
          return item.get('id') != inpar.id;
        });
        return state.set('peoples', newList);
      },
    },
    async: {
      // 异步删除方法。这里用setTimeout代表异步ajax请求。请求删除数据
      async removePeopleById(state, inpar, { mutations, getState }) {
        let data = await new Promise(reslove => {
          setTimeout(() => {
            reslove({ result: 'ok' });
          }, 1000);
        });
        if (data.result === 'ok') {
          // mutations.removePeopleById调用过后会返回最新的state,
          // 这里一定要返回最新的state。
          // state = mutations.removePeopleById(inpar);
          // 或者调用完mutation方法后使用getState方法获取最新的state并返回
          mutations.removePeopleById(inpar);
          state = getState();
        }
        return state;
      },
    },
  },
});
```

使用 connect 将 model 和组件绑定

```
// app.js
import React from 'react';
import { connect } from '@lugia/lugiax';
import { userModel } from './model';
class UserList extends React.Component {
  state = { loading: false };
  constructor(props) {
    super(props);
  }
  render() {
    const { list, removePeopleById, asyncRemovePeopleById } = this.props;
    return (
      <div>
        {list.map(item => {
          console.log('item', item);
          return (
            <p>
              <span>姓名：{item.name}</span>
              <span>&nbsp;年龄：{item.age}</span>
              <span>&nbsp;爱好：{item.hobby}</span>
              <span
                onClick={() => {
                  asyncRemovePeopleById(item);
                }}
              >
                &nbsp;异步删除
              </span>
              <span
                onClick={() => {
                  removePeopleById(item);
                }}
              >
                &nbsp;同步删除
              </span>
            </p>
          );
        })}
      </div>
    );
  }
}
const App = connect(
  userModel,
  state => {
    const list = state.get('peoples').toJS ? state.get('peoples').toJS() : state.get('peoples');
    return { list };
  },
  mutations => {
    console.log(mutations);
    const { removePeopleById, asyncRemovePeopleById } = mutations;
    return {
      removePeopleById,
      asyncRemovePeopleById,
    };
  }
)(UserList);
export default App;
```

### **使用 content 将单个 model 和组件进行绑定 - componentDidMount 调用多个 mutation 渲染页面**

在 lugaix 中注册 mode

```
// model.js
import lugaix from '@lugia/lugiax';
const userState = {
  peoples: [],
  subject: [],
  schoolInfo: {},
};
const modelName = 'userInfo';
export const userModel = lugaix.register({
  model: modelName,
  state: userState,
  mutations: {
    async: {
      // 这里用setTimeout代表异步ajax请求
      async getPeoples(state, inpar, { mutations, getState }) {
        let data = await new Promise(reslove => {
          const peoples = [
            { id: '1', name: '小名', age: 18, hobby: '睡觉' },
            { id: '2', name: '小王', age: 18, hobby: '打游戏' },
            { id: '3', name: '小张', age: 18, hobby: '看电视' },
            { id: '4', name: '小李', age: 18, hobby: '看小说' },
          ];
          setTimeout(() => {
            reslove({ result: 'ok', data: peoples });
          }, 1000);
        });
        if (data.result === 'ok') {
          // 更新state是一定要获取最新的state
          state = getState();
          return state.set('peoples', data.data);
        }
      },
      // 这里用setTimeout代表异步ajax请求
      async getSubject(state, inpar, { mutations, getState }) {
        const subject = [
          '理学',
          '工学',
          '文学',
          '艺术学',
          '历史学',
          '哲学',
          '经济学',
          '管理学',
          '法学',
          '教育学',
        ];
        let data = await new Promise(reslove => {
          setTimeout(() => {
            reslove({ result: 'ok', data: subject });
          }, 1000);
        });
        if (data.result === 'ok') {
          // 更新state是一定要获取最新的state
          state = getState();
          return state.set('subject', data.data);
        }
      },
      // 这里用setTimeout代表异步ajax请求
      async getsChoolInfo(state, inpar, { mutations, getState }) {
        const schoolInfo = {
          name: '北京清华大学',
          schoolMotto: '自强不息,厚德载物',
          address: '北京市海淀区双清路30号。',
        };
        let data = await new Promise(reslove => {
          setTimeout(() => {
            reslove({ result: 'ok', data: schoolInfo });
          }, 1000);
        });
        if (data.result === 'ok') {
          // 更新state是一定要获取最新的state
          state = getState();
          return state.set('schoolInfo', data.data);
        }
      },
    },
  },
});
```

使用 connect 将 model 和组件绑定

```
// app.js
import React from 'react';
import { connect } from '@lugia/lugiax';
import { userModel } from './model';
class UserList extends React.Component {
  state = { loading: false };
  constructor(props) {
    super(props);
  }
  componentDidMount() {
    const { asyncGetPeoples, asyncGetSubject, asyncGetsChoolInfo } = this.props;
    asyncGetPeoples();
    asyncGetSubject();
    asyncGetsChoolInfo();
  }
  render() {
    const { peoples, subject, schoolInfo } = this.props;
    return (
      <div>
        <div>
          <h2>学校信息</h2>
          <p>
            <span>&nbsp;校名：{schoolInfo.name}</span>
            <span>&nbsp;校训：{schoolInfo.schoolMotto}</span>
            <span>&nbsp;地址：{schoolInfo.address}</span>
          </p>
        </div>
        <div>
          <h2>学校专业</h2>
          <p>
            {subject.map(item => {
              return <span>【{item}】</span>;
            })}
          </p>
        </div>
        <h2>学员信息</h2>
        {peoples.map(item => {
          return (
            <p>
              <span>姓名：{item.name}</span>
              <span>&nbsp;年龄：{item.age}</span>
              <span>&nbsp;爱好：{item.hobby}</span>
            </p>
          );
        })}
      </div>
    );
  }
}
const App = connect(
  userModel,
  state => {
    const peoples = state.get('peoples').toJS ? state.get('peoples').toJS() : state.get('peoples');
    const subject = state.get('subject').toJS ? state.get('subject').toJS() : state.get('subject');
    const schoolInfo = state.get('schoolInfo').toJS
      ? state.get('schoolInfo').toJS()
      : state.get('schoolInfo');
    return { peoples, subject, schoolInfo };
  },
  mutations => {
    const { asyncGetPeoples, asyncGetSubject, asyncGetsChoolInfo } = mutations;
    return {
      asyncGetPeoples,
      asyncGetSubject,
      asyncGetsChoolInfo,
    };
  }
)(UserList);
export default App;
```

### **使用 content 将多个 model 和组件进行绑定**

在 lugaix 中注册 mode

```
// model.js
import lugaix from '@lugia/lugiax';
const userModelName = 'userModel';
const subjectModelName = 'subjectModel';
const schoolModelName = 'schoolModel';const userState = {
  peoples: [],
};
const subjectState = {
  subject: [],
};
const schoolState = {
  schoolInfo: {},
};export const userModel = lugaix.register({
  model: userModelName,
  state: userState,
  mutations: {
    async: {
      // 这里用setTimeout代表异步ajax请求
      async getPeoples(state, inpar, { mutations, getState }) {
        let data = await new Promise(reslove => {
          const peoples = [
            { id: '1', name: '小名', age: 18, hobby: '睡觉' },
            { id: '2', name: '小王', age: 18, hobby: '打游戏' },
            { id: '3', name: '小张', age: 18, hobby: '看电视' },
            { id: '4', name: '小李', age: 18, hobby: '看小说' },
          ];
          setTimeout(() => {
            reslove({ result: 'ok', data: peoples });
          }, 1000);
        });
        if (data.result === 'ok') {
          // 更新state是一定要获取最新的state
          state = getState();
          return state.set('peoples', data.data);
        }
      },
    },
  },
});export const subjectModel = lugaix.register({
  model: subjectModelName,
  state: subjectState,
  mutations: {
    async: {
      // 这里用setTimeout代表异步ajax请求
      async getSubject(state, inpar, { mutations, getState }) {
        const subject = [
          '理学',
          '工学',
          '文学',
          '艺术学',
          '历史学',
          '哲学',
          '经济学',
          '管理学',
          '法学',
          '教育学',
        ];
        let data = await new Promise(reslove => {
          setTimeout(() => {
            reslove({ result: 'ok', data: subject });
          }, 1000);
        });
        if (data.result === 'ok') {
          // 更新state是一定要获取最新的state
          state = getState();
          return state.set('subject', data.data);
        }
      },
    },
  },
});export const schoolModel = lugaix.register({
  model: schoolModelName,
  state: schoolState,
  mutations: {
    async: {
      // 这里用setTimeout代表异步ajax请求
      async getsChoolInfo(state, inpar, { mutations, getState }) {
        const schoolInfo = {
          name: '北京清华大学',
          schoolMotto: '自强不息,厚德载物',
          address: '北京市海淀区双清路30号。',
        };
        let data = await new Promise(reslove => {
          setTimeout(() => {
            reslove({ result: 'ok', data: schoolInfo });
          }, 1000);
        });
        if (data.result === 'ok') {
          // 更新state是一定要获取最新的state
          state = getState();
          return state.set('schoolInfo', data.data);
        }
      },
    },
  },
});
```

使用 connect 将 model 和组件绑定

```
// app.js
import React from 'react';
import { connect } from '@lugia/lugiax';
import { userModel, subjectModel, schoolModel } from './model';
class UserList extends React.Component {
  state = { loading: false };
  constructor(props) {
    super(props);
  }
  componentDidMount() {
    const { asyncGetPeoples, asyncGetSubject, asyncGetsChoolInfo } = this.props;
    asyncGetPeoples();
    asyncGetSubject();
    asyncGetsChoolInfo();
  }
  render() {
    const { peoples, subject, schoolInfo } = this.props;
    return (
      <div>
        <div>
          <h2>学校信息</h2>
          <p>
            <span>&nbsp;校名：{schoolInfo.name}</span>
            <span>&nbsp;校训：{schoolInfo.schoolMotto}</span>
            <span>&nbsp;地址：{schoolInfo.address}</span>
          </p>
        </div>
        <div>
          <h2>学校专业</h2>
          <p>
            {subject.map(item => {
              return <span>【{item}】</span>;
            })}
          </p>
        </div>
        <h2>学员信息</h2>
        {peoples.map(item => {
          return (
            <p>
              <span>姓名：{item.name}</span>
              <span>&nbsp;年龄：{item.age}</span>
              <span>&nbsp;爱好：{item.hobby}</span>
            </p>
          );
        })}
      </div>
    );
  }
}
const App = connect(
  [userModel, subjectModel, schoolModel],
  state => {
    // 绑定多个model获取state状态如下，模块状态和传入model的顺序一致
    const [userModelState, subjectModelState, schoolModelState] = state;
    const peoples = userModelState.get('peoples').toJS
      ? userModelState.get('peoples').toJS()
      : userModelState.get('peoples');
    const subject = subjectModelState.get('subject').toJS
      ? subjectModelState.get('subject').toJS()
      : subjectModelState.get('subject');
    const schoolInfo = schoolModelState.get('schoolInfo').toJS
      ? schoolModelState.get('schoolInfo').toJS()
      : schoolModelState.get('schoolInfo');
    return { peoples, subject, schoolInfo };
  },
  mutations => {
    // 绑定多个model获取mutation如下，模块mutation和传入model的顺序一致
    const [userMutation, subjectMutation, schoolMutation] = mutations;
    return {
      asyncGetPeoples: userMutation.asyncGetPeoples,
      asyncGetSubject: subjectMutation.asyncGetSubject,
      asyncGetsChoolInfo: schoolMutation.asyncGetsChoolInfo,
    };
  }
)(UserList);
export default App;
```

### **connect 方法中 options 的使用**

### areStateEqual connect 单个 model 示例

在 lugaix 中注册 mode

```
// model.js
import lugaix from '@lugia/lugiax';
import { fromJS } from 'immutable';
const userState = {
  peoples: [],
  subject: [],
  schoolInfo: {},
};
const modelName = 'userInfo';
export const userModel = lugaix.register({
  model: modelName,
  state: userState,
  mutations: {
    sync: {
      removePeoplesById(state, inpar, { getState }) {
        const newList = state.get('peoples').filter(item => {
          return item.get('id') !== inpar;
        });
        return state.set('peoples', newList);
      },
    },
    async: {
      // 这里用setTimeout代表异步ajax请求
      async getPeoples(state, inpar, { mutations, getState }) {
        let data = await new Promise(reslove => {
          const peoples = [
            { id: '1', name: '小名', age: 18, hobby: '睡觉' },
            { id: '2', name: '小王', age: 18, hobby: '打游戏' },
            { id: '3', name: '小张', age: 18, hobby: '看电视' },
            { id: '4', name: '小李', age: 18, hobby: '看小说' },
          ];
          setTimeout(() => {
            reslove({ result: 'ok', data: peoples });
          }, 1000);
        });
        if (data.result === 'ok') {
          // 更新state是一定要获取最新的state
          state = getState();
          return state.set('peoples', fromJS(data.data));
        }
      },
      // 这里用setTimeout代表异步ajax请求
      async getSubject(state, inpar, { mutations, getState }) {
        const subject = [
          '理学',
          '工学',
          '文学',
          '艺术学',
          '历史学',
          '哲学',
          '经济学',
          '管理学',
          '法学',
          '教育学',
        ];
        let data = await new Promise(reslove => {
          setTimeout(() => {
            reslove({ result: 'ok', data: subject });
          }, 1000);
        });
        if (data.result === 'ok') {
          // 更新state是一定要获取最新的state
          state = getState();
          return state.set('subject', fromJS(data.data));
        }
      },
      // 这里用setTimeout代表异步ajax请求
      async getsChoolInfo(state, inpar, { mutations, getState }) {
        const schoolInfo = {
          name: '北京清华大学',
          schoolMotto: '自强不息,厚德载物',
          address: '北京市海淀区双清路30号。',
        };
        let data = await new Promise(reslove => {
          setTimeout(() => {
            reslove({ result: 'ok', data: schoolInfo });
          }, 1000);
        });
        if (data.result === 'ok') {
          // 更新state是一定要获取最新的state
          state = getState();
          return state.set('schoolInfo', fromJS(data.data));
        }
      },
    },
  },
});
```

使用 connect 将 model 和组件绑定

```
// app.js
import React from 'react';
import { connect } from '@lugia/lugiax';
import { userModel } from './model';
class UserList extends React.Component {
  state = { loading: false };
  constructor(props) {
    super(props);
  }
  componentDidMount() {
    const { asyncGetPeoples, asyncGetSubject, asyncGetsChoolInfo } = this.props;
    asyncGetPeoples();
    asyncGetSubject();
    asyncGetsChoolInfo();
  }
  removePeoplesById(id) {
    const { removePeoplesById } = this.props;
    removePeoplesById(id);
  }
  render() {
    const { peoples, subject, schoolInfo } = this.props;
    return (
      <div>
        <div>
          <h2>学校信息</h2>
          <p>
            <span>&nbsp;校名：{schoolInfo.name}</span>
            <span>&nbsp;校训：{schoolInfo.schoolMotto}</span>
            <span>&nbsp;地址：{schoolInfo.address}</span>
          </p>
        </div>
        <div>
          <h2>学校专业</h2>
          <p>
            {subject.map(item => {
              return <span>【{item}】</span>;
            })}
          </p>
        </div>
        <h2>学员信息</h2>
        <p style={{ color: 'red' }}>用areStateEqual实现当学员少于2条时也面部刷新</p>
        {peoples.map(item => {
          return (
            <p>
              <span>姓名：{item.name}</span>
              <span>&nbsp;年龄：{item.age}</span>
              <span>&nbsp;爱好：{item.hobby}</span>
              <span
                onClick={() => {
                  this.removePeoplesById(item.id);
                }}
              >
                &nbsp;删除
              </span>
            </p>
          );
        })}
      </div>
    );
  }
}const App = connect(
  userModel,
  state => {
    const peoples = state.get('peoples').toJS ? state.get('peoples').toJS() : state.get('peoples');
    const subject = state.get('subject').toJS ? state.get('subject').toJS() : state.get('subject');
    const schoolInfo = state.get('schoolInfo').toJS
      ? state.get('schoolInfo').toJS()
      : state.get('schoolInfo');
    return { peoples, subject, schoolInfo };
  },
  mutations => {
    const { asyncGetPeoples, asyncGetSubject, asyncGetsChoolInfo, removePeoplesById } = mutations;
    return {
      asyncGetPeoples,
      asyncGetSubject,
      asyncGetsChoolInfo,
      removePeoplesById,
    };
  },
  {
    // 返回true 渲染组件  返回false 不渲染组件
    // 当connect单个model时 oldState表示这个model修改前的State值
    // 当connect单个model时 newState表示这个model修改后的State值
    areStateEqual(oldState, newState) {
      // 当peoples数据少于2条不渲染组件
      const flag = newState.get('peoples').size > 2;
      if (!flag) {
        console.log('当peoples数据少于2条组件渲染被阻止');
      }
      return flag;
    },
  }
)(UserList);
export default App;
```

运行结果
![图片](https://uploader.shimo.im/f/sUJu3mpU3b8dkXuT.jpg!thumbnail)

### areStateEqual connect 多个 model 示例

在 lugaix 中注册 mode

```
// model.js
import lugaix from '@lugia/lugiax';
import { fromJS } from 'immutable';
const userModelName = 'userModel';
const subjectModelName = 'subjectModel';
const schoolModelName = 'schoolModel';const userState = {
  peoples: [],
};
const subjectState = {
  subject: [],
};
const schoolState = {
  schoolInfo: {},
};export const userModel = lugaix.register({
  model: userModelName,
  state: userState,
  mutations: {
    sync: {
      removePeoplesById(state, inpar, { getState }) {
        const newList = state.get('peoples').filter(item => {
          return item.get('id') !== inpar;
        });
        return state.set('peoples', newList);
      },
    },
    async: {
      // 这里用setTimeout代表异步ajax请求
      async getPeoples(state, inpar, { mutations, getState }) {
        let data = await new Promise(reslove => {
          const peoples = [
            { id: '1', name: '小名', age: 18, hobby: '睡觉' },
            { id: '2', name: '小王', age: 18, hobby: '打游戏' },
            { id: '3', name: '小张', age: 18, hobby: '看电视' },
            { id: '4', name: '小李', age: 18, hobby: '看小说' },
          ];
          setTimeout(() => {
            reslove({ result: 'ok', data: peoples });
          }, 1000);
        });
        if (data.result === 'ok') {
          // 更新state是一定要获取最新的state
          state = getState();
          return state.set('peoples', fromJS(data.data));
        }
      },
    },
  },
});export const subjectModel = lugaix.register({
  model: subjectModelName,
  state: subjectState,
  mutations: {
    async: {
      // 这里用setTimeout代表异步ajax请求
      async getSubject(state, inpar, { mutations, getState }) {
        const subject = [
          '理学',
          '工学',
          '文学',
          '艺术学',
          '历史学',
          '哲学',
          '经济学',
          '管理学',
          '法学',
          '教育学',
        ];
        let data = await new Promise(reslove => {
          setTimeout(() => {
            reslove({ result: 'ok', data: subject });
          }, 1000);
        });
        if (data.result === 'ok') {
          // 更新state是一定要获取最新的state
          state = getState();
          return state.set('subject', fromJS(data.data));
        }
      },
    },
  },
});export const schoolModel = lugaix.register({
  model: schoolModelName,
  state: schoolState,
  mutations: {
    async: {
      // 这里用setTimeout代表异步ajax请求
      async getsChoolInfo(state, inpar, { mutations, getState }) {
        const schoolInfo = {
          name: '北京清华大学',
          schoolMotto: '自强不息,厚德载物',
          address: '北京市海淀区双清路30号。',
        };
        let data = await new Promise(reslove => {
          setTimeout(() => {
            reslove({ result: 'ok', data: schoolInfo });
          }, 1000);
        });
        if (data.result === 'ok') {
          // 更新state是一定要获取最新的state
          state = getState();
          return state.set('schoolInfo', fromJS(data.data));
        }
      },
    },
  },
});
```

使用 connect 将 model 和组件绑定

```
// app.js
import React from 'react';
import { connect } from '@lugia/lugiax';
import { userModel, subjectModel, schoolModel } from './model';
class UserList extends React.Component {
  state = { loading: false };
  constructor(props) {
    super(props);
  }
  componentDidMount() {
    const { asyncGetPeoples, asyncGetSubject, asyncGetsChoolInfo } = this.props;
    asyncGetPeoples();
    asyncGetSubject();
    asyncGetsChoolInfo();
  }
  removePeoplesById(id) {
    const { removePeoplesById } = this.props;
    removePeoplesById(id);
  }
  render() {
    const { peoples, subject, schoolInfo } = this.props;
    return (
      <div>
        <div>
          <h2>学校信息</h2>
          <p>
            <span>&nbsp;校名：{schoolInfo.name}</span>
            <span>&nbsp;校训：{schoolInfo.schoolMotto}</span>
            <span>&nbsp;地址：{schoolInfo.address}</span>
          </p>
        </div>
        <div>
          <h2>学校专业</h2>
          <p>
            {subject.map(item => {
              return <span>【{item}】</span>;
            })}
          </p>
        </div>
        <h2>学员信息</h2>
        <p style={{ color: 'red' }}>用areStateEqual实现当学员少于2条时也面部刷新</p>
        {peoples.map(item => {
          return (
            <p>
              <span>姓名：{item.name}</span>
              <span>&nbsp;年龄：{item.age}</span>
              <span>&nbsp;爱好：{item.hobby}</span>
              <span
                onClick={() => {
                  this.removePeoplesById(item.id);
                }}
              >
                &nbsp;删除
              </span>
            </p>
          );
        })}
      </div>
    );
  }
}const App = connect(
  [userModel, subjectModel, schoolModel],
  state => {
    // 绑定多个model获取state状态如下，模块状态和传入model的顺序一致
    const [userModelState, subjectModelState, schoolModelState] = state;
    const peoples = userModelState.get('peoples').toJS
      ? userModelState.get('peoples').toJS()
      : userModelState.get('peoples');
    const subject = subjectModelState.get('subject').toJS
      ? subjectModelState.get('subject').toJS()
      : subjectModelState.get('subject');
    const schoolInfo = schoolModelState.get('schoolInfo').toJS
      ? schoolModelState.get('schoolInfo').toJS()
      : schoolModelState.get('schoolInfo');
    return { peoples, subject, schoolInfo };
  },
  mutations => {
    // 绑定多个model获取mutation如下，模块mutation和传入model的顺序一致
    const [userMutation, subjectMutation, schoolMutation] = mutations;
    return {
      asyncGetPeoples: userMutation.asyncGetPeoples,
      removePeoplesById: userMutation.removePeoplesById,
      asyncGetSubject: subjectMutation.asyncGetSubject,
      asyncGetsChoolInfo: schoolMutation.asyncGetsChoolInfo,
    };
  },
  {
    // 返回true 渲染组件  返回false 不渲染组件
    // 当connect多个model时 oldState是一个数组,存放着所有model的修改前的值，在数组中的顺序和传入model的一致
    // 当connect多个model时 newState是一个数组,存放着所有model的修改后的值，在数组中的顺序和传入model的一致
    areStateEqual(oldState, newState) {
      const [oldUserState, oldSubjectSt  ate, OldSchoolState] = oldState;
      const [newUserState, newSubjectState, newSchoolState] = newState;
      const flag = newUserState.get('peoples').size > 2;
      if (!flag) {
        console.log('当peoples数据少于2条组件渲染被阻止');
      }
      return flag;
    },
  }
)(UserList);
export default App;
```

运行结果
![图片](https://uploader.shimo.im/f/fDkUwetd2UwIZz7P.png!thumbnail)

### areStatePropsEqual 示例

在 lugaix 中注册 mode

```
// model.js
import lugaix from '@lugia/lugiax';
import { fromJS } from 'immutable';
const userModelName = 'userModel';
const subjectModelName = 'subjectModel';
const schoolModelName = 'schoolModel';const userState = {
  peoples: [],
};
const subjectState = {
  subject: [],
};
const schoolState = {
  schoolInfo: {},
};export const userModel = lugaix.register({
  model: userModelName,
  state: userState,
  mutations: {
    sync: {
      removePeoplesById(state, inpar, { getState }) {
        const newList = state.get('peoples').filter(item => {
          return item.get('id') !== inpar;
        });
        return state.set('peoples', newList);
      },
    },
    async: {
      // 这里用setTimeout代表异步ajax请求
      async getPeoples(state, inpar, { mutations, getState }) {
        let data = await new Promise(reslove => {
          const peoples = [
            { id: '1', name: '小名', age: 18, hobby: '睡觉' },
            { id: '2', name: '小王', age: 18, hobby: '打游戏' },
            { id: '3', name: '小张', age: 18, hobby: '看电视' },
            { id: '4', name: '小李', age: 18, hobby: '看小说' },
          ];
          setTimeout(() => {
            reslove({ result: 'ok', data: peoples });
          }, 1000);
        });
        if (data.result === 'ok') {
          // 更新state是一定要获取最新的state
          state = getState();
          return state.set('peoples', fromJS(data.data));
        }
      },
    },
  },
});export const subjectModel = lugaix.register({
  model: subjectModelName,
  state: subjectState,
  mutations: {
    async: {
      // 这里用setTimeout代表异步ajax请求
      async getSubject(state, inpar, { mutations, getState }) {
        const subject = [
          '理学',
          '工学',
          '文学',
          '艺术学',
          '历史学',
          '哲学',
          '经济学',
          '管理学',
          '法学',
          '教育学',
        ];
        let data = await new Promise(reslove => {
          setTimeout(() => {
            reslove({ result: 'ok', data: subject });
          }, 1000);
        });
        if (data.result === 'ok') {
          // 更新state是一定要获取最新的state
          state = getState();
          return state.set('subject', fromJS(data.data));
        }
      },
    },
  },
});export const schoolModel = lugaix.register({
  model: schoolModelName,
  state: schoolState,
  mutations: {
    async: {
      // 这里用setTimeout代表异步ajax请求
      async getsChoolInfo(state, inpar, { mutations, getState }) {
        const schoolInfo = {
          name: '北京清华大学',
          schoolMotto: '自强不息,厚德载物',
          address: '北京市海淀区双清路30号。',
        };
        let data = await new Promise(reslove => {
          setTimeout(() => {
            reslove({ result: 'ok', data: schoolInfo });
          }, 1000);
        });
        if (data.result === 'ok') {
          // 更新state是一定要获取最新的state
          state = getState();
          return state.set('schoolInfo', fromJS(data.data));
        }
      },
    },
  },
});
```

使用 connect 将 model 和组件绑定

```
// app.js
import React from 'react';
import { connect } from '@lugia/lugiax';
import { userModel, subjectModel, schoolModel } from './model';
class UserList extends React.Component {
  state = { loading: false };
  constructor(props) {
    super(props);
  }
  componentDidMount() {
    const { asyncGetPeoples, asyncGetSubject, asyncGetsChoolInfo } = this.props;
    asyncGetPeoples();
    asyncGetSubject();
    asyncGetsChoolInfo();
  }
  removePeoplesById(id) {
    const { removePeoplesById } = this.props;
    removePeoplesById(id);
  }
  render() {
    const { peoples, subject, schoolInfo } = this.props;
    return (
      <div>
        <div>
          <h2>学校信息</h2>
          <p>
            <span>&nbsp;校名：{schoolInfo.name}</span>
            <span>&nbsp;校训：{schoolInfo.schoolMotto}</span>
            <span>&nbsp;地址：{schoolInfo.address}</span>
          </p>
        </div>
        <div>
          <h2>学校专业</h2>
          <p>
            {subject.map(item => {
              return <span>【{item}】</span>;
            })}
          </p>
        </div>
        <h2>学员信息</h2>
        <p style={{ color: 'red' }}>用areStateEqual实现当学员少于2条时也面部刷新</p>
        {peoples.map(item => {
          return (
            <p>
              <span>姓名：{item.name}</span>
              <span>&nbsp;年龄：{item.age}</span>
              <span>&nbsp;爱好：{item.hobby}</span>
              <span
                onClick={() => {
                  this.removePeoplesById(item.id);
                }}
              >
                &nbsp;删除
              </span>
            </p>
          );
        })}
      </div>
    );
  }
}const App = connect(
  [userModel, subjectModel, schoolModel],
  state => {
    // 绑定多个model获取state状态如下，模块状态和传入model的顺序一致
    const [userModelState, subjectModelState, schoolModelState] = state;
    const peoples = userModelState.get('peoples').toJS
      ? userModelState.get('peoples').toJS()
      : userModelState.get('peoples');
    const subject = subjectModelState.get('subject').toJS
      ? subjectModelState.get('subject').toJS()
      : subjectModelState.get('subject');
    const schoolInfo = schoolModelState.get('schoolInfo').toJS
      ? schoolModelState.get('schoolInfo').toJS()
      : schoolModelState.get('schoolInfo');
    return { peoples, subject, schoolInfo };
  },
  mutations => {
    // 绑定多个model获取mutation如下，模块mutation和传入model的顺序一致
    const [userMutation, subjectMutation, schoolMutation] = mutations;
    return {
      asyncGetPeoples: userMutation.asyncGetPeoples,
      removePeoplesById: userMutation.removePeoplesById,
      asyncGetSubject: subjectMutation.asyncGetSubject,
      asyncGetsChoolInfo: schoolMutation.asyncGetsChoolInfo,
    };
  },
  {
    // 返回true 渲染组件  返回false 不渲染组件
    // oldProps更改之前的props  newProps更改之后的props
    areStatePropsEqual(oldProps, newProps) {
      const flag = newProps.peoples.length > 2;
      if (!flag) {
        console.log('当peoples数据少于2条组件渲染被阻止');
      }
      return flag;
    },
  }
)(UserList);
export default App;
```

运行结果
![图片](https://uploader.shimo.im/f/BbvsslrIiw0b0Yxo.png!thumbnail)

### areOwnPropsEqual 示例

在 lugaix 中注册 mode

```
// model.js
import lugaix from '@lugia/lugiax';
import { fromJS } from 'immutable';
const userModelName = 'userModel';
const subjectModelName = 'subjectModel';
const schoolModelName = 'schoolModel';const userState = {
  peoples: [],
};export const userModel = lugaix.register({
  model: userModelName,
  state: userState,
  mutations: {
    sync: {
      removePeoplesById(state, inpar, { getState }) {
        const newList = state.get('peoples').filter(item => {
          return item.get('id') !== inpar;
        });
        return state.set('peoples', newList);
      },
    },
    async: {
      // 这里用setTimeout代表异步ajax请求
      async getPeoples(state, inpar, { mutations, getState }) {
        let data = await new Promise(reslove => {
          const peoples = [
            { id: '1', name: '小名', age: 18, hobby: '睡觉' },
            { id: '2', name: '小王', age: 18, hobby: '打游戏' },
            { id: '3', name: '小张', age: 18, hobby: '看电视' },
            { id: '4', name: '小李', age: 18, hobby: '看小说' },
          ];
          setTimeout(() => {
            reslove({ result: 'ok', data: peoples });
          }, 1000);
        });
        if (data.result === 'ok') {
          // 更新state是一定要获取最新的state
          state = getState();
          return state.set('peoples', fromJS(data.data));
        }
      },
    },
  },
});
```

使用 connect 将 model 和组件绑定

```
// app.js
import React from 'react';
import { connect } from '@lugia/lugiax';
import { userModel, subjectModel, schoolModel } from './model';
class UserList extends React.Component {
  state = { loading: false };
  constructor(props) {
    super(props);
  }
  componentDidMount() {
    const { asyncGetPeoples } = this.props;
    asyncGetPeoples();
  }
  render() {
    const { peoples, count } = this.props;
    return (
      <div style={{ border: '1px solid red' }}>
        <h2>学员信息</h2>
        <p style={{ color: 'red' }}>当count大于2时。组件不渲染</p>
        <p>count值: {count}</p>
      </div>
    );
  }
}const App = connect(
  userModel,
  state => {
    const peoples = state.get('peoples').toJS ? state.get('peoples').toJS() : state.get('peoples');
    return { peoples };
  },
  mutations => {
    return {
      asyncGetPeoples: mutations.asyncGetPeoples,
    };
  },
  {
    // 返回true 渲染组件  返回false 不渲染组件
    // oldProps更改之前的外部传入的props  newProps更改之后的props
    areOwnPropsEqual(oldProps, newProps) {
      const flag = newProps.count <= 2;
      if (!flag) {
        console.log('当count大约2的时候，组件渲染被阻止');
      }
      return flag;
    },
  }
)(UserList);export default class Test extends React.Component {
  constructor() {
    super();
    this.state = { count: 0 };
  }
  change = () => {
    this.setState({ count: ++this.state.count });
  };
  render() {
    const { count } = this.state;
    return (
      <div>
        <App count={count} />
        <button onClick={this.change}>count++</button>
      </div>
    );
  }
}
```

运行结果
![图片](https://uploader.shimo.im/f/goQ5EgMkrg8q8IPa.png!thumbnail)

### withRef 示例

在 lugaix 中注册 mode

```
// model.js
import lugaix from '@lugia/lugiax';
import { fromJS } from 'immutable';
const userModelName = 'userModel';
const userState = {
  peoples: [],
};
export const userModel = lugaix.register({
  model: userModelName,
  state: userState,
  mutations: {
    sync: {
      removePeoplesById(state, inpar, { getState }) {
        const newList = state.get('peoples').filter(item => {
          return item.get('id') !== inpar;
        });
        return state.set('peoples', newList);
      },
    },
    async: {
      // 这里用setTimeout代表异步ajax请求
      async getPeoples(state, inpar, { mutations, getState }) {
        let data = await new Promise(reslove => {
          const peoples = [
            { id: '1', name: '小名', age: 18, hobby: '睡觉' },
            { id: '2', name: '小王', age: 18, hobby: '打游戏' },
            { id: '3', name: '小张', age: 18, hobby: '看电视' },
            { id: '4', name: '小李', age: 18, hobby: '看小说' },
          ];
          setTimeout(() => {
            reslove({ result: 'ok', data: peoples });
          }, 1000);
        });
        if (data.result === 'ok') {
          // 更新state是一定要获取最新的state
          state = getState();
          return state.set('peoples', fromJS(data.data));
        }
      },
    },
  },
});
```

使用 connect 将 model 和组件绑定

```
// app.js
import React from 'react';
import { connect } from '@lugia/lugiax';
import { userModel, subjectModel, schoolModel } from './model';
class UserList extends React.Component {
  displayName = 'userComponent';
  state = { loading: false };
  constructor(props) {
    super(props);
  }
  componentDidMount() {
    const { asyncGetPeoples } = this.props;
    asyncGetPeoples();
  }
  showComponentName = () => {
    return `我是${this.displayName}组件`;
  };
  render() {
    const { peoples, count } = this.props;
    return (
      <div style={{ border: '1px solid red' }}>
        <h2>学员信息</h2>
        <p style={{ color: 'red' }}>当count大于2时。组件不渲染</p>
        <p>count值: {count}</p>
      </div>
    );
  }
}
const App = connect(
  userModel,
  state => {
    const peoples = state.get('peoples').toJS ? state.get('peoples').toJS() : state.get('peoples');
    return { peoples };
  },
  mutations => {
    return {
      asyncGetPeoples: mutations.asyncGetPeoples,
    };
  },
  {
    withRef: true,
  }
)(UserList);export default class Test extends React.Component {
  constructor() {
    super();
    this.state = { componentName: '' };
  }
  getInstance = () => {
    //this.componentInstance.target 包裹组件的实例
    const componentName = this.componentInstance.target.showComponentName();
    this.setState({ componentName });
  };
  render() {
    return (
      <div>
        <p>{this.state.componentName}</p>
        <App
          ref={target => {
            this.componentInstance = target;
          }}
        />
        <button onClick={this.getInstance}>获取connect包裹组件的实例</button>
      </div>
    );
  }
}
```

# 路由管理

## createApp

### 描述

> 使用 createApp 方法，创建根路由

### 方法使用简述

```javascript
createApp(
  routerMap: {
    [path: string]: {
      render?: Function,
      exact?: boolean,
      strict?: boolean,
      component?: React.Component,
      onPageLoad?: Function,
      onPageUnLoad?: Function,
      verify?: Function,
      redirect?:{
         to: '/login',
         verify?: Function
      }
    }
  },
  history: Object,
  param?:{
    loading?: React.Component,
    onBeforeGo?: async function，
    verifyUrl?: Function
  }
)
```

### 参数描述

- routerMap

      * path
          * 路由匹配路径
          * 数据类型
              * object
      * render
          * 内联渲染，而且不会产生上文说的重复装载问题。
          * 数据类型
              * function
      * exact
          * exact表示location.pathnam值必须和path完全一致才渲染对应的组件,为true是表示严格匹配，要求路径与location.pathname必须完全匹配。
          * 默认值为false
          * 数据类型
              * boolean
          * 匹配规则

| 路由 | url      | exact | 结果 |
| :--: | :------- | :---: | :--- |
| /one | /one/two | true  | no   |
| /one | /one/two | false | yes  |

> 备注：如果没有子路由的情况，建议大家配都加一个 exact；如果有子路由，建议在子路由中加 exact，父路由不加；

    * strict
        * strict属性主要就是匹配反斜杠，规定是否匹配末尾包含反斜杠的路径，如果strict为true，则如果path中不包含反斜杠结尾，则他也不能匹配包含反斜杠结尾的路径，这个需要和exact结合使用
        * 默认值false
        * 数据类型
            * boolean
        * 匹配规则

| 路由  | url      | exact | 结果 |
| :---: | :------- | :---: | :--- |
| /one/ | /one     | true  | no   |
| /one/ | /one/    | true  | yes  |
| /one/ | /one/two | true  | yes  |

![图片](https://uploader.shimo.im/f/zmO0Xrk7bnEWDAF2.png!thumbnail)

- component
  - 路径对应显示的组件
  - 数据类型
    - function 或者 reactComponent
- onPageLoad
  - 页面加载完成后执行
  - 数据类型
    - function
- onPageUnLoad
  - 页面卸载时执行
  - 数据类型
    - function
- verify
  - 单个路由同步验证方法，返回 false 时渲染空页面，也可以在函数内部调用重定向的方法跳转的其他路由。当返回 true 会渲染 routerMap.component 或者 routerMap.render 对应的组件。
  - 数据类型
    - function({url}): boolean
      - url 表示当前路由地址

> 备注：verfy 必须是同步方法。

- redirect
  - 当 redirect.verify 函数返回 false 重定向到 redirect.to 指定的路由。当返回 true 会渲染 routerMap.component 或者 routerMap.render 对应的组件。
  - 数据类型
    - object
      - to
        - 重定向的路由地址
        - 数据类型
          - string
      - verify
        - 函数重定向判断依据,如果函数返回值为 false，进行重定向跳转。如果返回值为 true 会渲染 routerMap.component 或者 routerMap.render 对应的组件。
        - 数据类型
          - function({url})：boolean
            - url 表示当前路由地址

> 备注：verfy 必须是同步方法。

- history
  - 用于导航的历史对象。
  - 数据类型
    - createBrowserHistory
    - createHashHistory
- param 其他扩展配置
  _ loading
  _ 页面为渲染之前的加载，loading 组件
  _ 数据类型
  _ function 或者 reactComponent
  _ onBeforeGo
  _ 异步路由校验，可在内部使用进行重定向其他路由。切换路由时这个函数就会被调用，
  _ 数据类型
  _ async function({url})
  _ url 表示当前路由地址
  _ verifyUrl
  _ 每个路由切换都会触发这个同步路由验证方法，返回 false 时渲染空页面，也可以在函数内部调用重定向的方法跳转的其他路由。当返回 true 会渲染 routerMap.component 或者 routerMap.render 对应的组件。
  _ 数据类型
  _ function({url})：boolean
  _ url 表示当前路由地址
  > 备注：verifyUrl 必须是同步方法。

### 路由校验 verifyUrl、onBeforeGo、verify 区别

- verifyUrl、verify 是同步验证方法，onBeforeGo 是异步验证方法
- 每次路由切换都会触发 onBeforeGo 函数或者 verifyUrl 函数。verify 函数只有在 location.pathname 和路由配置中的 path 相等或者被包含时才会被触发
- verify 函数执行顺序在 onBeforeGo 函数和 verifyUrl 函数之前
- 当有 verifyUrl 函数时 onBeforeGo 函时将不会执行，两者不能并存。

**路由校验流程图**

![图片](https://uploader.shimo.im/f/m4v4b99AyUG8OR0U.png!thumbnail)

路由优先级：verify > ( verifyUrl | onBeforeGo )

## 示例代码

### component 示例

在入口文件中创建根路由

```
// index.js 入口文件
import React from 'react';
import { createBrowserHistory } from 'history';
import { createApp, go, goBack, goForward, render, replace } from '@lugia/lugiax-router';
import { UserInfo, UserList, UserAdd } from './main';
const history = createBrowserHistory();
render(() => {
  const App = createApp(
    {
      '/': {
        exact: true,
        component: UserList,
      },
      '/info': {
        exact: true,
        component: UserInfo,
      },
      '/add': {
        exact: true,
        component: UserAdd,
      },
    },
    history
  );
  return <App />;
}, 'root');
```

导出所有组件

```
// main.js
import React from 'react';
class UserInfo extends React.Component<any> {
  render() {
    return <div>用户详情页面</div>;
  }
}class UserList extends React.Component<any> {
  render() {
    return <div>用户里列表</div>;
  }
}class UserAdd extends React.Component<any> {
  render() {
    return <div>用户详添加页</div>;
  }
}
export { UserInfo, UserList, UserAdd };
```

### render 示例

```
// index.js 入口文件
import React from 'react';
import { createBrowserHistory } from 'history';
import { createApp, go, goBack, goForward, render, replace } from '@lugia/lugiax-router';
import UserAdd from './user/add';const history = createBrowserHistory();
render(() => {
  const App = createApp(
    {
      '/': {
        exact: true,
        render: async () => {
          return import('./user/list');
        },
      },
      '/info': {
        exact: true,
        render: async () => {
          return import('./user/info');
        },
      },
      '/add': {
        exact: true,
        render: async () => {
          return import('./user/add');
        },
      },
    },
    history
  );
  return <App />;
}, 'root');
```

userList 组件

```
// /user/list.js
import React from 'react';
export default class UserList extends React.Component<any> {
  render() {
    return <div>用户里列表</div>;
  }
}
```

userAdd 组件

```
// /user/add.js
import React from 'react';
export default class UserAdd extends React.Component<any> {
  render() {
    return <div>用户详添加页</div>;
  }
}
```

userInfo 组件

```
// /user/info.js
import React from 'react';
export default class UserInfo extends React.Component<any> {
  render() {
    return <div>用户详情页面</div>;
  }
}
```

### redirect 示例

用户未登录，重定到登录页面

```
// index.js 入口文件
import React from 'react';
import { createBrowserHistory } from 'history';
import { createApp, go, goBack, goForward, render, replace, Redirect } from '@lugia/lugiax-router';
import { UserInfo, UserList, UserAdd, NoAuthority, Login } from './main';const history = createBrowserHistory();
render(() => {
  const App = createApp(
    {
      '/login': {
        exact: true,
        component: Login,
      },
      '/': {
        exact: true,
        redirect: {
          to: '/login',
          verify() {
            return window.loginState;
          },
        },
        component: UserList,
      },
      '/info': {
        exact: true,
        redirect: {
          to: '/login',
          verify() {
            return window.loginState;
          },
        },
        component: UserInfo,
      },
      '/add': {
        exact: true,
        redirect: {
          to: '/login',
          verify() {
            return window.loginState;
          },
        },
        component: UserAdd,
      },
    },
    history
  );
  return <App />;
}, 'root');
```

```
import React from 'react';
import { go, Link, createRoute, goBack, goForward, replace } from '@lugia/lugiax-router';class UserInfo extends React.Component<any> {
  render() {
    return (
      <div>
        <Head />
        用户详情页面
      </div>
    );
  }
}class UserList extends React.Component<any> {
  render() {
    return (
      <div>
        <Head />
        用户里列表
      </div>
    );
  }
}class UserAdd extends React.Component<any> {
  render() {
    return (
      <div>
        <Head />
        用户详添加页
      </div>
    );
  }
}class Login extends React.Component<any> {
  login = () => {
    // 模拟异步请求
    new Promise(resolve => {
      setTimeout(() => {
        resolve(true);
      }, 500);
    }).then(() => {
      window.loginState = true;
      go({ url: '/' });
    });
  };
  render() {
    return (
      <div>
        请单击按钮登录<button onClick={this.login}>登录</button>
      </div>
    );
  }
}class Head extends React.Component<any> {
  render() {
    return (
      <div>
        <Link to="/add" id="add">
          添加用户
        </Link>
        <Link to="/" id="list">
          用户列表
        </Link>
        <Link to="/info" id="info">
          用户详情
        </Link>
      </div>
    );
  }
}
export { UserInfo, UserList, UserAdd, Login, Head };
```

### verify 示例

```
import React from 'react';
import { createBrowserHistory } from 'history';
import { createApp, go, goBack, goForward, render, replace, Redirect } from '@lugia/lugiax-router';
import { UserInfo, UserList, UserAdd, NoAuthority } from './main';const history = createBrowserHistory();
render(() => {
  const App = createApp(
    {
      '/info': {
        exact: true,
        component: UserInfo,
      },
      '/add': {
        exact: true,
        component: UserAdd,
      },
      '/': {
        verify({ url }) {
          if (url === '/') {
            return true;
          } else {
            go({ url: '/' });
            return false;
          }
        },
        component: UserList,
      },
    },
    history
  );
  return <App />;
}, 'root');
```

```
import React from 'react';
import { go, Link, createRoute, goBack, goForward, replace } from '@lugia/lugiax-router';class UserInfo extends React.Component<any> {
  render() {
    return (
      <div>
        <Head />
        用户详情页面
      </div>
    );
  }
}class UserList extends React.Component<any> {
  render() {
    return (
      <div>
        <Head />
        用户里列表
      </div>
    );
  }
}class UserAdd extends React.Component<any> {
  render() {
    return (
      <div>
        <Head />
        用户详添加页
      </div>
    );
  }
}class Head extends React.Component<any> {
  render() {
    return (
      <div>
        <Link to="/add" id="add">
          添加用户
        </Link>
        <Link to="/" id="list">
          用户列表
        </Link>
        <Link to="/info" id="info">
          用户详情
        </Link>
      </div>
    );
  }
}
export { UserInfo, UserList, UserAdd, Head };
```

### verifyUrl 示例

通 verifyUrl 函数 当 url 时/add 时让页面重定到其他路由

```
import React from 'react';
import { createBrowserHistory } from 'history';
import { createApp, go, goBack, goForward, render, replace, Redirect } from '@lugia/lugiax-router';
import { UserInfo, UserList, UserAdd, NoAuthority } from './main';const history = createBrowserHistory();
render(() => {
  const App = createApp(
    {
      '/info': {
        exact: true,
        component: UserInfo,
      },
      '/add': {
        exact: true,
        component: UserAdd,
      },
      '/': {
        component: UserList,
      },
    },
    history,
    {
      verifyUrl({ url }) {
        console.log('verifyUrl');
        if (url === '/add') {
          console.log('当路由等于/add时路由不能跳转');
          go({ url: '/' });
          return false;
        }
        return true;
      },
    }
  );
  return <App />;
}, 'root')
```

```
import React from 'react';
import { go, Link, createRoute, goBack, goForward, replace } from '@lugia/lugiax-router';class UserInfo extends React.Component<any> {
  render() {
    return (
      <div>
        <Head />
        用户详情页面
      </div>
    );
  }
}class UserList extends React.Component<any> {
  render() {
    return (
      <div>
        <Head />
        用户里列表
      </div>
    );
  }
}class UserAdd extends React.Component<any> {
  render() {
    return (
      <div>
        <Head />
        用户详添加页
      </div>
    );
  }
}class Head extends React.Component<any> {
  render() {
    return (
      <div>
        <Link to="/add" id="add">
          添加用户
        </Link>
        <Link to="/" id="list">
          用户列表
        </Link>
        <Link to="/info" id="info">
          用户详情
        </Link>
      </div>
    );
  }
}
export { UserInfo, UserList, UserAdd, Head };
```

### onBeforeGo 示例

```
import React from 'react';
import { createBrowserHistory } from 'history';
import { createApp, go, goBack, goForward, render, replace, Redirect } from '@lugia/lugiax-router';
import { UserInfo, UserList, UserAdd, NotFound } from './main';const history = createBrowserHistory();
render(() => {
  const App = createApp(
    {
      '/info': {
        exact: true,
        component: UserInfo,
      },
      '/add': {
        exact: true,
        component: UserAdd,
      },
      '/notfound': {
        exact: true,
        component: NotFound,
      },
      '/': {
        component: UserList,
      },
    },
    history,
    {
      async onBeforeGo({ url }) {
        if (url === '/notfound') return;
        const data = await new Promise(resolve => {
          setTimeout(() => {
            const number = Math.floor(Math.random() * 10 + 1);
            resolve(number % 2 === 0);
          }, 1000);
        });
        // 能被2整除是重定向到 notfound页面
        if (data) {
          go({ url: '/notfound' });
        }
      },
    }
  );
  return <App />;
}, 'root');
```

```
import React from 'react';
import { go, Link, createRoute, goBack, goForward, replace } from '@lugia/lugiax-router';class UserInfo extends React.Component<any> {
  render() {
    return (
      <div>
        <Head />
        用户详情页面
      </div>
    );
  }
}class UserList extends React.Component<any> {
  render() {
    return (
      <div>
        <Head />
        用户里列表
      </div>
    );
  }
}class UserAdd extends React.Component<any> {
  render() {
    return (
      <div>
        <Head />
        用户详添加页
      </div>
    );
  }
}class NotFound extends React.Component<any> {
  render() {
    return (
      <div>
        <Head />
        未找到页面
      </div>
    );
  }
}class Head extends React.Component<any> {
  render() {
    return (
      <div>
        <Link to="/add" id="add">
          添加用户
        </Link>
        <Link to="/" id="list">
          用户列表
        </Link>
        <Link to="/info" id="info">
          用户详情
        </Link>
      </div>
    );
  }
}
export { UserInfo, UserList, UserAdd, Head, NotFound };
```

## createRoute

### 描述

> 使用 createRoute 方法，创建子路由

### 方法使用简述

```
  createRoute({
    [path: string]:{
      render?: Function,
      exact?: boolean,
      strict?: boolean,
      component?: React.Component,
      onPageLoad?: Function,
      onPageUnLoad?: Function,
      verify?: Function,
      redirect?:{
         to: '/login',
         verify?: Function
      }
    }
  })
```

### 参数描述

    * path
        * 路由匹配路径
        * 数据类型
            * object
    * render
        * 内联渲染，而且不会产生上文说的重复装载问题。
        * 数据类型
            * function
    * exact
        * exact表示location.pathnam值必须和path完全一致才渲染对应的组件,为true是表示严格匹配，要求路径与location.pathname必须完全匹配。
        * 默认值为false
        * 数据类型
            * boolean
        * 匹配规则

| 路由 | url      | exact | 结果 |
| :--: | :------- | :---: | :--- |
| /one | /one/two | true  | no   |
| /one | /one/two | false | yes  |

> 备注：如果没有子路由的情况，建议大家配都加一个 exact；如果有子路由，建议在子路由中加 exact，父路由不加；

    * strict
        * strict属性主要就是匹配反斜杠，规定是否匹配末尾包含反斜杠的路径，如果strict为true，则如果path中不包含反斜杠结尾，则他也不能匹配包含反斜杠结尾的路径，这个需要和exact结合使用
        * 默认值false
        * 数据类型
            * boolean
        * 匹配规则

| 路由  | url      | exact | 结果 |
| :---: | :------- | :---: | :--- |
| /one/ | /one     | true  | no   |
| /one/ | /one/    | true  | yes  |
| /one/ | /one/two | true  | yes  |

![图片](https://uploader.shimo.im/f/zmO0Xrk7bnEWDAF2.png!thumbnail)

    * component
        * 路径对应显示的组件
        * 数据类型
            * function 或者 reactComponent
    * onPageLoad
        * 页面加载完成后执行
        * 数据类型
            * function
    * onPageUnLoad
        * 页面卸载时执行
        * 数据类型
            * function
    * verify
        * 单个路由同步验证方法，返回false 时渲染空页面，也可以在函数内部调用重定向的方法跳转的其他路由。当返回true会渲染routerMap.component 或者 routerMap.render对应的组件。
        * 数据类型
            * function({url}): boolean
                * url表示当前路由地址

> 备注：verfy 必须是同步方法。

    * redirect
        * 当redirect.verify函数返回false重定向到redirect.to指定的路由。当返回true会渲染routerMap.component 或者 routerMap.render对应的组件。
        * 数据类型
            * object
                * to
                    * 重定向的路由地址
                    * 数据类型
                        * string
                * verify
                    * 韩树重定向判断依据,如果函数返回值为false，进行重定向跳转。如果返回值为true会渲染routerMap.component 或者 routerMap.render对应的组件。
                    * 数据类型
                        * function({url})：boolean
                            * url表示当前路由地址

> 备注：verfy 必须是同步方法。

## 示例代码

### 使用 createRoute 创建子路由

```
import React from 'react';
import { createBrowserHistory } from 'history';
import { createApp, go, goBack, goForward, render, replace, Redirect } from '@lugia/lugiax-router';
import UserModel, { NotFound } from './main';
const history = createBrowserHistory();
render(() => {
  const App = createApp(
    {
      '/': {
        exact: true,
        component: UserModel,
        verify() {
          console.log('verify');
          return true;
        },
      },
      '/user': {
        component: UserModel,
        verify() {
          console.log('verify');
          return true;
        },
      },
      '/404': {
        exact: true,
        component: NotFound,
      },
      NotFound: {
        component: () => (
          <Redirect
            to={{
              pathname: '/404',
            }}
          />
        ),
      },
    },
    history
  );
  return <App />;
}, 'root');
```

```
import React from 'react';
import { go, Link, createRoute, goBack, goForward, Redirect } from '@lugia/lugiax-router';class UserInfo extends React.Component<any> {
  render() {
    return (
      <div>
        <Head />
        用户详情页面
      </div>
    );
  }
}class UserList extends React.Component<any> {
  render() {
    return (
      <div>
        <Head />
        用户里列表
      </div>
    );
  }
}class UserAdd extends React.Component<any> {
  render() {
    return (
      <div>
        <Head />
        用户详添加页
      </div>
    );
  }
}export class NotFound extends React.Component<any> {
  render() {
    return <div>未找到页面</div>;
  }
}class Head extends React.Component<any> {
  render() {
    return (
      <div>
        <Link to="/user/add" id="add">
          添加用户
        </Link>
        <Link to="/user" id="list">
          用户列表
        </Link>
        <Link to="/user/info" id="info">
          用户详情
        </Link>
      </div>
    );
  }
}const routerMap = {
  '/': {
    exact: true,
    component: UserList,
  },
  '/user/': {
    exact: true,
    component: UserList,
  },
  '/user/info': {
    exact: true,
    component: UserInfo,
  },
  '/user/add': {
    exact: true,
    component: UserAdd,
  },
  NotFound: {
    component: () => (
      <Redirect
        to={{
          pathname: '/404',
        }}
      />
    ),
  },
};export default () => {
  return (
    <React.Fragment>
      {createRoute(routerMap)}
    </React.Fragment>
  );
};
```

> render、exact、strict、component、onPageLoad、onPageUnLoad、verify、 redirect 等用法和 careteApp 相同

# Lugiax 插件

解决问题：我们经常会通 lugaix 管理模型的 State。但是使用 lugaix 存储 State 数据时会有这么一个问题，如果用户刷新了网页，那么我们通过 lugaix 存储的 State 数据就会被全部清空。

# 持久化插件

## 如何安装

### npm

```
npm install @lugia/lugiax-persistence
```

### yarn

> 推荐使用 yarn 管理 npm 依赖

```
yarn add @lugia/lugiax-persistence
```

## 如何使用

### 使用 lugaix 内置默认持久化控制器

示例代码

```javascript
// 注册持久化model
import lugaix from '@lugia/lugiax';
import wrapPersistence from '@lugia/lugiax-persistence';
const userState = {
  peoples: [
    { id: '1', name: '小名', age: 18, hobby: '睡觉' },
    { id: '2', name: '小王', age: 18, hobby: '打游戏' },
    { id: '3', name: '小张', age: 18, hobby: '看电视' },
    { id: '4', name: '小李', age: 18, hobby: '看小说' },
  ],
};
const modelName = 'userInfo';
export const userModel = lugaix.register(
  wrapPersistence({
    model: modelName,
    state: userState,
    mutations: {
      sync: {
        // 同步删除方法
        removePeopleById(state, inpar, { mutations }) {
          // 获取的数据类型为 Immutable
          const list = state.get('peoples'); // Immutable数据的操作方法请参考官网
          const newList = list.filter(item => {
            return item.get('id') != inpar.id;
          });
          return state.set('peoples', newList);
        },
      },
      async: {
        // 异步删除方法。这里用setTimeout代表异步ajax请求。请求删除数据
        async removePeopleById(state, inpar, { mutations, getState }) {
          const data = await new Promise(reslove => {
            setTimeout(() => {
              reslove({ result: 'ok' });
            }, 1000);
          });
          if (data.result === 'ok') {
            // mutations.removePeopleById调用过后会返回最新的state,
            // 这里一定要返回最新的state。
            // state = mutations.removePeopleById(inpar);
            // 或者调用完mutation方法后使用getState方法获取最新的state并返回
            mutations.removePeopleById(inpar);
            state = getState();
          }
          return state;
        },
      },
    },
  })
);
```

```javascript
// 绑定model并使用
import React from 'react';
import { connect } from '@lugia/lugiax';
import { userModel } from './model';
class UserList extends React.Component {
  state = { loading: false };
  render() {
    const { list, removePeopleById, asyncRemovePeopleById } = this.props;
    return (
      <div>
                
        {list.map(item => {
          console.log('item', item);
          return (
            <p>
                            <span>姓名：{item.name}</span>
                            <span>&nbsp;年龄：{item.age}</span>
                            <span>&nbsp;爱好：{item.hobby}</span>
                            <span
                onClick={() => {
                  asyncRemovePeopleById(item);
                }}
              >
                                &nbsp;异步删除               
              </span>
                            
              <span
                onClick={() => {
                  removePeopleById(item);
                }}
              >
                                &nbsp;同步删除               
              </span>
                          
            </p>
          );
        })}
              
      </div>
    );
  }
}
const App = connect(
  userModel,
  state => {
    const list = state.get('peoples').toJS ? state.get('peoples').toJS() : state.get('peoples');
    return { list };
  },
  mutations => {
    console.log(mutations);
    const { removePeopleById, asyncRemovePeopleById } = mutations;
    return {
      removePeopleById,
      asyncRemovePeopleById,
    };
  }
)(UserList);
export default App;
```

### 使用用户自定义持久化控制器

示例代码

```javascript
// lugaix-persist.js
import { registerPersistence } from '@lugia/lugiax-persistence';
const domainName = 'my_data';
const objectStringToJson = (str: string) => {
  try {
    return JSON.parse(str);
  } catch (ex) {
    console.warn('持久化的JSON字符串转为成对象时报错，请核实数据！', ex.message);
    return {};
  }
};
const getStateBySession = modelName => {
  let persistData = objectStringToJson(window.sessionStorage.getItem(domainName));
  if (!persistData) {
    persistData = {};
  }
  return persistData[modelName];
};
const saveStateBySession = (modelName, state) => {
  try {
    const persistData = objectStringToJson(window.sessionStorage.getItem(domainName)) || {};
    persistData[modelName] = state.toJS() || {};
    window.sessionStorage.setItem(domainName, JSON.stringify(persistData));
    return true;
  } catch (ex) {
    console.warn('持久化数据保存失败！', ex.message);
    return false;
  }
};
const Persistence = {
  getStore: getStateBySession,
  saveStore: saveStateBySession,
};
registerPersistence(Persistence, 'mySession');
```

```
// 注册持久化model
// model.js
import lugaix from '@lugia/lugiax';
import wrapPersistence from '@lugia/lugiax-persistence';
const userState = {
  peoples: [
    { id: '1', name: '小名', age: 18, hobby: '睡觉' },
    { id: '2', name: '小王', age: 18, hobby: '打游戏' },
    { id: '3', name: '小张', age: 18, hobby: '看电视' },
    { id: '4', name: '小李', age: 18, hobby: '看小说' },
  ],
};
const modelName = 'userInfo';
export const userModel = lugaix.register(
  wrapPersistence(
    {
      model: modelName,
      state: userState,
      mutations: {
        sync: {
          // 同步删除方法
          removePeopleById(state, inpar, { mutations }) {
            // 获取的数据类型为 Immutable
            const list = state.get('peoples');
            // Immutable数据的操作方法请参考官网
            const newList = list.filter(item => {
              return item.get('id') != inpar.id;
            });
            return state.set('peoples', newList);
          },
        },
        async: {
          // 异步删除方法。这里用setTimeout代表异步ajax请求。请求删除数据
          async removePeopleById(state, inpar, { mutations, getState }) {
            const data = await new Promise(reslove => {
              setTimeout(() => {
                reslove({ result: 'ok' });
              }, 1000);
            });
            if (data.result === 'ok') {
              // mutations.removePeopleById调用过后会返回最新的state,
              // 这里一定要返回最新的state。
              // state = mutations.removePeopleById(inpar);
              // 或者调用完mutation方法后使用getState方法获取最新的state并返回
              mutations.removePeopleById(inpar);
              state = getState();
            }
            return state;
          },
        },
      },
    },
    { name: 'mySession' }
  )
);
```

```javascript
// 绑定model并使用
import React from 'react';
import { connect } from '@lugia/lugiax';
import { userModel } from './model';
class UserList extends React.Component {
  state = { loading: false };
  render() {
    const { list, removePeopleById, asyncRemovePeopleById } = this.props;
    return (
      <div>
                
        {list.map(item => {
          console.log('item', item);
          return (
            <p>
                            <span>姓名：{item.name}</span>
                            <span>&nbsp;年龄：{item.age}</span>
                            <span>&nbsp;爱好：{item.hobby}</span>
                            <span
                onClick={() => {
                  asyncRemovePeopleById(item);
                }}
              >
                                &nbsp;异步删除               
              </span>
                            
              <span
                onClick={() => {
                  removePeopleById(item);
                }}
              >
                                &nbsp;同步删除               
              </span>
                          
            </p>
          );
        })}
              
      </div>
    );
  }
}
const App = connect(
  userModel,
  state => {
    const list = state.get('peoples').toJS ? state.get('peoples').toJS() : state.get('peoples');
    return { list };
  },
  mutations => {
    console.log(mutations);
    const { removePeopleById, asyncRemovePeopleById } = mutations;
    return {
      removePeopleById,
      asyncRemovePeopleById,
    };
  }
)(UserList);
export default App;
```

## API 说明

### registerPersistence

注册获取和存储数据持久化控制器

#### 方法使用简述

```javascript
import { registerPersistence } from '@lugia/lugiax-persistence';
registerPersistence(
  (persistDataFn: {
    getStore(model: string): Object,
    saveStore(model: string, state: Object): boolean,
  }),
  (name: string = 'default')
);
```

> 数据持久化控制器是用来指定用户存储 state 的方法和获取 state 的方法

```plain
数据持久化控制器结构
{
    getStore (model: string): Object,
    saveStore (model: string, state: Object): boolean
}
```

### getPersistence

根据名称控制器名字获取数据持久化控制器

#### 方法使用简述

```javascript
import { getPersistence } from '@lugia/lugiax-persistence';
getPersistence((name: string));
```

### clearPersistence

清除所有数据持久化控制器

#### 方法使用简述

```javascript
import { clearPersistence } from '@lugia/lugiax-persistence';
clearPersistence();
```

### unRegisterPersistence()

根据控制器名字注销数据持久化控制器

#### 方法使用简述

```javascript
import { unRegisterPersistence } from '@lugia/lugiax-persistence';
unRegisterPersistence((name: string));
```

### @lugia/lugiax-persistence 导出默认方法

将 state 变成持久化的 state

#### 方法使用简述

```javascript
import lugaix from '@lugia/lugiax';
import wrapPersistence from '@lugia/lugiax-persistence';
lugaix.register(
  wrapPersistence(
    { model: string,state: Object,mutations?: Mutations},
    { name: string ='default'}
  )
)
```

# API 说明

## lugiax

### lugiax.register()

在 lugiax 中注册 model。注册好的 model 可以通过 bind、bindTo、connect 方法和组件进行绑定。

### lugiax.getState()

获取 lugiax 中的所有 model 的 state 状态。

### lugiax.getModelData(nodeName:string)

通过模块名称获取，这个 model 中的 state

### lugiax.clear()

清空 lugiax 中所有的 model 对象。

### lugiax.on()

mutation 被调用后都会触发 on 方法中的回调函数。

### bind()

将表单类型组件和单个 model 进行双向绑定，需开发者指定 model 中 mutations 和组件事件的调用关系

### bindTo()

将表单类型组件和单个 model 进行双向绑定，是对 bind 的一种封装。会自动生成 mutations 和组件事件的调用关系。

### connect()

将组件和一个或者多个 model 进行单向绑定。

## lugiax-router

### createApp

创建根路由

### createRoute

创建子路由

### render

将 component 组件渲染到指定 id 的 dom 元素

```
render(component, domId)
```

### go

路由调转方法，当参数是 url 时，跳转到对应路由。如果参数是 count 时，当 count 为正数时路由前进到对应的次数路由。如果 count 为负数时路由后退到对应的次数的路由。

```
go({ count: -2, });
go({ url: '/sport'});
```

### goBack

路由后退一次

```
等价于 go({ count: -1 });
```

### goForward

路由向前一次

```
等价于 go({ count: 1 });
```

### replace

把当前 A 的路由地址替换成参数中 url 的路由地址。

```
replace({ url: '/sport', });
```

### Redirect

等价于 react-router 中的 Redirect

### Link

路由跳转的标签，to 属性用来表示跳转指定的路由，最终标签会被便以为 a 标签展示在页面上

```
 <Link to="/sport" id="sport"> 运动 </Link>
```
