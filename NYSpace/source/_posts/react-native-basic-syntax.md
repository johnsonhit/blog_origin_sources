title: React Native 基础
date: 2016/10/09
comments: true
tags: 
- Web 
- React Native
- JavaScript
categories: 
- Cross-platform
---

## 前言
纵观软件开发模式的历史，从过去的 C/S 到 B/S，到如今移动互联网如春笋之势迅猛崛起，互联网的展示形态愈来愈多样化。然而，互联网形态不管以怎样形态示人，其本质仍然是服务端与客户端的数据通信。所以，一个 Web 就紧紧只能局限于网页吗？又或是，互联网软件工程的发展只能是移动终端的天下吗？

必然不是的。跨平台的需求，一直从未舍弃。而愈来愈多的跨平台框架，也火爆开发圈。除了 Facebook 的 React Native ，阿里巴巴也开源了自己研发的 weex ，结合 Vue.js 进行跨平台开发。而微信也开始做微信小应用这样的尝试。所以，现在要跟上潮流，打通任督二脉，探索未来应用开发的发展。

而在这么美好的大饼实现之前，必然是要先务实地学习如何使用这些开源轮子做开发。（以上自认真心扯）

本系列分为 React Native 、weex+Vue.js 、微信小应用三个开源框架的基础学习。希望对读者有用。

如果有需要交流，请见文末联系方式。

## React Native 基础语法
本篇文章都以 ES6 语法规范为本。

### State 和 Props 简介

```
Props 
Most components can be customized when they are created, with different parameters. These creation parameters are called props.

State 
There are two types of data that control a component: props and state. props are set by the parent and they are fixed throughout the lifetime of a component. For data that is going to change, we have to use state.
```
Props
大部分组件创建时，可以由不同的参数进行自定义。这些创建的参数被称之为属性 props 。


### 属性

```JavaScript
class Video extends React.Component {
    static defaultProps = {
        autoPlay: false,
        maxLoops: 10,
    };  // 注意这里有分号
    static propTypes = {
        autoPlay: React.PropTypes.bool.isRequired,
        maxLoops: React.PropTypes.number.isRequired,
        posterFrameSrc: React.PropTypes.string.isRequired,
        videoSrc: React.PropTypes.string.isRequired,
    };  // 注意这里有分号
    render() {
        return (
            <View />
        );
    } // 注意这里既没有分号也没有逗号
}
```
其中，**defaultProps** 是为属性赋初始值，**propTypes** 是为属性赋予数据类型。
并且属于类的成员变量


### 状态
```JavaScript
class Video extends React.Component {
    state = {
        loopsRemaining: this.props.maxLoops,
    }
}
```

需要理解的是，props是一个父组件传递给子组件的数据流，这个数据流可以一直传递到子孙组件。而state代表的是一个组件内部自身的状态（可以是父组件、子孙组件）。
改变一个组件自身状态，从语义上来说，就是这个组件内部已经发生变化，有可能需要对此组件以及组件所包含的子孙组件进行重渲染。
而props是父组件传递的参数，可以被用于显示内容，或者用于此组件自身状态的设置（部分props可以用来设置组件的state），不仅仅是组件内部state改变才会导致重渲染，父组件传递的props发生变化，也会执行。
既然两者的变化都有可能导致组件重渲染，所以只有理解pros与state的意义，才能很好地决定到底什么时候用props或state
State
如果component的某些状态需要被改变，并且会影响到component的render，那么这些状态就应该用state表示。 例如：一个购物车的component，会根据用户在购物车中添加的产品和产品数量，显示不同的价格，那么“总价”这个状态，就应该用state表示。

Props
如果component的某些状态由外部所决定，并且会影响到component的render，那么这些状态就应该用props表示。 例如：一个下拉菜单的component，有哪些菜单项，是由这个component的使用者和使用场景决定的，那么“菜单项”这个状态，就应该用props表示，并且由外部传入。

### 模块化
模块功能主要由两个命令构成： export 和 import 。 export 命令用于规定模块的对外接口，import 命令用于输入其他模块提供的功能。

- export 命令

```JavaScript
如果 export default {} 定义，那么在文件中引入组件时 import a from 'a' 

如果 export const b = {} 定义，那么在文件中引入组件时 import { b } from 'b'
```
- export default 命令

从前面的例子可以看出，使用 import 命令的时候，用户需要知道所要加载的变量名或函数名，否则无法加载。但是，用户肯定希望快速上手，未必愿意阅读文档，去了解模块有哪些属性和方法。

使用 export 命令定义了模块的对外接口以后，其他JS文件就可以通过import命令加载这个模块（文件）。

```JavaScript
// constant-scene.js
export const kSceneWayBillMain = 'ny.scene.waybill.main';
export const kSceneWayBillPrePickupList = 'ny.scene.waybill.prepickup';

// scene-login.js
import {kSceneLogin, kSceneWayBillMain} from './constant-scene';
// some code ...
class SaboRN extends React.Component {
  constructor(props) {
    super(props);
  }
  render() {
    var route = null;
    if (AppContext.token == null) {
      route = {title: '登录', scene: kSceneLogin};
    } else {
      route = {title: '运单', scene: kSceneWayBillMain};
    }
    return (
      <Navigator
        initialRoute={route}
        renderScene={ this._renderScene.bind(this) }
      />
    )
  }

  _renderScene(route, navigator) {
    var scene = null;
    if (route.scene == kSceneLogin) {
      scene = <LoginScene navigator={navigator} />
    } else {
      scene = <MainScene navigator={navigator} />
    }
    return (
      scene
    );
  }
}
```
以上代码就是在入口 js 文件中使用 constant-scene.js 定义的常量。


## Component 实践

### 自定义一个 Component

```JavaScript
import React, { Component } from 'react';
import { View, TouchableOpacity,Text } from 'react-native';
import { SCREEN_HEIGHT, SCREEN_WIDTH } from './constant-global-definition';

let lineHeight = 0.25;
let buttonHeight = 50;
let themeColor = '#FECE56';
let grayLight = '#EEEEEE';
let grayMiddle = '#999999';
let grayDark = '#666666';
let fontColor = '#333333';
let white = '#FFFFFF';

export class Line extends React.Component {
  static propTypes = {
    left: React.PropTypes.number.isRequired,
    top: React.PropTypes.number.isRequired,
    width: React.PropTypes.number.isRequired,
    height: React.PropTypes.number,
    color: React.PropTypes.string,
  };
  static defaultProps = {
    left:0,
    top:0,
    width: SCREEN_WIDTH,
    height: lineHeight,
    color: grayLight,
  }
  render() {
    let left = this.props.left;
    let top = this.props.top;
    let width = this.props.width;
    let height = this.props.height;
    let color = this.props.color;

    return(
      <View style={{height: lineHeight, backgroundColor: color, width: width, marginLeft: left, marginTop: top}} />
    );
  }
}

export class CommonButton extends React.Component {
  static propTypes = {
    left: React.PropTypes.number.isRequired,
    top: React.PropTypes.number.isRequired,
    width: React.PropTypes.number.isRequired,
    height: React.PropTypes.number,
    title: React.PropTypes.string,
    titleColor: React.PropTypes.string,
    backgroundColor: React.PropTypes.string,
    cornerRadius: React.PropTypes.number,
  };
  static defaultProps = {
    left:0,
    top:0,
    width: SCREEN_WIDTH,
    buttonHeight: buttonHeight,
    title: 'Button Title',
    titleColor: white,
    backgroundColor: themeColor,
    buttonPressed: () => {console.log('pressed'); return;},
    cornerRadius: 5,
  }

  render() {
    padding = this.props.cornerRadius;
    var buttonStyle = {
      padding:padding,
      marginTop: this.props.top,
      marginLeft: this.props.left,
      width: this.props.width,
      height: this.props.buttonHeight,
      backgroundColor: this.props.backgroundColor,
      borderRadius : this.props.cornerRadius,
    };
    paddingEdge = 2 * padding;
    var textStyle = {
      width: this.props.width - paddingEdge,
      height: this.props.buttonHeight - paddingEdge,
      lineHeight: this.props.buttonHeight - paddingEdge,
      color: this.props.titleColor,
      textAlign: 'center',
      backgroundColor: this.props.backgroundColor,
    };

    return(
      <View style={buttonStyle}>
        <TouchableOpacity onPress={this.props.buttonPressed}>

          <Text style={textStyle}>{this.props.title}</Text>
        </TouchableOpacity>
      </View>
    );
  }
}

```
以上就是定义了默认的一条线和另一个按钮的组件，并将这两个组件对外开放使用。其中，

### Navigator 注意

Navigator 并不是原生的 UINavigationController 。它是 React Native 自定的导航实例，但是可以根据 Navigator route 来判断当前的页面，并且父类可以通过 props 将 Navigator 传递给子类，并且由子类进行页面跳转的控制。上面的代码就是 Navigator 的初始化，而在子类中，获得 Navigator 则需要利用 props。

```JavaScript
  _goToMain() {
    route = {title:'运单', kSceneWayBillMain};
    this.props.navigator.push(route);
  }
```

## 桥接
原生代码和 JS 代码进行通信需要创建具体的 Module 类，并且，必须保证 JS 的模块名，和原生代码的类名保持一致，否则在应用运行时，将找不到模块而崩溃。

### 在 JS 文件中定义桥接模块
```JavaScript
import { NativeModules } from 'react-native';

var RNBridgeModule = NativeModules.RNBridgeModule;

export function addEvent(eventName, location) {
  RNBridgeModule.addEvent(eventName, location);
}
```

### 在 JS 文件中使用模块方法
```
import { addEvent } from './app-native-module'
// some code ...
addEvent('aaa', 'bbb');
```
注意，在定义桥接模块的时候，直接将 addEvent 方法对外暴露，所以直接使用方法即可。

### 创建 Objective-C 程序文件
```objc
// 头文件
#import <Foundation/Foundation.h>
#import <RCTBridgeModule.h>

@interface RNBridgeModule : NSObject <RCTBridgeModule>

+ (void)setup;
+ (instancetype)sharedManager;

@end

// 实现文件
#import "RNBridgeModule.h"

@implementation RNBridgeModule

+ (instancetype)sharedManager {
    static RNBridgeModule *manager = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        manager = [[self alloc] init];
    });
    return manager;
}

+ (void)setup {
    [RNBridgeModule sharedManager];
}

RCT_EXPORT_MODULE();

RCT_EXPORT_METHOD(addEvent:(NSString *)event location:(NSString *)location) {
    NSLog(@"%@ %@", event, location);
}

@end
```
在头文件中需要引入 RCTBridgeModule 头文件，并且要遵循 RCTBridgeModule 协议。

实现文件中，必须写上 **RCT_EXPORT_MODULE();** 方法。

**In addition to implementing the RCTBridgeModule protocol, your class must also include the RCT_EXPORT_MODULE() macro. This takes an optional argument that specifies the name that the module will be accessible as in your JavaScript code (more on this later).**

然后，自定义自己的桥接函数。这样在原生调用的时候，就能跟踪到， JS 的模块方法之中。

## Debug
原生应用调试如果一直输出 log 会很难定位到具体代码，那么就开启 Remote Debug 比较好，然后利用 Chrome 的调试工具进行 JS 的调试。

## 报错整理
### React Native 相关
- 却少符号

比如，结尾的}
```
[1] index.ios.js: Unexpected token (18:7)
[2] SyntaxError /Users/niyao/N.Y./Demo/ny_native/ny_react/scene-login.js: Unexpected token, expected } (47:28)

```

- 元素的属性（实例的方法，属性）

```
Unhandled JS Exception: Element type is invalid: expected a string (for built-in components) or a class/function (for composite components) but got: undefined. Check the render method of `Navigator`.
```

- 模块化的写法不对导致如下错误

```
Unable to resolve module login-view from /Users/niyao/N.Y./Demo/ny_native/ny_react/index.ios.js: Unable to find this module in its module map or any of the node_modules directories under /Users/node_modules/login-view and its parent directories

This might be related to https://github.com/facebook/react-native/issues/4968
To resolve try the following:
  1. Clear watchman watches: `watchman watch-del-all`.
  2. Delete the `node_modules` folder: `rm -rf node_modules && npm install`.
  3. Reset packager cache: `rm -fr $TMPDIR/react-*` or `npm start -- --reset-cache`.
2016-11-10 16:39:58.732 [fatal][tid:main] Unable to resolve module login-view from /Users/niyao/N.Y./Demo/ny_native/ny_react/index.ios.js: Unable to find this module in its module map or any of the node_modules directories under /Users/node_modules/login-view and its parent directories

This might be related to https://github.com/facebook/react-native/issues/4968
To resolve try the following:
  1. Clear watchman watches: `watchman watch-del-all`.
  2. Delete the `node_modules` folder: `rm -rf node_modules && npm install`.
  3. Reset packager cache: `rm -fr $TMPDIR/react-*` or `npm start -- --reset-cache`.
```

- JSX 没有结束

```
SyntaxError /Users/niyao/N.Y./Demo/ny_native/ny_react/index.ios.js: Adjacent JSX elements must be wrapped in an enclosing tag (26:6)
```

- JSX 的属性语法错误

```
[fatal][tid:main] SyntaxError /Users/niyao/N.Y./Demo/ny_native/ny_react/index.ios.js: Unterminated JSX contents (41:47)
```

- 空对象

```
 Unhandled JS Exception: undefined is not a function (evaluating 'navigator.getCurrentRoutes(0)')
```

- 绑定 this

在ES5下，React.createClass会把所有的方法都bind一遍，这样可以提交到任意的地方作为回调函数，而this不会变化。但官方现在逐步认为这反而是不标准、不易理解的。

在ES6下，你需要通过bind来绑定this引用，或者使用箭头函数（它会绑定当前scope的this引用）来调用
```
Unhandled JS Exception: undefined is not an object (evaluating 'this.props.navigator')
```
http://stackoverflow.com/questions/31304017/react-native-navigatorios-undefined-is-not-an-object-evaluating-this-props-n

### Native 报错
iOS 忘了设置 status bar 的 plist 属性
```
[RCTStatusBarManager.m:118] RCTStatusBarManager module requires that the                 UIViewControllerBasedStatusBarAppearance key in the Info.plist is set to NO
```

- state

要传进状态对象，指定需要修改的 Key 的对象
```
Unhandled JS Exception: setState(...): takes an object of state variables to update or a function which returns an object of state variables.
// wrong
  addCount() {
    console.log(this.state.count);
    this.setState(this.state.count+1);
  }
// right
  addCount() {
    console.log(this.state.count);
    this.setState({count: this.state.count+1});
  }
```

- RN CSS 样式出错

```
Unhandled JS Exception: "lineHight" is not a valid style property.
StyleSheet inputStyle: {
  "height": 44,
  "backgroundColor": "#fff",
  "fontSize": 14,
  "lineHight": 44,
  "padding": 5
}
Valid style props: [
  "alignItems",
  "alignSelf",
  "backfaceVisibility",
  "backgroundColor",
  "borderBottomColor",
  "borderBottomLeftRadius",
  "borderBottomRightRadius",
  "borderBottomWidth",
  "borderColor",
  "borderLeftColor",
  "borderLeftWidth",
  "borderRadius",
  "borderRightColor",
  "borderRightWidth",
  "borderStyle",
  "borderTopColor",
  "borderTopLeftRadius",
  "borderTopRightRadius",
  "borderTopWidth",
  "borderWidth",
  "bottom",
  "color",
  "decomposedMatrix",
  "elevation",
  "flex",
  "flexDirection",
  "flexWrap",
  "fontFamily",
  "fontSize",
  "fontStyle",
  "fontVariant",
  "fontWeight",
  "height",
  "justifyContent",
  "left",
  "letterSpacing",
  "lineHeight",
  "margin",
  "marginBottom",
  "marginHorizontal",
  "marginLeft",
  "marginRight",
  "marginTop",
  "marginVertical",
  "maxHeight",
  "maxWidth",
  "minHeight",
  "minWidth",
  "opacity",
  "overflow",
  "overlayColor",
  "padding",
  "paddingBottom",
  "paddingHorizontal",
  "paddingLeft",
  "paddingRight",
  "paddingTop",
  "paddingVertical",
  "position",
  "resizeMode",
  "right",
  "rotation",
  "scaleX",
  "scaleY",
  "shadowColor",
  "shadowOffset",
  "shadowOpacity",
  "shadowRadius",
  "textAlign",
  "textAlignVertical",
  "textDecorationColor",
  "textDecorationLine",
  "textDecorationStyle",
  "textShadowColor",
  "textShadowOffset",
  "textShadowRadius",
  "tintColor",
  "top",
  "transform",
  "transformMatrix",
  "translateX",
  "translateY",
  "width",
  "writingDirection",
  "zIndex"
]
```

### node 服务错误
目录错了
```
NYSpace npm start
npm ERR! Darwin 15.6.0
npm ERR! argv "/usr/local/Cellar/node/5.11.0/bin/node" "/usr/local/bin/npm" "start"
npm ERR! node v5.11.0
npm ERR! npm  v3.8.7

npm ERR! missing script: start
```

端口占用
```
> SaboRN@0.0.1 start /Users/niyao/Company/Projects/YouCaiDriver/Sabo/SaboRN
> node node_modules/react-native/local-cli/cli.js start

Scanning 681 folders for symlinks in /Users/niyao/Company/Projects/YouCaiDriver/Sabo/SaboRN/node_modules (17ms)
 ┌────────────────────────────────────────────────────────────────────────────┐ 
 │  Running packager on port 8081.                                            │ 
 │                                                                            │ 
 │  Keep this packager running while developing on any JS projects. Feel      │ 
 │  free to close this tab and run your own packager instance if you          │ 
 │  prefer.                                                                   │ 
 │                                                                            │ 
 │  https://github.com/facebook/react-native                                  │ 
 │                                                                            │ 
 └────────────────────────────────────────────────────────────────────────────┘ 
Looking for JS files in
   /Users/niyao/Company/Projects/YouCaiDriver/Sabo/SaboRN 

[11:33:01 AM] <START> Building Dependency Graph
[11:33:01 AM] <START> Crawling File System
 ERROR  Packager can't listen on port 8081
```

## 奇葩记录
### 
StatusBar 必须在 render() 方法返回才能生效
```JavaScript
import React, { Component } from 'react';
import {
  Dimensions,
  Platform,
  View,
  ListView,
  TextInput,
  StyleSheet,
  ScrollView,
  Text,
  RefreshControl,
  Image,
  ToolbarAndroid,
  InteractionManager,
  TouchableOpacity,
  TouchableHighlight,
  Linking,
  StatusBar,
} from 'react-native';

export default class LoginScene extends React.Component {
  constructor(props) {
    super(props);
    console.log('login constructor');
    this.state = {
      animated: false,
      hidden: true,
      showHideTransition: 'slide',
    }
  }
  render() {
    return(
      <View>
      <StatusBar
        hidden={true}
        showHideTransition={'slide'}
        animated={false}
        />
      </View>
    );
  }

  renderStatusBar() {
    return(
      <StatusBar
        hidden={this.state.hidden}
        showHideTransition={this.state.showHideTransition}
        animated={this.state.animated}
        />
    );
  }
}

var loginStyles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
  }
});

```

### 函数引用
this.goToMain 相当于函数名的引用，this.goToMain() 则会在渲染之后直接执行
```JavaScript
  <CommonButton left={kMargin} top={80} width={kLineWidth} title='登录' buttonPressed={this.goToMain} />
```

### 方法绑定 this

```JavaScript
import React,{ Component } from 'react';
import {
  View,
  Text,
  StyleSheet,
} from 'react-native';

import { CommonButton } from './custom-component'
import { addEvent } from './app-native-module'

export default class MainScene extends React.Component {
  constructor(props) {
    super(props);
  }

  render() {
    return(
      <View>
      {this.renderContent()}
      </View>
    );
  }

  renderContent() {
    return(
      <View style={mainStyles.container}>
        <Text>Main Scene</Text>
        <CommonButton
          title='登出'
          buttonPressed={this._logout.bind(this)}
        />

        <CommonButton
          top={30}
          title='Native 测试'
          buttonPressed={this._addEvent.bind(this)}
         />
      </View>
    );
  }

  _logout() {
    this.props.navigator.pop();
  }

  _addEvent() {
    addEvent('aaa', 'bbb');
  }
}

var mainStyles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
  }
});

```
不绑定就会报错，具体原因见报错整理。
ren
```
This might be related to https://github.com/facebook/react-native/issues/4968
To resolve try the following:
  1. Clear watchman watches: `watchman watch-del-all`.
  2. Delete the `node_modules` folder: `rm -rf node_modules && npm install`.
  3. Reset packager cache: `rm -fr $TMPDIR/react-*` or `npm start -- --reset-cache`.
2016-11-25 15:35:23.465 [fatal][tid:main] Unable to resolve module react/lib/ReactUpdates from /Users/niyao/N.Y./Project/react-native-redux-app/ny_react/node_modules/react-native/Libraries/react-native/react-native.js: Unable to find this module in its module map or any of the node_modules directories under /Users/node_modules/react/lib/ReactUpdates and its parent directories

This might be related to https://github.com/facebook/react-native/issues/4968
To resolve try the following:
  1. Clear watchman watches: `watchman watch-del-all`.
  2. Delete the `node_modules` folder: `rm -rf node_modules && npm install`.
  3. Reset packager cache: `rm -fr $TMPDIR/react-*` or `npm start -- --reset-cache`.
```

## 小结
本篇文章主要总结了，笔者在学习中认为重要的概念，以及开发过程中遇到的很奇葩的问题。
文章会不定期修改完善，如果有问题，或者有需修改的地方，可以联系我 yao.ni@ele.me

也非常欢迎关注我个人的微信公众号～

![](https://niyaoyao.github.io/images/qrcod_ny_1.jpg)

并且自己也有

## 参考资料
http://www.cnblogs.com/LuckyWinty/p/5309088.html
http://bbs.reactnative.cn/topic/15/react-react-native-%E7%9A%84es5-es6%E5%86%99%E6%B3%95%E5%AF%B9%E7%85%A7%E8%A1%A8