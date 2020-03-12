<template>
  <div id="app">
    <choose-user v-if="$root.me == null" :userlist = 'userlist'></choose-user>
    <user-list :islogin='islogin' :users='users' :room = 'room' :chooseUser="chooseUser" :chooseRoom="chooseRoom" v-if="$root.me !=null" :unreadlist = 'unreadlist' :unreadroomlist = 'unreadroomlist'></user-list>
    <chat-user v-if="ischat&&touser != null" :touser='touser' :closeChat="closeChat" :newMsg = 'newMsg' :unreadMsg = 'unreadMsg'></chat-user>
    <chat-room v-if="ischat&&toroom != null" :toroom='toroom' :newroomMsg = "newroomMsg" :unreadroomMsg = 'unreadroomMsg' :closeChat="closeChat" ></chat-room>
  </div>
</template>

<script>
import chooseUser from './components/chooseUser'
import userList from './components/userList'
import axios from 'axios'
import socket from './socket'
import chatUser from './components/chatUser'
import chatRoom from './components/chatRoom'

export default {
  name: 'App',
  components: {
    chooseUser,
    userList,
    chatUser,
    chatRoom,
  },
  data() {
    return {
      userlist:[],
      islogin:false,
      users:[],
      room:[],
      ischat:false,
      touser:null,
      toroom:null,
      unreadlist:[],
      unreadroomlist:[],
      newMsg:null,
      newroomMsg:null,
      unreadMsg:[],
      unreadroomMsg:[],
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
        socket.emit('users',data.data)
        socket.emit('room',data.data)
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
    socket.on('disconnect',()=>{
      console.log('断开连接')
    })

    // 获取用户列表
    socket.on('users',(data)=>{
      this.users = data
    })
    // 获取群列表
    socket.on('room',(data)=>{
      this.room = data  
    })

    // 未读个人信息
    socket.on('unreadMsg',(data)=>{
      data.forEach((item,index) => {
        
        // 设置未读的红点      
        // 点击用户，在添加信息
        this.unreadMsg.push(item)
        this.unreadlist.push(item.sendid)
      });
    })
    // 未读群消息
    socket.on('unreadroomMsg',(data)=>{
      data.forEach((item,index) => {
        // 设置未读的红点      
        // 点击用户，在添加信息
        console.log("==============")
        console.log(item)
        this.unreadroomMsg.push(item)
        this.unreadroomlist.push(item.toroomid)
      });
    })

    // 在线发送个人消息
    socket.on('accept',(msg)=>{
      // 判断是否在当前聊天页面且对象一致
      if(this.ischat ==true && msg.sendid ==this.touser.id){
        this.newMsg = msg
        // 修改readtime
        socket.emit('readMsg',{
            sendid:this.touser.id ,
            toid:this.$root.me.id,     
        })
      }
      // 聊天对象不一致时
      else{
        // 点击用户，在保存在loction
        // 修改未读状态 
        this.unreadMsg.push(msg)
        this.unreadlist.push(msg.sendid)

      }
    })

    socket.on('acceptroom',(msg)=>{
      console.log("++++++++++")
      // 判断聊天界面是否一致
      console.log(msg)
      if(this.ischat == true && msg.toid == this.toroom.id){
        this.newroomMsg = msg
        // 修改readtime
        socket.emit('readroomMsg',{
            toid:this.$root.me.id,     
        })
      }
      else{
        // 点击群聊，在保存在loction
        // 修改未读状态 
        this.unreadroomMsg.push(msg)
        this.unreadroomlist.push(msg.toid)
      }
    })
   
  },
  methods: {
    chooseUser: function(user){    
      this.touser = user
      this.ischat = true
      // 消除红点
      let index = this.unreadlist.indexOf(user.id)
      if(index >= 0){
        this.unreadlist.splice(index,1)
      }
      
    },  
    chooseRoom: function(room){    
      this.toroom = room
      this.ischat = true
      // 消除红点
      let index = this.unreadroomlist.indexOf(room.id)
      if(index >= 0){
        this.unreadroomlist.splice(index,1)
      }
    },
    closeChat:function(){
      this.ischat = false
    }
  },
  computed: {
    userObj:function(){
      let obj = {}
      this.users.forEach((item,index)=>{
        obj[item.id] = item
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
