<script setup>
import * as THREE from 'three'
import * as YUKA from 'yuka'
import  Stats  from "three/examples/jsm/libs/stats.module"

// import  GLTFLoader  from "three/examples/jsm/loaders/GLTFLoader.js"

import  { Octree }  from "three/examples/jsm/math/Octree.js"
// import { OctreeHelper } from "three/examples/jsm/helpers/OctreeHelper.js"

import { Capsule } from "three/examples/jsm/math/Capsule.js"

import { GUI } from "three/examples/jsm/libs/lil-gui.module.min.js"

import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js"

import { onMounted } from 'vue'
// import Stats from 'stats-js'

onMounted (()=>{
    // 创建场景
    const scene = new THREE.Scene();
    // 创建相机
    const camera = new THREE.PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight,
      0.1,
      1000
    )
    camera.position.set(0, 5, 10)

    // 创建渲染器
    const container = document.getElementById("container");
    const renderer = new THREE.WebGL1Renderer({ antialias: true })
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.VSMShadowMap;
    renderer.outputEncoding = THREE.sRGBEncoding;
    renderer.toneMapping = THREE.ACESFilmicToneMapping;
    container.appendChild(renderer.domElement)
    
    const stats = new Stats();
    stats.domElement.style.position = "absolute";
    stats.domElement.style.top = '0px';
    container.appendChild(stats.domElement);
    
    const controls = new OrbitControls(camera, renderer.domElement);
    controls.target.set(0, 0, 0);

    // 创建一个平面
    const planeGeometry = new THREE.PlaneGeometry(20, 20, 1, 1);
    const planMaterial = new THREE.MeshBasicMaterial({
      color: 0xffffff,
      side: THREE.DoubleSide
    });
    const plane = new THREE.Mesh(planeGeometry, planMaterial);
    plane.receiveShadow = true;
    plane.rotation.x = -Math.PI / 2;
    plane.position.y = -0.5;
    plane.updateMatrixWorld()
    scene.add(plane);
    //创建灯光
    const light = new THREE.SpotLight(0xffffff, 1, 100, Math.PI / 6, 0.5);
    light.position.set(10, 30, 10);
    light.castShadow = true;
    scene.add(light);

    //创建物体
    const coneGeometry = new THREE.ConeGeometry(0.2, 1, 32);
    coneGeometry.rotateX(Math.PI / 2);
    const coneMaterial = new THREE.MeshStandardMaterial({color:0xff0000});
    const cone = new THREE.Mesh(coneGeometry, coneMaterial);
    // cone.matrixAutoUpdate = false;
    cone.receiveShadow = true;
    cone.castShadow = true;
    scene.add(cone);

    // 创建YUKA的车
    const vehicle = new YUKA.Vehicle();
    vehicle.maxSpeed = 5
    // 设置渲染对象
    vehicle.setRenderComponent(cone, callback);
    function callback(entity, renderComponent){
        // renderComponent.matrix.copy(entity.worldMatrix)
        renderComponent.position.copy(entity.position);
        renderComponent.quaternion.copy(entity.rotation);
      }
    // 创建目标小球
    const sphereGeometry = new THREE.SphereGeometry(0.2, 32, 32);
    const sphereMateial = new THREE.MeshStandardMaterial({
      color: 0xff00ff
    })
    const sphere = new THREE.Mesh(sphereGeometry, sphereMateial);
    sphere.receiveShadow = true;
    sphere.castShadow = true;
    scene.add(sphere)

    // 创建目标
    const target = new YUKA.GameEntity();
    target.setRenderComponent(sphere, callback)
    target.position.set(Math.random() * 20 - 10, 0, Math.random() * 20 - 10)
  
    
    const entityManager = new YUKA.EntityManager();
    entityManager.add(vehicle)
    entityManager.add(target)

    // 到达行为
    const arriveBehavior = new YUKA.ArriveBehavior(target.position, 3);
    vehicle.steering.add(arriveBehavior);

    const ndc = new THREE.Vector2();
    const raycaster = new THREE.Raycaster();
    // projector = new THREE.Projector();
    // Mesh.updateMatrixWorld();
    // 点击前往
    window.addEventListener("pointerdown",(event)=>{
      ndc.x =( event.clientX / window.innerWidth ) * 2 - 1;
      ndc.y = -( event.clientY / window.innerHeight ) * 2 + 1;
      console.log('ndc', ndc);
      raycaster.setFromCamera(ndc, camera);
    
      // const intersects = raycaster.intersectObject(plane);
      const intersects = raycaster.intersectObjects( scene.children, true );  
      console.log(intersects)
      if(intersects.length > 0){
        const point = intersects[0].point;
        console.log(point)
        target.position.set(point.x, 0, point.z)
        // target.position.copy(point)
      }
    })

    // 障碍物品

    const obstacles = [];
    for(let i = 0; i < 5; i++) {
       const boxGeometry = new THREE.BoxGeometry(3,3,3);
       const boxMaterial = new THREE.MeshStandardMaterial({
        color: 0x00ff00
       });
       const box = new THREE.Mesh(boxGeometry, boxMaterial);
       box.position.set(Math.random() * 30 - 15, 1.5, Math.random() * 30 - 5);
       box.receiveShadow = true;
       box.castShadow = true;
       scene.add(box);
       // 创建障碍物
       const obstacle = new YUKA.GameEntity();
       obstacle.position.copy(box.position);
       // 设置障碍我半径
       boxGeometry.computeBoundingSphere();
       obstacle.boundingRadius = boxGeometry.boundingSphere.radius;
       obstacles.push(obstacle)
       entityManager.add(obstacle)

      }
    // 避障
    const obstacleAvoidanceBehavior = new YUKA.ObstacleAvoidanceBehavior(obstacles)
    vehicle.steering.add(obstacleAvoidanceBehavior)
    vehicle.smoother = new YUKA.Smoother(30)
    const time = new YUKA.Time();
    function animate() {
      const delta = time.update().getDelta();
      renderer.render(scene, camera);
      stats.update();
      controls.update();
      entityManager.update(delta);
      requestAnimationFrame(animate);
    }
    animate();

    

});


</script>

<template>
  <div id="container">
  </div>
</template>

<style scoped>
/* .logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
} */
</style>
