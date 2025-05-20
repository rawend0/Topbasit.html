<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>3D İnsan ve Zombi Örneği (Three.js)</title>
<style>body { margin: 0; overflow: hidden; }</style>
</head>
<body>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/examples/js/loaders/GLTFLoader.js"></script>

<script>
  const scene = new THREE.Scene();
  scene.background = new THREE.Color(0x222222);

  const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
  camera.position.set(0, 1.7, 5);

  const renderer = new THREE.WebGLRenderer({antialias:true});
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  const light = new THREE.HemisphereLight(0xffffff, 0x444444, 1);
  scene.add(light);

  const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
  directionalLight.position.set(3, 10, 10);
  scene.add(directionalLight);

  // Zemin
  const floorGeometry = new THREE.PlaneGeometry(50, 50);
  const floorMaterial = new THREE.MeshStandardMaterial({color: 0x555555});
  const floor = new THREE.Mesh(floorGeometry, floorMaterial);
  floor.rotation.x = -Math.PI/2;
  scene.add(floor);

  // GLTF Loader
  const loader = new THREE.GLTFLoader();

  // Oyuncu modeli (insan)
  let player, playerMixer;
  loader.load('https://threejs.org/examples/models/gltf/RobotExpressive/RobotExpressive.glb', function(gltf) {
    player = gltf.scene;
    player.scale.set(1.5,1.5,1.5);
    player.position.set(0,0,0);
    scene.add(player);

    playerMixer = new THREE.AnimationMixer(player);
    const action = playerMixer.clipAction(gltf.animations[0]);
    action.play();
  });

  // Zombi modeli
  const zombies = [];
  const zombieMixers = [];

  // Zombi pozisyonları
  const zombiePositions = [
    {x:5, z:-5},
    {x:-5, z:-7},
    {x:3, z:-10}
  ];

  zombiePositions.forEach(pos => {
    loader.load('https://raw.githubusercontent.com/KhronosGroup/glTF-Sample-Models/master/2.0/Zombie/glTF/Zombie.gltf', function(gltf) {
      const zombie = gltf.scene;
      zombie.scale.set(1.2,1.2,1.2);
      zombie.position.set(pos.x, 0, pos.z);
      scene.add(zombie);

      const mixer = new THREE.AnimationMixer(zombie);
      const action = mixer.clipAction(gltf.animations[0]);
      action.play();

      zombies.push(zombie);
      zombieMixers.push(mixer);
    });
  });

  // Hareket ve animasyon için değişkenler
  const clock = new THREE.Clock();
  const keys = {};
  const moveSpeed = 0.05;

  window.addEventListener('keydown', e => keys[e.key.toLowerCase()] = true);
  window.addEventListener('keyup', e => keys[e.key.toLowerCase()] = false);

  function animate() {
    requestAnimationFrame(animate);
    const delta = clock.getDelta();

    // Animasyonları güncelle
    if(playerMixer) playerMixer.update(delta);
    zombieMixers.forEach(mixer => mixer.update(delta));

    // Oyuncu hareketi
    if(player){
      if(keys['w'] || keys['arrowup']) player.position.z -= moveSpeed;
      if(keys['s'] || keys['arrowdown']) player.position.z += moveSpeed;
      if(keys['a'] || keys['arrowleft']) player.position.x -= moveSpeed;
      if(keys['d'] || keys['arrowright']) player.position.x += moveSpeed;

      // Kamera takip
      camera.position.x = player.position.x;
      camera.position.z = player.position.z + 5;
      camera.position.y = player.position.y + 2;
      camera.lookAt(player.position);

      // Zombiler oyuncuya doğru koşuyor
      zombies.forEach(zombie => {
        const dirX = player.position.x - zombie.position.x;
        const dirZ = player.position.z - zombie.position.z;
        const dist = Math.sqrt(dirX*dirX + dirZ*dirZ);
        if(dist > 0.1){
          zombie.position.x += (dirX/dist)*0.02;
          zombie.position.z += (dirZ/dist)*0.02;
          zombie.lookAt(player.position);
        }
      });
    }

    renderer.render(scene, camera);
  }

  animate();

  window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth/window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  });
</script>
</body>
</html>
