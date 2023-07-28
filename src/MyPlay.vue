<template>
  <div class="container" ref="container">
    <div class="restart" @click="resetGame">重新开始</div>
    <div class="message" ></div>
    <div class="score" >您的得分为：{{ score }}</div>
    <div id="overlay" v-if="showDiv">
      <div class="restart" @click="resetGame">重新开始</div>
    </div>
    <!-- <div id="video-container">
      <canvas id="video-canvas"></canvas>
    </div> -->
  </div>
</template>

<script setup>
import * as THREE from 'three';
//toRaw把Proxy(Group)数据类型还原回Group数据类型
import {ref,onMounted ,defineProps, toRaw,onBeforeUnmount} from "vue";
import {OrbitControls}from "three/examples/jsm/controls/OrbitControls";//控制视角移动
import {GLTFLoader} from 'three/examples/jsm/loaders/GLTFLoader';//导入模型
import {DRACOLoader} from 'three/examples/jsm/loaders/DRACOLoader';//解压缩
import {RGBELoader} from 'three/examples/jsm/loaders/RGBELoader';
import gsap from 'gsap';

//接收父组件的传参
const props = defineProps(['model']);
console.log("新的组件里")
console.log(toRaw(props.model))
const model=toRaw(props.model)
const scene=new THREE.Scene();
const showDiv = ref(false);
const container=ref(null);
const score=ref(0);
let resetGame
const camera=new THREE.PerspectiveCamera(
  45,
  window.innerWidth/window.innerHeight,
  0.01,
  1000,
);
camera.position.set(5,5,5);
scene.add(camera);
const renderer=new THREE.WebGLRenderer({
  antialias:true,
})
renderer.setSize(window.innerWidth,window.innerHeight);
const controls = new OrbitControls(camera,renderer.domElement);
controls.enableDamping=true;
const render=()=>{
  renderer.render(scene,camera);
  requestAnimationFrame(render);
  controls&&controls.update();
};
//加载hdr环境图
const rgbeLoader=new RGBELoader()
//图片太大了，设置异步加载
rgbeLoader.loadAsync('/imgs/bg2.hdr').then((texture)=>{
    texture.mapping=THREE.EquirectangularReflectionMapping;//不然就是一张图，这个映射成一个圆柱
    scene.background=texture;
    scene.environment=texture;
})

//添加平面
const textureLoader=new THREE.TextureLoader();
const floorTexture=textureLoader.load('./imgs/roud.jpg');
const planeMaterial =new THREE.MeshBasicMaterial({
  map:floorTexture,
});
const planeGeometry=new THREE.PlaneBufferGeometry(130,130,1,1);//宽高10，宽高分为1个小区间 
const plane=new THREE.Mesh(planeGeometry,planeMaterial);
plane.rotation.x = -Math.PI / 2;
plane.position.set(0,-0.1,0);
scene.add(plane);
//添加碰撞几何体
const boxGeometry=new THREE.BoxBufferGeometry(26,5,26,1,1,1);
const boxMetarial=new THREE.MeshBasicMaterial({
  color:0xffffff,
  transparent: true, 
  opacity: 0 ,
})
const box1=new THREE.Mesh(boxGeometry,boxMetarial);
const box2=new THREE.Mesh(boxGeometry,boxMetarial);
const box3=new THREE.Mesh(boxGeometry,boxMetarial);
const box4=new THREE.Mesh(boxGeometry,boxMetarial);
const box5=new THREE.Mesh(boxGeometry,boxMetarial);
const box6=new THREE.Mesh(boxGeometry,boxMetarial);
const box7=new THREE.Mesh(boxGeometry,boxMetarial);
box2.position.x+=36;
box3.position.x-=36;
box4.position.x+=36;
box4.position.z+=36;
box5.position.x-=36;
box5.position.z+=36;
box6.position.z+=36;
box7.position.z-=35;
box7.position.x+=0.5;
const box_1 = new THREE.Box3().setFromObject(box1);
const box_2 = new THREE.Box3().setFromObject(box2);
const box_3 = new THREE.Box3().setFromObject(box3);
const box_4 = new THREE.Box3().setFromObject(box4);
const box_5 = new THREE.Box3().setFromObject(box5);
const box_6 = new THREE.Box3().setFromObject(box6);
const box_7 = new THREE.Box3().setFromObject(box7);
// const boxHelper_1 = new THREE.Box3Helper(box_1, 0xff0000); // 使用 Box3Helper 创建一个可视化的包围盒辅助器
// const boxHelper_2 = new THREE.Box3Helper(box_2, 0xff0000);
// const boxHelper_3 = new THREE.Box3Helper(box_3, 0xff0000);
// const boxHelper_4 = new THREE.Box3Helper(box_4, 0xff0000);
// const boxHelper_5 = new THREE.Box3Helper(box_5, 0xff0000);
// const boxHelper_6 = new THREE.Box3Helper(box_6, 0xff0000);
// const boxHelper_7 = new THREE.Box3Helper(box_7, 0xff0000);
// scene.add(boxHelper_1);
// scene.add(boxHelper_2);
// scene.add(boxHelper_4);
// scene.add(boxHelper_3);
// scene.add(boxHelper_5);
// scene.add(boxHelper_6);
// scene.add(boxHelper_7);

//随机刷新金币位置
let box_corn
const getCoinPosition=(corn)=>{
  let x;
  let y;
  let flag=true;
  while(flag){
    x=Math.random()*126-63;
    y=Math.random()*126-63;
    if(x>=-14&&x<=14&&y>=-14&&y<=-14){
      continue
    }
    if(x>=22&&x<=50&&y>=-14&&y<=-14){
      continue
    }
    if(x>=-50&&x<=-22&&y>=-14&&y<=-14){
      continue
    }
    if(x>=-14&&x<=14&&y>=-50&&y<=-22){
      continue
    }
    if(x>=-14&&x<=14&&y>=22&&y<=50){
      continue
    }
    if(x>=-50&&x<=-22&&y>=22&&y<=50){
      continue
    }
    if(x>=22&&x<=50&&y>=22&&y<=50){
      continue
    }
    break
  }
  console.log(x,y)
  corn.position.set(x,1.5,y)
  box_corn = new THREE.Box3().setFromObject(corn);
  
}
let animation1;
onMounted(()=>{
  // // 获取视频元素和Canvas元素
  // const video = document.createElement('video');
  // video.src = '../public//video/bom.mp4';
  // video.autoplay = true;
  // video.loop = true;
  // const canvas = document.getElementById('video-canvas');
  // const context = canvas.getContext('2d');
  // // 当视频准备就绪时，开始绘制到Canvas
  // video.addEventListener('loadedmetadata', () => {
  //   canvas.width = video.videoWidth;
  //   canvas.height = video.videoHeight;

  //   function drawVideoFrame() {
  //     context.clearRect(0, 0, canvas.width, canvas.height);
  //     context.globalAlpha = 0.5; // 设置半透明度
  //     context.drawImage(video, 0, 0, canvas.width, canvas.height);
  //     requestAnimationFrame(drawVideoFrame);
  //   }
  //   drawVideoFrame();
  // });
  // // 将Canvas覆盖在页面上的其他元素之上
  // const videoContainer = document.getElementById('video-container');
  // videoContainer.style.backgroundColor = 'rgba(0, 0, 0, 0.5)';

  render();
  container.value.appendChild(renderer.domElement);
  const loader=new GLTFLoader();
  const dracoLoader=new DRACOLoader();
  dracoLoader.setDecoderPath("./draco/gltf/");
  loader.setDRACOLoader(dracoLoader);
  let corn
  loader.load('./model/enviro.glb',(gltf)=>{
      let enviro=gltf.scene;
      enviro.rotation.set(0,Math.PI/2,0);
      enviro.scale.set(2.1, 2.1, 2.1);
      scene.add(enviro);
  })
  loader.load('./model/tree1.gltf',(gltf)=>{
      let tree1=gltf.scene;
      tree1.scale.set(2.1, 2.1, 2.1);
      let tree2=tree1.clone();
      let tree3=tree1.clone();
      let tree4=tree1.clone();
      tree1.position.set(13,0,13);
      scene.add(tree1);
      tree2.position.set(-13,0,13);
      scene.add(tree2);
      tree3.position.set(13,0,-13);
      scene.add(tree3);
      tree4.position.set(-13,0,-13);
      scene.add(tree4);
  })
  loader.load('./model/corn.glb',(gltf)=>{
      corn=gltf.scene;
      corn.position.set(0, 1.5, 0);
      corn.scale.set(2,2,2);
      scene.add(corn);
      getCoinPosition(corn);
      
      
    //添加车辆相关
      let height
      if(model.children[0].name==='Light'){//是特斯拉
        height=1.39
      }else{
        height=0.05
      }
    model.position.set(16,height,13);
    scene.add(model);
    //实现车辆移动
    window.addEventListener('keydown', onKeyPress, false);
    window.addEventListener('keyup', onKeyRelease, false);
    const moveDistance = 0.17; // 每次移动的距离
    const turnAngle = Math.PI / 16; // 每次转弯的角度
    const movement = {
      up: false,
      down: false,
      left: false,
      right: false
    };
    const box_car = new THREE.Box3();
    // const boxHelper_car = new THREE.Box3Helper(box_car, 0xff0000);
    // scene.add(boxHelper_car);

    //重置游戏
    resetGame=()=>{
      cancelAnimationFrame(animation1);
      movement.up=false;
      movement.down=false;
      movement.left=false;
      movement.right=false;
      model.position.set(16,height,15);
      model.rotation.set(0,0,0);
      box_car.setFromObject(model);
      setTimeout(()=>{
        animate();
      },100)
      setTimeout(() => {
        showDiv.value = false;
      }, 50);
      getCoinPosition(corn);
      score.value=0;
    }
    function onKeyPress(event) {
      switch (event.keyCode) {
        case 37: // 左箭头键
          movement.left = true;
          break;
        case 40: // 下箭头键
          movement.up = true;
          break;
        case 39: // 右箭头键
          movement.right = true;
          break;
        case 38: // 上箭头键
          movement.down = true;
          break;
      }
    }
    function onKeyRelease(event) {
      switch (event.keyCode) {
        case 37: // 左箭头键
          movement.left = false;
          break;
        case 40: // 上箭头键
          movement.up = false;
          break;
        case 39: // 右箭头键
          movement.right = false;
          break;
        case 38: // 下箭头键
          movement.down = false;
          break;
      }
    }
    let speed=1;
    let relativeCameraOffset;//根据车辆设定响应偏移距离
    console.log(model.children[0].name)

    if(model.children[0].name==='Sketchfab_model'){
      //是lexus
      relativeCameraOffset = new THREE.Vector3(0, 6, -16);
    }else{
      relativeCameraOffset = new THREE.Vector3(0, 1.8, -5.5); // 根据需求设置摄像机的相对位置
    }
    // 在动画循环中更新车辆模型的位置和旋转
    function animate() {
      corn.rotation.y+=0.02;//设置金币旋转
      let rotationTween = null;
      if((movement.left===movement.right&&movement.left===true)||(movement.up===movement.down&&movement.up===true)){
        alert('请不要同时按住左右||前后')
        movement.left=false;
        movement.right=false;
        movement.up=false;
        movement.down=false;
      }
      // 根据键盘事件更新车辆模型的位置和旋转
      if ((movement.right&&!rotationTween&&movement.up===true)||(movement.left&&!rotationTween&&movement.down===true)) {
        // 向左转弯
        rotationTween = gsap.to(model.rotation, {
          y: "+=" + turnAngle,
          duration: 0.3,
          onComplete: () => {
            rotationTween.kill();
            rotationTween = null; // 动画完成后重置 rotationTween
          }
        });
      }
      if (movement.up) {
        // 向后移动
        const forward = new THREE.Vector3(0, 0, -1*speed);
        forward.applyQuaternion(model.quaternion);
        model.position.add(forward.multiplyScalar(moveDistance));
      }
      if ((movement.left&&!rotationTween&&movement.up===true)||(movement.right&&!rotationTween&&movement.down===true)) {
        // 向左转弯
        rotationTween = gsap.to(model.rotation, {
          y: "-=" + turnAngle,
          duration: 0.3,
          onComplete: () => {
            rotationTween.kill();
            rotationTween = null; // 动画完成后重置 rotationTween
          }
        });
      }
      if (movement.down) {
        // 向前移动
        const backward = new THREE.Vector3(0, 0, speed);
        backward.applyQuaternion(model.quaternion);//方向与车头朝向相适应
        model.position.add(backward.multiplyScalar(moveDistance));
      }

      //摄像头跟随车尾
      const cameraOffset = relativeCameraOffset.clone().applyMatrix4(model.matrixWorld);//相对位置转换为全局位置
      camera.position.copy(cameraOffset);
      const offset = new THREE.Vector3(0, 2, 0);
      const targetPosition = new THREE.Vector3();
      targetPosition.copy(model.position).add(offset);
      // 位置摆正,注释model上两个单位
      camera.lookAt(targetPosition);

      // 在动画中更新car的包围框，检测是否发生碰撞
      box_car.setFromObject(model);
      if (box_car.intersectsBox(box_1)||box_car.intersectsBox(box_2)||box_car.intersectsBox(box_3)||box_car.intersectsBox(box_4)||box_car.intersectsBox(box_5)||box_car.intersectsBox(box_6)||box_car.intersectsBox(box_7)) {
        // 发生碰撞
        if(speed>0.01){
          document.getElementsByClassName('message')[0].innerHTML='请尽快远离障碍物！！！';
          speed=0.01;
        }
      } else{
        if(speed<1){
          document.getElementsByClassName('message')[0].innerHTML='';
          speed=1;
        }
      }
      //检测是否吃到金币
      if (box_car.intersectsBox(box_corn)){
        score.value++;
        getCoinPosition(corn);
      }
      renderer.render(scene, camera);
      animation1=requestAnimationFrame(animate);
      //检测是否运行到边界
      if(Math.abs(model.position.x)>63||Math.abs(model.position.z)>63){
        showDiv.value = true;
        cancelAnimationFrame(animation1);
      }
    }
    animate();
  });

})
window.addEventListener("resize",()=>{
    renderer.setSize(window.innerWidth,window.innerHeight);
    camera.aspect=window.innerWidth/window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setPixelRatio(window.devicePixelRatio);
  });
  onBeforeUnmount(()=>{
      cancelAnimationFrame(animation1);
    })
</script>

<style>
*{
  margin: 0;
  padding: 0;
}
.restart{
  position: fixed;
  top: 0;
  left: 0;
  width: 200px;
  line-height: 30px;
  color:darkslateblue;
  cursor: pointer;
  padding-left: 20px;
  padding-top: 10px;
}
.restart:hover{
  color:skyblue;
}
.message{
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
  font-size: 30px;
  color: crimson;
}
.score{
  position: fixed;
  top: 0;
  right: 20px;
  border-radius: 8px;
  background: skyblue;
  width: 150px;
  line-height: 30px;
  color:darkslateblue;
  cursor: pointer;
  padding-left: 20px;
}
#overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(139, 0, 0, 0.5);
  z-index: 9999;
}
#overlay>.restart{
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 40px;
}
/* #video-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 9999;
}

#video-canvas {
  width: 100%;
  height: 100%;
} */
</style>
