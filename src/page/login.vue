<template>
  <div id="test1" >
  </div>
</template>

<script>

import * as THREE from 'three';

import { TTFLoader } from 'three/examples/jsm/loaders/TTFLoader.js';
import { Font } from 'three/examples/jsm/loaders/FontLoader.js';
import { TextGeometry } from 'three/examples/jsm/geometries/TextGeometry.js';


let camera, cameraTarget, scene, renderer;
let group, textMesh1, textMesh2, textGeo, material;
let firstLetter = true;

let text = '堵管检测系统';
const height = 20,
    size = 70,
    hover = 30,
    curveSegments = 4,
    bevelThickness = 2,
    bevelSize = 1.5;

let font = null;
const mirror = true;

let targetRotation = 0;
let targetRotationOnPointerDown = 0;

let pointerX = 0;
let pointerXOnPointerDown = 0;

let windowHalfX = window.innerWidth / 2;


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
    init () {
      let this_ = this

      this_.container = document.createElement( 'div' );
      camera = new THREE.PerspectiveCamera( 30, window.innerWidth / window.innerHeight, 1, 1500 );
      camera.position.set( 0, 400, 700 );
      cameraTarget = new THREE.Vector3( 0, 150, 0 );
      // SCENE
      scene = new THREE.Scene();
      scene.background = new THREE.Color( 0x000000 );
      scene.fog = new THREE.Fog( 0x000000, 250, 1400 );
      // LIGHTS

      const dirLight = new THREE.DirectionalLight( 0xffffff, 0.125 );
      dirLight.position.set( 0, 0, 1 ).normalize();
      scene.add( dirLight );

      const pointLight = new THREE.PointLight( 0xffffff, 1.5 );
      pointLight.position.set( 0, 100, 90 );
      pointLight.color.setHSL( Math.random(), 1, 0.5 );
      scene.add( pointLight );

      material = new THREE.MeshPhongMaterial( { color: 0xffffff, flatShading: true } );

      group = new THREE.Group();
      group.position.y = 100;

      scene.add( group );

      const loader = new TTFLoader();

      loader
          .setPath('~@/../static/fonts/')
          .load( 'ttf/STXINGKA.TTF', function ( json ) {

        font = new Font( json );
        this_.createText();

      } );

      const plane = new THREE.Mesh(
          new THREE.PlaneGeometry( 10000, 10000 ),
          new THREE.MeshBasicMaterial( { color: 0xffffff, opacity: 0.5, transparent: true } )
      );
      plane.position.y = 100;
      plane.rotation.x = - Math.PI / 2;
      scene.add( plane );

      // RENDERER

      renderer = new THREE.WebGLRenderer( { antialias: true } );
      renderer.setPixelRatio( window.devicePixelRatio );
      renderer.setSize( window.innerWidth, window.innerHeight );
      this_.container.appendChild( renderer.domElement );

      // EVENTS

      this_.container.style.touchAction = 'none';
      this_.container.addEventListener( 'pointerdown', this_.onPointerDown );

      document.addEventListener( 'keypress', this_.onDocumentKeyPress );
      document.addEventListener( 'keydown', this_.onDocumentKeyDown );

      window.addEventListener( 'resize', this_.onWindowResize );

    },

    onWindowResize () {
      let this_ = this
      windowHalfX = window.innerWidth / 2;

      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();

      renderer.setSize( window.innerWidth, window.innerHeight );
    },

     onDocumentKeyDown( event ) {

     if ( firstLetter ) {

    firstLetter = false;
    text = '';

  }

  const keyCode = event.keyCode;

  // backspace

  if ( keyCode === 8 ) {

    event.preventDefault();

    text = text.substring( 0, text.length - 1 );
    this.refreshText();

    return false;

  }

},

 onDocumentKeyPress( event ) {
      let this_ = this;

  const keyCode = event.which;

  // backspace

  if ( keyCode === 8 ) {

    event.preventDefault();

  } else {

    const ch = String.fromCharCode( keyCode );
    text += ch;

    this_.refreshText();

  }

},

 createText() {

  textGeo = new TextGeometry( text, {

    font: font,

    size: size,
    height: height,
    curveSegments: curveSegments,

    bevelThickness: bevelThickness,
    bevelSize: bevelSize,
    bevelEnabled: true

  } );

  textGeo.computeBoundingBox();
  textGeo.computeVertexNormals();

  const centerOffset = - 0.5 * ( textGeo.boundingBox.max.x - textGeo.boundingBox.min.x );

  textMesh1 = new THREE.Mesh( textGeo, material );

  textMesh1.position.x = centerOffset;
  textMesh1.position.y = hover;
  textMesh1.position.z = 0;

  textMesh1.rotation.x = 0;
  textMesh1.rotation.y = Math.PI * 2;

  group.add( textMesh1 );

  if ( mirror ) {

    textMesh2 = new THREE.Mesh( textGeo, material );

    textMesh2.position.x = centerOffset;
    textMesh2.position.y = - hover;
    textMesh2.position.z = height;

    textMesh2.rotation.x = Math.PI;
    textMesh2.rotation.y = Math.PI * 2;

    group.add( textMesh2 );

  }

},

 refreshText() {
      let this_ = this;
  group.remove( textMesh1 );
  if ( mirror ) group.remove( textMesh2 );

  if ( ! text ) return;

  this_.createText();

},

 onPointerDown( event ) {

  if ( event.isPrimary === false ) return;

  pointerXOnPointerDown = event.clientX - windowHalfX;
  targetRotationOnPointerDown = targetRotation;

  document.addEventListener( 'pointermove', onPointerMove );
  document.addEventListener( 'pointerup', onPointerUp );

},

 onPointerMove( event ) {

  if ( event.isPrimary === false ) return;

  pointerX = event.clientX - windowHalfX;

  targetRotation = targetRotationOnPointerDown + ( pointerX - pointerXOnPointerDown ) * 0.02;

},

 onPointerUp() {
  if ( event.isPrimary === false ) return;
  document.removeEventListener( 'pointermove', this.onPointerMove );
  document.removeEventListener( 'pointerup', this.onPointerUp );

},


 animate() {
  requestAnimationFrame( this.animate );
  group.rotation.y += ( targetRotation - group.rotation.y ) * 0.05;
  camera.lookAt( cameraTarget );
  renderer.render( scene, camera );
},


  }

})

</script>
