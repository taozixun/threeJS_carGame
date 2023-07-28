<template>
  <a-spin tip="Loading..." :spinning="isSpin">
    <div class="total_container">
      <audio ref="myAudio" :src="musicSRC"></audio>
      <div class="box">
        <button @click="playMusic">播放音乐</button>
        <button @click="loadComponent2">改装车辆</button>
        <button @click="loadComponent">进入游戏</button>
      </div>
      <div class="buttonBox" v-show="isShow">
        <a-button class="song" type="primary" shape="circle" :size="size" @click="lastMusic">
          {{ '<' }}
        </a-button>
        <a-button class="song" type="primary" shape="circle" :size="size" @click="nextMusic">
          {{ '>' }}
        </a-button>
      </div>
      <component :is="currentComponent" v-if="currentComponent" @data-updated="handleDataFromChild" :model="tzx"></component>
    </div>
  </a-spin>
</template>
  
  <script>
  import MyPlay from './MyPlay.vue';
  import MyCar from './MyCar.vue';
  import { ref } from 'vue';
  // 监听键盘按键事件
  function handleKeyDown(event) {
    // 阻止默认的键盘事件行为
    event.preventDefault();
  }
  // 添加键盘按键事件监听器
  document.addEventListener("keydown", handleKeyDown);
  export default {
    data() {
      return {
        isSpin:false,
        currentComponent: null,
        tzx:  ref(null),
        isShow:false,
        musicSRC:'/music/1.mp3',
        musicFlag:1,
        isReady:false,
      };
    },
    methods: {
      loadComponent() {
        if(!this.isReady){
          alert("请先改装车辆！")
          return
        }
        this.currentComponent=null
        // 动态加载另一个组件
        setTimeout(()=>{//设置定时是为了等this.tzx更新好了再加载新的组件
          import('./MyPlay.vue').then((module) => {
          this.currentComponent = module.default || module;
          this.isSpin=true
          setTimeout(()=>{
            this.isSpin=false
          },1500)
        });
        },0)
        
      },
      loadComponent2() {
        this.isReady=true;
        // 动态加载另一个组件
        import('./MyCar.vue').then((module) => {
          this.currentComponent = module.default || module;
          this.isSpin=true
          setTimeout(()=>{
            this.isSpin=false
          },3600)
        });
      },
      handleDataFromChild(model){
        //判断从子组件传来的数据
        if(model==='table'){//点击table
          this.isShow=true;
          return
        }
        if(model==='quit'){//离开table
          this.isShow=false;
          return
        }
        console.log(model)
        this.tzx = model;
        console.log(this.tzx)
      },
      playMusic() {
        // 获取音频元素的引用
        const audioElement = this.$refs.myAudio;
        if(audioElement.paused){
          // 播放音乐
          audioElement.play();
        }else{
          audioElement.pause();
        }
      },
      lastMusic(){
        this.musicFlag--;
        if(this.musicFlag===0){
          this.musicFlag=3;
        }
        this.musicSRC='/music/'+this.musicFlag+'.mp3';
        setTimeout(()=>{
          const audioElement = this.$refs.myAudio;
          audioElement.play();
        },0)
      },
      nextMusic(){
        this.musicFlag++;
        if(this.musicFlag===4){
          this.musicFlag=1;
        }
        this.musicSRC='/music/'+this.musicFlag+'.mp3';
        setTimeout(()=>{
          const audioElement = this.$refs.myAudio;
          audioElement.play();
        },0)
      }
    },
    components: {
      MyPlay,
      MyCar
    },
  };
  </script>
<style>
.box{
  position: absolute;
  /* display: flex;
  width: 100%;
  justify-content: flex-start; */
  margin-left: 40vw;
}
.box>button{
  margin: 5px;
  padding: 3px;
  background: skyblue;
  cursor: pointer;
}
.box>button:hover{
  background: steelblue;
}
.total_container{
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background-image: url("../public/imgs/background.jpeg");
  background-size: cover;
  background-repeat: no-repeat;
  /* color: aliceblue;
  text-align: center;  */
  }
  .buttonBox{
    position: absolute;
    display: flex;
    width: 100%;
    justify-content: space-between;
    pointer-events: none;
    margin-top: 45vh;
  }
  .buttonBox>.song{
    pointer-events:all;

  }
</style>
  