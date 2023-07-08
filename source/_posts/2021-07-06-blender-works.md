---
layout: article
title: '[Blender] 作品集'
date: 2021-07-06 21:51
permalink: /blender-works/
tags:
- Blender
- 作品集
---
按下鍵盤左右鍵切換模型
<canvas id="c" tabindex="0" style="width: 100%"></canvas>
<script type="module">
  import * as THREE from 'https://threejsfundamentals.org/threejs/resources/threejs/r127/build/three.module.js';
  import {OrbitControls} from 'https://threejsfundamentals.org/threejs/resources/threejs/r127/examples/jsm/controls/OrbitControls.js';
  import {GLTFLoader} from 'https://threejsfundamentals.org/threejs/resources/threejs/r127/examples/jsm/loaders/GLTFLoader.js';

  const canvas = document.querySelector('#c');
  canvas.addEventListener('keydown', (e) => {
    if (e.keyCode == 37) {
        index = (index + models.length - 1) % models.length;
        loadModel();
    } else if (e.keyCode == 39) {
        index = (index + 1) % models.length;
        loadModel();
    }
  });
  const renderer = new THREE.WebGLRenderer({canvas});
  renderer.setSize(canvas.clientWidth, canvas.clientHeight);

  const fov = 45;
  const aspect = canvas.clientWidth / canvas.clientHeight;
  const near = 0.1;
  const far = 100;
  const camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
  camera.position.set(1, 1, 3);

  const controls = new OrbitControls(camera, canvas);

  const clock = new THREE.Clock();

  const scene = new THREE.Scene();
  scene.background = new THREE.Color(0xB3DDD5);

  // {
  //   const skyColor = 0xB1E1FF;
  //   const groundColor = 0xB97A20;
  //   const intensity = 1;
  //   const light = new THREE.HemisphereLight(skyColor, groundColor, intensity);
  //   scene.add(light);
  // }

  // {
  //   const color = 0xFFFFFF;
  //   const intensity = 1;
  //   const light = new THREE.DirectionalLight(color, intensity);
  //   light.position.set(5, 10, 2);
  //   scene.add(light);
  //   scene.add(light.target);
  // }

  {
    const color = 0xFFFFFF;
    const intensity = 1;
    const light = new THREE.AmbientLight(color, intensity);
    scene.add(light);
  }

  const models = ['Finn', 'Jake', 'PB', 'Spear', 'tree sword'];
  let index = 0;
  let root;
  let mixer;
  const gltfLoader = new GLTFLoader();

  function loadModel() {
    if (root) {
      scene.remove(root);
    }
    gltfLoader.load(`/assets/${models[index]}.glb`, (gltf) => {
      root = gltf.scene;
      scene.add(root);

      // compute the box that contains all the stuff
      // from root and below
      const box = new THREE.Box3().setFromObject(root);

      const boxSize = box.getSize(new THREE.Vector3()).length();
      const boxCenter = box.getCenter(new THREE.Vector3());

      // update the Trackball controls to handle the new size
      controls.maxDistance = boxSize * 10;
      controls.target.copy(boxCenter);
      controls.update();

      mixer = new THREE.AnimationMixer(root);
      gltf.animations.forEach(function(clip) {
        mixer.clipAction(clip).play();
      });
    });
  }
  loadModel();

  function render() {
    if (mixer) {
      mixer.update(clock.getDelta());
    }
    renderer.render(scene, camera);
    requestAnimationFrame(render);
  }
  requestAnimationFrame(render);
</script>