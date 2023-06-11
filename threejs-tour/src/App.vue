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
      const geometry = new THREE.BufferGeometry();
      const position = new THREE.Float32BufferAttribute([1, 0, -8], 3);
      geometry.setAttribute("position", position);
      const material = new THREE.PointsMaterial({ map: tglowStar });
      const points = new THREE.Points(geometry, material);
      scene.add(points);
    },
    addStar(scene) {
      const geometry = new THREE.BufferGeometry();
      const position = new THREE.Float32BufferAttribute([0, 0, -30], 3);
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
