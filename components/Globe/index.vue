<template>
  <div class="Globe" ref="root" />
</template>

<script setup lang="ts">

import * as T3 from "three";
import { OrbitControls } from "three/addons";
import { createEarth,createSunWind,createTexture, createSun, createEarthClouds, createSkyBox, drawDefaultMarker, type IGlobePoint, cord } from "./utils";

interface IComponentProps {
  markers?: Array<IGlobePoint>
  drawMarker?: (v: IGlobePoint) => T3.Mesh
  drawMarkers?: (s: T3.Mesh, v: Array<IGlobePoint>) => void
}

const { markers, drawMarker, drawMarkers } = defineProps<IComponentProps>()

const root = ref<HTMLDivElement | undefined>();
const animationId = ref<number>(0)

const scene = new T3.Scene();
const camera = new T3.PerspectiveCamera();

const camera1 = new T3.PerspectiveCamera();



const renderer = new T3.WebGLRenderer({
  antialias: true
});
const raycaster = new T3.Raycaster()
const mouse = new T3.Vector2()
const touch = new T3.Vector2()

const getW = () => unref(root)?.clientWidth ?? innerWidth
const getH = () => unref(root)?.clientHeight ?? innerHeight

const setupCamera = () => {
  const w = getW();
  const h = getH();
  camera.aspect = w / h;
  camera.updateProjectionMatrix();
  renderer.setSize(w, h);
};



camera.rotation.y = 0.01;

//CONTROLING
////SPEED_Rot
const rotation_speed = 0.01;
////CAMERA
const controls = new OrbitControls(camera, renderer.domElement);
controls.minDistance = 50
controls.maxDistance = 450
controls.enablePan = false;
controls.update()
controls.saveState()

camera.position.z = 20;


// controls = new THREE.OrbitControls( camera, renderer.domElement );
// controls.target.set( x, y, z );


// const terminator = calcTerminator();
// console.log(terminator)


scene.background = createSkyBox()

//SUN
const point = new T3.Object3D();
point.position.set(0,0,0);
const sun = createSun()
const sunwind = createSunWind()
const animateSunRotation = () => {
  sun.position.set(0,0,0);
  sun.rotation.y -= rotation_speed;
  sunwind.rotation.y +=rotation_speed;
  sunwind.position.set(0,0,0);
};


//EARTHPOSITIONFUNCTION


// function getEarthTiltAngle(date) {
//   const dayOfYear = Math.floor((date - new Date(date.getFullYear(), 0, 1)) / (1000 * 60 * 60 * 24)) + 1;
//   const tiltAngle = 23.44 * Math.cos(((2 * Math.PI) / 365) * (dayOfYear - 81));
//   return tiltAngle * Math.PI/180;
// }

// const timeinput = prompt("ГГГГ-ММ-ДД ЧЧ:ММ:SS (например, 2022-01-01 00:00:00):");
// if(!timeinput){
//   console.log('Нет данных')
// }
// else {
//   const inputDate=new Date(timeinput);
//   if(isNaN(inputDate))
// }




//EARTH ROTATE
let year = prompt("Введите год (YYYY):");
let month = prompt("Введите месяц (1-12):");
let day = prompt("Введите день (1-31):");
let hour = prompt("Введите час (0-23):");
let minute = prompt("Введите минуту (0-59):");
let second = prompt("Введите секунду (0-59):");

let inputDate = new Date(parseInt(year), parseInt(month) - 1, parseInt(day), parseInt(hour), parseInt(minute), parseInt(second));

let startOfYear = new Date(inputDate.getFullYear(), 0, 0);
let diff = inputDate.getTime() - startOfYear.getTime();
let oneDay = 1000 * 60 * 60 * 24;
let dayOfYear = Math.floor(diff / oneDay);



let sunAngle = (dayOfYear * (360 / 365)) * Math.PI / 180;
let totalSeconds = parseInt(hour) * 3600 + parseInt(minute) * 60 + parseInt(second);
let earthRotationAngle = ((totalSeconds / (24 * 60 * 60)) * 360) * Math.PI / 180;







//EARTH
const earth = createEarth()
const earthClouds = createEarthClouds()
const tiltAngle = 23.4 * Math.PI / 180;

const animateEarthRotation = () => {
  earth.position.set(150,0,0);
  earth.rotation.x = tiltAngle;
  earth.rotation.y = 271; 
};
//
const orbitGeometry = new T3.TorusGeometry(150, 1, 64, 64);
const orbitMaterial = new T3.MeshBasicMaterial({ color: 0xffffff, transparent: true, opacity: 0.3 });
const earthOrbit = new T3.Mesh(orbitGeometry, orbitMaterial);
scene.add(earthOrbit);
earthOrbit.position.set(0, 0, 0);
earthOrbit.rotation.x = T3.MathUtils.degToRad(90);




const animate = () => {
  animationId.value = requestAnimationFrame(animate);
  animateEarthRotation();
  animateSunRotation();
  renderer.render(scene, camera);
};
animate();



// //
// //READLATITUDELONGETUDE
// //
// const latlon = require('latlon');
// const data = latlon.readFileSync('data.json');
// const cord = JSON.parse(data);
// //

const textureDed1 = createTexture(1920,1080,cord)

const geometryDed = new T3.SphereGeometry(10,32,32)
const materialDed = new T3.MeshBasicMaterial({
  map:textureDed1,
  opacity:0.8,
  transparent:true,
})
const result = new T3.Mesh(geometryDed,materialDed)
earth.add(result)

//ADD THINGS
earth.add(earthClouds)
earthClouds.add(result)
scene.add(earth)
scene.add(sun)
scene.add(point)
sun.add(sunwind)
// scene.add(create_earth_orbit)
//position of earth



//AROUND_point
point.add(earth);
function _render(){
  point.rotation.y = sunAngle;
    window.requestAnimationFrame(_render);
    renderer.render(scene, camera);
  }
_render();

//lights
const light = new T3.PointLight('white',100000);
const light1 = new T3.PointLight('white',4500,60);
const light2 = new T3.PointLight('white',4500,60);
const light3 = new T3.PointLight('white',4500,60);
const light4 = new T3.PointLight('white',4500,60);
const light5 = new T3.PointLight('white',4500,60);
const light6 = new T3.PointLight('white',4500,60);
light1.rotation.x = T3.MathUtils.degToRad(-90);;
light1.position.set(0,50,0)
light.add(sun);
light.position.set(0,0,0)
light2.position.set(0,-50,0)
light3.position.set(50,0,0)
light4.position.set(-50,0,0)
light5.position.set(0,0,-50)
light6.position.set(0,0,50)
scene.add(light,light1,light2,light3,light4,light5,light6)
//



if (markers) {
  if (!!drawMarkers) {
    drawMarkers(earthClouds, markers)
  } else {
    earthClouds.add(...(markers.map(marker => drawMarker?.(marker) ?? drawDefaultMarker(marker))))
  }
}

const onResize = () => {
  setupCamera();
};

const onInteract = (event: MouseEvent) => {
  event.preventDefault()

  const w = getW();
  const h = getH();

  mouse.x = (event.clientX / w) * 2 - 1
  mouse.y = - (event.clientY / h) * 2 + 1
  raycaster.setFromCamera(mouse, camera)

  const intersects = raycaster.intersectObjects(earthClouds.children)
  intersects.forEach(i => {
    console.log(i);
    
  })
}

onMounted(() => {
  unref(root)!.appendChild(renderer.domElement);
  setupCamera();

  addEventListener("resize", onResize);
  addEventListener('click', onInteract)

  const animate = () => {
    animationId.value = requestAnimationFrame(animate);
    renderer.render(scene, camera);
  };
  animate();
});

onBeforeUnmount(() => {
  removeEventListener("resize", onResize);
  removeEventListener('click', onInteract);

  cancelAnimationFrame(unref(animationId))

  let i = earthClouds.children.length
  while (i--) {
    const c: T3.Mesh = earthClouds.children[i] as T3.Mesh

    if (Array.isArray(c.material)) {
      c.material.forEach(m => m.dispose())
    } else {
      c.material.dispose()
    }

    c.geometry.dispose()

    earthClouds.remove(earthClouds.children[i])
  }
});
</script>

<style scoped>
.Globe {
  width: 100%;
  height: 100%;
}
</style>
