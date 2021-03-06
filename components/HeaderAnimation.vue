<script setup>
import * as THREE from 'three'
import * as lil from 'lil-gui'
import { gsap } from 'gsap'

const textures = [
  '/textures/texture.png',
  '/textures/texture-1.png',
  '/textures/texture-2.png',
  '/textures/texture-3.png',
  '/textures/texture-4.png',
  '/textures/texture-5.png',
  '/textures/texture-6.png',
  '/textures/texture-7.png',
  '/textures/texture-8.png'
]

onMounted(() => {
  const loadingMangaer = new THREE.LoadingManager()
  const textureLoader = new THREE.TextureLoader(loadingMangaer)
  const colorTextures = []

  for (const texture of textures) {
    colorTextures.push(textureLoader.load(texture))
  }
  const alphaTexture = textureLoader.load('/textures/card-alpha-map.jpg')

  // Scene
  const scene = new THREE.Scene() 

  // Object

  //Cards 
  const cardGeometry = new THREE.PlaneGeometry( 2, 2 )
  const cardMaterials = []
  for (const colorTexture of colorTextures) {
    cardMaterials.push(new THREE.MeshBasicMaterial({
      map: colorTexture,
      transparent: true,
      alphaMap: alphaTexture
    }))
  }

  const numCard = 50
  const cards = [ ]

  for (let i = 0; i < numCard; i++) {
    const mesh = new THREE.Mesh(cardGeometry, cardMaterials[i % cardMaterials.length])
    cards[i] = { mesh: mesh, speed: (Math.random() * 10) + 10 }
    const movement = Math.random() - 0.5
    cards[i].mesh.rotation.z = -movement * 0.25
    cards[i].mesh.position.x = movement * 13
    cards[i].mesh.position.y = -10 + (Math.random() -0.5) * 10
    scene.add(cards[i].mesh)
  }
  //Front cards
  const cardsSettings = [
    { 
      position: new THREE.Vector3( -4, 5, 1.4 ),
      targetPosition: new THREE.Vector3( -4, -0.5, 1.4 ),
      rotation: new THREE.Vector3( 0, 0, 0.2 ),
      material: cardMaterials[5]
    },
    {
      position: new THREE.Vector3( -2.6, 5, 1 ),
      targetPosition: new THREE.Vector3( -2.6, -0.8, 1 ), 
      rotation: new THREE.Vector3( 0, 0, 0.1 ),
      material: cardMaterials[1]
    },
    {
      position: new THREE.Vector3( 0, 5, 1.8 ),
      targetPosition: new THREE.Vector3( 0, -1, 1.8 ),
      rotation: new THREE.Vector3( 0, 0, 0 ),
      material: cardMaterials[2]
    },
    {
      position: new THREE.Vector3( 2.5, 5, 1 ),
      targetPosition: new THREE.Vector3( 2.5, -0.8, 1 ),
      rotation: new THREE.Vector3( 0, 0, -0.1 ),
      material: cardMaterials[8]
    },
    {
      position: new THREE.Vector3( 3.8, 10, 1.5 ),
      targetPosition: new THREE.Vector3( 3.8, -0.6, 1.5 ),
      rotation: new THREE.Vector3( 0, 0, -0.2 ),
      material: cardMaterials[4]
    }
  ]

  const frontCard = []

  for (let i = 0; i < cardsSettings.length; i++) {
    frontCard[i] = new THREE.Mesh(cardGeometry, cardsSettings[i].material)
    frontCard[i].position.set(cardsSettings[i].position.x, cardsSettings[i].position.y, cardsSettings[i].position.z)
    frontCard[i].rotation.set(cardsSettings[i].rotation.x, cardsSettings[i].rotation.y, cardsSettings[i].rotation.z)
    const timeline = gsap.timeline()
    timeline.delay(2)
    timeline.to(frontCard[i].position, { duration: 3, x: cardsSettings[i].targetPosition.x, y: cardsSettings[i].targetPosition.y, z: cardsSettings[i].targetPosition.z } )
    scene.add(frontCard[i])
  }

  //Particules
  const parameters = {}
  parameters.particulesCount = 500

  let geometry = null
  let material = null
  let points = null

  const generateParticules = (renderer) => {
    if (points != null) {
      geometry.dispose()
      material.dispose()
      scene.remove(points)
    }
    geometry = new THREE.BufferGeometry()
    const position = new Float32Array(parameters.particulesCount * 3)
    const randomnes = new Float32Array(parameters.particulesCount * 3)
    const color = new Float32Array(parameters.particulesCount * 3)

    for (let i = 0; i < parameters.particulesCount; i++) {
      const i3 = i * 3
      const radius = Math.random() * parameters.radius
      const randomX = (Math.random() - 0.5)
      const randomY = (Math.random() - 0.5)
      const randomZ = (Math.random() - 1)

      position[i3] = randomX * 20
      position[i3 + 1] = 0
      position[i3 + 2] = randomZ

      randomnes[i3    ] = Math.random() - 0.5
      randomnes[i3 + 1] = Math.random() * 0.5
      randomnes[i3 + 2] = Math.random() + 0.2

      color[i3] = 0.9
      color[i3 + 1] = 0.9
      color[i3 + 2] = 0.9
    }

    geometry.setAttribute('position', new THREE.BufferAttribute(position, 3))
    geometry.setAttribute('aRandomness', new THREE.BufferAttribute(randomnes, 3))
    geometry.setAttribute('color', new THREE.BufferAttribute(color, 3))

    material = new THREE.ShaderMaterial({
      vertexColors: true,
      fragmentShader: `
        varying vec3 vColor;

        void main()
        {
          
          gl_FragColor = vec4(vColor,  1.0);
        }
      `,
      vertexShader: `
        uniform float uTime;
        uniform float uSize;

        attribute vec3 aRandomness;
        attribute float aScale;

        varying vec3 vColor; 

        float easing(float t){
            return t * t * (3.0 - 2.0 * t);
        }

        void main () {
          vec4 modelPosition = modelMatrix * vec4(position, 1.0);
          float angle = atan(modelPosition.x, modelPosition.z);
          float distanceToCenter = length(modelPosition.xz);
          modelPosition.y -= uTime * aRandomness.z;
          modelPosition.xy += aRandomness.xy;
          modelPosition.x += easing(sin(uTime * aRandomness.z - aRandomness.x) + 0.5) * 0.5 ;
          modelPosition.y = mod(modelPosition.y, 7.5) - 3.;
          vec4 viewPosition = viewMatrix * modelPosition;
          vec4 projectedPosition = projectionMatrix * viewPosition;
          gl_Position = projectedPosition;

          gl_PointSize = uSize;
          gl_PointSize *= (1.0 / - viewPosition.z);
          vColor = color;
        }
      `,
      uniforms: {
        uTime: { value: 0 },
        uSize: { value: 15 * renderer.getPixelRatio() } 
      }
    })

    points = new THREE.Points(geometry, material)
    points.scale.y = 2
    scene.add(points)
  }

  // Sizes 
  const sizes = {
    width: window.innerWidth,
    height: 500
  }

  // Camera
  const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height, 0.1, 100)
  camera.position.z = 3
  scene.add(camera)

  //Raycaster
  const raycaster = new THREE.Raycaster()

  const mouse = new THREE.Vector2()
  window.addEventListener('mousemove', (event) =>
  {
      mouse.x = event.clientX / sizes.width * 2 - 1
      mouse.y = - (event.clientY / sizes.height) * 2 + 1
  })

  // Renderer
  const canvas = document.querySelector('canvas.webgl')
  const renderer = new THREE.WebGLRenderer({
      canvas: canvas
  })
  renderer.setClearColor(0x000000, 0.0)
  renderer.setSize(sizes.width, sizes.height)
  renderer.render(scene, camera)
  generateParticules(renderer)
  
  const clock = new THREE.Clock()

  let previousTime = 0.0
  let currentIntersect = []
  const tick = () =>
  {
    const elapsedTime = clock.getElapsedTime()
    const deltaTime = elapsedTime - previousTime
    previousTime = elapsedTime

    material.uniforms.uTime.value = elapsedTime

    if (elapsedTime < 4){
      for(let card of cards) {
        card.mesh.position.y += deltaTime * card.speed
        card.mesh.position.x += deltaTime * (-card.mesh.rotation.z * 7)
      }
    }

    raycaster.setFromCamera(mouse, camera)

    const intersects = raycaster.intersectObjects(frontCard)
    if(elapsedTime > 5 && intersects.length > currentIntersect.length)
    {
      for (let intersect of intersects) {
        if (!currentIntersect.includes(intersect.object)) {
          gsap.to(intersect.object.position, { duration: 0.5, y: intersect.object.position.y + 0.2} )
          currentIntersect.push(intersect.object)
        }
      }
    }
    else
    {
      for (let i = 0; i < currentIntersect.length; i++) {
        if (intersects.find(intersect => currentIntersect[i] === intersect.object) == null) {
          const cardIndex = frontCard.findIndex(card => card.uuid === currentIntersect[i].uuid)
          gsap.to(currentIntersect[i].position, { duration: 0.5, y: cardsSettings[cardIndex].targetPosition.y } )
          currentIntersect.splice(i, 1)
        }
      }
    }

    // Render
    renderer.render(scene, camera)
    window.requestAnimationFrame(tick)
  }

tick()
})
</script>

<template>
<div class="animation-background">
  <canvas id="header-animation" class="webgl">
</canvas>
</div>
</template>

<style scoped>
.animation-background {
  background: linear-gradient(217deg, #FF5F7E, rgba(255,0,0,0) 70.71%),
    linear-gradient(127deg, #F9A36D, rgba(0,255,0,0) 70.71%),
    linear-gradient(336deg, #9C67FD, rgba(0,0,255,0) 70.71%);
}
</style>