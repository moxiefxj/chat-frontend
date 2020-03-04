<template>
  <div id="app">
    <choose-user v-if="$root.me == null" :userlist = 'userlist'></choose-user>
    <user-list :islogin='islogin' :users='users' :chooseUser="chooseUser" v-if="$root.me !=null" :unreadlist = 'unreadlist'></user-list>
    <chat-user v-if="ischat" :touser='touser' :closeChat="closeChat"></chat-user>
  </div>
</template>

<script>
import chooseUser from './components/chooseUser'
import userList from './components/userList'
import axios from 'axios'
import socket from './socket'
import chatUser from './components/chatUser'

export default {
  name: 'App',
  components: {
    chooseUser,
    userList,
    chatUser,
  },
  data() {
    return {
      userlist:[],
      islogin:false,
      users:[],
      ischat:false,
      touser:null,
      unreadlist:[],
    }
  },
  // async:异步操作
  async beforeMount() {
    let res = await axios.get('http://localhost:3000/api/userlist')
    this.userlist = res.data
  },
  mounted() {
    // 监听登录成功，设置为true
    socket.on('login',(data)=>{
      if(data.state == 'ok'){
        this.islogin = true
        socket.emit('users')
      }
    })
    //监听登出事件
    socket.on('logout',(data)=>{
      console.log(data)
      this.islogin = false
      // 断开连接
      socket.disconnect()
    })
    // 监听断开连接事件
    socket.on('disconnect',(data)=>{
      console.log('断开连接')
    })

    // 获取用户列表
    socket.on('users',(data)=>{
      this.users = data
    })

    // 未读信息
    socket.on('unreadMsg',(data)=>{
      data.forEach((item,index) => {
        // 设置未读的红点
        // 将聊天的内容分别添加到本地存储
        this.unreadlist.push(item.sendname) 
      });
    })
  },
  methods: {
    chooseUser: function(user){     
      this.touser = user
      this.ischat = true
    },
    closeChat:function(){
      this.ischat = false
    }
  },

}
</script>

<style>
  *{
    margin: 0;
    padding: 0;
  }
</style>
