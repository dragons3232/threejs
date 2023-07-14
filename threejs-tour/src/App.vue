<template>
  <div id="threejs" />
</template>

<script>
import * as THREE from "three";
import { OBJLoader } from "three/examples/jsm/loaders/OBJLoader";
import { MTLLoader } from "three/examples/jsm/loaders/MTLLoader";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { FontLoader } from "three/examples/jsm/loaders/FontLoader";
import { TextGeometry } from "three/examples/jsm/geometries/TextGeometry.js";
import ThreeGlobe from "three-globe";

const texLoader = new THREE.TextureLoader();
// const tglowStar = texLoader.load("merkababloom.png");
const tsparkle = texLoader.load("sparkle.png");
const tsparkle2 = texLoader.load("sparkle2.png");
const tsparkle3 = texLoader.load("sparkle3.png");
// const tstar = texLoader.load("star.png");
const tsmoke = texLoader.load("smoke_particle.png");

const sparkles = [tsparkle, tsparkle2, tsparkle3];
const littles = [tsparkle, tsparkle2];
const smokeMaxZ = 200;
const shootingSpeeds = [0.5, 0.7, 0.8, 1];
const starSpeeds = [];

export default {
  name: "App",
  components: {},
  mounted() {
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight,
      0.1,
      1000
    );

    const renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);

    const container = document.getElementById("threejs");
    container.appendChild(renderer.domElement);

    this.threejs = {
      renderer,
      scene,
      camera,
    };

    camera.position.z = 5;
    const nstars = this.addStar(scene);
    const gstars = this.glowStar(scene);
    const shootingStars = this.shootingStar(scene);

    this.addCube();
    this.addCylinder();
    this.addText();
    // this.model3d(scene);

    const globe = new ThreeGlobe()
      .globeImageUrl("//unpkg.com/three-globe/example/img/earth-blue-marble.jpg")
      .bumpImageUrl("//unpkg.com/three-globe/example/img/earth-topology.png");

    globe.position.z = -3;
    globe.scale.set(0.02, 0.02, 0.02);

    const directionalLight = new THREE.DirectionalLight(0xffffff, 2);
    directionalLight.position.set(0, 0, 1); // change light position to see the specularMap's effect

    // scene.add(globe);
    scene.add(directionalLight);

    function animate() {
      requestAnimationFrame(animate);

      globe.rotation.y += 0.002;

      shootingStars.map((e, index) => {
        const z = e.position.z + starSpeeds[index];
        if (z > smokeMaxZ) {
          e.position.z = -smokeMaxZ;
        } else {
          e.position.z = z;
        }
      });

      nstars.map((e) => {
        e.visible = Math.floor(Math.random() * 100) >= 20;
      });

      gstars.map((e) => {
        e.visible = Math.floor(Math.random() * 100) >= 5;
      });

      renderer.render(scene, camera);
    }

    animate();

    window.addEventListener("resize", () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();

      renderer.setSize(window.innerWidth, window.innerHeight);
    });
  },
  methods: {
    addCube() {
      const { scene } = this.threejs;
      const geometry = new THREE.BoxGeometry(1, 1, 1);
      const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
      const cube = new THREE.Mesh(geometry, material);
      scene.add(cube);

      cube.rotation.set(-1, 0, 0);
      cube.position.set(-3, 0, 0);
    },
    addCylinder() {
      const { scene } = this.threejs;

      const geometry = new THREE.CylinderGeometry(1, 1, 3, 32);
      const material = new THREE.MeshBasicMaterial({ color: 0xffff00 });
      const cylinder = new THREE.Mesh(geometry, material);
      scene.add(cylinder);

      cylinder.position.set(3, 0, 0);

      return cylinder;
    },
    addText() {
      const { scene } = this.threejs;
      const loader = new FontLoader();

      loader.load("fonts/helvetiker_regular.typeface.json", function (font) {
        const geometry = new TextGeometry("ThreeJs Galaxy", {
          font,
          size: 150,
          height: 30,
          curveSegments: 12,
          bevelEnabled: true,
          bevelThickness: 10,
          bevelSize: 8,
          bevelOffset: 0,
          bevelSegments: 5,
        });

        const material = new THREE.MeshPhongMaterial({ color: 0x6ed47c });
        const text = new THREE.Mesh(geometry, material);
        scene.add(text);

        text.scale.set(0.003, 0.003, 0.003);
        text.position.x = -2;
        text.position.y = 2;
        text.rotateY(0.1);
        text.rotateZ(0.15);
      });
    },
    model3dMtl(obj, materialImage, scale, rotation) {
      const { scene } = this.threejs;
      const objLoader = new OBJLoader();
      const mtlLoader = new MTLLoader();

      mtlLoader.load(
        materialImage,
        (materials) => {
          materials.preload();
          objLoader.setMaterials(materials);
          objLoader.load(
            obj,
            (object) => {
              object.scale.set(scale, scale, scale);
              if (rotation) object.rotation.set(rotation[0], rotation[1], rotation[2]);
              object.position.x = -1;
              scene.add(object);
            },
            (xhr) => {
              console.log((xhr.loaded / xhr.total) * 100 + "% loaded");
            },
            (error) => console.log(error)
          );
        },
        (xhr) => {
          console.log((xhr.loaded / xhr.total) * 100 + "% loaded");
        },
        (error) => console.log(error)
      );
    },
    model3dGlbGltf(obj, scale, rotation) {
      const { scene } = this.threejs;
      const gltfLoader = new GLTFLoader();
      gltfLoader.load(
        obj,
        (gltf) => {
          const object = gltf.scene;
          object.scale.set(scale, scale, scale);
          if (rotation) object.rotation.set(rotation[0], rotation[1], rotation[2]);
          object.position.x = -1;
          scene.add(object);
        },
        (xhr) => {
          console.log((xhr.loaded / xhr.total) * 100 + "% loaded");
        },
        (error) => console.log(error)
      );
    },
    model3d(obj, materialImage, scale, rotation) {
      const { scene } = this.threejs;
      const objLoader = new OBJLoader();

      if (obj.split(".").reverse()[0] == "gltf" || obj.split(".").reverse()[0] == "glb") {
        return this.model3dGlbGltf(obj, scale, rotation);
      }

      if (materialImage.split(".").reverse()[0] == "mtl") {
        return this.model3dMtl(obj, materialImage, scale, rotation);
      }

      const plume = texLoader.load(materialImage);
      const material = new THREE.MeshPhongMaterial({
        map: plume,
      });

      objLoader.load(obj, (model) => {
        model.traverse(function (child) {
          if (child instanceof THREE.Mesh) {
            child.material = material;
          }
        });
        model.scale.set(scale, scale, scale);
        if (rotation) model.rotation.set(rotation[0], rotation[1], rotation[2]);
        model.position.x = -1;
        scene.add(model);
      });
    },
    glowStar(scene) {
      const stars = [];
      const starMap = this.randomPoints(320, [18, 15, 12, 10]);

      for (let i = 0; i < starMap.length / 3; i++) {
        const j = i * 3;
        const p = [starMap[j], starMap[j + 1], starMap[j + 2]];
        const geometry = new THREE.BufferGeometry();
        const position = new THREE.Float32BufferAttribute(p, 3);
        geometry.setAttribute("position", position);
        const material = new THREE.PointsMaterial({
          map: sparkles[Math.floor(Math.random() * 10) % sparkles.length],
          transparent: true,
        });
        const points = new THREE.Points(geometry, material);
        scene.add(points);
        stars.push(points);
      }
      return stars;
    },
    randomPoints(numOfStars, zIds) {
      const quarter = numOfStars / 4;
      const zIndices = zIds || [80, 70, 60, 50];

      const starMap = [];
      for (let i = 0; i <= numOfStars; i++) {
        const x = Math.floor(Math.random() * 100);
        const y = Math.floor(Math.random() * 100) % 50;
        const z = zIndices[Math.floor(Math.random() * 10) % zIndices.length];

        const division = Math.floor(i / quarter);
        const signX = division == 0 || division == 1;
        const signY = division == 1 || division == 3;

        starMap.push(signX ? -x : x);
        starMap.push(signY ? -y : y);
        starMap.push(-z);
      }
      return starMap;
    },
    addStar(scene) {
      const stars = [];
      const starMap = this.randomPoints(80);

      for (let i = 0; i < starMap.length / 3; i++) {
        const j = i * 3;
        const p = [starMap[j], starMap[j + 1], starMap[j + 2]];
        const geometry = new THREE.BufferGeometry();
        const position = new THREE.Float32BufferAttribute(p, 3);
        geometry.setAttribute("position", position);
        const material = new THREE.PointsMaterial({
          map: littles[Math.floor(Math.random() * 10) % littles.length],
        });
        const points = new THREE.Points(geometry, material);
        scene.add(points);
        stars.push(points);
      }
      return stars;
    },
    shootingStar(scene) {
      const stars = [];
      const starMap = this.randomPoints(40, [
        smokeMaxZ,
        smokeMaxZ - 60,
        smokeMaxZ - 120,
        smokeMaxZ - 150,
      ]);

      for (let i = 0; i < starMap.length / 3; i++) {
        const j = i * 3;
        const p = [starMap[j], starMap[j + 1], starMap[j + 2]];
        const geometry = new THREE.BufferGeometry();
        const position = new THREE.Float32BufferAttribute(p, 3);
        geometry.setAttribute("position", position);
        const material = new THREE.PointsMaterial({
          map: tsmoke,
          transparent: true,
        });
        const points = new THREE.Points(geometry, material);
        scene.add(points);
        stars.push(points);

        const sIndex = Math.floor(Math.random() * 10) % shootingSpeeds.length;
        starSpeeds.push(shootingSpeeds[sIndex]);
      }
      return stars;
    },
  },
};
</script>

<style>
body {
  margin: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
