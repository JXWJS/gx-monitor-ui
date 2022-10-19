<template>
  <div id="test1"></div>
</template>

<script>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { RoomEnvironment } from 'three/examples/jsm/environments/RoomEnvironment.js'
// import {AxesHelper} from "three";


export default( {
  name: 'RoboticArm',
  data () {
    return {
      camera: null,
      scene: null,
      renderer: null,
      controls: null,
      container: null
    }
  },
  watch:{
    asyncArray:function (){}
  },
  mounted () {
    this.init()
    this.animate()
    this.$nextTick( () => {
      document.getElementById('test1').appendChild(this.container)
    })
  },
  methods: {
    init () {
      let this_ = this
      this_.container = document.createElement('div')

      this_.camera = new THREE.PerspectiveCamera(10, window.innerWidth / window.innerHeight, 0.1, 20)
      this_.camera.position.set(-30, 5, 12)
      this_.scene = new THREE.Scene()

      // this_.scene.add(new AxesHelper())

      // model
      new GLTFLoader()
          .setPath('/static/model/')
          .load('arm8.gltf', function (gltf) {
            gltf.scene.scale.set(1,1.1,1.2)
            this_.scene.add(gltf.scene)

            this_.robot_J7 = gltf.scene.getObjectByName( 'base7' );
            this_.robot_J6 = gltf.scene.getObjectByName( 'base6' );
            this_.robot_J5 = gltf.scene.getObjectByName( 'base5' );
            this_.robot_J4 = gltf.scene.getObjectByName( 'base4' );
            this_.robot_J3 = gltf.scene.getObjectByName( 'base3' );
            this_.robot_J2 = gltf.scene.getObjectByName( 'base2' );

          })

      this_.renderer = new THREE.WebGLRenderer({antialias: true,  alpha: true})
      this_.renderer.setPixelRatio(window.devicePixelRatio)
      this_.renderer.setSize(1200, 650)
      this_.renderer.toneMapping = THREE.ACESFilmicToneMapping
      this_.renderer.toneMappingExposure = 1
      this_.renderer.outputEncoding = THREE.sRGBEncoding
      this_.container.appendChild(this_.renderer.domElement)

      const environment = new RoomEnvironment()
      const paramGenerator = new THREE.PMREMGenerator(this_.renderer)

      this_.scene.background = new THREE.Color(0x01067c)
      this_.scene.environment = paramGenerator.fromScene(environment).texture

      this_.controls = new OrbitControls(this_.camera, this_.renderer.domElement)
      this_.controls.enableDamping = true
      this_.controls.minDistance = 1
      this_.controls.maxDistance = 10
      this_.controls.target.set(0.80, 0.6, 0)
      this_.controls.update()

      window.addEventListener('resize', this.onWindowResize)
    },

    onWindowResize () {
      let this_ = this
      this_.camera.aspect = window.innerWidth / window.innerHeight
      this_.camera.updateProjectionMatrix()

      this_.renderer.setSize(window.innerWidth, window.innerHeight)
    },
    animate () {
      let this_ = this
      requestAnimationFrame(this.animate)

      if(this.robot_J5!==undefined) {
        this.robot_J2.rotation.z += 0.001
        this.robot_J3.rotation.y -= 0.001
        this.robot_J4.rotation.y += 0.001
        this.robot_J5.rotation.x += 0.001
        this.robot_J6.rotation.z += 0.001
        this.robot_J7.rotation.z += 0.001

      }
      this_.controls.update() // required if damping enabled
      this.render()
    },
    render () {
      let this_ = this
      this_.renderer.render(this_.scene, this_.camera)
    }
  }

})

</script>
