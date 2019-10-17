1: 样式

```Javascript
<DataTile style={[styles.tileContainer, {marginBottom: bottom}]} title={item.name} value={item.value}/>
```

2: 默认属性

```swift
static propTypes = {
    // viewBehavior: React.PropTypes.string,
    keyboardType: PropTypes.string,
    includeLayoutAnimation: PropTypes.bool,
    text: PropTypes.string,
    onPress: PropTypes.func,
};

static defaultProps = {
    // viewBehavior: 'padding',
    keyboardType: 'numeric',
    includeLayoutAnimation: true,
    text: 'Done',
    onPress: () => {},
};
```

3:结构

```swift
let { bottom, width, opacity } = this.state;
```

4:颜色

```swift
color: "rgba(255, 255, 255, 0.5)",
```

5: 变量声明 

```swift
let var const  [LINK: https://juejin.im/post/5991549af265da3e345f6e6d?utm_source=gold_browser_extension]

var 声明的变量作用域是最近的函数块；
    
let 声明的变量作用域是最近的闭合块(包含在 {} 之内的作用域)，往往会小于函数块。以 let 关键字创建的变量虽然同样被提升到作用域头部，但是并不能在实际声明前使用.
    
const const 关键字一般用于常量声明，用 const 关键字声明的常量需要在声明时进行初始化并且不可以再进行修改，并且 const 关键字声明的常量被限制于块级作用域中进行访问。
```

6: 对象的引用

```swift
    function changeStuff(a, b, c) { 
        a = a * 10; 
        b.item = "changed”;    //此时, 相当于objc中的引用.
        c = {item: "changed"}; //这里相当于重新new了一个新值
    } 
    
    var num = 10; 
    var obj1 = {item: "unchanged"}; 
    var obj2 = {item: "unchanged"}; 

    changeStuff(num, obj1, obj2); 

    console.log(num);
    console.log(obj1.item);
    console.log(obj2.item); 
    // 输出结果 10 changed unchanged
```

7: 

```swift
getDescription() { return `${this.name} leads ${this.country}`; }
```

8: Promise: 

```swift
或者: 
Promise实例生成以后，可以用then方法分别指定Resolved状态和Reject状态的回调函数。
po.then(function(value) {
  // success
}, function(value) {
  // failure
});
```
           

9: string 非空的判断

```swift
(string) == (string != ‘’ && string != null)
```

10: 关于代码规范: [https://github.com/airbnb/javascript/tree/master/react#basic-rules]

```swift
*Always use double quotes (") for JSX attributes, but single quotes (') for all other JS. eslint: jsx-quotes
// good
<Foo bar="bar" />   
// good
<Foo style={{ left: '20px' }} />

*Omit the value of the prop when it is explicitly true. eslint: react/jsx-boolean-value
// bad
<Foo
hidden={true}
/>
    
// good
<Foo
hidden
/>
```

11: 当组件需要再次更新时会调用下面方法:

```swift
componentWillReceiveProps(nextProps) {
    //refresh the state value to trigger 'render()' method
    this.setState({
        value: nextProps.item.value
    })
}

shouldComponentUpdate(nextProps, nextState) {
    return true;
}

```

12: Style 合成

```swift
<Animated.View style = e.dashView, {opacity: this.state.fadeAnim}]
```

13: 声明为剪头函数的方法会自动绑定当前 self 

```swift
_onQuestionPricePressed = () => {
  
} 
```

14：方法默认参数值

```swift
    static sendPUTRequst(url = '', param = {}) {
        return BTRNApiClientManager.sendPUTRequst(url, param);
    }
```
