#### 创建组件的三种方法
##### 函数式定义的无状态组件
创建纯展示组件，这种组件只负责根据传入的props来展示，不涉及到state状态操作，具体方式
```
ReactDOM.render(
    <Test title="hello,world" />,
    document.getElementById('root')
);

function Test(props) {
    const {title} = props;
    return <div>{title}</div>
}
```
###### 特点
* 组件不会被实例化，整体渲染性能得到了提升
因为组件被精简成render方法的函数来实现，由于是无状态组件，所以无状态组件就不会在有组件实例化的过程，无实例化过程也就不需要分配多余的内存，从而性能都到提升
* 组件不能访问this对象
无状态组件由于没有实例过程，也就无法访问组件this中的对象，如this.state this.ref,均不能访问，若想访问，则不要使用这种方法创建组件
* 组件无法访问生命周期方法
因为无状态组件是不需要组件生命周期管理和状态管理，所以底层实现这种形式的组件时是不会实现组件的生命周期方法，所以无状态组件是不能参与组件的各个生命周期管理的
* 无状态组件只能访问输入的props，同样的props会得到同样的渲染效果，不会有副作用
##### ES5原生方式定义的组件
```
var Test = React.createClass({
    //定义传入props中的属性类型
    propTypes: {
        title: React.PropTypes.string
    },
    //组件默认的props对象
    defaultProps: {
        title: 'test'
    },
    //组件相关的状态对象
    getInitialState: function () {
        return {
            text: this.porps.title || 'placeholder'
        }
    },
    handleChange: function(e) {
        this.setState({
            text: e.target.value
        });
    },
    render: function() {
        return (
            <div>
                Type something:
                <input onChange={this.handleChange} value={this.state.text}></input>
            </div>
        )
    }
})
```
##### ES6形式定义的组件
```
class Test extends React.Component {
    constructor(props) {
        super(props);
        //设置initial state
        this.state = {
            text: props.title || "placeholder"
        };
        this.handleChange = this.handleChange.bind(this);
    }

    handleChange(e) {
        this.setState({
            text: e.target.value+'wj'
        })
    }

    render() {
        return (
            <div>
                Type something:
                <input onChange={this.handleChange}
               value={this.state.text} />
            </div>
        )
    }

};
```
##### React.createClass与React.Component区别
* 函数this自绑定
React.createClass创建的组件，其每一个成员函数的this都有React自动绑定，任何时候使用，直接使用this.method即可，函数中的this会被正确设置
React.Component创建的组件，其成员函数不会自动绑定this，需要开发者手动绑定，否则this不能获取当前组件实例对象。
```
constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this); //构造函数中绑定
}
<div onClick={this.handleClick.bind(this)}></div> //使用bind来绑定
<div onClick={()=>this.handleClick()}></div> //使用arrow function来绑定
```
* 组件属性类型propTypes及其默认props属性defaultProps配置不同
React.createClass在创建组件时，有关组件props的属性类型及组件默认的属性会作为组件实例的属性来配置，其中defaultProps是使用getDefaultProps的方法来获取默认组件属性的
React.Component在创建组件时配置这两个对应信息时，他们是作为组件类的属性，不是组件实例的属性，也就是所谓的类的静态属性来配置的
```
const TodoItem = React.createClass({
    propTypes: { // as an object
        name: React.PropTypes.string
    },
    getDefaultProps(){   // return a object
        return {
            name: ''    
        }
    }
    render(){
        return <div></div>
    }
})

class TodoItem extends React.Component {
    static propTypes = {//类的静态属性
        name: React.PropTypes.string
    };

    static defaultProps = {//类的静态属性
        name: ''
    };

    ...
}
```
* 组件初始状态state的配置不同
React.createClass创建的组件，其状态state是通过getInitialState方法来配置组件相关的状态；
React.Component创建的组件，其状态state是在constructor中像初始化组件属性一样声明的。
* Mixins的支持不同
React.createClass在创建组件时可以使用mixins属性，以数组的形式来混合类的集合。
```
var SomeMixin = {  
  doSomething() {

  }
};
const Contacts = React.createClass({  
  mixins: [SomeMixin],
  handleClick() {
    this.doSomething(); // use mixin
  },
  render() {
    return (
      <div onClick={this.handleClick}></div>
    );
  }
});
```
React.Component这种形式并不支持Mixins，至今React团队还没有给出一个该形式下的官方解决方案；但是React开发者社区提供一个全新的方式来取代Mixins,那就是Higher-Order Components(HOC),高阶组件
#### props和state
peops 的主要作用是让使用该组件的父组件可以传入参数来配置该组件。它是外部传进来的配置参数，组件内部无法控制也无法修改。除非外部组件主动传入新的 props，否则组件的 props 永远保持不变。
state 的主要作用是用于组件保存、控制、修改自己的可变状态。state 在组件内部初始化，可以被组件自身修改，而外部不能访问也不能修改。你可以认为 state 是一个局部的、只能被组件自身控制的数据源。state 中状态可以通过 this.setState 方法进行更新，setState 会导致组件的重新渲染。

##### props的使用
######  不可修改父属性
通过对组件设置defaultProps属性可以在父组件未定义属性值时使用默认属性，但对于父组件已经定义了属性值，该组件定义的同名属性也要被覆盖。
```
Test.defaultProps = {
    title:"这是一个测试"
}
```
###### props值传递
props可以在父子组件之间传递值，且是单向的，即只能从父组件传递到子组件

###### prop验证
```
Test.propTypes = {
    title: React.PropTypes.string,
}
```
以上定义了Test组件的title属性必须是string类型
###### children
this.props.children可以处理任何数据（组件、字符串、函数等等）React.children有5个方法：
* React.Children.map()  React.Children.map()有些类似Array.prototype.map()。
* React.Children.forEach()  跟React.Children.map()一样，区别在于无返回
* React.Children.count()  React.Children.count()用来计数，返回child个数。
* React.Children.only()  验证children里只有唯一的孩子并返回他。否则这个方法抛出一个错误。
* React.Children.toArray()  将children转换成Array，对children排序时需要使用
通常与React.cloneElement()结合使用来操作this.props.children。
```
function Test(props) {
const {children} = props;
    return (
         <ul>
         {
              children.map(function(child, index){
                   return <SpanItem key="index" text={child}></SpanItem>
            })
         }
         </ul>
     )
}

function SpanItem(porps) {
const {key, text} = porps;
    return (
         <li>{text}</li>
     )
}
```

###### 利用es6进行结构赋值
```
function Test(props) {
    const {...user} = props;
    return (
         <div>
              <span>{user.user.name}</span>
              <span>{user.user.age}</span>
         </div>
    )
}
```

##### state的使用
判断是否可以作为state的条件
* 变量如果是通过props从父组件获取，就不是一个状态
* 如果这个变量可以通过其他状态的state或者属性props通过数据处理得到，就不是一个状态
* 如果变量没有在render中使用到，就不是一个状态
* 变量在整个生命周期中都保持不变时，就不是一个状态

**state是可变的，是组件内部维护的一组用于反映组件UI变化的状态集合**
**props对于使用它的组件来说，是只读的，想要修改props，只能通过组件的父组件内修改**

通过setState可以修改state的值
state的值在修改了之后并不会立即被修改，而是也有一个类似的队列，setState通过一个队列机制实现state的更新。当执行setState时，会把需要更新的state合并后放入状态队列，而不会立刻更新this.state，利用这个队列机制可以高效的批量的更新state。

#### react的生sd命周期
即组件在某个时间点会自动调用的函数
* Initialization：初始化阶段
* Mounting：真实DOM已插入
* Updation：重新渲染
* Unmounting：已移除真实DOM

##### Mounting 
###### componentWillMounting()
组件即将挂载到页面时自动执行，在render()之前执行，只会执行一次
###### componentDidMounting()
组件在被挂在到页面之后自动执行，在render()之后执行，数据在第一次查询将在这里执行（网页初始化数据），只会执行一次

##### Updation
###### shouldComponentUpdate()
组件在被更新之前自定执行，在render更新之前，可以理解为需要更新render吗，如果需要返回true，如果不需要返回false
###### componentWillUpdate()
组件更新之前自动执行，在shouldComponentUpdate()函数之后，如果shouldComponentUpdate()函数返回true才会执行该函数
###### componentDidUpdate()
组件重新被挂载到页面之后自动执行，上面几个周期函数在props和state更新之后都会执行
###### componentWillReceiveProps()
当一个子组件从父组件接受参数（props更改），一般会出现在子组件
注：
　　只要父组件的render函数被跟新时，子组件这个生命周期函数会被执行。
　　如果这个组件第一次存在父组件中，不会执行。
　　如何这个组件之前已经存在于父组件中，才会执行。

##### Unmounting
###### componentWillUnmount()
当这个组件即将被删除时自动执行