<template>
  <div ref="sceneContainer" class="scene-container"></div>
</template>

<script>
import { ref, onMounted, reactive } from "vue";
import * as THREE from "three";

export default {
  name: "ThreeScene",
  setup() {
    const sceneContainer = ref(null);
    const teeth = reactive({});
    const mouse = { x: 0, y: 0 };
    const isDragging = ref(false);

    onMounted(() => {
      const scene = new THREE.Scene();
      const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
      const renderer = new THREE.WebGLRenderer({ antialias: true });
      renderer.setSize(window.innerWidth, window.innerHeight);
      sceneContainer.value.appendChild(renderer.domElement);

      const light = new THREE.AmbientLight(0x404040);
      scene.add(light);
      const directionalLight = new THREE.DirectionalLight(0xffffff, 0.6);
      scene.add(directionalLight);

      const toothGeometry = new THREE.CylinderGeometry(0.3, 0.5, 1, 32);
      const toothGeometryReversed = new THREE.CylinderGeometry(0.3, 0.4, 0.9, 32);
      const toothMaterial = new THREE.MeshLambertMaterial({ color: 0xffffff });

      let index = 0;
      const createTeeth = (yPosition, zPosition, direction, reverse = false) => {
        const arcRadius = 3;
        const angleStep = Math.PI / 16;
        for (let i = 0; i < 16; i++) {
          const tooth = new THREE.Mesh(reverse ? toothGeometryReversed : toothGeometry, toothMaterial);
          const angle = angleStep * i * direction;
          const xPosition = arcRadius * Math.cos(angle);
          const zPos = reverse ? zPosition - arcRadius * Math.sin(angle) : zPosition + arcRadius * Math.sin(angle);
          tooth.position.set(xPosition, yPosition, zPos);
          tooth.rotation.y = -angle;
          teeth[index++] = tooth;
          scene.add(tooth);
        }
      };

      // Create lower teeth
      createTeeth(-0.5, -1, 1);

      // Create upper teeth
      createTeeth(2.5, -1, -1, true);

      const gumMaterial = new THREE.MeshLambertMaterial({ color: 0xff6666 });

      // Lower gum
      const lowerGumGeometry = new THREE.TorusGeometry(3, 0.5, 16, 100, Math.PI);
      const lowerGum = new THREE.Mesh(lowerGumGeometry, gumMaterial);
      lowerGum.rotation.x = Math.PI / 2;
      lowerGum.position.y = -1;
      scene.add(lowerGum);

      // Upper gum
      const upperGumGeometry = new THREE.TorusGeometry(3, 0.5, 16, 100, Math.PI);
      const upperGum = new THREE.Mesh(upperGumGeometry, gumMaterial);
      upperGum.rotation.x = Math.PI / 2;
      upperGum.rotation.y = Math.PI;
      upperGum.position.y = 3;
      scene.add(upperGum);

      const cheekMaterial = new THREE.MeshLambertMaterial({ color: 0xff9999, side: THREE.DoubleSide });
      const cheekGeometry = new THREE.PlaneGeometry(12, 8);

      // Left cheek
      const leftCheek = new THREE.Mesh(cheekGeometry, cheekMaterial);
      leftCheek.position.set(-6, 0.5, 1);
      leftCheek.rotation.y = Math.PI / 2;
      scene.add(leftCheek);

      // Right cheek
      const rightCheek = new THREE.Mesh(cheekGeometry, cheekMaterial);
      rightCheek.position.set(6, 0.5, 1);
      rightCheek.rotation.y = -Math.PI / 2;
      scene.add(rightCheek);

      const tongueMaterial = new THREE.MeshLambertMaterial({ color: 0xff4d4d });
      const tongueGeometry = new THREE.PlaneGeometry(4, 6);
      const tongue = new THREE.Mesh(tongueGeometry, tongueMaterial);
      tongue.position.set(0, -1.5, 1);
      tongue.rotation.x = -Math.PI / 2;
      scene.add(tongue);

      // Position the camera closer to the teeth
      camera.position.set(0, 1, 5);
      camera.lookAt(0, 1, 0);

      const animate = function () {
        requestAnimationFrame(animate);
        renderer.render(scene, camera);
      };

      animate();

      window.addEventListener("resize", () => {
        camera.aspect = window.innerWidth / window.innerHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(window.innerWidth, window.innerHeight);
      });

      window.addEventListener("mousemove", (event) => {
        if (isDragging.value) {
          mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
          mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
        }
      });

      window.addEventListener("mousedown", (event) => {
        if (event.button === 2) {
          // Right mouse button
          isDragging.value = true;
        }
      });

      window.addEventListener("mouseup", () => {
        isDragging.value = false;
      });

      window.addEventListener("wheel", (event) => {
        camera.position.z += event.deltaY * 0.01;
        camera.position.z = THREE.MathUtils.clamp(camera.position.z, 2, 10); // Limit zoom
      });

      const update = function () {
        if (isDragging.value) {
          camera.position.x = mouse.x * 2;
          camera.position.y = mouse.y * 2 + 1;
          camera.lookAt(scene.position);
        }
        requestAnimationFrame(update);
      };

      update();
      const moveBacteria = (particle) => {
        const direction = new THREE.Vector3((Math.random() - 0.5) * 0.02, (Math.random() - 0.5) * 0.02, (Math.random() - 0.5) * 0.02);
        const moveBacteria = () => {
          particle.position.add(direction);
          requestAnimationFrame(moveBacteria);
        };
        moveBacteria();
      };
      window.addEventListener("apply-bacteria", (event) => {
        const bacteria = event.detail;
        console.log(`Bacteria applied: ${bacteria}`);

        if (bacteria === "Streptococcus mutans") {
          const bacteriaMaterial = new THREE.MeshLambertMaterial({ color: 0xff0000 });
          const bacteriaGeometry = new THREE.SphereGeometry(0.2, 16, 16);
          const bacteriaParticles = [];

          for (let i = 0; i < 50; i++) {
            const bacterium = new THREE.Mesh(bacteriaGeometry, bacteriaMaterial);
            bacterium.position.set((Math.random() - 0.5) * 6, (Math.random() - 0.5) * 6, (Math.random() - 0.5) * 6);
            bacteriaParticles.push(bacterium);
            scene.add(bacterium);
          }

          bacteriaParticles.forEach(moveBacteria);

          setTimeout(() => {
            bacteriaParticles.forEach((particle) => scene.remove(particle));
          }, 5000);
        }

        if (bacteria === "Lactobacillus") {
          const bacteriaMaterial = new THREE.MeshLambertMaterial({ color: 0x00ff00 });
          const bacteriaGeometry = new THREE.CylinderGeometry(0.1, 0.1, 0.5, 16);
          const bacteriaParticles = [];

          for (let i = 0; i < 50; i++) {
            const bacterium = new THREE.Mesh(bacteriaGeometry, bacteriaMaterial);
            bacterium.position.set((Math.random() - 0.5) * 6, (Math.random() - 0.5) * 6, (Math.random() - 0.5) * 6);
            bacterium.rotation.x = Math.random() * Math.PI;
            bacterium.rotation.y = Math.random() * Math.PI;
            bacteriaParticles.push(bacterium);
            scene.add(bacterium);
          }
          bacteriaParticles.forEach(moveBacteria);

          setTimeout(() => {
            bacteriaParticles.forEach((particle) => scene.remove(particle));
          }, 5000);
        }

        if (bacteria === "Actinomyces") {
          const bacteriaMaterial = new THREE.MeshLambertMaterial({ color: 0x0000ff });
          const bacteriaGeometry = new THREE.CylinderGeometry(0.05, 0.05, 0.5, 16);
          const bacteriaParticles = [];

          for (let i = 0; i < 50; i++) {
            const bacterium = new THREE.Mesh(bacteriaGeometry, bacteriaMaterial);
            bacterium.position.set((Math.random() - 0.5) * 6, (Math.random() - 0.5) * 6, (Math.random() - 0.5) * 6);
            bacterium.rotation.x = Math.random() * Math.PI;
            bacterium.rotation.y = Math.random() * Math.PI;
            bacteriaParticles.push(bacterium);
            scene.add(bacterium);
          }
          bacteriaParticles.forEach(moveBacteria);

          setTimeout(() => {
            bacteriaParticles.forEach((particle) => scene.remove(particle));
          }, 5000);
        }
      });

      window.addEventListener("evolve-transmission", (event) => {
        const method = event.detail;
        console.log(`Evolving transmission method: ${method}`);
        if (method === "Saliva Spread") {
          const salivaMaterial = new THREE.MeshLambertMaterial({ color: 0x88c0d0, opacity: 0.5, transparent: true });
          const salivaGeometry = new THREE.SphereGeometry(3.5, 32, 16);
          const saliva = new THREE.Mesh(salivaGeometry, salivaMaterial);
          saliva.position.set(0, 0, 0);
          scene.add(saliva);
          setTimeout(() => {
            scene.remove(saliva);
          }, 5000); // Saliva layer disappears after 5 seconds
        }
        // Apply changes to the 3D scene if needed
        if (method === "Food Residue") {
          const foodMaterial = new THREE.MeshLambertMaterial({ color: 0xa0522d });
          const foodGeometry = new THREE.SphereGeometry(0.3, 16, 16);
          const foodParticles = [];

          for (let i = 0; i < 10; i++) {
            const food = new THREE.Mesh(foodGeometry, foodMaterial);
            food.position.set((Math.random() - 0.5) * 6, (Math.random() - 0.5) * 6, (Math.random() - 0.5) * 6);
            foodParticles.push(food);
            scene.add(food);
          }
          setTimeout(() => {
            foodParticles.forEach((particle) => scene.remove(particle));
          }, 5000); // Food residue disappears after 2 seconds
        }
        // Apply changes to the 3D scene if needed
        if (method === "Plaque Formation") {
          const plaqueMaterial = new THREE.MeshLambertMaterial({ color: 0x8b4513, opacity: 0.8, transparent: true });
          const plaqueGeometry = new THREE.CylinderGeometry(0.35, 0.35, 0.1, 32);
          const plaqueParticles = [];

          for (let i = 0; i < 16; i++) {
            const tooth = teeth[i];
            const plaque = new THREE.Mesh(plaqueGeometry, plaqueMaterial);
            plaque.position.set(tooth.position.x, tooth.position.y + 0.55, tooth.position.z);
            plaque.rotation.x = Math.PI / 2;
            plaqueParticles.push(plaque);
            scene.add(plaque);
          }
          for (let i = 16; i < 32; i++) {
            const tooth = teeth[i];
            const plaque = new THREE.Mesh(plaqueGeometry, plaqueMaterial);
            plaque.position.set(tooth.position.x, tooth.position.y - 0.55, tooth.position.z);
            plaque.rotation.x = Math.PI / 2;
            plaqueParticles.push(plaque);
            scene.add(plaque);
          }
          setTimeout(() => {
            plaqueParticles.forEach((particle) => scene.remove(particle));
          }, 10000); // Plaque formation disappears after 10 seconds
        }
      });

      window.addEventListener("evolve-symptom", (event) => {
        const symptom = event.detail;
        console.log(`Evolving symptom: ${symptom}`);
        if (symptom === "Enamel Erosion") {
          const erosionMaterial = new THREE.MeshLambertMaterial({ color: 0xd3d3d3 });
          for (let i = 0; i < 32; i++) {
            const tooth = teeth[i];
            tooth.material = erosionMaterial;
          }
        }
        if (symptom === "Cavity Formation") {
          const cavityMaterial = new THREE.MeshLambertMaterial({ color: 0x000000 });
          const cavityGeometry = new THREE.SphereGeometry(0.2, 16, 16);
          const cavities = [];

          for (let i = 0; i < 32; i++) {
            const tooth = teeth[i];
            const cavity = new THREE.Mesh(cavityGeometry, cavityMaterial);
            cavity.position.set(tooth.position.x + (Math.random() - 0.5) * 0.2, tooth.position.y + (Math.random() - 0.5) * 0.2, tooth.position.z + (Math.random() - 0.5) * 0.2);
            cavities.push(cavity);
            scene.add(cavity);
          }
        }
        if (symptom === "Gum Recession") {
          const lowerGum = scene.getObjectByName("lowerGum");
          const upperGum = scene.getObjectByName("upperGum");
          if (lowerGum) lowerGum.position.y -= 0.2;
          if (upperGum) upperGum.position.y += 0.2;
        }
      });

      window.addEventListener("evolve-ability", (event) => {
        const ability = event.detail;
        console.log(`Evolving ability: ${ability}`);
        if (ability === "Acid Resistance") {
          const acidResistantMaterial = new THREE.MeshLambertMaterial({ color: 0xffd700 });
          for (let i = 0; i < 32; i++) {
            const tooth = teeth[i];
            tooth.material = acidResistantMaterial;
          }
        }
        if (ability === "Biofilm Fortification") {
          const biofilmMaterial = new THREE.MeshLambertMaterial({ color: 0x556b2f, opacity: 0.5, transparent: true });
          const biofilmGeometry = new THREE.CylinderGeometry(0.4, 0.4, 1.2, 32);
          const biofilms = [];

          for (let i = 0; i < 32; i++) {
            const tooth = teeth[i];
            const biofilm = new THREE.Mesh(biofilmGeometry, biofilmMaterial);
            biofilm.position.set(tooth.position.x, tooth.position.y, tooth.position.z);
            biofilm.rotation.x = Math.PI / 2;
            biofilms.push(biofilm);
            scene.add(biofilm);
          }

          setTimeout(() => {
            biofilms.forEach((biofilm) => scene.remove(biofilm));
          }, 5000); // Biofilm disappears after 2 seconds
        }
        if (ability === "Antibiotic Resistance") {
          const antibioticResistantMaterial = new THREE.MeshLambertMaterial({ color: 0x8b0000 });
          for (let i = 0; i < 32; i++) {
            const tooth = teeth[i];
            tooth.material = antibioticResistantMaterial;
          }
        }
      });

      window.addEventListener("update-game-status", (event) => {
        const { infectedTeeth, destroyedTeeth } = event.detail;
        console.log(`Infected Teeth: ${infectedTeeth}, Destroyed Teeth: ${destroyedTeeth}`);
        // Update the 3D scene to reflect the current game status
      });
    });

    return { sceneContainer, teeth };
  },
};
</script>

<style scoped>
.scene-container {
  width: 100%;
  height: 100vh;
  overflow: hidden;
  background-color: #f8e0b2;
}
</style>
