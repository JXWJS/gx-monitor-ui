<template>
  <div id="test1" >
    <div id="test11" style="position: absolute;top: 30px; width: 10%;
  height: 200px;">
    </div>

  </div>
</template>

<script>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import { RGBELoader  } from 'three/examples/jsm/loaders/RGBELoader.js'
import { RoomEnvironment } from 'three/examples/jsm/environments/RoomEnvironment.js'
// import {AxesHelper} from "three";


import * as echarts from 'echarts';

export default( {
  name: 'RoboticArm',

  data () {
    return {
      camera: null,
      scene: null,
      renderer: null,
      controls: null,
      container: null,
      myChart: null,
      dataList: null,
      J1: "",
      J2: "",
      J3: "",
      J4: "",
      J5: "",
      J6: "",
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
    setInterval(this.initCharts, 1000);
    this.test()
  },
  methods: {
    init () {
      let this_ = this
      this_.container = document.createElement('div')

      this_.camera = new THREE.PerspectiveCamera(10, window.innerWidth / window.innerHeight, 0.1, 20)
      this_.camera.position.set(-30, 5, -12)
      this_.scene = new THREE.Scene()

      // this_.scene.add(new AxesHelper())

      new RGBELoader()
          .setPath( '~@/../static/model/' )
          .load( 'royal_esplanade_1k.hdr', function ( texture ) {
            texture.mapping = THREE.EquirectangularReflectionMapping;

            this_.scene.background = texture;
            this_.scene.environment = texture;

            this_.render();
            // model
            new GLTFLoader()
                .setPath('~@/../static/model/')
                .load('arm8.gltf', function (gltf) {
                  gltf.scene.scale.set(1, 1.1, 1.2)
                  this_.scene.add(gltf.scene)

                  this_.robot_J7 = gltf.scene.getObjectByName('base7');
                  this_.robot_J6 = gltf.scene.getObjectByName('base6');
                  this_.robot_J5 = gltf.scene.getObjectByName('base5');
                  this_.robot_J4 = gltf.scene.getObjectByName('base4');
                  this_.robot_J3 = gltf.scene.getObjectByName('base3');
                  this_.robot_J2 = gltf.scene.getObjectByName('base2');

                  this_.render();
                });
          });


      this_.renderer = new THREE.WebGLRenderer({antialias: true,  alpha: true})
      this_.renderer.setPixelRatio(window.devicePixelRatio)
      this_.renderer.setSize(890, 645)
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
        this.robot_J2.rotation.z = this_.J1
        this.robot_J3.rotation.y = this_.J2
        this.robot_J4.rotation.y = this_.J3
        this.robot_J5.rotation.x = this_.J4
        this.robot_J6.rotation.z = this_.J5
        this.robot_J7.rotation.z = this_.J6

        this_.dataList = [this.robot_J2.rotation.z,this.robot_J3.rotation.y ,this.robot_J4.rotation.y ,
          this.robot_J5.rotation.x ,
          this.robot_J6.rotation.z ,
          this.robot_J7.rotation.z ];

      }

      this_.controls.update() // required if damping enabled
      this.render()
    },
    render () {
      let this_ = this
      this_.renderer.render(this_.scene, this_.camera)
    },

    test() {
      let this_ = this
      setInterval(function (){
        this_.$post('data.html',{})
            .then((response) => {
              let arr = response.split("1:")[1].split("Frame")[0].split("Joint ");
              this_.J1 = parseFloat(arr[0])
              this_.J2 = parseFloat(arr[1].split("2:")[1])
              this_.J3 = parseFloat(arr[2].split("3:")[1])
              this_.J4 = parseFloat(arr[3].split("4:")[1])
              this_.J5 = parseFloat(arr[4].split("5:")[1])
              this_.J6 = parseFloat(arr[5].split("6:")[1])
            })
      }, 1000);
    },

    initCharts() {
      let this_ = this
      // 初始化echarts实例
      this_.myChart = this.$echarts.init(document.getElementById("test11"))
      // 绘制图表
      this_.myChart.setOption({
        title: {
          text: 'Robotic Arm Data'
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'shadow'
          }
        },
        grid: {
          top: 80,
          bottom: 30
        },
        xAxis: {
          type: 'value',
          position: 'top',
          axisLine: {
            lineStyle: {
              color: '#645206',
              width: 2,
              // type: 'dashed'
            }
          }
        },
        yAxis: {
          type: 'category',
          axisLine: { show: false },
          axisLabel: { show: false },
          axisTick: { show: false },
          splitLine: { show: false },
          data: [
            'J1',
            'J2',
            'J3',
            'J4',
            'J5',
            'J6'
          ]
        },
        series: [
          {
            name: 'Cost',
            type: 'bar',
            stack: 'Total',
            data: this_.dataList,
            itemStyle: {
              normal: {
                //这里是重点
                color: function (params) {
                  //注意，如果颜色太少的话，后面颜色不会自动循环，最好多定义几个颜色
                  let colorList = ['#c23531', '#2f4554', '#61a0a8', '#d48265', '#91c7ae', '#749f83'];
                  return colorList[params.dataIndex]
                }
              }
            }
          },




        ]

      })
    },
  }

})

</script>
