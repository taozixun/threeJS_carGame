<template>
    <div class="home">
      <div class="canvasContainer" ref="canvasContainer"></div>
       <div class="home-content"  ref="homeContent">
          <div class="home-content-title">
              <h2>汽车展示与选配</h2>
          </div>
          <h3>选择车型</h3>
          <div class="select">
            <div
              class="select-item"
              v-for="(item, index) in cars"
              :key="index"
              @click="selectCar(item)"
            >
              <div class="select-item-text">{{item}}</div>
            </div>
          </div>
          <h3>选择车身颜色</h3>
          <div class="select">
            <div
              class="select-item"
              v-for="(item, index) in colors"
              :key="index"
              @click="selectColor(index)"
            >
              <div
                class="select-item-color"
                :style="{ backgroundColor: item }"
              ></div>
            </div>
          </div>
    
          <h3>选择车衣材质</h3>
          <div class="select">
            <div
              class="select-item"
              v-for="(item, index) in materials"
              :key="index"
              @click="selectMaterial(index)"
            >
              <div class="select-item-text">{{ item.name }}</div>
            </div>
          </div>
    
          <h3>选择玻璃</h3>
          <div class="select">
            <div
              class="select-item"
              v-for="(item, index) in glassType"
              :key="index"
              @click="selectGlasses(index)"
            >
              <div class="select-item-text">{{ item }}</div>
            </div>
          </div>
        </div>
        <div class="goback" @click="transBack">
            {{goback}}
        </div>
    </div>
    </template>
    
    <script setup>
      import * as THREE from 'three';
      import {OrbitControls}from "three/examples/jsm/controls/OrbitControls";//控制视角移动
      import {ref,onMounted,defineEmits,onBeforeUnmount  } from "vue";
      import gsap from 'gsap';
      import {GLTFLoader} from 'three/examples/jsm/loaders/GLTFLoader';//导入模型
      import {DRACOLoader} from 'three/examples/jsm/loaders/DRACOLoader';//解压缩
      import * as dat from 'dat.gui';
      // import { Reflector } from "three/examples/jsm/objects/Reflector";
      import {RGBELoader} from 'three/examples/jsm/loaders/RGBELoader';
      const emits = defineEmits(['data-updated']);//传递数据
      const homeContent = ref(null);
      const goback = ref(" ");
      const gui =new dat.GUI();
      //加载hdr环境图
      const rgbeLoader=new RGBELoader()
      //图片太大了，设置异步加载
      rgbeLoader.loadAsync('/imgs/bg2.hdr').then((texture)=>{
          texture.mapping=THREE.EquirectangularReflectionMapping;//不然就是一张图，这个映射成一个圆柱
          scene.background=texture;
          scene.environment=texture;
      })
      let sc20;
      let bmw;
      let lexus;
      let Tsla;
      let nissan;
      let canvasContainer=ref(null);
      const scene =new THREE.Scene();
      const camera=new THREE.PerspectiveCamera(
        75,
        window.innerWidth/window.innerHeight,
        0.01,
        1000
      );
      camera.position.set(0,2,6);
      scene.add(camera);
      const renderer=new THREE.WebGLRenderer({
        antialias:true,//抗锯齿
      });
    renderer.setSize(window.innerWidth,window.innerHeight);
    //添加控制器
    const controls = new OrbitControls(camera,renderer.domElement);
    controls.maxPolarAngle = (Math.PI / 2.1);
    controls.minDistance=3.5;
    controls.maxDistance=10;
    controls.enableDamping=true;
    let animation2
    const animate=()=>{
      renderer.render(scene,camera);
      animation2=requestAnimationFrame(animate);
      controls&&controls.update();
    }
    //物体数组
    let wheels=[];
    let carBody=[];
    let frontCar=[];
    let holdCar=[];
    let glassesCar=[];
    //创建材质
    const bodyMaterial = new THREE.MeshPhysicalMaterial({
      color:0xffffff,
      metalness:1,
      roughness:0.5,
      clearcoat:1,//设置清漆
      clearcoatRoughness:0,
    })
    const frontMaterial = new THREE.MeshPhysicalMaterial({
      color:0xffffff,
      metalness:1,
      roughness:0.5,
      clearcoat:1,//设置清漆
      clearcoatRoughness:0,
    })
    const holdMaterial = new THREE.MeshPhysicalMaterial({
      color:0xffffff,
      metalness:1,
      roughness:0.5,
      clearcoat:1,//设置清漆
      clearcoatRoughness:0,
    })
    const wheelsMaterial = new THREE.MeshPhysicalMaterial({
      color:0xffffff,
      metalness:1,
      roughness:0.1,
    })
    const glassesMaterial = new THREE.MeshPhysicalMaterial({
      color:0xffffff,
      transmission:1,
      metalness:0,
      roughness:0,
      transparent:true,
    });
    let colors=[
      "red",
      "black",
      'skyblue',
      'gray',
      'pink',
      'orange',
      'white',
    ];
    let materials=[
      {
        name:"磨砂",
        value:1,
      },
      {
        name:"水晶",
        value:0,
      }
    ];
    let glassType=['普通','熏黑'];
    let cars=['bmw','sc20','lexus','Tsla','nissan'];
    const selectColor=(index)=>{
      bodyMaterial.color.set(colors[index]);
      frontMaterial.color.set(colors[index]);
      holdMaterial.color.set(colors[index]);
      wheelsMaterial.color.set(colors[index]);
      // glassesMaterial.color.set(colors[index]);
    }
    const selectMaterial=(index)=>{
      bodyMaterial.clearcoatRoughness=materials[index].value;
      frontMaterial.clearcoatRoughness=materials[index].value;
      holdMaterial.clearcoatRoughness=materials[index].value;
      bodyMaterial.roughness=materials[index].value===1?1:0.5;
      frontMaterial.roughness=materials[index].value===1?1:0.5;
      holdMaterial.roughness=materials[index].value===1?1:0.5;
    }
    const selectGlasses=(index)=>{
      if(index===1){
        glassesMaterial.color.set('black');
      }else{
        glassesMaterial.color.set('white');
      }
      
    }
    let selectCar;
    // 使用补间动画移动相机
    let timeLine1 = gsap.timeline();
    let timeline2 = gsap.timeline();
    let isWork=false;
    // 定义相机移动函数
    function transBack(){
      const sendDataToParent = () => {
        emits('data-updated', "quit");
      };
      sendDataToParent()
      timeline2.to(controls.target, {
        x: 0,
        y: 0,
        z: 0,
        duration: 1,
        ease: "power2.inOut",
      });
      timeLine1.to(camera.position, {
        x: 0,
        y: 2,
        z: 6,
        duration: 1,
        ease: "power2.inOut",
      });
      controls.enableZoom = true; //控制器禁用缩放
      homeContent.value.style.display='';
      goback.value=' ';
      isWork=false;
    }
    function translateCamera(position,target) {
      const sendDataToParent = () => {
        emits('data-updated', "table");
      };
      sendDataToParent()
      timeline2.to(controls.target, {
        x: target.x,
        y: target.y,
        z: target.z,
        duration: 1,
        ease: "power2.inOut",
      });
      timeLine1.to(camera.position, {
        x: position.x,
        y: position.y,
        z: position.z,
        duration: 1,
        ease: "power2.inOut",
      });
      controls.enableZoom = false; //控制器禁用缩放
      homeContent.value.style.display='none';
      setTimeout(()=>{
        goback.value='返回展台';
      },800)
    }
    let myModel
    onMounted(()=>{
      canvasContainer.value.appendChild(renderer.domElement);
      renderer.setClearColor('#000');
      scene.background=new THREE.Color('#ccc');
      scene.environment=new THREE.Color('#ccc');
      animate();
    
      // //添加网格地面
      // const gridHelper=new THREE.GridHelper(20,20);
      // gridHelper.material.opacity=0.5;
      // gridHelper.material.transparent=true;
      // scene.add(gridHelper)
      // //添加圆柱展示台
      // const CylinderGeometry = new THREE.CylinderGeometry( 4, 4, 1, 32 );
      // const CylinderMaterial = new THREE.MeshBasicMaterial( {color: 0xffff00} );
      // const cylinder = new THREE.Mesh( CylinderGeometry, CylinderMaterial );
      // scene.add( cylinder );
      // // 添加镜面反射展台
      // let reflectorGeometry = new THREE.CircleGeometry(4.1,32);
      // let reflectorPlane = new Reflector(reflectorGeometry, {
      //   textureWidth: window.innerWidth/3,
      //   textureHeight:window.innerHeight/3,
      //   color: 0x332222,
      // });
      // reflectorPlane.rotation.x = -Math.PI / 2;
      // reflectorPlane
      // scene.add(reflectorPlane);
      // 添加视频光阵
      let video = document.createElement("video");
      video.src = "./video/zp2.mp4";
      video.loop = true;
      video.muted = true;//只有静音才能自动播放，执行下一条语句
      video.play();
      let videoTexture = new THREE.VideoTexture(video);//加载视频材质要先创建好视频的dom
      const videoGeoPlane = new THREE.PlaneBufferGeometry(16, 9);
      const videoMaterial = new THREE.MeshBasicMaterial({
        map: videoTexture,
        alphaMap: videoTexture,//黑色部分透明
        transparent: true,
      });
      const videoMesh = new THREE.Mesh(videoGeoPlane, videoMaterial);
      videoMesh.position.set(0, 0.01, 0);
      videoMesh.rotation.set(-Math.PI / 2, 0, 0);
      scene.add(videoMesh);
    
      //添加地面
      const textureLoader=new THREE.TextureLoader();
      const floorTexture=textureLoader.load('./imgs/floor.jpeg');
      const planeMaterial =new THREE.MeshBasicMaterial({
        map:floorTexture,
        roughness:1,
        metalness: 0, 
        side:THREE.DoubleSide,//每个面同时渲染正面和背面
      });
      const planeGeometry=new THREE.PlaneBufferGeometry(50,50,1,1);//宽高10，宽高分为1个小区间 
      const plane=new THREE.Mesh(planeGeometry,planeMaterial);
      plane.rotation.x = Math.PI / 2;
      plane.position.set(0,-0.1,0);
      scene.add(plane);
    
    
      const loader=new GLTFLoader();
      const dracoLoader=new DRACOLoader();
      dracoLoader.setDecoderPath("./draco/gltf/");
      loader.setDRACOLoader(dracoLoader);
    
    
      //添加场景模型
      loader.load('./model/workspace.glb',(gltf)=>{
          let workspace=gltf.scene;
          workspace.position.set(-5,0,-3);
          workspace.rotation.set(0,Math.PI/2,0);
          workspace.scale.set(1.6, 1.6, 1.6);
          scene.add(workspace);
          //给workspace绑定点击事件
          const raycaster = new THREE.Raycaster();// 创建一个 Raycaster 对象
          window.addEventListener('click', onMouseClick, false);//第三个参数默认false代表冒泡，true代表捕获，冒泡的话，首先在触发元素上触发，再向上传递
    
          function onMouseClick(event) {
            // 根据点击位置计算鼠标的归一化设备坐标
            const mouse = new THREE.Vector2();
            mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
            mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
    
            // 更新射线的起点和方向
            raycaster.setFromCamera(mouse, camera);
    
            // 进行射线检测，检测与模型的交互
            const intersects = raycaster.intersectObject(workspace, true);
    
            // 如果射线与模型相交
            if (intersects.length > 0&&!isWork) {
              isWork=true;
              // 执行点击事件的操作
              translateCamera(
                new THREE.Vector3(-1,2,-3),
                new THREE.Vector3(-5,0,-3.35)
              );
            }
          }
      })
      loader.load('./model/ws2.glb',(gltf)=>{
          let ws2=gltf.scene;
          ws2.position.set(-7,0,8);
          ws2.rotation.set(0,Math.PI/2,0);
          ws2.scale.set(2, 2, 2);
          scene.add(ws2);
      })
      loader.load('./model/轿车03.gltf',(gltf)=>{
          let ws2=gltf.scene;
          ws2.position.set(15,0,10);
          ws2.rotation.set(0,Math.PI,0);
          ws2.scale.set(1.5, 1.5, 1.5);
          scene.add(ws2);
      })
    
      //添加汽车模型
      let loaderArr=[];
      let carNameArr=[];
      let carObjArr=[];
      let nowChoose='sc20';
      const f1=()=>{
        for(let i=0;i<carNameArr.length;i++){
          if(carObjArr[i]){
            scene.remove(carObjArr[i]);
            carObjArr[i]=null;
            break;
          }
        }
          loader.load('./model/sc20.glb',(gltf)=>{
          sc20=gltf.scene;
          sc20.scale.set(1.3, 1.3, 1.3);
          carObjArr[carNameArr.indexOf('sc20')]=sc20;
          sc20.traverse((child)=>{//traverse(three.js)遍历所有子对象及孙，孙孙对象。。。。
            if(child.isMesh){
              // console.log(child.name);
              if(child.name.includes('Wheel_5')){
                wheels.push(child);
                child.material=wheelsMaterial;
              }
              if(child.name.includes('Door')){
                carBody.push(child);
                child.material=bodyMaterial;
              }
              if(child.name.includes('Bodyshell')){
                frontCar.push(child);
                child.material=frontMaterial;
              }
              if(child.name.includes('Interior_Zone')){
                holdCar.push(child);
                child.material=holdMaterial;
              }
              if(child.name.includes('Glass')){
                glassesCar.push(child);
                child.material=glassesMaterial;
              }
              if(child.name.includes('Bodyshell_1')){
                child.material=new THREE.MeshPhysicalMaterial({
                  color:0x000000,
                  roughness:1,
                })
              }
              if(child.name.includes('Wheel_4')){
                child.material=new THREE.MeshPhysicalMaterial({
                  color:0x000000,
                  roughness:1,
                })
              }
            }
          })
          scene.add(sc20);
          myModel=sc20;
        })
      }
      loaderArr.push(f1);
      carNameArr.push('sc20');
      carObjArr.push(sc20);
      const f2=()=>{
        for(let i=0;i<carNameArr.length;i++){
          if(carObjArr[i]){
            scene.remove(carObjArr[i]);
            carObjArr[i]=null;
            break;
          }
        }
        loader.load('./model/bmw01.glb',(gltf)=>{
        bmw=gltf.scene;
        bmw.scale.set(1.52, 1.52, 1.52);
        carObjArr[carNameArr.indexOf('bmw')]=bmw;
        bmw.traverse((child)=>{//traverse(three.js)遍历所有子对象及孙，孙孙对象。。。。
          if(child.isMesh){
            // console.log(child.name)
            if(child.name.includes('轮毂')){
              wheels.push(child);
              child.material=wheelsMaterial;
            }
            if(child.name.includes('Mesh002')){
              carBody.push(child);
              child.material=bodyMaterial;
            }
            if(child.name.includes('前脸')){
              frontCar.push(child);
              child.material=frontMaterial;
            }
            if(child.name.includes('引擎盖_1')){
              holdCar.push(child);
              child.material=holdMaterial;
            }
            if(child.name.includes('挡风玻璃')){
              glassesCar.push(child);
              child.material=glassesMaterial;
            }
          }
        })
        scene.add(bmw);
        myModel=bmw;
      })
    
      }
      loaderArr.push(f2);
      carNameArr.push('bmw');
      carObjArr.push(bmw);
      const f3=()=>{
        for(let i=0;i<carNameArr.length;i++){
          if(carObjArr[i]){
            scene.remove(carObjArr[i]);
            carObjArr[i]=null;
            break;
          }
        }
        loader.load('./model/lecius.glb',(gltf)=>{
        lexus=gltf.scene;
        lexus.scale.set(0.5, 0.5, 0.5);
        carObjArr[carNameArr.indexOf('lexus')]=lexus;
        lexus.traverse((child)=>{//traverse(three.js)遍历所有子对象及孙，孙孙对象。。。。
          if(child.isMesh){
            // console.log(child.name)
            if(child.name.includes('2017_Lexus_LC_500_b_0')){
              wheels.push(child);
              child.material=wheelsMaterial;
            }
            if(child.name.includes('body')){
              carBody.push(child);
              child.material=bodyMaterial;
            }
            if(child.name.includes('Color_A')){
              glassesCar.push(child);
              child.material=glassesMaterial;
            }
          }
        })
        scene.add(lexus);
        myModel=lexus;
      })
    
      }
      loaderArr.push(f3);
      carNameArr.push('lexus');
      carObjArr.push(lexus);
      const f4=()=>{
        for(let i=0;i<carNameArr.length;i++){
          if(carObjArr[i]){
            scene.remove(carObjArr[i]);
            carObjArr[i]=null;
            break;
          }
        }
        loader.load('./model/tesla.glb',(gltf)=>{
        Tsla=gltf.scene;
        Tsla.position.set(0,1.34,0);
        Tsla.scale.set(2, 2, 2);
        carObjArr[carNameArr.indexOf('Tsla')]=Tsla;
        Tsla.traverse((child)=>{//traverse(three.js)遍历所有子对象及孙，孙孙对象。。。。
          if(child.isMesh){
            console.log(child.name)
            if((child.name.includes('Cylinder')&&child.name.includes('_2'))||child.name==='Cylinder004_1'){
              // child.material=new THREE.MeshPhysicalMaterial({
              //     color:0x00ff00,
              //     roughness:1,
              //   })
              wheels.push(child);
              child.material=wheelsMaterial;
            }
            if(child.name==='Cylinder001_1'||child.name==='Cylinder002_1'||child.name==='Cylinder003_1'||child.name==='Cylinder004'){
              child.material=new THREE.MeshPhysicalMaterial({
                  color:0x000000,
                  roughness:1,
                })
            }
            if(child.name.includes('Cube001_3')){
                  child.material=new THREE.MeshPhysicalMaterial({
                  color:0xff0000,
                  roughness:0,
                })
            }
            if(child.name==='Cube001'){
              child.material=bodyMaterial;
            }
            if(child.name.includes('Cube001_1')){
              glassesCar.push(child);
              child.material=glassesMaterial;
            }
          }
        })
        scene.add(Tsla);
        myModel=Tsla;
      })
    
      }
      loaderArr.push(f4);
      carNameArr.push('Tsla');
      carObjArr.push(Tsla);
      const f5=()=>{
        for(let i=0;i<carNameArr.length;i++){
          if(carObjArr[i]){
            scene.remove(carObjArr[i]);
            carObjArr[i]=null;
            break;
          }
        }
        loader.load('./model/nissan.glb',(gltf)=>{
          nissan=gltf.scene;
          nissan.scale.set(1.5, 1.5, 1.5);
        carObjArr[carNameArr.indexOf('nissan')]=nissan;
        nissan.traverse((child)=>{//traverse(three.js)遍历所有子对象及孙，孙孙对象。。。。
          if(child.isMesh){
            console.log(child.name)
            if(child.name.includes('_1')){
              child.material=bodyMaterial;
            }
            if(child.name==='Plane006'){
              child.material=bodyMaterial;
            }
            if(child.name.includes('Plane006_1')||child.name.includes('Plane006_3')){
              glassesCar.push(child);
              child.material=glassesMaterial;
            }
          }
        })
        scene.add(nissan);
        myModel=nissan;
      })
    
      }
      loaderArr.push(f5);
      carNameArr.push('nissan');
      carObjArr.push(nissan);
      loaderArr[0]();
      selectCar=(item)=>{
        if(item===nowChoose){
          return;
        }
        nowChoose=item;
        loaderArr[carNameArr.indexOf(item)]();
      }
      //聚光灯光源
      const spotLight = new THREE.SpotLight(0xffffff, 0.5);
      spotLight.position.set(5, 5, 5);
      spotLight.castShadow = true;
      spotLight.shadow.radius=20;
      spotLight.shadow.mapSize.set(1024,1024);//阴影贴图分辨率
      spotLight.angle=Math.PI/6;
      spotLight.distance=0;//设置灯光衰减到0的距离，线性计算光强
      spotLight.penumbra=0;//光边缘虚化
      spotLight.decay=0;//沿光照距离的衰减量，2为现实世界，要事先设置renderer.physicallyCorrectLights为true
      scene.add(spotLight);
      gui.add(spotLight.position,"x").min(-10).max(10).step(0.1)
      gui.add(spotLight.position,"y").min(0).max(20).step(0.1)
      gui.add(spotLight.position,"z").min(-10).max(10).step(0.1)
    
    
      window.addEventListener("resize",()=>{
        renderer.setSize(window.innerWidth,window.innerHeight);
        camera.aspect=window.innerWidth/window.innerHeight;
        camera.updateProjectionMatrix();
        renderer.setPixelRatio(window.devicePixelRatio);
      });
    });
    onBeforeUnmount(()=>{
      console.log('组件销毁了')
      console.log(myModel)
      // 向父组件传递数据
      const sendDataToParent = () => {
        emits('data-updated', myModel);
      };
      sendDataToParent()
      //销毁gui
      gui.destroy();
      cancelAnimationFrame(animation2);
    })
    </script>
    
    <style>
    *{
      margin: 0;
      padding: 0;
    }
    
    .home-content {
      position: fixed;
      top: 0;
      left: 20px;
    }
    
    .select-item-color {
      width: 20px;
      height: 20px;
      border: 1px solid #ccc;
      margin: 5px;
      display: inline-block;
      cursor: pointer;
      border-radius: 10px;
    }
    .select {
      display: flex;
    }
    .select-item-text{
      margin-right: 5px;
      cursor: pointer;
    }
    .goback{
      cursor: pointer;
      position: fixed;
      top: 0;
      left: 20px;
    }
    </style>
    