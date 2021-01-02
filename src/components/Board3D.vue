<template>
    <div id="scene" ref="container"></div>
</template>

<script>
import * as Three from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

export default {
  name: 'Board3D',
  props: [
    'gridSize',
    'width',
    'height'
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
      offColor :0x0,
      color: 0x0000ff,
      color_r: 0,
      color_g: 0,
      color_b: 255,
      color_dest_r: 0,
      color_dest_g: 0,
      color_dest_b: 255,
      color_dr:0,
      color_dg:0,
      color_db:0
    }
  },
  watch: {
    gridSize: function (val) {
      this.gridDivisions=val;
      this.updateGrid();
    },
    width: function() {
      this.updateDimensions();
    },
    height: function() {
      this.updateDimensions();
    },

  },
  methods: {
    addVoxel() {

    },
    removeVoxel() {

    },
    updateVoxel() {

    },
    updateDimensions() {

      let self=this;
      this.voxels.forEach(function(voxel) {
        self.scene.remove(voxel);
      });

      this.voxels = [];


      /*
      for(let x=0;x<this.width;x++) {
        for(let y=0;y<this.height;y++) {
          let voxel = new Three.Mesh(this.cubeGeo, this.cubeMaterial);
          voxel.position.x =  (x)*50;
          voxel.position.z =  (y)*50;

          voxel.position.x+= 50/2;
          voxel.position.z+= 50/2;
          voxel.position.y+= 50/2;

          voxel.position.x-=(50*this.width)/2;
          voxel.position.z-= (50*this.height)/2;

          this.voxels.push(voxel);
          this.objects.push(voxel);
          this.scene.add(voxel);
        }
      }
      */
      this.render();
    },
    updateGrid() {
      if(this.gridHelper) {
        this.scene.remove(this.gridHelper);
      }
      this.gridHelper = new Three.GridHelper(1000, this.gridDivisions);
      //this.scene.add(this.gridHelper);
      this.render();
    },
    setMaterial() {

    },

    RGB2Hex(r, g, b) {
        var hex = b;
        hex |= (g << 8);
        hex |= (r << 16);
        return hex;
    },

    color_change(steps) {
        if (Math.round(this.color_r) == this.color_dest_r &&
            Math.round(this.color_g) == this.color_dest_g &&
            Math.round(this.color_b) == this.color_dest_b
            ) {
            this.color_dest_r = this.rand()%255;
            this.color_dest_g = this.rand()%255;
            this.color_dest_b = this.rand()%255;

            this.color_dr = (this.color_dest_r - this.color_r) / steps;
            this.color_dg = (this.color_dest_g - this.color_g) / steps;
            this.color_db = (this.color_dest_b - this.color_b) / steps;
        }

        this.color_r += this.color_dr;
        this.color_g += this.color_dg;
        this.color_b += this.color_db;

        this.color = this.RGB2Hex(Math.round(this.color_r), Math.round(this.color_g), Math.round(this.color_b));
    },

    fill(pattern) {
        if (!pattern) {
            this.cube_clear();
        } else {
            this.cube_clear(this.color);
        }
    },

    setvoxel(x, y, z, c) {
        if (typeof c === 'undefined') {
            c = this.color;
        }
        this.cube_set_color(x, y, z, c);
    },

    rand() {
        return Math.floor(Math.random() * 999999);
    },

    distance(x1, y1, z1, x2, y2, z2) {
        return Math.sqrt((x1-x2)*(x1-x2) + (y1-y2)*(y1-y2) + (z1-z2)*(z1-z2));
    },

    init() {

      //conteneur pour le canvas de sortie de la scene
      let container = this.$refs.container;

      this.container = container;
      this.voxels = [];
      this.objects= [];

      this.delay = 0;
      this.start = 0;
      this.resX = 8;
      this.resY = 8;
      this.resZ = 8;
      this.distanceLED = 40;

      var d = this.distanceLED;
      var s = 40;
      var s2 = 15;
      var x,y,z;

      this.offset_x = -d*(this.resX-1)/2;
      this.offset_y = -d*(this.resY-1)/2+30;
      this.offset_z = -150;


      this.camera = new Three.PerspectiveCamera(45, container.clientWidth/container.clientHeight, 1, 10000);
      this.camera.position.set( 250, 2000, 2500 );
      this.camera.lookAt( 0, 0, 0 );

      this.scene = new Three.Scene();
      this.scene.background = new Three.Color( 0x000000 );


      var textureGlow  = Three.ImageUtils.loadTexture('images/textures/led1.png');
      var texturePoint = Three.ImageUtils.loadTexture('images/textures/led4.png');
      var textureOff   = Three.ImageUtils.loadTexture('images/textures/ledoff.png');

      this.materialGlow  = new Three.SpriteMaterial({ map: textureGlow,  color: 0x0, transparent: true } );
      this.materialPoint = new Three.SpriteMaterial({ map: texturePoint, color: 0x0, transparent: true } );
      this.materialOff   = new Three.SpriteMaterial({ map: textureOff,   color: 0xffffff, transparent: true, opacity: 0.2} );

      // grid
      this.gridHelper = new Three.GridHelper( 1000, 20 );
      //this.scene.add( this.gridHelper );

      //un cube
      //let geometry = new Three.BoxGeometry(1000/32, 1000/32, 1000/32);
      //let material = new Three.MeshNormalMaterial();

      this.cubeGeo = new Three.BoxBufferGeometry( 50, 50, 50 );
      this.cubeMaterial = new Three.MeshLambertMaterial( { color: 0xfeb74c, map: new Three.TextureLoader().load( 'square-outline-textured.png' ) } );
      //this.raycaster = new Three.Raycaster();
      //this.mouse = new Three.Vector2();

      // plan pour detecter la collision
      //let planegeometry = new Three.PlaneBufferGeometry( 1000, 1000 );
      //planegeometry.rotateX( - Math.PI / 2 );
      //this.plane = new Three.Mesh( planegeometry, new Three.MeshBasicMaterial( { visible: false } ) );
      //this.scene.add( this.plane );
      //this.objects.push( this.plane );

      // lights
      //var ambientLight = new Three.AmbientLight( 0x606060 );
      //this.scene.add( ambientLight );

      //var directionalLight = new Three.DirectionalLight( 0xffffff );
      //directionalLight.position.set( 1, 0.75, 0.5 ).normalize();
      //this.scene.add( directionalLight );

      this.renderer = new Three.WebGLRenderer({antialias: true});
      this.renderer.setSize(container.clientWidth, container.clientHeight);
      this.controls = new OrbitControls( this.camera, this.renderer.domElement );

        // floor
      this.floorMaterial = new Three.MeshPhongMaterial({
          color: 0,
          shininess: 2
      });
      this.floorGeometry = new Three.PlaneGeometry(2000, 2000, 10, 10);
      this.floor = new Three.Mesh(this.floorGeometry, this.floorMaterial);
      this.floor.position.y = this.offset_y - 30;
      this.floor.rotation.x = -Math.PI / 2;
      //this.scene.add(this.floor);


      for ( z = 0; z < this.resZ; z++) {
        for ( y = 0; y < this.resY; y++) {
            for ( x = 0; x < this.resX; x++) {
                var sprite = new Three.Sprite(this.materialOff);
                sprite.position.set(this.offset_x + d*x, this.offset_y + d*y, this.offset_z + d*z);
                sprite.scale.set(s2, s2, s2);
                this.scene.add(sprite);

                sprite = new Three.Sprite(this.materialGlow.clone());
                sprite.material.opacity = 0;
                sprite.position.set(this.offset_x + d*x, this.offset_y + d*y, this.offset_z + d*z);
                sprite.scale.set(s, s, s);
                this.scene.add(sprite);
            }
        }
      }

      for (z = 0; z < this.resZ; z++) {
        for (x = 0; x < this.resX; x++) {
            var light = new Three.PointLight(0x0000ff, 0.0, 700);
            light.position.set(this.offset_x + d*x, this.offset_y, this.offset_z + d*z);
            this.scene.add(light);
        }
      }


      container.appendChild(this.renderer.domElement);

      //this.updateDimensions();
    },

    onWindowResize(){
      this.renderer.setSize(this.$refs.container.clientWidth, this.$refs.container.clientHeight);
      this.camera.aspect = this.$refs.container.clientWidth / this.$refs.container.clientHeight;
      this.camera.updateProjectionMatrix();
    },
    render() {
      var i;
      var elapsed = Date.now() - this.start;
      if (elapsed >= this.delay) {
          if (this.start > 0) {
              i = Math.floor(elapsed/this.delay);
              if (i < 1) i = 1;
          } else {
              i = 1;
          }

          do {
              this.delay = this.animation();
              i--;
          } while (this.delay == 0 || i > 0);

          this.start = Date.now();
      }

      this.renderer.render(this.scene, this.camera);
    },
    animate() {
      requestAnimationFrame(this.animate);
      this.render();
    },
    animation() {
      var ripple_interval = 1.3;
      var i = 0;
      var delay = 5;
      var self = this;
      self.fill(0x00);
      for (var z=0; z<this.resZ; z++) {
          for (var x=0; x<this.resX; x++) {
              var d = self.distance(3.5, 3.5, 0, x, z, 0)/9.899495*this.resZ;
              var height = Math.floor(4+Math.sin(d/ripple_interval + i/50)*4);
              self.setvoxel(x, height, z, this.color-10*height);
          }
      }

      self.color_change(1000);
      i++;
      return delay;
    },

    cube_check_coords(x, y, z) {
        return (x >= 0 && x < this.resX && y >= 0 && y < this.resY && z >= 0 && z < this.resZ);
    },

    cube_get_color(x, y, z) {
      if (!this.cube_check_coords(x, y, z)) {
        return -1;
      }
      var i = 2 + (x + y*this.resY + z*this.resX*this.resY)*2;
      return this.scene.children[i+1].material.color.getHex();
    },

    get_scene_offset(x, y, z) {
      return 2 + (x + y*this.resY + z*this.resX*this.resY)*2;
    },

    cube_set_color(x, y, z, color) {
      if (!this.cube_check_coords(x, y, z)) {
          return;
      }

      var i = this.get_scene_offset(x, y, z);

      if (color == 0x0) {
          // turn off the LED
          this.scene.children[i].material = this.materialOff;
          this.scene.children[i+1].material.opacity = 0;
          this.scene.children[i+1].material.color.setHex(color);
      } else {
        // turn on the LED
        this.scene.children[i].material = this.materialPoint.clone();
        this.scene.children[i].material.color.setHex(color);
        var hsl = this.scene.children[i].material.color.getHSL();
        this.scene.children[i].material.color.setHSL(hsl.h, hsl.s, 0.7);

        this.scene.children[i+1].material.opacity = 1;
        this.scene.children[i+1].material.color.setHex(color);
      }

      // update the light
      var light = this.scene.children[2 + this.resX*this.resY*this.resZ*2 + x + z*this.resZ];
      var lightOff = true;

      for (var yy = 0; yy < this.resY; yy++) {
          var ledGlow = this.scene.children[this.get_scene_offset(x, yy, z)+1];
          if (ledGlow.material.opacity == 1) {
              if (lightOff) {
                  light.intensity = 1.0;
                  light.color.setHex(ledGlow.material.color.getHex());
                  light.position.y = yy*this.distanceLED;

                  lightOff = false;
              } else {
                  var mixinColor = ledGlow.material.color.clone();
                  light.color.lerp(mixinColor, 1/yy);
              }
          }
      }

      if (lightOff) {
          light.intensity = 0;
      }
    },

    cube_clear(color) {
      if (typeof color === 'undefined') {
          color = 0x0;
      }

      for (var z = 0; z < this.resZ; z++) {
        for (var y = 0; y < this.resY; y++) {
          for (var x = 0; x < this.resX; x++) {
            this.cube_set_color(x, y, z, color);
          }
        }
      }
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