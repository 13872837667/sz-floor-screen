<script setup>
import { onMounted, ref, onUnmounted } from 'vue'
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

const sceneContainer = ref(null)
let scene, camera, renderer, controls, loader

/**
 * 获取 Mesh 在世界空间中的包围盒
 * @param {THREE.Mesh} mesh
 * @returns {THREE.Box3}
 */
function getWorldBoundingBox(mesh) {
  // 确保几何体有包围盒
  if (!mesh.geometry.boundingBox) {
    mesh.geometry.computeBoundingBox()
  }
  const localBox = mesh.geometry.boundingBox.clone()
  // 将局部包围盒转换到世界空间
  const worldBox = localBox.applyMatrix4(mesh.matrixWorld)
  return worldBox
}

/**
 * 调暗地面材质（颜色变暗，漫反射减弱）
 * @param {THREE.Object3D} model
 * @param {number} groundThreshold 地面判定阈值（世界Y坐标最小值低于此值视为地面）
 * @param {number} darkenFactor 颜色压暗系数（0-1，越小越暗）
 */
function darkenGroundMaterials(model, groundThreshold = 0.2, darkenFactor = 0.5) {
  model.traverse((child) => {
    if (child.isMesh) {
      // 获取该Mesh在世界中的包围盒
      const worldBox = getWorldBoundingBox(child)
      // 若最低点接近地面（Y值较小），则视为地面部分
      if (worldBox.min.y <= groundThreshold) {
        const materials = Array.isArray(child.material) ? child.material : [child.material]
        materials.forEach(mat => {
          if (mat && mat.isMaterial) {
            // 针对标准材质进行颜色压暗
            if (mat.color) {
              mat.color.multiplyScalar(darkenFactor)
            }
            // 降低自发光强度，使地面更暗
            if (mat.emissiveIntensity !== undefined) {
              mat.emissiveIntensity *= 0.3
            }
            // 略微增加粗糙度，减少镜面反射，使视觉上更暗沉
            if (mat.roughness !== undefined) {
              mat.roughness = Math.min(mat.roughness * 1.2, 1.0)
            }
            // 需要重新赋值以触发渲染更新
            mat.needsUpdate = true
          }
        })
      }
    }
  })
}

const initThree = () => {
  if (!sceneContainer.value) return

  // 场景
  scene = new THREE.Scene()

  // 相机
  camera = new THREE.PerspectiveCamera(
    45,
    sceneContainer.value.clientWidth / sceneContainer.value.clientHeight,
    0.1,
    2000
  )

  // 渲染器
  renderer = new THREE.WebGLRenderer({
    antialias: true,
    alpha: true,
    logarithmicDepthBuffer: true
  })
  renderer.setSize(sceneContainer.value.clientWidth, sceneContainer.value.clientHeight)
  renderer.setPixelRatio(window.devicePixelRatio)
  sceneContainer.value.appendChild(renderer.domElement)

  // 控制器
  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true

  // ========== 灯光调整（使地面灯光更暗） ==========
  // 大幅降低环境光强度，从 0.3 降至 0.12，让整体光线更暗，尤其影响地面
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.12)
  scene.add(ambientLight)

  // 可选：增加一个极弱的顶部方向光，避免非地面物体过暗看不清（强度很低，不影响地面暗化效果）
  const fillLight = new THREE.DirectionalLight(0xccddff, 0.25)
  fillLight.position.set(5, 10, 7)
  scene.add(fillLight)

  // 加载模型
  loader = new GLTFLoader()
  loader.load(
    'http://s2b2c-prod.oss-cn-shenzhen.aliyuncs.com/0000/file/apk/map.glb',
    (gltf) => {
      const model = gltf.scene

      // 移除 GLB 模型中自带的灯光
      model.traverse((child) => {
        if (child.isLight) {
          child.intensity = 0
          child.visible = false
        }
      })

      scene.add(model)

      // 水平居中模型，并将底部放置在地面上
      const box = new THREE.Box3().setFromObject(model)
      const center = box.getCenter(new THREE.Vector3())
      model.position.x -= center.x
      model.position.y -= box.min.y // 底部置于 y=0
      model.position.z -= center.z

      model.rotation.x = Math.PI / 6 // 绕x轴旋转
      model.position.x += 5          // 沿x轴方向水平移动

      // 自动缩放模型到稳定的尺寸
      const size = box.getSize(new THREE.Vector3())
      const maxDim = Math.max(size.x, size.y, size.z)
      const scale = 600 / maxDim
      model.scale.set(scale, scale, scale)

      // ====== 关键：调暗地面材质（颜色压暗50%） ======
      // 在模型位置、缩放确定后，其世界矩阵已更新，可准确识别地面部分
      darkenGroundMaterials(model, 0.25, 0.5) // 阈值0.25，颜色亮度减半

      // ====== 相机位置修复（与原逻辑一致） ======
      const scaledHeight = size.y * scale
      const scaledWidth = size.x * scale
      const cameraHeight = scaledHeight * 0.4
      const cameraDistance = Math.max(scaledWidth, scaledHeight) * 0.05
      camera.position.set(0, cameraHeight, cameraDistance)
      controls.target.set(0, scaledHeight * 0.4, 0)
      camera.lookAt(controls.target)
      controls.update()
    },
    (xhr) => {
      console.log((xhr.loaded / xhr.total * 100) + '% loaded')
    },
    (error) => {
      console.error('发生错误', error)
    }
  )

  const animate = () => {
    requestAnimationFrame(animate)
    controls.update()
    renderer.render(scene, camera)
  }

  animate()
}

const handleResize = () => {
  if (!sceneContainer.value || !camera || !renderer) return
  const width = sceneContainer.value.clientWidth
  const height = sceneContainer.value.clientHeight
  camera.aspect = width / height
  camera.updateProjectionMatrix()
  renderer.setSize(width, height)
}

onMounted(() => {
  initThree()
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
  if (renderer) {
    renderer.dispose()
  }
  if (controls) {
    controls.dispose()
  } 
  scene = null
  camera = null
  renderer = null
  controls = null
  loader = null
})
</script>

<template>
  <div class="three-container" ref="sceneContainer"></div>
</template>

<style lang="scss" scoped>
.three-container {
  flex: 1;
  width: 100%;
  position: relative;
  overflow: hidden;
}
</style>