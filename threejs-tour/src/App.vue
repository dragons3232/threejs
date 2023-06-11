<template>
  <div id="threejs" />
</template>

<script>
import * as THREE from "three";
import { OBJLoader } from "three/examples/jsm/loaders/OBJLoader";

const texLoader = new THREE.TextureLoader();
// const tglowStar = texLoader.load("merkababloom.png");
const tsparkle = texLoader.load("sparkle.png");
const tsparkle2 = texLoader.load("sparkle2.png");
const tsparkle3 = texLoader.load("sparkle3.png");
// const tstar = texLoader.load("star.png");

const sparkles = [tsparkle, tsparkle2, tsparkle3];
const littles = [tsparkle, tsparkle2];

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

    const geometry = new THREE.CylinderGeometry(1, 1, 3, 32);
    const material = new THREE.MeshBasicMaterial({ color: 0xffff00 });
    const cylinder = new THREE.Mesh(geometry, material);
    // scene.add(cylinder);

    camera.position.z = 5;
    const nstars = this.addStar(scene);
    const gstars = this.glowStar(scene);

    this.model3d(scene);

    function animate() {
      requestAnimationFrame(animate);

      cylinder.rotation.x += 0.01;
      cylinder.rotation.y += 0.01;

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
    model3d(scene) {
      const objLoader = new OBJLoader();
      const plume = texLoader.load("plume.png");
      const material = new THREE.MeshBasicMaterial({
        map: plume,
      });

      objLoader.load("plume.obj", (model) => {
        model.traverse(function (child) {
          if (child instanceof THREE.Mesh) {
            child.material = material;
          }
        });
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
