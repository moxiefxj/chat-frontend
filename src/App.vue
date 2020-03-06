<template>
  <div id="app">
    <choose-user v-if="$root.me == null" :userlist = 'userlist'></choose-user>
    <user-list :islogin='islogin' :users='users' :chooseUser="chooseUser" v-if="$root.me !=null" :unreadlist = 'unreadlist'></user-list>
    <chat-user v-if="ischat" :touser='touser' :closeChat="closeChat" :newMsg = 'newMsg'></chat-user>
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
      newMsg:null,

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
        // 将sendid/toid改成由头像的对象
        item.sendid = this.userObj[item.sendid]
        item.toid = this.userObj[item.toid]
        this.unreadlist.push(item.sendid)
        
        let strKey = 'chat-user-'+this.$root.me.id+'-'+item.sendid.id
        // 先解析本地存储的数据，在添加
        // 解析：JSON.parse(localStorage[strKey]).push(item);
        // console.log(localStorage[strKey])
        localStorage[strKey] = localStorage[strKey]?localStorage[strKey]:'[]'
        let newArr = JSON.parse(localStorage[strKey])
        newArr.push(item)
        localStorage[strKey] = JSON.stringify(newArr);  
      });
    })

    socket.on('accept',(msg)=>{
      console.log(msg)
      // 判断是否在当前聊天页面且对象一致
      if(this.ischat ==true && msg.sendid.id ==this.touser.id){
        this.newMsg = msg
      }
      // else if(msg.toid.username == this.touser.username && msg.toid.isgroup=='true'){
      //   this.newMsg = msg
      // }
      else{

        let strKey = 'chat-user-'+msg.toid.id+'-'+msg.sendid.id
        // 先解析本地存储的数据，在添加
        // 解析：JSON.parse(localStorage[strKey]).push(item);
        localStorage[strKey] = localStorage[strKey]?localStorage[strKey]:'[]'
        let newArr = JSON.parse(localStorage[strKey])
        newArr.push(msg)
        localStorage[strKey] = JSON.stringify(newArr);

        // 小红点显示
        let unreadUser = msg.toid.isgroup !=null?msg.toid.id:msg.sendid.id
        // 修改未读状态   true 为群
        this.unreadlist.push(unreadUser)

      }
    })
  },
  methods: {
    chooseUser: function(user){    
      this.touser = user
      this.ischat = true
      // 消除红点
      let index = this.unreadlist.indexOf(user.id)
      this.unreadlist.splice(index,1)
    },
    closeChat:function(){
      this.ischat = false
    }
  },
  computed: {
    userObj:function(){
      let obj = {}
      this.users.forEach((item,index)=>{
        obj[item.username] = item
      })
      return obj;
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
