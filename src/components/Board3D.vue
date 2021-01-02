<template>
    <div id="scene" ref="container"></div>
</template>

<script>
import * as Three from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

export default {
  name: 'Board3D',
  props: [
    'gridSize'
  ],
  data() {
    return {
      camera: null,
      scene: null,
      renderer: null,
      mesh: null,
      container: null,
      raycaster: null,
      plane: null,
      mouse: null,
      gridHelper: null,
      rollOverMesh: null,
      voxels: [],
      objects: [],
    }
  },
  watch: {
    gridSize: function (val) {
      this.gridDivisions=val;
      this.updateGrid();
    },
  },
  methods: {
    addVoxel() {

    },
    removeVoxel() {

    },
    updateVoxel() {

    },
    updateGrid() {
      if(this.gridHelper) {
        this.scene.remove(this.gridHelper);
      }
      this.gridHelper = new Three.GridHelper(1000, this.gridDivisions);
      this.scene.add(this.gridHelper);
      this.render();
    },
    setMaterial() {

    },

    init() {

      //conteneur pour le canvas de sortie de la scene
      let container = this.$refs.container;

      this.container = container;
      this.voxels = [];
      this.objects= [];

      this.camera = new Three.PerspectiveCamera(45, container.clientWidth/container.clientHeight, 1, 10000);
      this.camera.position.set( 250, 2000, 2500 );
      this.camera.lookAt( 0, 0, 0 );

      this.scene = new Three.Scene();
      this.scene.background = new Three.Color( 0xf0f0f0 );

      // roll-over helpers
      var rollOverGeo = new Three.BoxBufferGeometry( 50, 50, 50 );
      var rollOverMaterial = new Three.MeshBasicMaterial( { color: 0xff0000, opacity: 0.5, transparent: true } );
      this.rollOverMesh = new Three.Mesh( rollOverGeo, rollOverMaterial );
      this.scene.add( this.rollOverMesh );

      // grid
      this.gridHelper = new Three.GridHelper( 1000, 20 );
      this.scene.add( this.gridHelper );

      //un cube
      //let geometry = new Three.BoxGeometry(1000/32, 1000/32, 1000/32);
      //let material = new Three.MeshNormalMaterial();

      this.cubeGeo = new Three.BoxBufferGeometry( 50, 50, 50 );
      this.cubeMaterial = new Three.MeshLambertMaterial( { color: 0xfeb74c, map: new Three.TextureLoader().load( 'square-outline-textured.png' ) } );
      this.raycaster = new Three.Raycaster();
      this.mouse = new Three.Vector2();

      // plan pour detecter la collision
      let planegeometry = new Three.PlaneBufferGeometry( 1000, 1000 );
      planegeometry.rotateX( - Math.PI / 2 );
      this.plane = new Three.Mesh( planegeometry, new Three.MeshBasicMaterial( { visible: false } ) );
      this.scene.add( this.plane );
      this.objects.push( this.plane );

      // lights
      var ambientLight = new Three.AmbientLight( 0x606060 );
      this.scene.add( ambientLight );

      var directionalLight = new Three.DirectionalLight( 0xffffff );
      directionalLight.position.set( 1, 0.75, 0.5 ).normalize();
      this.scene.add( directionalLight );
/*
      for(let x=0;x<32;x++) {
        for(let y=0;y<32;y++) {
          let voxel = new Three.Mesh(geometry, material);
          voxel.position.x =  (x-1)*1000/32;
          voxel.position.z =  (y-1)*1000/32;
          voxel.position.x+= (1000/32)/2;
          voxel.position.y+= (1000/32)/2;
          voxel.position.z+= (1000/32)/2;
          this.voxels.push(voxel);
          this.objects.push(voxel);
          this.scene.add(voxel);
        }
      }
*/

      this.renderer = new Three.WebGLRenderer({antialias: true});
      this.renderer.setSize(container.clientWidth, container.clientHeight);
      this.controls = new OrbitControls( this.camera, this.renderer.domElement );

      container.appendChild(this.renderer.domElement);
      document.addEventListener( 'mousemove',this.onMouseMove, false );
      document.addEventListener( 'mousedown', this.onMouseDown, false );
    },
    onWindowResize(){
      this.renderer.setSize(this.$refs.container.clientWidth, this.$refs.container.clientHeight);
      this.camera.aspect = this.$refs.container.clientWidth / this.$refs.container.clientHeight;
      this.camera.updateProjectionMatrix();
    },
    onMouseMove( event ) {
      //if(!this.$refs.scenecontainer) return;
      event.preventDefault();

      //pas bon
      this.mouse.set( ( event.clientX / window.innerWidth ) * 2 - 1, - ( event.clientY / window.innerHeight ) * 2 + 1 );

      this.raycaster.setFromCamera( this.mouse, this.camera );
      var intersects = this.raycaster.intersectObjects( this.objects );
      if ( intersects.length > 0 ) {
        var intersect = intersects[ 0 ];
        this.rollOverMesh.position.copy( intersect.point ).add( intersect.face.normal );
        this.rollOverMesh.position.divideScalar( 50 ).floor().multiplyScalar( 50 ).addScalar( 25 );
      }
      this.render();
    },
    onMouseDown( event ) {

      if(event.button == 2) return;
      event.preventDefault();

      //pas bon
      this.mouse.set( ( event.clientX /  window.innerWidth ) * 2 - 1, - ( event.clientY / window.innerHeight ) * 2 + 1 );

      this.raycaster.setFromCamera( this.mouse, this.camera );

      var intersects = this.raycaster.intersectObjects( this.objects );

      if ( intersects.length > 0 ) {

        var intersect = intersects[0];

        // delete cube
        if ( this.isShiftDown ) {
          if ( intersect.object !== this.plane ) {
            this.scene.remove( intersect.object );
            this.objects.splice( this.objects.indexOf( intersect.object ), 1 );
            this.voxels.splice( this.voxels.indexOf( intersect.object ), 1 );
          }
          // create cube
        } else {
          var voxel = new Three.Mesh( this.cubeGeo, this.cubeMaterial );
          voxel.position.copy( intersect.point ).add( intersect.face.normal );
          voxel.position.divideScalar( 50 ).floor().multiplyScalar( 50 ).addScalar( 25 );
          this.scene.add( voxel );
          this.voxels.push( voxel );
        }
        this.render();
      }
    },
    render() {
      this.renderer.render(this.scene, this.camera);
    },
    animate() {
      requestAnimationFrame(this.animate);
      this.render();
    }
  },
  mounted() {
    this.init();
    this.animate();
  },
  created(){
    window.addEventListener('resize', () => {this.onWindowResize()})
  }
}
</script>

<style scoped>
#scene {
  height: 500px;
  width: 500px;
}
</style>