# vue开发规范以及部分说明文档

### 代码规范

1.默认使用 vue 官方推荐的最佳实践配置文档,具体配置位置在项目目录下 eslintrc.js 该配置主要用作详细的 vue 的语法呈现约束,包括部分 ES5-ES7 的代码编写约束.请查阅[官方风格推荐及约束](https://cn.vuejs.org/v2/style-guide/index.html).
2.语法检查:eslintrc.js 中的配置能约束常见的编写语法错误,及引用错误,因此推荐直接在编辑器中用 eslint 的自动修复功能(如用 vscode 则需要安装 eslint 插件,vetur 插件)
---

### 代码风格
 补充:
 * 变量名称采用驼峰式,如 : helloWorld
 * css类名采用短横杆式,如: .tab-title
 * 函数或属性方法尽量保持最小原则,每个函数只实现单一功能
 * 按工程目录做好代码的细分
 * 如编写新的组件,请把该组件以目录的形式呈现,并配备readme.
 * 一定要写备注(变量命名,函数功能简介,流程说明) !!!
 * [带demo的补充文档说明.](https://github.com/pablohpsilva/vuejs-component-style-guide/blob/master/README-CN.md)

---

### 工程目录约定

1.根目录

| .vscode                      | public         | src          |
|------------------------------|----------------|--------------|
| 编辑器配置文件(如使用vscode) | 渲染模板及logo | 代码编写区域 |

2.src文件夹


| assets | components | router | store | views | doc | api | styles |
| :----: | :----: | :----: | :----: | :----: | :----: | :----: | :----: |
| 静态文件 | 公共组件 | 路由 | vuex | 路由页面 | 文档 | 接口请求 | 公共样式文件 |

---

#### `vue`开发标准

* `.vue name` 字段采用大写开头 (PascalCase)，比如：`name: "EnLoading"`,`name: "EnAddress"`

参考：[vue风格指南](https://cn.vuejs.org/v2/style-guide/#%E7%BB%84%E4%BB%B6%E6%96%87%E4%BB%B6%E5%BC%BA%E7%83%88%E6%8E%A8%E8%8D%90)

```javascript
// Good
export default {
  name: "MyLoading", 
}

// bad
export default {
  name: "myLoading"
}
export default {
  name: "my-loading"
}
```

* `.vue props | methods | computed | data` 字段，采用驼峰命令(camelCase)

```javascript
// Good
export default {
  props: {
    fullViewPage: {
      type: Object,
      // 当type:Object的时候 default: () => {}
      // 当type:Array default: () => []
      default: () => {}
    } 
  },
  data() {
    return {
      pageLayout: {
        showLogin: true
      }
    }
  },
  computed: {
    showLogin() {
      return this.pageLayout
    }
  },
  watch: {
    pageLayout() {},
    "pageLayout.showLogin": { // 当监听的是对象里面一级，采用这种写法
      handler() {}
    } 
  },
  methods: {
    showForgetView() {}
  } 

}
//bad
export default {
  props: {
    "full-View-Page": {
      type: Object,
      default: {} // 直接写{} 是错误写法，需要通过方法返回
    } 
  }
  // ...
}
```

* `v-on | $emit` 事件监听和触发，采用横线连接(kebab-case)命名：比如：`left-click`

参考：[vue风格指南](https://cn.vuejs.org/v2/guide/components-custom-events.html)

```javascript
// Good
export default {
  name: "MyCard",
  methods: {
    onLeftClick() {
      this.$emit("left-click");
    }
  }
}
// <my-card @left-click="onLeftClick">

```

* 异步写法 所有异步写法采用`asyn/await`，尽可能消除回调

```javascript
// Good
export default {
  methods: {
    async test() {
      try{
        // 单个
        const result = await fn();
        // 多个顺序执行
        const result = await fn();
        const result = await fn();
        const result = await fn();
        // 多个不需要顺序执行
        const result1 = fn();
        const result2 = fn();
        const result3 = fn();
        const allResult = Promise.all([result1, result2, result3]) 
      } catch(e) {
        // 处理异常
      } finally {
        // 不管是出错，还是正确，都要处理的逻辑
      }   
    }
  }
}

// bad
export default {
  methods: {
    test() {
        // 单个
        fn().then();
        // 多个顺序执行
        fn().then(() => {
          fn().then(() => {
            fn();
          });
        });
        // 多个不需要顺序执行
        const result1 = fn();
        const result2 = fn();
        const result3 = fn();
        Promise.all([result1, result2, result3]).then()
    }
  }
}
```

* 访问深度嵌套的属性

参考：[@babel/plugin-proposal-optional-chaining](https://babeljs.io/docs/en/next/babel-plugin-proposal-optional-chaining)

```javascript
const obj = {
  foo: {
    bar: {
      baz: 42,
    },
  },
};

// Good
const baz = obj?.foo?.bar?.baz // 42

const safe = obj?.qux?.baz; // undefined

const ohter = obj?.qux?.ohter || "" // ""

// bad
if (obj && obj.foo && obj.foo.bar) {
  const baz = obj.foo.bar.baz
}

```

---

#### 平台结构详细说明

#### 1. apis 接口请求  
* `apis/index.js` 处理apis的导出问题(需不需要自动挂载)  
* `apis/http.js` 请求通用逻辑封装
* `apis/BaseService.js` 接口父类
* apis下面按模块划分目录，下面举例  
    * `apis/user/index.js` 处理这个模块的接口导出  
    * `apis/user/[name]Service.js` 这个模块下面的Service模块  
**实例**  

```javascript
// apis/http.js
// 从en-js获取Http对象，web端就是axios，这个是方便以后多平台扩展，保持统一的api调用方式
// 同时会处理共性的拦截逻辑，比如状态等之类的处理
import axios from 'axios'

// 创建一个http配置，尽量不使用axios的全局配置
const http = axios.create({
  ...config
});

// 业务拦截请求
http.interceptors.request.use(onFulfiled, onRejected);
// ...

// 业务响应拦截
http.interceptors.response.use(onFulfiled, onRejected);
// ...

export default http;
```

```javascript
import Qs from 'qs'
import http from './http'

// apis/BaseService.js
class BaseService {
  // 处理get请求的参数序列化配置
  serializerConfig = {
    arrayFormat: "indices",
    allowDots: true,
  }
  
  // 一个模块的公共路径
  module = ""
  /**
   * 构造函数
   * @param module 模块名称，比如：用户模块的增删改查 增加、修改：/user/add,查询/user/list,，删除/user/del。那么module=“user”
   * @param customHttp 自定义的http，可以扩展多个不同http。默认使用http.js种的http对象
   */
  constructor(module, customHttp) {
    this.module = module
    this.http = customHttp || http
  }

  /**
   * get参数序列化配置
   * @param params
   * @param config
   * @returns {string}
   */
  serializer(params, config = {}) {
    config = { ...this.serializerConfig, ...config }
    return Qs.stringify(params, config)
  }

  /**
   * Get请求封装，用于带参数的Get请求
   * @param url 请求url
   * @param params 请求参数
   * @config 序列化配置，可以替换serializer配置，一般情况不用使用
   * @returns {*}
   */
  query(url, params, config) {
    const { http } = this
    return http.get(url, {
      params,
      paramsSerializer: params => this.serializer(params, config),
    })
  }
  
  // 如果后台路径命名有一定标准，可以扩展标准化命令请求，比如查询列表，查询单个资源，增加/修改单个资源，删除单个资源
}
```

```javascript
// apis/common/index.js
import UploadService from "./UploadService";
import PublicService from "./PublicService";

export const uploadService = new UploadService()

export const publicService = new PublicService()

```

```javascript
// BaseService方案1的使用
// 其他模块集成实现
// /apis/exapmle/ExampleService.js 简单实例
class ExampleService extends BaseService {
  // 新增修改通同扩展可以使用
  pk = "id"
  /*
   * 构造函数
   * @http http请求，不传则使用父类中的http对象
   */
  constructor(http) {
    super("example", http);
  }
  // 一个简单请求是咧
  queryList(params) {
    const { module } = this;
    return this.query(`/${module}/list`, params);
  }
}

export default ExampleService;

// 类写法优势，缓存控制
// apis/common/PublickService.js 举例了枚举缓存使用

class PublicService extends BaseService {
  pk = "id"

  cache = null

  constructor(http) {
    super("public", http)
  }


  async enum() {
    if (this.cache) {
      return this.cache
    }
    const { http, module } = this
    const res = await http.get(`/${module}/enum`)
    if (res.code === 200) {
      const data = res.data || {}
      Object.values(data).forEach((v) => {
        v.forEach((t) => {
          t.value = t.id
          delete t.id
        })
      })
      this.cache = data
    } else {
      this.cache = {}
    }

    return this.cache
  }

  async enumByKey(key) {
    if (!this.cache) {
      await this.enum()
    }
    return this.cache[key] || []
  }
}

```

> **开发标准**

1、每个模块下面统一都由`index.js`导出类的实例化

2、模块采用`class`方式，文件名由[模块名] + [Service] 组成，命名方式采用大写开头(PascalCase)，比如：`LoginService.js`,`ExampleService.js`

3、类名称同文件名，比如：`LoginService`,`ExampleService`

---

#### 2. assets 资源目录
   *  images 图片目录
   *  css css目录
   *  js js目录  

---

#### 3. components
   *  目录存放平台架构通用组件  
   
> **开发标准**
>
1、组件文件命名方式都采用单词大写开头 (PascalCase)或者横线连接 (kebab-case)命名，比如：`my-loading`, `MyLoading`等

单文件组件的文件名应该要么始终是单词大写开头 (PascalCase)，要么始终是横线连接 (kebab-case)。

参考：[vue风格指南](https://cn.vuejs.org/v2/style-guide/#%E7%BB%84%E4%BB%B6%E6%96%87%E4%BB%B6%E5%BC%BA%E7%83%88%E6%8E%A8%E8%8D%90)

---

#### 4. mock 模拟拦截请求使用
  * `index.js` 所有mock模块导出入口
  * `modules` 模块文件夹，各个模块的数据放在这个文件夹下面  
**实例**

```javascript
// mock/index.js
import "./modules/example";
```

```javascript
// mock/modules/example.js
import Mock from "mockjs";
import Qs from "qs"; // 解析请求body

const login = (opt) => {
  const body = JSON.parse(opt.body);
  const { data } = body;
  if (data.userName === "admin" && data.password === "e10adc3949ba59abbe56e057f20f883e") {
    return {
      code: 200,
      msg: "ok",
      data: {
        mobilePhone: "18811112222",
        nickName: "管理员",
        userId: 10000,
        userName: "admin"
      }
    };
  }
  return {
    code: 400,
    msg: "账号密码错误"
  };
};
const userList = [];
for (let i = 0; i < 10; i++) {
  const user = {
    userId: i + 1,
    userName: `user${i + 1}`,
    state: 1
  };
  if (i > 6) {
    user.state = 0;
  }
  userList.push(user);
}
const userList = [];
for (let i = 0; i < 10; i++) {
  const user = {
    userId: i + 1,
    userName: `user${i + 1}`,
    state: 1
  };
  if (i > 6) {
    user.state = 0;
  }
  userList.push(user);
}
const getUserList = () => {
  const r = userList;
  return {
    msgType: "N",
    resCode: "AU1001",
    rspMsg: "123",
    rspData: {
      pagination: {
        total: userList.length
      },
      list: r
    }
  };
};
const getUser = (opt) => {
  const { url } = opt;
  const userId = url.replace(/\/example\//, "");
  const index = userList.findIndex((v) => v.userId === Number(userId));
  if (index > -1) {
    return {
      code: 200,
      msg: "ok",
      data: userList[index]
    };
  }
  return {
    code: 400,
    msg: "未查询到相关数据",
    data: {}
  };
};
const addUser = (opt) => {
  const body = Qs.parse(opt.body, {
    arrayFormat: "indices",
    allowDots: true
  });
  const data = body;
  data.state = data.state ? Number(data.state) : 1;
  // const { data } = body;
  const userId = Number(userList[userList.length - 1].userId) + 1;
  data.userId = userId;
  userList.push({
    ...data
  });
  return {
    msgType: "N",
    rspMsg: "添加成功",
    rspData: {
      ...data
    }
  };
};
const updateUser = (opt) => {
  const body = Qs.parse(opt.body, {
    arrayFormat: "indices",
    allowDots: true
  });
  const data = body;
  data.userId = data.userId ? Number(data.userId) : "";
  data.state = data.state ? Number(data.state) : 1;
  const { userId } = data;
  const index = userList.findIndex((v) => v.userId === userId);
  userList[index] = {
    ...data
  };
  return {
    msgType: "N",
    resCode: "AU1001",
    rspMsg: "删除成功",
    data: {
      ...data
    }
  };
};
const delUser = (opt) => {
  const body = JSON.parse(opt.body);
  const userId = body.userId;
  const index = userList.findIndex((v) => v.userId.toString() === userId);
  const user = userList[index];
  userList.splice(index, 1);
  return {
    msgType: "N",
    resCode: "AU1001",
    rspMsg: "删除成功",
    rspData: {
      ...user
    }
  };
};
Mock.mock(/auth\/login/, /post|get/i, login);
Mock.mock(/example\/list/, "get", getUserList);
Mock.mock(/example\/update/, "post", updateUser);
Mock.mock(/example\/add/, "post", addUser);
Mock.mock(/example\/delete/, "post", delUser);
Mock.mock(/example/, "get", getUser);
```

>**开发标准**  

1、路径采用正则表达式，不要使用字符串

```javascript
// Good
Mock.mock(/example\/list/, "get", getUserList);

// bad
Mock.mock("example/list", "get", getUserList);
```

---

#### 5. router
   *  index.js 对外整合的router
   *  permission.js 路由权限
   *  util.js 路由工具
   *  [fileName] 文件夹 分模块规划路由
        * 每个模块下面必须由`index.js`导出这个模块的所有路由的数组数据
   *  `router/index.js` 只会对当前路径下的文件夹中`index.js`加载。只加载一级，在内部的子目录，不再做任何处理。
   *  路由定义标准说明
        *  文件组织，一个模块的路由维护在一个文件里面
        *  合理的组织`webpackChunkName`，可以以模块未单位组织  
**实例**

```
  {
    path: "/hotel/query",
    name: "hotelquery",
    component: () => import(/* webpackChunkName: "business-travel" */ "@/views/businessTravel/hotel/query"),
    meta: {
      title: "酒店摘选", // 标题
      fullPage: false, // 是否使用全屏布局
      keepAlive: true // 是否需要keepAlive, 多级路由，需要路由组件支持
    }
  }
```

>**开发标准**

1、异步组件组要定义`webpackChunkName` 

2、`webpackChunkName`命名规则：根据模块，以及模块大小来命名。  

比如：  

2.1、 同一个模块下的路由名字统一  

2.2、 如果同一个模块下的路由非常大，统一命名的话js包会大，那么可以根据合理的方式进行拆分命名

```
// Good
  {
    path: "/hotel/query",
    name: "hotelquery",
    component: () => import(/* webpackChunkName: "business-travel" */ "@/views/businessTravel/hotel/query"),
    meta: {
      title: "酒店摘选", // 标题
      fullPage: false, // 是否使用全屏布局
      keepAlive: true // 是否需要keepAlive, 多级路由，需要路由组件支持
    }
  }

// bad
  {
    path: "/hotel/query",
    name: "hotelquery",
    component: () => import("@/views/businessTravel/hotel/query"),
    meta: {
      title: "酒店摘选", // 标题
      fullPage: false, // 是否使用全屏布局
      keepAlive: true // 是否需要keepAlive, 多级路由，需要路由组件支持
    }
  }
```

---

#### 6. store
   *  index.js 全局状态管理，公共的状态可以在这个中管理，比如登录之后的信息，权限等
   *  persistedstate.js 配置需要持久化的路径，支持`_.get(val, path) // lodash 方法`
   *  modules 下面分模块划分，所有模块状态的vuex全部放到modules下面  
**实例**

```javascript
// index.js 首次创建的模块vuex，需要在index.js导入
import _ from "lodash";
import Vue from "vue";
import Vuex from "vuex";
import createPersistedState from "vuex-persistedstate";
import persistedStateList from "./persistedstate"; // 需要持久化的key
import main from "./modules/main"; // 导入的模块，需要引入

Vue.use(Vuex);
const debug = process.env.NODE_ENV !== "production";

export default new Vuex.Store({
  state: {},
  getters: {
  },
  actions: {},
  mutations: {},
  modules: {
    main // 导入的模块，需要导入
  },
  plugins: [
    createPersistedState({
      key: "MY-MANAGE",
      storage: window.localStorage,
      reducer(val) {
        const r = {};
        persistedStateList.reduce((o, path) => {
          const v = _.get(val, path);
          if (!_.isUndefined(v)) {
            _.set(r, path, v);
          }
          return r;
        }, r);
        return r;
      }
    })
  ],
  strict: debug
});

```

```javascript
// store/modules/main 可以是单独文件，也可以自己分为：index.js,action.js,mutation.js,state.js,getter.js
// index.js 
import state from "./state";
import mutations from "./mutations";
import actions from "./action";
import getters from "./getters";

export default {
  namespaced: true, // 开启模块名称
  state,
  getters,
  actions,
  mutations
};
// 其他js可以参考现有实例
```

```javascript
// /store/persistedstate.js
export default [
  "isLogin",
  "userInfo",
  "userInfo.name", // 支持路径语法,路径语法规则来自于：lodash的_.get(data, path) 方法
  "main",
];
```

>**开发标准**

1、模块的vuex导出的时候需要设置`namespaced: true`

```javascript
// Good
export default {
  namespaced: true,
  state,
  getters,
  actions,
  mutations
};

// bad
export default {
  state,
  getters,
  actions,
  mutations
};
```

2、需要持久化的手动维护在`persistedstate.js`，根据业务规则来维护，不要把不需要的维护进去。

3、vuex使用，通过`mapActions,mapGetter`等去引入使用

参考：[vue风格指南](https://cn.vuejs.org/v2/style-guide/#%E9%9D%9E-Flux-%E7%9A%84%E5%85%A8%E5%B1%80%E7%8A%B6%E6%80%81%E7%AE%A1%E7%90%86%E8%B0%A8%E6%85%8E%E4%BD%BF%E7%94%A8)

---

#### 7. styles 公共scss，由`sass-resources-loader`全局注入
   * `vars.scss` 下个版本重新定义使用

---

#### 8. views 页面的目录
   *  按模块存放业务页面
   *  业务根据模块划分到各个业务目录下面
   *  业务模块自己的组件存放在`components`下面
   *  学习文件夹规范请参考：[https://yq.aliyun.com/articles/645697](https://yq.aliyun.com/articles/645697)
*  main.js 入口文件
*  其他文件输入工程化构建文件
   *  .env开头文件，属于环境变量文件，可以了解一下：[https://cli.vuejs.org/zh/guide/mode-and-env.html](https://cli.vuejs.org/zh/guide/mode-and-env.html)  
   主要是`.env.local`or`.env.[mode].local` 这个文件，它是被git忽略的文件，可以配置本地链条换件，这样不用去修改Git上的环境文件

---

### 关于代码编写中的一些 tips

1.css 相关

```
  * 除app.vue外,需强制给每一个组件的style添加scoped,避免css/scss的命名混入
  * 使用 /deep/ 如:  .home /deep/ .title 在父组件重置带有作用域的css类
  * styles文件夹内包含两种scss文件,其中globalVar.scss内应该存放全局的通用变量,如标题字体大小,颜色类型等
  * styleReset.scss内存放的应该是第三方库或组件需要重置的选择器对应的样式
  * 如无特殊原因,请禁止使用!important来重置样式
```

2.dom 操作

```
  * 避免直接用原生的方式操作dom,推荐使用this.$refs
  * 如需在组件初始化时操作dom,请使用this.$nextTick
```

3.数据更新而视图不更新的问题排查

```
  * 是否给组件的data新增了原来没有的属性,如果是请使用this.$set来进行hack
  * 数组操作中是否返回了新的数组
```

4.vue 的各种生命周期及方法命名不能直接使用箭头函数.

5.关于 router-view 的同组建渲染更新请用以下方式:

```
<router-view :key='$route.fullPath'></router-view>
```

6.组件尽量减少没必要的嵌套,很多原生 DOM 属性会向下传递.

```
<!-- bad -->
<div class="content-view">
    <router-view></router-view>
</div>
<!-- good -->
<router-view class="content-view"></router-view>
```

7.异步数据的加载处理: 如果模版中绑定了 obj.xx 时，需要注意 obj 是否是异步数据，默认值是否为 null。安全起见，可在组件最外层加 v-if 判断.这样能避免没必要的报错.

```
<template>
    <div v-if="!!obj">
        <p>{{obj.name}}</p>
        <p>{{obj.age}}</p>
    </div>
</template>
<script>
export default {
    data() {
        return {
            obj: null
        }
    }
}
</script>
```

8.编写组件时,当该组件有业务逻辑处理时,一定要添加 name 属性,这样能在出现错误时快速定位错误位置.如:

```
<script>
export default {
  name: "HelloWorld",
  data(){
    return {
      title:'helloworld'
    }
  }
};
</script>
```
---
### 关于单页路由的说明


1.所有的路由页面呈现均在app.vue中做第一个层级路由或页面.比如:
```
<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">
        Home
      </router-link>
      |
      <router-link to="/about">
        About
      </router-link>
    </div>
    <keep-alive>
      <router-view />
    </keep-alive>
  </div>
</template>
```
2.具体的页面开发层级为: app.vue -> view(具体页面) -> components

3.一条完整的自定义路由大概如下:
```

{
    path: "/",
    name: "home",
    component: Home,
    meta: {
      title: "home",
      fullPage:false,
      // fullPage:true,
      keepAlive: true,
      permissions: [ "admin",
        "user",
        "editor"
      ]
    },
  }
```
path表示根路径;name表示路由名称,component对应的是组件(如需异步懒加载页面可变为:()=>import('home'));
meta中fullPage表示当前页面是否全屏显示,如不是则显示顶部和左部菜单,keepAlive表示是否缓存当前页面,如回退时保留数据;
permissions表示当前页面的角色访问权限.

---

### vue 工程可视化工具的使用
当全局环境安装了 vue-cli3 及以上时,可用 vue ui 直接进入工程管理页面,导入当前项目,能可视化修改和管理项目,包含:

- 项目语法校验
- 部分错误自动修复
- 完整的依赖管理
- 项目编译,并查看相关性能分析(如模拟各种网络下的访问速度加载时间等)

---

### 相关文档查询

1. [vue基础教程](https://cn.vuejs.org/v2/guide/)
2. [vuex基础教程](https://vuex.vuejs.org/zh/)
3. [vue-router基础教程](https://router.vuejs.org/zh/)
4. [vue api查询](https://cn.vuejs.org/v2/api/)
4. [vue-clio查询](https://cli.vuejs.org/zh/)
5. [vue约定规范文档](https://cn.vuejs.org/v2/style-guide/index.html)
6. [ES6标准入门](http://es6.ruanyifeng.com/)
7. [git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/)
8. [gitlab使用教程](https://www.yiibai.com/gitlab/gitlab_git_commands.html)
