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
    // 创建矩阵
    const path = new YUKA.Path();
    path.add(new YUKA.Vector3(0, 0, 0));
    path.add(new YUKA.Vector3(0, 0, 10));
    path.add(new YUKA.Vector3(10, 0, 10));
    path.add(new YUKA.Vector3(10, 0, 0));
    path.add(new YUKA.Vector3(0, 0, 0));
    // 设置循环模式
    path.loop = true
    // 将路径当前的位置设置为车辆位置
    vehicle.position.copy(path.current());

    // 路径跟随
    const follw = new YUKA.FollowPathBehavior(path)
    vehicle.steering.add(follw)
    
    // 保持路径中的行为
    const onPathBehavior = new YUKA.OnPathBehavior(path, 0.1);
    onPathBehavior.weight = 10;
    vehicle.steering.add(onPathBehavior)
     
    // 创建对实体管理的对象
    const entityManager = new YUKA.EntityManager();
    entityManager.add(vehicle)

    showPath(path)
    function showPath(path){
      console.log(path)
      const positions = [];
      for (let i = 0; i<path._waypoints.length; i++) {
        positions.push(path._waypoints[i].x, path._waypoints[i].y, path._waypoints[i].z)
      }
      const geometry = new THREE.BufferGeometry();
      geometry.setAttribute(
        "position",
        new THREE.Float32BufferAttribute(positions, 3)
      );
      const material = new THREE.LineBasicMaterial({
        color:0x0000ff
      });
      const line = new THREE.Line(geometry, material)
      scene.add(line)
    }

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
