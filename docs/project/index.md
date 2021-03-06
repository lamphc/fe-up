## 黑马头条

> 「黑马头条」对标“CSDN”、“博客园”等竞品，致力成为更加贴近年轻 IT 从业者（学员）的科技资讯类应用。

### 技术栈

vue 全家桶+vant 组件库+amfe-flexible 适配+vue-lazyload 图片懒加载+axios 请求+socket.io-client 即时通讯库

### 主要业务

#### 嵌套路由

> 实现子页面动态切换

思路：

1. 公共布局组件 layout
2. 嵌套路由 children

#### 登录

> 问题：你的项目中，是如何实现登录功能的？

**关键技术点描述实现的功能**=》给面试官留下提问的口子，引导面试官提问

答：实现登录功能，最核心的技术是 JWT（也就是 token）。我们通过 token 来记录用户的登录状态。并把 token 存储到 vuex 中。

1. 首先，我们调用登录的 API 接口，把用户填写的表单信息发送给后端（在真正发起请求之前，为了让用户体验更好，可以在对表单的数据进行合法性校验）。
2. 后端校验通过之后，会返回一个 token 字符串，里面记录了用户的基本信息。由于这个 token 字符串是 base64 格式的，很容易被逆向还原，因此它没有安全性可言，所以 token 中不会包含用户的敏感信息。
3. 为了在项目中能够方便地获取和使用 token，我们会把 token 存储到 vuex 中。
4. 由于 vuex 中的数据都是存储在内存中的，页面一刷新，状态就丢失了。因此，我们通过 vuex-persistedstate 这个插件，可以把 vuex 中的所有数据，持久化存储到本地。
5. 今后，用户在调用接口的时候，我们会通过 axios 的请求拦截器，判断用户请求的是否为有权限的接口。如果是，则会通过 config.headers 对象，为这次请求添加 Authorization 的请求头，值就是 vuex 中存储的 token 值。这样，有权限的接口才能调用成功。
6. 如果 token 过期了，会触发 axios 的响应拦截器。通过 err.response.status 可以判断响应状态码是否为 401，如果是，则证明 token 过期了，我们需要清空 vuex 中的数据。并通过路由的编程式导航 API，跳转到 login 登录页面，让用户重新进行登录。

> 项目中的 axios 咋用的？

说明：可以从 axios 的作用、如何封装、如何使用 3 个维度来回答。

1. axios 在项目中，主要用来发起 Ajax 的数据请求，专门用来实现前后端数据交互的。
2. 我们在项目中会对 axios 做进一步的封装。会创建一个 src/utils/request.js 的模块，里面调用 axios.create() 函数，来创建 axios 的实例对象。在调用 create 函数期间可以全局配置请求的 baseURL 根路径等。
3. 对于功能类似的 API 接口，我们会在 src/api 目录下进行接口的封装。这样可以提高 API 接口的复用性。
4. 在项目中还会用到 axios 的拦截器。它主要的应用场景是：
   a. 基于请求拦截器展示 loading 提示，基于响应拦截器隐藏 loading 提示
   b. 基于请求拦截器，为有权限的接口统一添加 Authorization 的 token 认证
   c. 基于响应拦截器，判断响应状态码是否为 401，来处理 token 失效后的问题

#### 新闻列表

> 讲讲移动端列表页面的实现和优化？

答：主要实现上拉加载更多和下拉刷新，图片懒加载等，优化列表首屏渲染性能。

1. 上拉加载和下拉刷新是依赖 vant 组件库提供的 List 列表组件实现。其中，上拉加载更多原理可以通过触底检测或通过 IntersectionObserver 函数检测底部加载中提示是否在可视区，进行分页数据的请求和渲染
2. 图片懒加载借助 lazy-load 插件实现，原理是只加载当前可视区域内的图片，原理也可以通过 IntersectionObserver 函数检测图片是否进入可视区，如果进入进行图片的请求和渲染
3. 组件缓存是通过 vue 内置组件 keep-alive 实现，被缓存组件会执行对应 activated、deactivated 这两个生命周期钩子函数。其中，借助 activated 可以记录列表滚动位置和执行滚动方法，实现阅读记忆功能

#### 频道管理

> 频道管理中怎么处理用户自选和默认频道的数据管理？

答：主要借助计算属性对频道数据进行动态计算，得到用户自选频道数据

1. 根据后台 API 获取所有频道数据和用户频道数据
2. 使用计算属性，根据用户频道数据过滤所有频道数据（所有频道数据 - 用户频道数据 = 可选频道数据），如果当前频道数据在用户频道数据中不存在，就属于可选频道数据

#### 搜索新闻

> 讲讲搜索业务的实现流程和优化？

答：搜索业务主要根据搜索关键词进行数据的搜索展现

1. 通过双向绑定获取搜索关键词
2. 根据搜索关键词调用后台接口，获取联想词列表。这里为了避免用户输入期间，多次调用联想词接口，进行了函数防抖处理
   a. 函数防抖：延迟时间内，没有再次重复触发，就会执行一次（如果重复触发，会清除定时器，重新计时）。主要借助 setTimeout 函数实现
   b. 函数节流：延迟时间内，保证会执行一次。主要通过闭包记录上一次执行回调函数时间，如果当前时间 - 上一次执行函数时间 > 延迟时间，就执行回调
3. 用户回车确认搜索跳转结果页之前，记录搜索关键词到本地 localStorage 中，进行排重处理

#### 文章详情

> 文章阅读记忆如何实现？一个组件被缓存，怎么刷新页面数据？

答：主要是缓存组件内容，记录滚动位置，下次进入根据记录位置，执行滚动

1. 使用 vue 内置组件 keep-alive 缓存文章详情内容
2. 用户阅读内容滚动页面时，通过 scroll 事件记录滚动高度
3. 下一次用户再次进入当前缓存页，执行滚动到上次阅读位置
4. 如果用户再次进入时，在 activated 钩子函数判断文章详情 ID 参数发生变化，重新调用接口刷新数据并重置记录的阅读位置

#### 个人中心

> 讲讲上传的实现思路？

答：主要借助 input 元素或三方上传组件（组件库提供），配合后台 API 接口实现文件上传

1. 准备 input[type=file]，进行文件选择，通过 chang 事件获取 file 文件对象
2. 通过 formData 存储 file 文件对象，调用接口上传文件到服务器

#### 小智聊天

> 即时聊天如何实现？

答：IM 即时聊天，主要依赖 WebSocket 技术实现，一般使用社区比较成熟的方案 socket.io-client

1. 安装 socket.io-client
2. 使用插件提供的 IO 方法传入后台 ws 地址，建立长连接
3. 通过 IO 实例的 on 方法监听事件，接收服务器的数据
4. 通过 IO 实例的 emit 方法触发事件，向服务器发送数据
   注意：
5. 使用 socket.io-client 需要前后台配合
6. 处理聊天内容超过一屏时滚动
   a. 思路：获取聊天列表滚动高度 dom.list.scrollHeight，执行滚动

#### 移动打包

> 前端如何打包 vue 开发的页面为移动 app？

答：借助 HbuilderX 提供的在线云打包（无需本地配置打包环境）

> 移动开发的主要方式和技术栈？

答：

1. 原生开发
   a. IOS 开发：oc（Objective-C）、swift 语言
   b. Android 开发：java 语言
2. webapp 开发：也叫 html5（H5）开发，主要技术 html5+css3+js
3. HybridApp 混合开发：原生+H5 结合，原生 APP 中借助 webview 控件嵌入 H5 页面形式
4. 跨平台开发
   a. 典型代表 react-native 框架，一套 js 代码跨平台运行
   b. 小程序

## 人资中台

> iHRM 是一款基于 SaaS 平台的人力资源管理系统， 企业通过该系统可以完成公司组织架构管理、员工管理、角色管理、权限管理（审批、考勤、社保）等，为企业的人力资源管理提供一站式解决方案。

### 技术栈

vue admin template 基础架构（方案） =》vue 全家桶+element 组件库+自带功能组件（Excel 导入导出等）

### 主要业务

#### 环境变量配置

> vue-cli 怎么设置环境变量？ 什么是模式？

答：

vue-cli 创建的项目默认有三个模式

1. development 模式用于 vue-cli-service serve（执行这个命令）
2. test 模式用于 vue-cli-service test:unit
3. production 模式用于 vue-cli-service build
   说明：比较常用的是 development 和 production 模式，对应开发和生产环境

环境变量设置

1. 通过.env.[mode]文件设置（以开发环境为例）
   a. 设置开发环境环境变量，在项目根目录新建.env.development 文件
   b. 在.env.development 文件中：key=value
   c. 说明：VUE*APP*开头的变量可以通过 process.env 在项目中获取使用
2. 设置完，需要重启服务

#### 登录

#### 组织架构

> 什么是树形数据结构和转换方式？

答：前端在做树形组件的时候，组件需要的 prop 数据为树形数据 ，但是后端接口返回的是平铺的数组结构，所以需要我们前端自己做一步数据转化以适配组件的需要，转树形数据主要是
自己写了一个属性数组处理方法，大概思路是寻找父节点，找到自己的父节点然后把自己放到父节点的 children 属性中

#### 角色管理

#### 员工管理

> 讲讲 excel 导入和导出的实现？

答：

1. 公司的导入导出因为数据量并不大，所以采用的主要方式的前端主导的导入方案，主要是借助插件，通过插件把 excel 表格转化成 js 的数据类型，然后按照后端接口的要求调用接口完成导入，这里主要的问题还是数据处理，插件处理之后的数据和后端接口要求的数据并不一致

2. 导出的话主要是调用的后端接口，告诉后端需要导出的数据，然后利用插件把后端接口返回的数据转化成插件要求的格式，然后触发浏览器的下载功能进行下载

#### 员工详情

> 员工头像数据上云实现

答：核心是使用三方云服务，把图片或文件存储到云服务器，云服务器的存储 url 存储到公司自由服务器

#### 权限设计和实现

> 后台系统如何进行权限控制和实现？

答：基于 RBAC 设计权限管理，对路由进行动、静划分，根据登录人权限标识 code 动态添加对应 code 代表的路由页面到路由堆栈

#### 首页开发

> 讲讲国际化实现思路？

答：国际化的实现思路主要是通过 i18n 国际化插件实现的

1. 首先需要提前准备好国际化需要的语言包，也就是语言之间的对应关系
2. 在项目里安装了 i18n 国际化包，在 Vue 原型上挂载了转化方法
3. 在组件中需要做国际化的地方 通过转化方法对对应的文字进行方法调用转化为特定的语言显示

## 小兔鲜儿

> Vue3.0 小兔鲜儿电商是对标网易严选的综合品类的网上商城项目，现在除了顶级的一些电商平台之外，越来越多的企业都会做自己的垂直领域的商城，如小米商城、华为商城等，市场空间巨大，人才较为紧缺。本项目涵盖了电商的核心业务，首页加载性能优化，本地线上购物车，订单管理，第三方支付。

### 技术栈

Vue3+ @vueuse/core + vuex + vue-router

### 主要业务

#### 首页开发

> 谈谈电商首页如何做性能优化？

答：电商页面由于比较长，加载的资源比较多，所以我们做的主要是网络资源加载层面的优化

1. 针对于组件数据，我们做了懒加载处理，只有进入到视口区域的组件才正式发送接口请求，把之前在生命周期中发送的请求接管过来，由自己控制

2. 针对于图片资源繁多的情况，我们封装了图片懒加载指令，对于图片资源做加载控制，防止资源加载堵塞

#### 购物车

> 谈谈购物车是如何实现的遇到了哪些难点？

答：

1. 购物车主要是采用了 vuex 进行的数据管理，因为购物车的数据在很多个组件中都需要操作，用 vuex 可以方便操作数据
2. 购物车主要功能包括购物车的加入、删除、单选、多选等操作，基本上都是围绕 vuex 展开的
3. 为了刷新不丢失数据，我要做了 vuex 数据持久化

#### 下单支付

> 在线支付的流程和实现？

答： 在线支付的主要流程是有三部分构成，前端 后端 支付宝三方服务。

1. 我们自己的前端准备订单 id 等必要参数通知我们自己的后端需要支付了
2. 我们自己的后端拿到参数之后去和支付宝三方服务进行接口调用 进行支付流程
3. 支付宝服务完成支付之后会通知我们自己后端支付状态等信息，再由我们自己的后端打开我们提前约定好的落地页面
4. 前端负责展示支付结果
