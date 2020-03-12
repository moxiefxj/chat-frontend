<template>
    <div class="userlist">
        <div class="nav">
            <div class="headerimg" :class="{online:islogin}">
                <img :src="$root.me.headerimg" alt="">
            </div>
            <div class="title">消 息</div>
            <div class="headerimg"></div>
        </div>
        <div class="users">
            <div @click="chooseUser(item)" class="useritem" v-for="(item, index) in users" :key="'useritem'+index">
                <div class="left" :class="{online:item.isonline==1 ,unread:unreadlist.indexOf(item.id)!=-1}">
                    <img :src="item.headerimg">
                </div>
                <div class="right">
                    <span class="username">{{item.username}}</span>
                    <span class="msg"></span>
                </div>      
            </div>
            <div @click="chooseRoom(item)" class="useritem" v-for="(item, index) in room" :key="index">
                <div class="left online" :class="{unread:unreadroomlist.indexOf(item.id)!=-1}">
                    <img :src="item.roomimg">
                </div>
                <div class="right">
                    <span class="username">{{item.roomname}}</span>
                    <span class="msg"></span>
                </div>      
            </div>

        </div>
    </div>
</template>
<script>
export default {
    props:['islogin','users','chooseUser','unreadlist','room','chooseRoom','unreadroomlist'],
    
}
</script>

<style scoped>
    .unread{
        position: relative;
    }
    .unread::before{
        position: absolute;
        content: "";
        display: block;
        width: 10px;
        height: 10px;
        border-radius: 5px;
        background: red;
        bottom: 20px;
        right: 5px;

    }
 
    .useritem .left{
        filter: grayscale(1)
    }
    .useritem{
        display: flex;
        height: 80px;
        background: #eee;
        border-bottom:1px solid #ccc;
        text-align: center;
        padding: 0 10px;
        padding-top:10px;

    }
    .useritem .left img{
        width: 60px;
        height: 60px;
        border-radius: 50%;
    }
    .nav{
        height: 80px;
        width: 100vw;
        background: seagreen;
        display: flex;
        align-items: center;
        justify-content: space-between;
    }
    .headerimg{
        height: 50px;
        width: 50px;
        filter:grayscale(1);
        margin: 0 10px;
        
    }
    .online{
        filter:grayscale(0) !important
    }
    .headerimg img{
        height: 50px;
        width: 50px;
        border-radius: 50%;
        float: left;
    }
    .nav .title{
        font-weight: 900;
        font-size:18px;
    }
</style>