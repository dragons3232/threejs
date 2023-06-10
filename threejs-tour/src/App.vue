<template>
  <div id="threejs" />
</template>

<script>
import * as THREE from "three";

const texLoader = new THREE.TextureLoader();
const tglowStar = texLoader.load("merkababloom.png");
const tstar = texLoader.load("star.png");

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
    this.addStar(scene);
    this.glowStar(scene);

    function animate() {
      requestAnimationFrame(animate);

      cylinder.rotation.x += 0.01;
      cylinder.rotation.y += 0.01;

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
    glowStar(scene) {
      const starMap = this.randomPoints(40, [18, 15, 12, 10]);
      const geometry = new THREE.BufferGeometry();
      const position = new THREE.Float32BufferAttribute(starMap, 3);
      geometry.setAttribute("position", position);
      const material = new THREE.PointsMaterial({ map: tglowStar });
      const points = new THREE.Points(geometry, material);
      scene.add(points);
    },
    randomPoints(numOfStars, zIds) {
      const quarter = numOfStars / 4;
      const zIndices = zIds || [120, 100, 90, 80, 60];

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
      const starMap = this.randomPoints(160);
      const geometry = new THREE.BufferGeometry();
      const position = new THREE.Float32BufferAttribute(starMap, 3);
      geometry.setAttribute("position", position);
      const material = new THREE.PointsMaterial({ map: tstar });
      const points = new THREE.Points(geometry, material);
      scene.add(points);
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
