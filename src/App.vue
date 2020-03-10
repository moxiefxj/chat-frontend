<template>
  <div id="app">
    <choose-user v-if="$root.me == null" :userlist = 'userlist'></choose-user>
    <user-list :islogin='islogin' :users='users' :room = 'room' :chooseUser="chooseUser" :chooseRoom="chooseRoom" v-if="$root.me !=null" :unreadlist = 'unreadlist'></user-list>
    <chat-user v-if="ischat&&touser != null" :touser='touser' :closeChat="closeChat" :newMsg = 'newMsg'></chat-user>
    <chat-room v-if="ischat&&toroom != null" :toroom='toroom' :closeChat="closeChat" ></chat-room>
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
      ischat:false,
      touser:null,
      toroom:null,
      unreadlist:[],
      newMsg:null,
      newroomMsg:null,
      room:[],

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
      console.log(data)
      this.users = data
    })
    // 获取群列表
    socket.on('room',(data)=>{
      this.room = data
      
    })


    // 未读信息
    socket.on('unreadMsg',(data)=>{
      data.forEach((item,index) => {
        // 设置未读的红点
        // 将聊天的内容分别添加到本地存储 
        // 将sendid/toid改成由头像的对象
        // item.sendid = this.userObj[item.sendid]
        // item.toid = this.userObj[item.toid]

        this.unreadlist.push(item.sendid)

        console.log(this.unreadlist)
        
        let strKey = 'chat-user-'+this.$root.me.id+'-'+item.sendid
        // 先解析本地存储的数据，在添加
        // 解析：JSON.parse(localStorage[strKey]).push(item);
        // console.log(localStorage[strKey])
        localStorage[strKey] = localStorage[strKey]?localStorage[strKey]:'[]'
        let newArr = JSON.parse(localStorage[strKey])
        newArr.push(item)
        console.log(newArr)
        localStorage[strKey] = JSON.stringify(newArr);  
      });
    })

    // 个人消息
    socket.on('accept',(msg)=>{
      
      // 判断是否在当前聊天页面且对象一致
      if(this.ischat ==true && msg.sendid ==this.touser.id){
        this.newMsg = msg
        socket.emit('readMsg',{
            selfid:this.$root.me.id         
        })
      }
      // 聊天对象不一致时
      else{
        let strKey = 'chat-user-'+msg.toid+'-'+msg.sendid
        // 先解析本地存储的数据，在添加
        // 解析：JSON.parse(localStorage[strKey]).push(item);
        localStorage[strKey] = localStorage[strKey]?localStorage[strKey]:'[]'
        let newArr = JSON.parse(localStorage[strKey])
        newArr.push(msg)
        localStorage[strKey] = JSON.stringify(newArr);
        console.log(newArr)

        // 修改未读状态 
        this.unreadlist.push(msg.sendid)

      }
    })

    socket.on('acceptroom',(msg)=>{
      // 判断聊天界面是否一致
      if(this.ischat == true && msg.sendid == this.toroom.id){

      }
    })
    // 群消息
    socket.on('acceptroom',(msg)=>{
      // 判断聊天界面是否一致
      if(this.ischat == true && msg.sendid == this.toroom.id){
        this.newroomMsg = msg
        socket.emit('readMsg',{
          selfid:this.$root.me.id
        })
      }
      else{
        
      }
    })
  },
  methods: {
    chooseUser: function(user){    
      this.touser = user
      this.ischat = true
      console.log(user)
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
      let index = this.unreadlist.indexOf(room.id)
      if(index >= 0){
        this.unreadlist.splice(index,1)
      }
    },
    chooseRoom: function(room){    
      this.toroom = room
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
