## 项目简介> 主要是通过做一个多人在线多房间群聊的小项目、来练手全栈技术的结合运用。**主要技术：** vue2全家桶 + socket.io + node(express) + mongodb(mongoose)**环境配置：** 需安装配置好 node,mongodb环境（[参考：[http://gjincai.github.io/categories/mongodb/](http://gjincai.github.io/categories/mongodb/)）; 建议安装 Robomogo 客户端来管理mongodb数据。**编译运行：** 1.开启MongoDB服务，新建命令行窗口1：```mongod```2.启动服务端node，新建命令行窗口2：```cd servernode index.js```3.启动前端vue页面```cd clientcnpm installnpm run dev```然后在浏览器多个窗口打开 `localhost:8080`，注册不同账号并登录、即可进行多用户多房间在线聊天。## 代码目录概览```js|--chat-vue-node    |--client               // 前端客户端：基于 vue-cli 搭建的所有聊天页面    |--server               // 后台服务端        |--api.js             // express 通过 mongoose 操作 mongodb 数据库的所有接口        |--db.js              // 数据库初始化、Schema数据模型        |--index.js           // 后台服务启动入口        |--package.json    .gitignore    README.md```## soeket.io 基础soeket.io 在该项目中用到的基本功能如下（详情请看GitHub中的chatGroup.vue、server/index.js这两文件代码）：```js// 客户端连接var socket = io.connect('http://localhost:8081')// 服务端监听到连接io.on('connection', function(socket){  // ......}// 客户端发送进入房间请求socket.emit('joinToRoom', data)// 服务端监听socket.on('joinToRoom', function (data) {  // ......  // 服务端处理进入房间、离开房间  socket.join(roomGroupId)  // 有人进入房间，向该群其它的成员发送更新在线成员  io.sockets.in(roomGroupId).emit('joinToRoom', chat)  io.sockets.in(roomGroupId).emit('updateGroupNumber', roomNum[roomGroupId])}// 客户端发送聊天消息socket.emit('emitChat', chat)// 服务端监听并向群内其它人群发该消息socket.on('emitChat', function (data) {  let roomGroupId = chat.chatToGroup  // 向特定的群成员转发消息  socket.in(roomGroupId).emit('broadChat', chat)})```## 数据结构设计主要有三个数据结构模型：```js// 用户信息的数据结构模型const accountSchema = new Schema({  account: {type: Number},    // 用户账号  nickName: {type: String},   // 用户昵称  pass: {type: Number},       // 密码  regTime: {type: Number}     // 注册时间})// 聊天群的数据结构模型：聊天群包含的成员const relationSchema = new Schema({  groupAccount: Number,       // 群账号  groupNickName: String,      // 群昵称  groupNumber: []             // 群成员})// 单个聊天群的聊天消息记录const groupSchema = new Schema({  account: Number,            // 聊天者的账号  nickName: String,           // 聊天者的昵称  chatTime: Number,           // 发消息的时间戳  chatMes: String,            // 聊天的消息内容  chatToGroup: Number,        // 聊天的所在群账号  chatType: String            // 消息类型：进入群、离开群、发消息})```## vue-router 路由设计页面路由的跳转全部由前端的 vue-router 处理，页面功能少而全、仅3个：注册登录页、个人中心页、群聊页```jsroutes: [  // {path: '/', name: 'Hello', component: Hello},  {path: '/', redirect: '/login', name: 'Hello', component: Hello},  {path: '/login', name: 'Login', component: Login},  {path: '/center', name: 'Center', component: Center},  {path: '/chatGroup', name: 'ChatGroup', component: ChatGroup}]// 未登录时，通过编程式跳去登录页：router.push({ path: 'login' })```## vuex 全局状态主要是通过vuex来全局管理个人账号的登录状态、当前所在群聊房间的信息：```jsexport default new Vuex.Store({  state: {    chatState: {      account: null,      nickName: null    },    groupState: null   //  点击进群的时候更新  },  mutations: {    updateChatState (state, obj) {      state.chatState = obj    },    updateGroupState (state, obj) {      state.groupState = obj    }  },  actions: {    updateChatState ({commit}, obj) {      commit('updateChatState', obj)    },    updateGroupState ({commit}, obj) {      commit('updateGroupState', obj)    }  },  getters: {    getChatState (state) {      return state.chatState    },    getGroupState (state) {      return state.groupState    }  }})```在全局中更新state、获取state:```js// 更新this.$store.dispatch('updateChatState', {account: null, nickName: null})// 获取this.$store.getters.getChatState```## 数据库接口api```jsmodule.exports = function (app) {  app.all("*", function(req, res, next) {    next()  })  // api login 登录  app.get('/api/user/login', function (req, res) { // ... })  // api register 注册  app.get('/api/user/register', function (req, res) { // ... })  // getAccountGroup 获取可进入的房间  app.get('/api/user/getAccountGroup', function (req, res) { // ... })  // getGroupNumber 获取当前房间的群成员  app.get('/api/user/getGroupNumber', function (req, res) { // ... })  // api getChatLog 获取当前房间的聊天记录  app.get('/api/getChatLog', function (req, res) { // ... })  app.get('*', function(req, res){    res.end('404')  })}```更多详细的实现，请看 [源码 chat-vue-node](https://github.com/gjincai/chat-vue-node) ，里面保留着开发摸索时的很多注释。