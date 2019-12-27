<template>
  <div class="talk">
    <ul class="talk_list">
      <li v-for="item in talkList" class="talk_li"><p  :class="{own_talk:item.id==id}">{{item.talk}}</p></li>
    </ul>
    <input type="text" v-model="talkVal" class="input" @keydown.enter="send">
    <div class="submit" @click="send">发送</div>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      talkVal:'',
      talkList:[

      ],
      id:Math.random()*10000
    }
  },
  sockets:{ 
    connect(){
      console.log('连接成功啦')
    },
    progress(val){
      this.talkList.push(val);
    }
  },
  watch: {
    talkList(){
      
      setTimeout(() => {
        document.getElementsByClassName('talk_li')[document.getElementsByClassName('talk_li').length-1].scrollIntoView({
          behavior: "smooth"
          
        }); 
      },1);
    }
  },
  methods: {
    send(){
      this.$socket.emit('message',{talk:this.talkVal,id:this.id});  
      this.talkVal='';
    }
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  @import url('./talk.css');
</style>
