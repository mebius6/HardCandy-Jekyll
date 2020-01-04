---
layout: post
title: "React-Native总结"
date: 2018-08-17
tags: 随笔

color: "rgb(154,133,255)"
---

### 1.windows-bug 启动不成功--清除 andriod 残余文件

- cd android
- gradlew clean

### 2.View Style

#### 支持 Flexbox、ShadowPropTypesIOS、Transforms 属性。

#### 背面可见性

- backfaceVisibility enum('visible', 'hidden')

#### 背景颜色

- backgroundColor string

#### 边框颜色

- borderColor string
- borderTopColor string
- borderRightColor string
- borderBottomColor string
- borderLeftColor string

#### 边框圆角半径

- borderRadius number
- borderTopLeftRadius number
- borderTopRightRadius number
- borderBottomLeftRadius number
- borderBottomRightRadius number

#### 边框样式

- borderStyle enum('solid', 'dotted', 'dashed')

#### 边框宽度

- borderWidth number
- borderTopWidth number
- borderRightWidth number
- borderBottomWidth number
- borderLeftWidth number

#### 不透明度

- opacity number

#### 填充

- overflow enum('visible', 'hidden')

#### 测试 ID（用来定位视图）

- testID string

### Image Style

#### 支持 Flexbox 和 Transforms

#### 调整大小模式

- resizeMode Object.keys(ImageResizeMode)

#### 背景颜色

- backgroundColor string

#### 边框属性

- borderColor string
- borderWidth number
- borderRadius number

#### 填充

- overflow enum('visible', 'hidden')

#### 色彩颜色

- tintColor string

#### 不透明度

- opacity number

### Text Style

#### 支持 View 的样式

#### 字体颜色

- color string

#### 字体

- fontFamily string

#### 一行省略

- numberOfLines number

#### 字体大小

- fontSize number

#### 字体样式

fontStyle enum('normal', 'italic')
字体粗细（指定字体的粗细。大多数字体都支持’normal’和’bold’值。并非所有字体都支持所有的数字值。如果某个值不支持，则会自动选择最接近的值。）

- fontWeight enum("normal", 'bold', '100', '200', '300', '400', '500', '600', '700', '800', '900')

#### 字间距

- letterSpacing number

#### 行间距

- lineHeight number

#### 字体对齐方式（指定文本的对齐方式。其中’justify’值仅 iOS 支持。）

- textAlign enum("auto", 'left', 'right', 'center', 'justify')

Flexbox Style

#### 宽高

- width number
- height number

#### 项目对齐

- alignItems enum('flex-start', 'flex-end', 'center', 'stretch')

#### 自身对齐

- alignSelf enum('auto', 'flex-start', 'flex-end', 'center', 'stretch')

#### 边框宽度

- borderBottomWidth number
- borderLeftWidth number
- borderRightWidth number
- borderTopWidth number
- borderWidth number

#### 弹性伸缩

- flex number

#### 弹性伸缩方向

- flexDirection enum('row', 'column')

#### 弹性伸缩包裹

- flexWrap enum('row', 'row-reverse', 'column', 'column-reverse)

##### 属性内容

- justifyContent enum('flex-start', 'flex-end', 'center', 'space-between', 'space-around')

#### 外边距

- margin number
- marginBottom number
- marginLeft number
- marginRight number
- marginTop number
- marginHorizontal number
- marginVertical number

#### 内边距

- padding number
- paddingBottom number
- paddingLeft number
- paddingRight number
- paddingTop number
- paddingHorizontal number
- paddingVertical number

#### 位置（绝对、相对）

- position enum('absolute', 'relative')

#### 上下左右

- right number
- top number
- left number
- bottom number

#### Transform 属性变化

- transform [
  - {perspective: number},
  - {rotate: string},
  - {rotateX: string},
  - {rotateY: string},
  - {rotateZ: string},
  - {scale: number},
  - {scaleX: number},
  - {scaleY: number},
  - {translateX: number},
  - {translateY: number},
  - {skewX: string},
  - {skewY: string}
    ]

#### 属性矩阵

- transformMatrix TransformMatrixPropType

### 3.adb server version (39) doesn't match this client (40)

#### Genymotion==>设置==>ADB=>Use custom Android SDK Tools=>选择安卓 adk 目录，然后 cmd 输入

- adb kill-server

- adb start-server

### 4.Genymotion 不能安装 APK 的解决方法

- 1、下载 Genymotion-ARM-Translation-Librarities 工具转换包；下载路径:https://pan.baidu.com/s/1sWe36TSZHPxqiaLn1TGPig

- 2、将下载号的工具包直接拖拽到 Genymotion 中，

- 3、进行操作

  - 1.adb shell

  - 2.cd /sdcard/Download/

  - 3.sh /system/bin/flash-archive.sh /sdcard/Download/Genymotion-ARM-Translation.zip

  - 4.重启模拟器。

### 5.运行 react-native init Hello 之后报错 "import type {CommandT} from './commands';"

- 更新的 react-native 0.56 版本在 Windows 上有 bug 不能正常运行，所以才会一直报错，然后我试了一下 react-native init xxx --version 0.55.4，即初始化时指定版本

- 可以进入项目的文件夹，然后

  - C:\Users\13708\Desktop\aaa\Hello>react-native -v
  - react-native-cli: 2.0.1
  - react-native: 0.56.0
  - 就可以看到自己项目的版本了

### 6.正确地引入 antd-mobile 的步骤：

- (1)npm intall react-dom --save
- (2)npm install antd-mobile --save
- (3)npm install babel-plugin-import --save-dev
- (4).babelrc 中为

```json
{

“plugins”：[["import",{"libraryName":"antd-mobile"}]]，

"presets":["react-native"]

}
```

- (5)在代码中引入 antd-mobile，然后正确的运行

### 7.指定端口启动项目

```js
react-native run-android --port 5000
react-native run-android=8082
```

### 7.touchouable 的区别

| 名称                     | 描述                                                                                                                                          | 方法                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| TouchableWithoutFeedback | 触摸后在视觉上没有任何点击效果反馈,<span style="color:#ff9900">该标签不可以添加 style 样式属性，加了也没效果</span>                           | onPress function 当触摸操作结束时调用，但如果被取消了则不调用（譬如响应者被一个滚动操作取代）<hr> onLongPress function 当用户长时间按压组件(长按效果)的时候调用该方法。<hr> disabled bool 如果设为 true，则禁止此组件的一切交互。<hr>onPressIn function 与 onPressOut function 分别是当用户开始点击按钮时与点击结束后被回调。                                                                                                                                                                                                                                                                                                        |
| TouchableOpacity         | 触摸后实现透明度的触摸反馈 <span style="color:#ff9900">完全同 TouchableHighlight，只是不可以修改颜色，只能修改透明度，可以添加样式效果</span> | 方法同 TouchableHighlight                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| TouchableHighlight       | 触摸后实现高亮的触摸反馈 <span style="color:#ff9900">该标签可以添加 style 样式属性，因为有一层 View 可以使用</span>                           | activeOpacity number 用于设置被按下时按钮的不透明度，默认是 0.85<hr>underlayColor color 用于设置被按下时按钮的颜色，默认是 black<hr> onShowUnderlay function 最外层的 View 显示的时候调用<hr>onHideUnderlay function 最外层的 View 被隐藏的时候调用<hr>style View#style 最外层的 View 可以设置样式<hr>onPress function 当触摸操作结束时调用，但如果被取消了则不调用（譬如响应者被一个滚动操作取代）<hr> onLongPress function 当用户长时间按压组件(长按效果)的时候调用该方法。<hr> disabled bool 如果设为 true，则禁止此组件的一切交互。<hr> onPressIn function 与 onPressOut function 分别是当用户开始点击按钮时与点击结束后被回调。 |
| TouchableNativeFeedback  | 在 Android 5.0 以后触摸实现水波纹的效果，（因此仅限 Android 平台，iOS 平台使用会报错）                                                        | TouchableNativeFeedback 在 TouchableWithoutFeedback 所支持的属性的基础上增加了按下去的水波纹效果。我们可以通过 background 属性来自定义原生触摸操作反馈的背景                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |

#### 示例

```js

// TouchableWithoutFeedback
<TouchableWithoutFeedback
  onPress={() => {
    alert("触摸效果");
  }}
  onLongPress={() => {
    alert("长按效果");
  }}
  disabled={false} //默认是false，如果是true表示关闭该组件的触摸功能
  onPressIn={() => {
    alert("触摸开始");
  }}
  onPressOut={() => {
    alert("触摸结束");
  }}
>
  <View>
    <Text>TouchableWithoutFeedback的测试</Text>
  </View>
</TouchableWithoutFeedback>

// TouchableHighlight
<TouchableHighlight
  style={{}}
  activeOpacity={0.6}
  underlayColor="red"
  onHideUnderlay={() => {
    alert("衬底被隐藏");
  }}
  onShowUnderlay={() => {
    alert("衬底显示");
  }}
>
  <View>
    <Text>TouchableHighlight的测试</Text>
  </View>
</TouchableHighlight>
```

#### 提示

提示一：无论是 TouchableWithoutFeedback 还是其他三种 Touchable 组件，都是在根节点都是只支持一个组件，如果你需要多个组件同时响应单击事件，可以用一个 View 将它们包裹着，它的这种根节点只支持一个组件的特性和 ScrollView 很类似。

```js
//错误示范：其他的Touchable系列同样
<TouchableHighlight
    style={{}}
   onPress={() => {alert('123')}}
>
    <View><Text>TouchableHighlight的测试</Text></View>
    <View><Text>这是错误的，TouchableHighlight中只能有一个总节点包裹</Text></View>
</TouchableHighlight>

//正确
<TouchableHighlight
    style={{}}
   onPress={() => {alert('123')}}
>
    <View>
        <View><Text>TouchableHighlight的测试</Text></View>
        <View><Text>这是错误的，TouchableHighlight中只能有一个总节点包裹</Text></View>
    </View>
</TouchableHighlight>
```

提示二： <Text>组件也有点击功能 可通过 onPress 实现触摸回调

```js
//Text也可以实现点击功能
<Text
    style={{}}
    onPress={()} => {alert('Text组件的点击功能')}
    >Text的点击功能
</Text>
```
