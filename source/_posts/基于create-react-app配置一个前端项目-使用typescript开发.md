---
title: 基于create-react-app配置一个前端项目(使用typescript开发)
categories:
- Web前端
date: 2020-06-02 21:01:48
tags:
- React
- TypeScript
---

顺便记录下遇到的坑

<!--more-->


# 使用`npx`命令创建新 react 项目
```
npx create-react-app ts-react-app --typescript
```

# 安装需要的 npm 包

```
//package.json
{
"dependencies":{
    "antd": "^4.3.0",
    "axios": "^0.19.2",
    "react-router-dom": "^5.2.0",
},
"devDependencies": {
    "less": "^3.11.2",
    "less-loader": "^6.1.0",
    "@types/react-router-dom": "^5.1.5",
    "babel-plugin-import": "^1.13.0",
    "react-app-rewired": "^2.1.6",
    "customize-cra": "^1.0.0",
    "http-proxy-middleware": "^1.0.4",
    "http-server": "^0.12.3", 、
  }
}
```

解释一下安装的依赖：
babel-plugin-import：用于 antd 的按需加载。
customize-cra 和 react-app-rewired：create-react-app 隐藏了 webpack 配置，这两个插件能让我们配置 webpack。
http-server 和 http-proxy-middleware：用于搭建 mock server。
less 和 less-loader：引入 antd 的 less 样式文件，这里有个坑，后面介绍。

使用了 react-app-rewired 之后，package.json 文件中的 script 字段也要做相应修改：
```
//package.json
{
...
    "scripts": {
-       "start": "react-scripts start",
+       "start": "react-app-rewired start",
-       "build": "react-scripts build",
+       "build": "react-app-rewired build",
-       "test": "react-scripts test",
+       "test": "react-app-rewired test --env=jsdom",
        "eject": "react-scripts eject"
    },
}
```

# 解决 antd 的按需加载
根目录下创建 config-overrides.js 文件，添加以下代码：
```
const {
    override,
    fixBabelImports,
    addLessLoader,
} = require("customize-cra");

module.exports = override(
    fixBabelImports('import', {
        libraryName: 'antd',
        libraryDirectory: 'es',
        style: true
    }),
    addLessLoader({
        lessOptions: {
            javascriptEnabled: true,
            modifyVars: { '@primary-color': '#1DA57A' },
        },
    })
)
```

前面说到这里有个坑，使用的 less-loader 版本不一样的话，config-overrides.js的写法也不一样，上面的写法是对应 less-loader 6.0 以上版本的。具体区别如下图，详情参见
[Customization of theme is broken with latest version of less-loader #23624](https://github.com/ant-design/ant-design/issues/23624)
{% asset_img issue.jpg GitHub Issue %}

# 组件写法
## 函数组件
```tsx
import React from "react";
import {Button} from 'antd';


interface Greeting {
    name: string,
    firstName?: string,
    lastName?: string
}


const Hello=(props:Greeting)=>{
    return (
        <div>
            <div>Hello {props.name}</div>
            <Button type="primary">按钮</Button>
        </div>
    )
}

// const Hello: React.FC<Greeting> = (props) => {
//     return (
//         <div>
//             <div>Hello {props.name}</div>
//             <div>{props?.firstName} {props?.lastName}</div>
//             <Button type="primary">按钮</Button>
//             <div>{props.children}</div>
//         </div>
//     );
// }

Hello.defaultProps={
    firstName:'',
    lastName:''
}

export default Hello;

```

## 类组件
```tsx
import React, {Component} from "react";
import {Button} from 'antd';


interface Greeting {
    name: string,
    firstName?: string,
    lastName?: string
}

interface State {
    count: number
}

class HelloClass extends Component<Greeting, State> {
    public state: State = {
        count: 0
    }

    static defaultProps = {
        firstName: '',
        lastName: ''
    }

    render() {
        return (
            <div>
                <div>Hello {this.props.name}</div>
                <Button onClick={() => {
                    this.setState({count: this.state.count + 1})
                }} type="primary">按钮</Button>
                <div>{this.state.count}</div>
            </div>
        );
    }
}


export default HelloClass;

```

## 高阶组件
```tsx
import React, {Component} from 'react';
import HelloClass from "./HelloClass";

interface Loading {
    loading: boolean
}

function HelloHOC<P>(WrappedComponent: React.ComponentType<P>) {
    return class extends Component<P & Loading> {
        render() {
            const {loading, ...props} = this.props;

            return loading ?
                <div>Loading...</div>
                : <WrappedComponent {...props as P} />;
        }
    }
}

export default HelloHOC(HelloClass);
```


## 使用 Hooks
```tsx
import React, {useState, useEffect} from "react";
import {Button} from 'antd';


interface Greeting {
    name: string,
    firstName?: string,
    lastName?: string
}


const HelloHooks = (props: Greeting) => {
    const [count, setCount] = useState(0);
    const [text, setText] = useState<string | null>(null);
    useEffect(() => {
        if (count > 5) {
            setText("大于 5")
        }
    }, [count]);

    return (
        <div>
            <div>Hello {props.name}</div>
            <Button onClick={() => {
                setCount(count + 1)
            }} type="primary">按钮</Button>
            <div>{count}</div>
            <div>{text}</div>
        </div>
    )
}


HelloHooks.defaultProps = {
    firstName: '',
    lastName: ''
}

export default HelloHooks;

```