# JavaScript设计模式与实践--工厂模式

## 1 什么是工厂模式?

工厂模式是用来创建对象的一种最常用的设计模式。我们不暴露创建对象的具体逻辑，而是将将逻辑封装在一个函数中，那么这个函数就可以被视为一个工厂。工厂模式根据抽象程度的不同可以分为：`简单工厂`，`工厂方法`和`抽象工厂`。

### 1.1 简单工厂模式

`简单工厂模式`又叫`静态工厂模式`，由一个工厂对象决定创建某一种产品对象类的实例。主要用来创建同一类对象。

在实际的项目中，我们常常需要根据用户的权限来渲染不同的页面，高级权限的用户所拥有的页面有些是无法被低级权限的用户所查看。所以我们可以在不同权限等级用户的构造函数中，保存该用户能够看到的页面。在根据权限实例化用户。使用ES6重写简单工厂模式时，我们不再使用构造函数创建对象，而是使用class的新语法，并使用static关键字将简单工厂封装到`User`类的静态方法中.代码如下：

```js
//User类
class User {
  //构造器
  constructor(opt) {
    this.name = opt.name;
    this.viewPage = opt.viewPage;
  }

  //静态方法
  static getInstance(role) {
    switch (role) {
      case 'superAdmin':
        return new User({ name: '超级管理员', viewPage: ['首页', '通讯录', '发现页', '应用数据', '权限管理'] });
        break;
      case 'admin':
        return new User({ name: '管理员', viewPage: ['首页', '通讯录', '发现页', '应用数据'] });
        break;
      case 'user':
        return new User({ name: '普通用户', viewPage: ['首页', '通讯录', '发现页'] });
        break;
      default:
        throw new Error('参数错误, 可选参数:superAdmin、admin、user')
    }
  }
}

//调用
let superAdmin = User.getInstance('superAdmin');
let admin = User.getInstance('admin');
let normalUser = User.getInstance('user');

```

`User`就是一个简单工厂，在该函数中有3个实例中分别对应不同的权限的用户。当我们调用工厂函数时，只需要传递`superAdmin`, `admin`, `user`这三个可选参数中的一个获取对应的实例对象。

简单工厂的优点在于，你只需要一个正确的参数，就可以获取到你所需要的对象，而无需知道其创建的具体细节。但是在函数内包含了所有对象的创建逻辑（构造函数）和判断逻辑的代码，每增加新的构造函数还需要修改判断逻辑代码。当我们的对象不是上面的3个而是30个或更多时，这个函数会成为一个庞大的超级函数，便得难以维护。**所以，简单工厂只能作用于创建的对象数量较少，对象的创建逻辑不复杂时使用**。

### 1.2 工厂方法模式

工厂方法模式的本意是将**实际创建对象的工作推迟到子类**中，这样核心类就变成了抽象类。但是在JavaScript中很难像传统面向对象那样去实现创建抽象类。所以在JavaScript中我们只需要参考它的核心思想即可。我们可以将工厂方法看作是一个实例化对象的工厂类。虽然ES6也没有实现`abstract`，但是我们可以使用`new.target`来模拟出抽象类。`new.target`指向直接被`new`执行的构造函数，我们对`new.target`进行判断，如果指向了该类则抛出错误来使得该类成为抽象类。

在简单工厂模式中，我们每添加一个构造函数需要修改两处代码。现在我们使用工厂方法模式改造上面的代码，刚才提到，工厂方法我们只把它看作是一个实例化对象的工厂，它只做实例化对象这一件事情！

```js
class User {
  constructor(name = '', viewPage = []) {
    if(new.target === User) {
      throw new Error('抽象类不能实例化!');
    }
    this.name = name;
    this.viewPage = viewPage;
  }
}

class UserFactory extends User {
  constructor(name, viewPage) {
    super(name, viewPage)
  }
  create(role) {
    switch (role) {
      case 'superAdmin': 
        return new UserFactory( '超级管理员', ['首页', '通讯录', '发现页', '应用数据', '权限管理'] );
        break;
      case 'admin':
        return new UserFactory( '普通用户', ['首页', '通讯录', '发现页'] );
        break;
      case 'user':
        return new UserFactory( '普通用户', ['首页', '通讯录', '发现页'] );
        break;
      default:
        throw new Error('参数错误, 可选参数:superAdmin、admin、user')
    }
  }
}

let userFactory = new UserFactory();
let superAdmin = userFactory.create('superAdmin');
let admin = userFactory.create('admin');
let user = userFactory.create('user');

```

### 1.3 抽象工厂模式

上面介绍了简单工厂模式和工厂方法模式都是直接生成实例，但是抽象工厂模式不同，抽象工厂模式并不直接生成实例， 而是用于`对产品类簇`的创建。

上面例子中的`superAdmin`，`admin`，`user`三种用户角色，其中user可能是使用不同的社交媒体账户进行注册的，例如：`wechat，qq，weibo`。那么这三类社交媒体账户就是对应的类簇。在抽象工厂中，类簇一般用父类定义，并在父类中定义一些抽象方法，再通过抽象工厂让子类继承父类。所以，**抽象工厂其实是实现子类继承父类的方法**。

上面提到的抽象方法是指声明但不能使用的方法。在其他传统面向对象的语言中常用`abstract`进行声明，但是在JavaScript中，`abstract`是属于保留字，但是我们可以通过在类的方法中抛出错误来模拟抽象类。

```js
function getAbstractUserFactory(type) {
  switch (type) {
    case 'wechat':
      return UserOfWechat;
      break;
    case 'qq':
      return UserOfQq;
      break;
    case 'weibo':
      return UserOfWeibo;
      break;
    default:
      throw new Error('参数错误, 可选参数:superAdmin、admin、user')
  }
}

let WechatUserClass = getAbstractUserFactory('wechat');
let QqUserClass = getAbstractUserFactory('qq');
let WeiboUserClass = getAbstractUserFactory('weibo');

let wechatUser = new WechatUserClass('微信小李');
let qqUser = new QqUserClass('QQ小李');
let weiboUser = new WeiboUserClass('微博小李');


```

## 2 工厂模式的项目实战应用

在实际的前端业务中，最常用的简单工厂模式。如果不是超大型的项目，是很难有机会使用到工厂方法模式和抽象工厂方法模式的。下面我介绍在Vue项目中实际使用到的`简单工厂模式`的应用。

在普通的`vue + vue-router`的项目中，我们通常将所有的路由写入到`router/index.js`这个文件中。下面的代码我相信vue的开发者会非常熟悉，总共有5个页面的路由：

```js
// index.js

import Vue from 'vue'
import Router from 'vue-router'
import Login from '../components/Login.vue'
import SuperAdmin from '../components/SuperAdmin.vue'
import NormalAdmin from '../components/Admin.vue'
import User from '../components/User.vue'
import NotFound404 from '../components/404.vue'

Vue.use(Router)

export default new Router({
  routes: [
    //重定向到登录页
    {
      path: '/',
      redirect: '/login'
    },
    //登陆页
    {
      path: '/login',
      name: 'Login',
      component: Login
    },
    //超级管理员页面
    {
      path: '/super-admin',
      name: 'SuperAdmin',
      component: SuperAdmin
    },
    //普通管理员页面
    {
      path: '/normal-admin',
      name: 'NormalAdmin',
      component: NormalAdmin
    },
    //普通用户页面
    {
      path: '/user',
      name: 'User',
      component: User
    },
    //404页面
    {
      path: '*',
      name: 'NotFound404',
      component: NotFound404
    }
  ]
})

```

当涉及权限管理页面的时候，通常需要在用户登陆根据权限开放固定的访问页面并进行相应权限的页面跳转。但是如果我们还是按照老办法将所有的路由写入到`router/index.js`这个文件中，那么低权限的用户如果知道高权限路由时，可以通过在浏览器上输入url跳转到高权限的页面。所以我们必须在登陆的时候根据权限使用`vue-router`提供的`addRoutes`方法给予用户相对应的路由权限。这个时候就可以使用简单工厂方法来改造上面的代码。

在`router/index.js`文件中，我们只提供`/login`这一个路由页面。

```js
//index.js

import Vue from 'vue'
import Router from 'vue-router'
import Login from '../components/Login.vue'

Vue.use(Router)

export default new Router({
  routes: [
    //重定向到登录页
    {
      path: '/',
      redirect: '/login'
    },
    //登陆页
    {
      path: '/login',
      name: 'Login',
      component: Login
    }
  ]
})

```

我们在`router/`文件夹下新建一个`routerFactory.js`文件，导出`routerFactory`简单工厂函数，用于根据用户权限提供路由权限，代码如下

```js
//routerFactory.js

import SuperAdmin from '../components/SuperAdmin.vue'
import NormalAdmin from '../components/Admin.vue'
import User from '../components/User.vue'
import NotFound404 from '../components/404.vue'

let AllRoute = [
  //超级管理员页面
  {
    path: '/super-admin',
    name: 'SuperAdmin',
    component: SuperAdmin
  },
  //普通管理员页面
  {
    path: '/normal-admin',
    name: 'NormalAdmin',
    component: NormalAdmin
  },
  //普通用户页面
  {
    path: '/user',
    name: 'User',
    component: User
  },
  //404页面
  {
    path: '*',
    name: 'NotFound404',
    component: NotFound404
  }
]

let routerFactory = (role) => {
  switch (role) {
    case 'superAdmin':
      return {
        name: 'SuperAdmin',
        route: AllRoute
      };
      break;
    case 'normalAdmin':
      return {
        name: 'NormalAdmin',
        route: AllRoute.splice(1)
      }
      break;
    case 'user':
      return {
        name: 'User',
        route:  AllRoute.splice(2)
      }
      break;
    default: 
      throw new Error('参数错误! 可选参数: superAdmin, normalAdmin, user')
  }
}

export { routerFactory }

```

在登陆页导入该方法，请求登陆接口后根据权限添加路由:

```js
//Login.vue

import {routerFactory} from '../router/routerFactory.js'
export default {
  //... 
  methods: {
    userLogin() {
      //请求登陆接口, 获取用户权限, 根据权限调用this.getRoute方法
      //..
    },
    
    getRoute(role) {
      //根据权限调用routerFactory方法
      let routerObj = routerFactory(role);
      
      //给vue-router添加该权限所拥有的路由页面
      this.$router.addRoutes(routerObj.route);
      
      //跳转到相应页面
      this.$router.push({name: routerObj.name})
    }
  }
};

```

在实际项目中，因为使用`this.$router.addRoutes`方法添加的路由刷新后不能保存，所以会导致路由无法访问。通常的做法是本地加密保存用户信息，在刷新后获取本地权限并解密，根据权限重新添加路由。这里因为和工厂模式没有太大的关系就不再赘述。

## 3 总结

上面说到的三种工厂模式和单例模式一样，都是属于创建型的设计模式。简单工厂模式又叫静态工厂方法，用来创建某一种产品对象的实例，用来创建单一对象；工厂方法模式是将创建实例推迟到子类中进行；抽象工厂模式是对类的工厂抽象用来创建产品类簇，不负责创建某一类产品的实例。在实际的业务中，需要根据实际的业务复杂度来选择合适的模式。对于非大型的前端应用来说，灵活使用简单工厂其实就能解决大部分问题。

### 3.1. 什么时候会用工厂模式？

将new操作简单封装，遇到new的时候就应该考虑是否用工厂模式；

### 3.2. 工厂模式的好处？

举个例子：

- 你去购买汉堡，直接点餐、取餐、不会自己亲自做；（买者不关注汉堡是怎么做的）

- 商店要`封装`做汉堡的工作，做好直接给买者；（商家也不会告诉你是怎么做的，也不会傻到给你一片面包，一些奶油，一些生菜让你自己做）

外部不许关心内部构造器是怎么生成的，只需调用一个工厂方法生成一个实例即可；

构造函数和创建者分离，符合开放封闭原则

### 3.3 实际的一些例子

- jQuery的`$(selector)` jQuery中`$('div')`和`new $('div')`哪个好用，很显然直接`$()`最方便 ,这是因为`$()`已经是一个工厂方法了;

```
class jQuery {
    constructor(selector) {
        super(selector)
    }
    //  ....
}

window.$ = function(selector) {
    return new jQuery(selector)
}


```

- React的`createElement()`

`React.createElement()`方法就是一个工厂方法

![](https://user-gold-cdn.xitu.io/2018/8/9/1651a5e388f90113?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- Vue的异步组件

通过`promise`的方式`resolve`出来一个组件

![](https://user-gold-cdn.xitu.io/2018/8/9/1651a5eac5fc6d96?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
