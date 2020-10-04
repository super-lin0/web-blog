## connect()

### ``react-redux``中的``connect``

- 基本作用

  ``connect``是一个桥梁，用来连接容器组件和展示组件。简单的来说：``connect``的核心是将开发者定义的组件，包装转换生成容器组件。所生成的容器组件能使用 ``Redux Store``中的哪些数据，全由``connect``的参数来决定。

- 基本用法

  ```javascript
  const WrappedComponent = connect([mapStateToProps],[mapDispatchToProps],[mergeProps],[options])(presentionalComponent);
  
  export default WrappedComponent;
  ```

  **mapStateToProps**

  - 是一个函数

  - 作用是给返回的组件``WrappedComponent``注入``props``，这个``props``来自``Redux store``中的状态。

  - 这个函数一定要返回一个JavaScript原生对象

  - 它完成从``store``中选取数据并通过``props``传递给将要创建的容器组件``WrappedComponent``

    ```javascript
    function mapStateToProps(state, [ownProps]) {
    	return {}
    }
    ```

  **mapDispatchToProps**

  - 作用是将``dispatch``作为``props``传递给``WrappedComponent``组件。

  - 可以是一个函数，也可以是一个对象。

  - 如果传递的是一个对象，那么键值应该是一个函数，用来描述action的生成。也就是说，每个定义在该对象中的函数都将被当作``Redux action creator``(构造器)，其中所定义的方法名将作为属性名被合并到组件的``props``中。

  - 默认情况下，``dispatch``方法会注入``WrappedComponent``组件的``props``上

    ```javascript
    const mapDispatchToProps = {
      minus: () => ({ type: "counter/minus" }),
      add: () => ({ type: "counter/add" }),
      sum: params => ({type: "counter/sum", payload: params})
    };
    ```

  - 如果传递的是一个函数，那么这个函数将接受``dispatch``方法以及容器组件的``props``作为参数，最终也返回一个对象。

    ```javascript
    const nmapDispatchToProps = (dispatch, ownProps) => return {
    	add: () => dispatch({
    		type: "add",
    		data: ownProps.data
    	})
    }
    ```

- 一个完整的例子

  ```javascript
  import React, { Component } from "react";
  import { connect } from "react-redux";
  
  class ReactReduxPage extends Component {
    render() {
      console.log("props:", this.props);
      const { counter, add, minus, dispatch } = this.props;
  
      return (
        <div>
          <p>{counter}</p>
          <button onClick={add}>+</button>
          <button onClick={minus}>-</button>
  				<button onClick={() => dispatch({type: "sum", payload: {}})}>sum</button>
        </div>
      );
    }
  }
  
  function mapStateToProps(state) {
    return { counter: state };
  }
  const mapDispatchToProps = {
    minus: () => ({ type: "minus" }),
    add: () => ({ type: "add" })
  };
  
  export default connect(mapStateToProps, mapDispatchToProps)(ReactReduxPage);
  
  ```

  

- 总结

  ``mapStateToProps``和``mapDispatch``定义了展示组件需要用到的``store``内容。其中``mapStateToProps``负责输入逻辑，就是将数据映射到展示组件的参数(props)上；后者负责输出逻辑，即将用户对展示组件的操作映射成``action``

  

### 系统管理的``connect``

系统管理中我们在``react-redux``中的``connect``基础上做了增强。除了上述功能之外，还包含了以下功能：

- 获取静态参数（某些下拉框数据可以在这里获取）
- 简化了调用接口时页面``loading``的写法

```javascript
import connect from "@/components/connect";

@connect(({ loginLog }) => ({ logs: loginLog }), {
  staticKey: ["tfm_sys_login_log|loginType"],
  loadings: ["loginLog/getLogs", "loginLog/exportLogs"],
})
```

重点区别在第二个参数上：

- ``staticKey``是一个数组，这个数组的项表示要查询的静态参数，拿上面案例来说。``"tfm_sys_login_log|loginType"``代表在``connect``的时候查询了``tfm_sts_bussness_parameter``表的``param_module``值为``tfm_sys_login_log``且``param_code``为``loginType``的数据，返回一个数组，注入到组件当中。获取方式：

```javascript
import { trimStaticData } from "@/utils";

const { publicData: { staticData }} = this.props;
const staticloginType = trimStaticData(staticData, "tfm_sys_login_log|loginType");
```

- ``loadings``也是一个数组，数组的值为组件中``dispatch``的``type``类型，例如上面的例子，就表示，在``dispatch``的``type``值为``loginLog/getLogs``和``loginLog/exportLogs``的时候页面出现全局的``loading``效果。

