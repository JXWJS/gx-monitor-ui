<template>
  <div id="test1" >
    <div  style="position: absolute;top: 30px; width: 10%;
  height: 200px;transform: translate(450%, 10%); ">
      <el-button-group>
        <el-button type="info"  icon="el-icon-d-arrow-left" @click="connectDevice(0)" >入口</el-button>
        <el-button type="info"  icon="el-icon-d-arrow-right" @click="show = !show">出口</el-button>
      </el-button-group>

      <transition name="slide-fade"
      enter-active-class='animate__animated animate__fadeInDown'
      leave-active-class='animate__animated animate__fadeOutUp'
      :duration="1000">
      <div v-if="show" style="color: white">
        <div class="elem" >

          <p>选择焊接类型：</p>
          <input type="radio" v-model="radioVal" value="laser" /><label for="laser">激光</label>
          <input type="radio" v-model="radioVal" value="tig" /><label for="tig">TIG</label>
          <el-button type="info"   @click="connectDevice(1)" >确认</el-button>
        </div>

      </div>
      </transition>

    </div>
  </div>
</template>

<script>

import * as THREE from 'three';

import Stats from 'three/examples/jsm/libs/stats.module.js';

import { FirstPersonControls } from 'three/examples/jsm/controls/FirstPersonControls.js';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { FontLoader } from 'three/examples/jsm/loaders/FontLoader.js';
import { TextGeometry } from 'three/examples/jsm/geometries/TextGeometry.js';
import { ShadowMapViewer } from 'three/examples/jsm/utils/ShadowMapViewer.js';


import { RGBELoader  } from 'three/examples/jsm/loaders/RGBELoader.js'
import axios from "axios";


const SHADOW_MAP_WIDTH = 2048, SHADOW_MAP_HEIGHT = 1024;

let SCREEN_WIDTH = window.innerWidth;
let SCREEN_HEIGHT = window.innerHeight;
const FLOOR = - 250;

let camera, controls, scene;
let container, stats;

const NEAR = 10, FAR = 3000;

let mixer;

const morphs = [];

let light;
let lightShadowMapViewer;

const clock = new THREE.Clock();

let showHUD = false;

export default( {
  name: 'Start',

  data () {
    return {
      camera: null,
      scene: null,
      renderer: null,
      controls: null,
      container: null,
      myChart: null,
      show: false,
      radioVal: 'laser'
    }
  },
  mounted () {
    this.init()
    this.animate()
    this.$nextTick( () => {
      document.getElementById('test1').appendChild(this.container)
    })
  },
  methods: {
    init() {
      let this_ = this

      this_.container = document.createElement('div');

      // CAMERA

      camera = new THREE.PerspectiveCamera(23, SCREEN_WIDTH / SCREEN_HEIGHT, NEAR, FAR);
      camera.position.set(700, 50, 1900);

      // SCENE

      scene = new THREE.Scene();
      // scene.background = new THREE.Color(0x121112);
      new RGBELoader()
          .setPath( '~@/../static/model/' )
          .load( 'royal_esplanade_1k.hdr', function ( texture ) {
                texture.mapping = THREE.EquirectangularReflectionMapping;

                scene.background = texture;
                scene.environment = texture; })



      scene.fog = new THREE.Fog(0x232323, 1000, FAR);

      // LIGHTS

      const ambient = new THREE.AmbientLight(0x444444);
      scene.add(ambient);

      light = new THREE.SpotLight(0xffffff, 1, 0, Math.PI / 5, 0.3);
      light.position.set(0, 1500, 1000);
      light.target.position.set(0, 0, 0);

      light.castShadow = true;
      light.shadow.camera.near = 1200;
      light.shadow.camera.far = 2500;
      light.shadow.bias = 0.0001;

      light.shadow.mapSize.width = SHADOW_MAP_WIDTH;
      light.shadow.mapSize.height = SHADOW_MAP_HEIGHT;

      scene.add(light);

      this.createHUD();
      this.createScene();

      // RENDERER

      this_.renderer = new THREE.WebGLRenderer({antialias: true});
      this.renderer.setPixelRatio(window.devicePixelRatio);
      this.renderer.setSize(SCREEN_WIDTH, SCREEN_HEIGHT);
      this_.container.appendChild(this.renderer.domElement);

      this.renderer.outputEncoding = THREE.sRGBEncoding;
      this.renderer.autoClear = false;

      //

      this.renderer.shadowMap.enabled = true;
      this.renderer.shadowMap.type = THREE.PCFShadowMap;

      // CONTROLS

      controls = new FirstPersonControls(camera, this.renderer.domElement);

      controls.lookSpeed = 0.0125;
      controls.movementSpeed = 5;
      controls.noFly = false;
      controls.lookVertical = true;



      controls.lookAt(scene.position);

      // STATS

      stats = new Stats();
      //container.appendChild( stats.dom );

      //

      window.addEventListener('resize', this.onWindowResize);
      window.addEventListener('keydown', this.onKeyDown);

    },

    onWindowResize() {

      SCREEN_WIDTH = window.innerWidth;
      SCREEN_HEIGHT = window.innerHeight;

      camera.aspect = SCREEN_WIDTH / SCREEN_HEIGHT;
      camera.updateProjectionMatrix();

      this.renderer.setSize(SCREEN_WIDTH, SCREEN_HEIGHT);

      controls.handleResize();

    },

    onKeyDown(event) {

      switch (event.keyCode) {

        case 84:	/*t*/
          showHUD = !showHUD;
          break;

      }

    },

    createHUD() {

      lightShadowMapViewer = new ShadowMapViewer(light);
      lightShadowMapViewer.position.x = 10;
      lightShadowMapViewer.position.y = SCREEN_HEIGHT - (SHADOW_MAP_HEIGHT / 4) - 10;
      lightShadowMapViewer.size.width = SHADOW_MAP_WIDTH / 4;
      lightShadowMapViewer.size.height = SHADOW_MAP_HEIGHT / 4;
      lightShadowMapViewer.update();

    },

    createScene() {

      // GROUND

      const geometry = new THREE.PlaneGeometry(100, 100);
      const planeMaterial = new THREE.MeshPhongMaterial({color: 0xffb851});

      const ground = new THREE.Mesh(geometry, planeMaterial);

      ground.position.set(0, FLOOR, 0);
      ground.rotation.x = -Math.PI / 2;
      ground.scale.set(100, 100, 100);

      ground.castShadow = false;
      ground.receiveShadow = true;

      scene.add(ground);

      // TEXT

      const loader = new FontLoader();
      loader
          .setPath('~@/../static/fonts/')
      loader.load( 'ttf/STXingkai_Regular.json', function ( font ) {

        const textGeo = new TextGeometry( '堵管检测系统', {

              font: font,

              size: 150,
              height: 100,
              curveSegments: 20,

              bevelThickness: 9,
              bevelSize: 5,
              bevelEnabled: true

            });

            textGeo.computeBoundingBox();
            const centerOffset = -0.5 * (textGeo.boundingBox.max.x - textGeo.boundingBox.min.x);

            const textMaterial = new THREE.MeshPhongMaterial({color: 0xff0000, specular: 0xffffff});

            const mesh = new THREE.Mesh(textGeo, textMaterial);
            mesh.position.x = centerOffset;
            mesh.position.y = FLOOR + 70;

            mesh.castShadow = true;
            mesh.receiveShadow = true;

            scene.add(mesh);

          });

      // CUBES

      const cubes1 = new THREE.Mesh(new THREE.BoxGeometry(1600, 220, 150), planeMaterial);

      cubes1.position.y = FLOOR - 50;
      cubes1.position.z = 20;

      cubes1.castShadow = true;
      cubes1.receiveShadow = true;

      scene.add(cubes1);

      const cubes2 = new THREE.Mesh(new THREE.BoxGeometry(1600, 170, 250), planeMaterial);

      cubes2.position.y = FLOOR - 50;
      cubes2.position.z = 20;

      cubes2.castShadow = true;
      cubes2.receiveShadow = true;

      scene.add(cubes2);

      // MORPHS

      mixer = new THREE.AnimationMixer(scene);

      function addMorph(mesh, clip, speed, duration, x, y, z, fudgeColor) {

        mesh = mesh.clone();
        mesh.material = mesh.material.clone();

        if (fudgeColor) {

          mesh.material.color.offsetHSL(0, Math.random() * 0.5 - 0.25, Math.random() * 0.5 - 0.25);

        }

        mesh.speed = speed;

        mixer.clipAction(clip, mesh).setDuration(duration).
            // to shift the playback out of phase:
            startAt(-duration * Math.random()).play();

        mesh.position.set(x, y, z);
        mesh.rotation.y = Math.PI / 2;

        mesh.castShadow = true;
        mesh.receiveShadow = true;

        scene.add(mesh);

        morphs.push(mesh);

      }

      const gltfloader = new GLTFLoader();
      gltfloader
          .setPath('~@/../static/model/')
          .load('Parrot.glb', function (gltf) {

        const mesh = gltf.scene.children[ 0 ];
        const clip = gltf.animations[ 0 ];

        addMorph( mesh, clip, 500, 1, 500 - Math.random() * 500, FLOOR + 350, 40 );

      } );

      gltfloader
          .setPath('~@/../static/model/')
          .load('Parrot.glb', function (gltf) {

        const mesh = gltf.scene.children[ 0 ];
        const clip = gltf.animations[ 0 ];

        addMorph( mesh, clip, 350, 1, 500 - Math.random() * 500, FLOOR + 350, 340 );

      } );

      gltfloader
          .setPath('~@/../static/model/')
          .load('Parrot.glb', function (gltf) {

        const mesh = gltf.scene.children[ 0 ];
        const clip = gltf.animations[ 0 ];

        addMorph( mesh, clip, 450, 0.5, 500 - Math.random() * 500, FLOOR + 300, 700 );

      } );

    },

    animate() {

      requestAnimationFrame(this.animate);

      this.render();
      stats.update();

    },

    render() {

      const delta = clock.getDelta();

      mixer.update(delta);

      for (let i = 0; i < morphs.length; i++) {

        const morph = morphs[i];

        morph.position.x += morph.speed * delta;

        if (morph.position.x > 2000) {

          morph.position.x = -1000 - Math.random() * 500;

        }

      }

      // controls.update(delta);

      this.renderer.clear();
      this.renderer.render(scene, camera);

      // Render debug HUD with shadow map

      if (showHUD) {

        lightShadowMapViewer.render(renderer);

      }

    },

    connectDevice(type) {
      if (type === 0 && this.show === true){
        this.show = !this.show;
      }

      // 开始加载
      let loading = this.$loading({
        lock: true,//lock的修改符--默认是false
        text: "连接设备就绪中，请稍候...",//显示在加载图标下方的加载文案
        background: "rgba(0,0,0,0.8)",//遮罩层颜色
        spinner: "el-icon-loading",//自定义加载图标类名
      });

      let url0 = "http://127.0.0.1:20000/data.html";
      let url1 = "http://127.0.0.1:20000/data.html";

      axios.get(type === 0 ? url0:url1 ,{})
          .then(res =>{
        if (res !== undefined) {
          // 获取数据显示成功后，关闭loading
          setTimeout( () => {
            loading.close();
            this.$message.success("设备已就绪！");
            if (type === 1) {
              this.show = !this.show
              if (this.radioVal === "laser" ){
                this.$router.push({ path: '/home/1' })
              }else {

              }
            }
          },1500)
        } else {
          loading.close();
          this.$message.error("设备不在线，请检查设备连接！");
        }
      });
    },
  }
})

</script>
