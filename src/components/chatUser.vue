<template>
    <div class="chatuser">
        <div class="header">
            <span class='back' @click="closeChat()">&lt;</span>
            <div>{{touser.username}}</div>
        </div>
        <div class="chatlist" ref="chatbox">
            <div class="chatItem" v-for="(item, index) in chatlist" :key="index" :class="{self:$root.me.id == item.sendid.id}">
                <div class="header">
                    <img :src="item.sendid.headerimg" alt="">
                </div>
                <div class="chatContent">
                    {{item.content}}
                </div>
            </div>
        </div>
        <div class="inputcom">
            <input type="text" v-model="inputData" @keydown.enter="sendEvent">
            <button @click="sendEvent">发送</button>
        </div>
    </div>
</template>

<script>
import socket from '../socket'

export default {
    props:['touser','closeChat','newMsg'],
    data() {
        return {
            chatlist:[],
            inputData:[],
        }
    },
    methods: {
        sendEvent:function(){
            let msg = {
                sendid:this.$root.me,
                toid:this.touser,
                content:this.inputData,
                chattime:new Date().getTime(),
            } 
            console.log(msg)
            // 发送到服务端
            socket.emit('sendMsg',msg)
            this.chatlist.push(msg)
            // 保存聊天记录到本地
            this.saveStorage()
        },
        // 保存到本地
        saveStorage(){
            let strKey = 'chat-user-'+this.$root.me.id+'-'+this.touser.id
            localStorage[strKey] = JSON.stringify(this .chatlist) 
        },
        getStorage(){
            let strKey = 'chat-user-'+this.$root.me.id+'-'+this.touser.id
            console.log(localStorage[strKey])
            localStorage[strKey] = localStorage[strKey]?localStorage[strKey]:'[]'
            this.chatlist = JSON.parse(localStorage[strKey]) 
        },
        toBottom(){
            let chatbox = this.$refs.chatbox
            chatbox.scrollTop = chatbox.scrollHeight - chatbox.clientHeight
        }
    },
    // 挂载前修改信息为已读
    beforeMount() {
        this.getStorage()
        
        // 挂载前修改信息为已读
        socket.emit('readMsg',{
            selfid:this.$root.me.id,
            userid:this.touser.id
        }) 
    },
    //挂载完成后，消息列表显示在最低端
    mounted() {
        this.toBottom()       
    },
    // 新消息push
    watch: {
        newMsg:function(val){
            this.chatlist.push(val)
            this.saveStorage()
        }
    },
    //更新完成后，消息列表显示在最低端
    updated() {
        this.toBottom()
    },
}
</script>

<style scoped>
    .chatItem{
        display: flex;
        margin: 5px 10px;
    }
    .chatItem.self{
        flex-direction: row-reverse;
        justify-content: flex-start;

    }
    .chatItem .header img{
        width: 50px;
        height: 50px;
        border-radius: 50%
    }
    .chatItem .chatContent{
        background: #bbb;
        border-radius: 5px;
        padding: 8px 10px;
        color: #fff;
        margin: 0 0px 0px 20px;
        line-height: 34px;
        position: relative;
    }
    .chatItem.self .chatContent{
        margin: 0 20px 0px 0px;
    }
    .chatItem .chatContent::before{
        display: block;
        content: "";
        position: absolute;
        width: 0;
        height: 0;
        border-right: 10px solid #bbb;
        border-top: 10px solid transparent;
        border-bottom: 5px solid transparent;
        top: 20px;
        left: -10px;
    }
    .chatItem.self .chatContent::before{
        display: block;
        content: "";
        position: absolute;
        width: 0;
        height: 0;
        border-left: 10px solid #bbb;
        border-top: 10px solid transparent;
        border-bottom: 5px solid transparent;
        top: 20px;
        right: -10px;
        left: initial;
        border-right:initial;
    }
    .chatuser{
        width: 100vw;
        height: 100vh;
        display: flex;
        flex-direction: column;
        position: fixed;
        top: 0;
        left: 0;
        background: #efefef;
    }
    .chatuser>.header{
        position: relative;
    }
    .chatuser .back{
        display: block;
        width: 40px;
        height: 40px;
        line-height: 40px;
        text-align:center;
        position: absolute;
        left: 0;
        top: 0;
    }

    .chatuser>.header{
        font-size: 18px;
        font-weight: 900;
        background: skyblue;
        height: 40px;
        text-align: center;
        line-height: 40px;

    }
    .chatlist{
        flex:1;
        overflow: scroll;
    }
    .inputcom{
        height: 50px;
        display: flex;
        background: #eee;
        justify-content: space-around;
    }
    .inputcom input{
        width: 270px;
        height: 40px;
        border-radius: 5px;
        outline: none;
        border: 1px solid #ccc;
        margin: 0 5px;
    }
    .inputcom button{
        width: 80px;
        height: 40px;
        border-radius: 5px;
        outline: none;
        border:  1px solid #ccc;
        margin: 0 5px;
    }
</style>