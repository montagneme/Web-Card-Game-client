<template>
  <div id="app">
    <div class="box">
      <transition name="alert">
          <span class="alert" v-if="alertState">{{alertVal}}</span>
      </transition>
      <transition name="id">
          <div class="id_box" v-if="!hasId">
            <p class="id_title">请输入ID-KEY</p>
            <input type="text" class="id" v-model="id" placeholder="ID-KEY">
            <div class="id_submit" @click="IdSubmit" >确认</div>
          </div>
      </transition>
      <transition name="start">
        <div class="start" v-if="isStart">{{startVal}}</div>
      </transition>
      <transition name="game">
        <div class="game" v-if="hasId">
          <div class="opp_coin"  v-if="oppName!=''">
              <img :src="coinImg" alt="" style="height: 20px;display: block;float: left;"> <span style="height: 20px;line-height: 20px;display: block;margin-left:5px;float: left;color:rgb(245, 187, 87);">{{oppCoin}}</span>
          </div>
          <div class="own_coin">
              <img :src="coinImg" alt="" style="height: 20px;display: block;float: left;"> <span style="height: 20px;line-height: 20px;display: block;margin-left:5px;float: left;color:rgb(245, 187, 87);">{{ownCoin}}</span>
          </div>
          <div class="own_name">{{ownName}}</div>
          <div class="opp_name" v-if="oppName!=''">{{oppName}}</div>
          <div class="opp_name" v-else>正在等待玩家进入...</div>
          
          <div class="center_box">
            <transition name="pk_ok">
                <div class="opppk_ok" v-if="oppPk && !pokerPk">
                  来啊
                </div>
            </transition>
            <transition name="pk_ok">
                <div class="ownpk_ok" v-if="ownPk && !pokerPk">
                  来啊
                </div>
            </transition>
            <span class="time" v-if="!winState">{{time}}</span>
            <transition name="win">
              <div class="win" v-if="winState && win==1">你赢了!</div>
              <div class="win" v-if="winState && win==0">你输了!</div>
              <div class="win" v-if="winState && win==2">平局</div>
            </transition>
          </div>
          <div class="double_num_box" v-if="gameOpen">


            <transition-group name="own_double" class="own_double_ani" tag="div">
                <img :src="doubleImg2" v-for="item in ownDoubleList" alt="" :key="item" class="own_double_img">
            </transition-group>
                  
            <transition-group name="opp_double" class="opp_double_ani" tag="div">
                <img :src="doubleImg" v-for="item in oppDoubleList" alt="" :key="item" class="opp_double_img">
            </transition-group>
            倍数：{{double}}倍<br>
            <img :src="coinImg" alt="" style="height: 20px;display: block;float: left;"> <span style="height: 20px;line-height: 20px;display: block;margin-left:5px;float: left;color:rgb(245, 187, 87);">{{doubleCoin}}</span>
          </div>
          <transition name="double">
              <div class="double" @click="Double" v-if="own.length!=0">加倍</div>
          </transition>
          <transition name="go">
              <div class="go" @click="Go"  v-if="own.length!=0">过</div>
          </transition>
          <transition name="go">
            <div class="pk" @click="Pk"  v-if="own.length!=0">PK</div>
          </transition>
          <div class="send_poker_box" v-if="gameOpen" @click="SendPoker" :style="{backgroundImage:'url('+sendImg+')'}"><span>剩余:{{pokerTotal}}</span></div>
          <transition-group name="pokers" tag="ul" class="own_pokers">
            <li v-for="(item,index) in own" :key="item">
              <img :src="'/img/'+item+'.jpg'" alt="" class="ownpokerImg" :class="{['ownpokerImg'+index]:true,ownpoker_pk:pokerPk}">
            </li>
          </transition-group>
          <transition-group name="pokers" tag="ul" class="opp_pokers">
            <li v-for="(item,index) in opp" :key="item">
              <img :src="'/img/'+item+'.jpg'" alt="" class="opppokerImg" :class="{['opppokerImg'+index]:true,opppoker_pk:pokerPk}">
            </li>
          </transition-group>
        </div>
      </transition>
    </div>
  </div>
</template>

<script>
import sendImg from '@/assets/send.png'
import doubleImg from '@/assets/double.png'
import doubleImg2 from '@/assets/double2.png'
import coinImg from '@/assets/coin.jpg'
export default {
  name: 'App',
  data(){
    return {
      sendImg,
      doubleImg,
      doubleImg2,
      coinImg,
      id:'',
      ownName:'',
      oppName:'',
      time:'...',
      hasId:false,
      user:Math.random()*10000,
      alertState:false,
      isStart:false,
      gameOpen:false,
      alertVal:'',
      pokerTotal:54,
      own:[],   //我方的牌
      opp:[],   //对方的牌
      startVal:'开始游戏',
      winState:false,
      win:0,    //0输了  1赢了  2平局
      pokerPk:false,
      double:1,
      doubleCoin:200,
      ownPk:false,
      oppPk:false,
      ownCoin:0,
      oppCoin:0,
      ownDoubleList:[],
      oppDoubleList:[],
    }
  },
  created() {
    
  },
  mounted() {

  },
  sockets:{ 
    connect(){
      console.log('连接成功')
    },
    //加入
    join(res){
      if(res.code==200){
        
        res.msg.forEach((user)=>{
          if(user==this.id){
            //自己进入房间
            this.Alert(user+'进入了房间');
            this.ownName=user;
            this.hasId=true;
          }else{
            //敌人进入房间
            this.Alert(user+'进入了房间');
            this.oppName=user;
          }
        });
        
      }else{
        this.Alert(res.msg);
      }
    },
    //离开
    leave(res){
      if(res.code==200){
        this.Alert(res.msg+'离开了房间');
        this.oppName='';
        this.gameOpen=false;
        this.own=[];
        this.opp=[];
        this.ownPk=false;
        this.oppPk=false;
        this.pokerTotal=54;
        this.time='...';
      }
    },
    //开始游戏
    start(res){
      this.startVal='游戏开始，请抽牌';
      this.isStart=res;
      this.gameOpen=res;   //控制抽牌
      this.double=1;
      this.doubleCoin=200;
      this.ownPk=false;
      this.oppPk=false;
      setTimeout(()=>{
        this.isStart=false;
      },1000);
    },
    //时间提示
    time(res){
      this.time=res;
    },
    //重新开始下一轮
    reStart(res){
      this.startVal='第'+res+'轮开始，请抽牌';
      console.log('第'+res+'轮');
      this.own=[];
      this.opp=[];
      this.double=1;
      this.doubleCoin=200;
      this.ownPk=false;
      this.oppPk=false;
      this.isStart=true;
      setTimeout(()=>{
        this.isStart=false;
      },1000);
    },
    //谁获胜
    win(res){
      setTimeout(()=>{
        this.pokerPk=true;
        this.winState=true;
        if(this.id==res){
          this.win=1;
        }else if(res==''){
          this.win=2;
        }else{
          this.win=0;
        }
        setTimeout(()=>{
          this.winState=false;
          this.win=0;
          this.pokerPk=false;
        },7500);
      },500);
    },
    //pk
    pk(res){
      res.forEach((user)=>{
        if(user==this.id){
          this.ownPk=true;
        }else{
          this.oppPk=true;
        }
      });
    },
    //加倍
    double(res){
      console.log(res);
      if(res.user==this.id){
        let index=this.ownDoubleList.length==0 ? 0 : this.ownDoubleList[this.ownDoubleList.length-1]+1;
        this.ownDoubleList.push(index);
        setTimeout(()=>{
          this.ownDoubleList.splice(this.ownDoubleList.indexOf(index),1);
        },500);
      }else{
        let index2=this.oppDoubleList.length==0 ? 0 : this.oppDoubleList[this.oppDoubleList.length-1]+1;
        this.oppDoubleList.push(index2);
        setTimeout(()=>{
          this.oppDoubleList.splice(this.oppDoubleList.indexOf(index2),1);
        },500);
      }
      this.double=res.double;
      this.doubleCoin=this.double*200;
    },
    //金钱获取
    coin(res){
      res.forEach((obj)=>{
        if(obj.user==this.id){
          this.ownCoin=obj.coin;
        }else{
          this.oppCoin=obj.coin;
        }
      });
    },
    //获取抽的牌
    sendPoker(res){
      if(res.total<=2){
        this.pokerTotal=54;
      }else{
        this.pokerTotal=res.total;
      }
      console.log(res);
      res.userPoker.forEach((obj)=>{
        if(obj.user==this.id){
          this.own=obj.poker;
        }else{
          this.opp=obj.poker;
        }
      });
    },
    //错误提示
    sendAlert(res){
      this.Alert(res);
    }
  },
  methods: {
    Pk(){
      this.$socket.emit('pk',this.id);
    },
    Go(){
      this.$socket.emit('go',this.id);
    },
    Double(){
      this.$socket.emit('double',this.id);
    },
    SendPoker(){
      this.$socket.emit('sendPoker',this.id);
    },
    IdSubmit(){
      if(this.id!=''){
        this.$socket.emit('join',this.id);
      }
    },
    Alert(val){
      this.alertState=true;
      this.alertVal=val;
      clearTimeout(this.alertTimer);
      this.alertTimer=setTimeout(()=>{
          this.alertState=false;
          this.alertVal='';
      },2000);
  },
  },
}
</script>

<style>
  @import url('./APP.css');
</style>
