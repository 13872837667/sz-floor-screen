<script setup>
import { onMounted, ref, onUnmounted } from 'vue'
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'

const sceneContainer = ref(null)
let scene, camera, renderer, controls, loader, animationId

// 射线检测相关变量
let raycaster, mouse, mouseDownPos = { x: 0, y: 0 }

// 弹窗状态
const showPopup = ref(false)
const selectedFloor = ref(null)

// --- 选中高亮相关变量 ---
let lastSelectedObject = null
const originalMaterials = new Map()

const highlightMaterial = new THREE.MeshStandardMaterial({
  color: 0x7FFF00, // 亮绿色 (Chartreuse)
  transparent: true,
  opacity: 0.7,
  emissive: 0x7FFF00,
  emissiveIntensity: 1.2, // 进一步增加自发光，使其呈现荧光亮绿
  side: THREE.DoubleSide,
  depthWrite: false, // 允许透明层级穿透，增强视觉叠加感
  blending: THREE.NormalBlending
})

const restoreMaterials = () => {
  if (lastSelectedObject) {
    lastSelectedObject.traverse(child => {
      if (child.isMesh && originalMaterials.has(child.uuid)) {
        child.material = originalMaterials.get(child.uuid)
      }
    })
    lastSelectedObject = null
    originalMaterials.clear()
  }
}

const applyHighlight = (object) => {
  restoreMaterials()
  lastSelectedObject = object
  object.traverse(child => {
    if (child.isMesh) {
      if (!originalMaterials.has(child.uuid)) {
        originalMaterials.set(child.uuid, child.material)
      }
      child.material = highlightMaterial
    }
  })
}
// -----------------------

const floorInfo = ref({
  name: '',
  totalTasks: 0,
  pendingTasks: 0,
  completedTasks: 0
})

// 模拟楼层数据
const floorDataMap = {
  '14F': { name: '14F 交换区', totalTasks: 120, pendingTasks: 100, completedTasks: 20 },
  '11F': { name: '11F 无菌注射器车间（眼罩）', totalTasks: 85, pendingTasks: 70, completedTasks: 15 },
  '10F': { name: '10F 工厂车间（棉柔巾）实验室', totalTasks: 100, pendingTasks: 90, completedTasks: 10 },
  '9F': { name: '9F 工厂车间（胶原/医用胶水）', totalTasks: 150, pendingTasks: 130, completedTasks: 20 },
  '8F': { name: '8F 工厂车间（手术包柔性生产）', totalTasks: 95, pendingTasks: 80, completedTasks: 15 },
  '7F': { name: '7F 工厂车间（手术包自动化）', totalTasks: 200, pendingTasks: 180, completedTasks: 20 },
  '6F': { name: '6F 工厂仓库', totalTasks: 60, pendingTasks: 50, completedTasks: 10 },
  '5F': { name: '5F 工厂仓库', totalTasks: 55, pendingTasks: 45, completedTasks: 10 },
  '4F': { name: '4F 工厂车间（手术包组装）', totalTasks: 110, pendingTasks: 95, completedTasks: 15 },
  '3F': { name: '3F 全棉时代 (2B&2C)', totalTasks: 130, pendingTasks: 110, completedTasks: 20 },
  '2F': { name: '2F 集团医疗 (内贸&外贸)', totalTasks: 140, pendingTasks: 120, completedTasks: 20 },
  '1F': { name: '1F 集团发货区', totalTasks: 180, pendingTasks: 150, completedTasks: 30 },
}

// 直接生成新对象，原对象不变
const newFloorDataMap = Object.fromEntries(
  Object.entries(floorDataMap).map(([key, value]) => [key.replace('F', '楼'), value])
);
console.log(newFloorDataMap,'newFloorDataMap')

const onMouseDown = (event) => {
  mouseDownPos = { x: event.clientX, y: event.clientY }
}

const onMouseUp = (event) => {
  // 如果移动距离超过 5 像素，认为是拖拽而非点击
  const moveDist = Math.sqrt(
    Math.pow(event.clientX - mouseDownPos.x, 2) + 
    Math.pow(event.clientY - mouseDownPos.y, 2)
  )
  if (moveDist > 5) return

  if (!sceneContainer.value || !camera || !scene) return

  // 获取点击位置
  const rect = sceneContainer.value.getBoundingClientRect()
  mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1
  mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1

  raycaster.setFromCamera(mouse, camera)
  const intersects = raycaster.intersectObjects(scene.children, true)

  if (intersects.length > 0) {
    // 寻找被点击的楼层对象
    let target = intersects[0].object
    console.log(target.name,'target')
    
    // --- 新增高亮逻辑 ---
    // 尝试寻找楼层根对象，通常是父级或者根据名称规则
    // 这里先简单高亮点击的对象及其父级，或者保持简单，高亮点击的 Mesh
    applyHighlight(target)
    // ------------------

    // while (target && target.parent && target.parent !== scene) {
    //   target = target.parent
    // }

    // 匹配楼层
    const data = newFloorDataMap[target.name]
    if(data){
      floorInfo.value = {
        name: data.name,
        totalTasks: data.totalTasks,
        pendingTasks: data.pendingTasks,
        completedTasks: data.completedTasks
      }
    }else{
      floorInfo.value = {
        name: target.name,
        totalTasks: 0,
        pendingTasks: 0,
        completedTasks: 0
      }
    }
    
    showPopup.value = true
  } else {
    // 点击空白处关闭弹窗并清除高亮
    showPopup.value = false
    restoreMaterials()
  }
}

const onMouseMove = (event) => {
  if (!sceneContainer.value || !camera || !scene) return
  const rect = sceneContainer.value.getBoundingClientRect()
  mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1
  mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1
  raycaster.setFromCamera(mouse, camera)
  const intersects = raycaster.intersectObjects(scene.children, true)
  if (renderer && renderer.domElement) {
    renderer.domElement.style.cursor = intersects.length > 0 ? 'pointer' : 'default'
  }
}

/**
 * 获取 Mesh 在世界空间中的包围盒
 * @param {THREE.Mesh} mesh
 * @returns {THREE.Box3}
 */
function getWorldBoundingBox(mesh) {
  if (!mesh.geometry.boundingBox) {
    mesh.geometry.computeBoundingBox()
  }
  const localBox = mesh.geometry.boundingBox.clone()
  const worldBox = localBox.applyMatrix4(mesh.matrixWorld)
  return worldBox
}

/**
 * 调暗地面材质
 * @param {THREE.Object3D} model
 * @param {number} groundThreshold
 * @param {number} darkenFactor
 */
function darkenGroundMaterials(model, groundThreshold = 0.2, darkenFactor = 0.5) {
  model.traverse((child) => {
    if (child.isMesh) {
      const worldBox = getWorldBoundingBox(child)
      if (worldBox.min.y <= groundThreshold) {
        const materials = Array.isArray(child.material) ? child.material : [child.material]
        materials.forEach(mat => {
          if (mat && mat.isMaterial) {
            if (mat.color) {
              mat.color.multiplyScalar(darkenFactor*0.1)
            }
            if (mat.emissiveIntensity !== undefined) {
              mat.emissiveIntensity *= 0.3
            }
            if (mat.roughness !== undefined) {
              mat.roughness = Math.min(mat.roughness * 1.2, 1.0)
            }
            mat.needsUpdate = true
          }
        })
      }
    }
  })
}

/**
 * 调整相机和控制器，使模型绕自身中心Y轴旋转，并让相机正对模型正面
 * @param {THREE.Object3D} model
 */
function setupCameraForModelRotation(model) {
  // 更新世界矩阵，确保包围盒计算准确
  scene.updateMatrixWorld(true)
  
  // 计算模型的最终世界包围盒
  const boundingBox = new THREE.Box3().setFromObject(model)
  const center = boundingBox.getCenter(new THREE.Vector3())
  const size = boundingBox.getSize(new THREE.Vector3())
  
  // 设置控制器目标点为模型中心
  controls.target.copy(center)
  
  // 相机距离系数进一步减小，让模型显得更大
  const maxHorizontalSize = Math.max(size.x, size.z)
  const distance = Math.max(maxHorizontalSize * 0.1, size.y * 0.2)
  
  // 初始相机角度：方位角0°（正对正面），俯仰角0.2弧度（约11度轻微俯视）
  const azimuth = 0
  const polar = 0.2
  
  // 计算相机位置
  const x = center.x + distance * Math.sin(azimuth) * Math.cos(polar)
  const y = center.y + distance * Math.sin(polar)
  const z = center.z + distance * Math.cos(azimuth) * Math.cos(polar)
  
  camera.position.set(x, y, z)
  camera.lookAt(center)
  
  // 更新控制器
  controls.update()
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
  controls.screenSpacePanning = true
  controls.maxPolarAngle = Math.PI / 2

  // 射线检测
  raycaster = new THREE.Raycaster()
  mouse = new THREE.Vector2()

  renderer.domElement.addEventListener('mousedown', onMouseDown)
  renderer.domElement.addEventListener('mouseup', onMouseUp)
  renderer.domElement.addEventListener('mousemove', onMouseMove)

  // 灯光调整
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.12)
  scene.add(ambientLight)

  const fillLight = new THREE.DirectionalLight(0xccddff, 0.25)
  fillLight.position.set(5, 10, 7)
  scene.add(fillLight)

  // 加载模型
  loader = new GLTFLoader()
  loader.load(
    'https://s2b2c-prod.oss-cn-shenzhen.aliyuncs.com/0000/file/apk/map.glb',
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

      // 模型缩放：放大倍数进一步增加 (150 / maxDim)
      const size = box.getSize(new THREE.Vector3())
      const maxDim = Math.max(size.x, size.y, size.z)
      const scale = 150 / maxDim   // 从300调整为150，模型再放大一倍
      model.scale.set(scale, scale, scale)

      // 旋转模型：让大楼正面面向屏幕
      // model.rotation.y = Math.PI / 4   // 45度，可根据实际模型微调
      model.rotation.x = Math.PI / 8 // 绕x轴旋转

      // 位置微调（可选）
      // model.position.x += 25

      // 调暗地面材质
      darkenGroundMaterials(model, 0.25, 0.1)

      // 设置相机和控制器
      setupCameraForModelRotation(model)
      camera.position.y += 2
      controls.target.y += 2
      controls.update()
    },
    (xhr) => {
      // console.log((xhr.loaded / xhr.total * 100) + '% loaded')
    },
    (error) => {
      console.error('发生错误', error)
    }
  )

  const animate = () => {
    if (!renderer || !scene || !camera) return
    
    animationId = requestAnimationFrame(animate)
    
    if (controls) {
      controls.update()
    }
    
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
  
  if (animationId) {
    cancelAnimationFrame(animationId)
  }

  if (renderer && renderer.domElement) {
    // 移除事件监听
    renderer.domElement.removeEventListener('mousedown', onMouseDown)
    renderer.domElement.removeEventListener('mouseup', onMouseUp)
    renderer.domElement.removeEventListener('mousemove', onMouseMove)
  }
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
  <div class="three-container" ref="sceneContainer">
    <!-- 楼层信息弹窗 -->
    <div v-if="showPopup" class="floor-popup">
      <div class="popup-header">
        <div class="title-box">
          <span class="title-arrow">»</span>
          <span class="title-text">楼层</span>
        </div>
        <div class="close-btn" @click="showPopup = false; restoreMaterials()">×</div>
      </div>
      
      <div class="popup-content">
        <div class="info-item">
          <span class="label">楼层名称</span>
          <span class="value">{{ floorInfo.name }}</span>
        </div>
        <div class="info-item">
          <span class="label">总任务数</span>
          <span class="value">{{ floorInfo.totalTasks }}</span>
        </div>
        <div class="info-item">
          <span class="label">待下发任务</span>
          <span class="value">{{ floorInfo.pendingTasks }}</span>
        </div>
        <div class="info-item">
          <span class="label">已下发任务</span>
          <span class="value">{{ floorInfo.completedTasks }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.three-container {
  flex: 1;
  width: 100%;
  position: relative;
  overflow: hidden;
}

.floor-popup {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 480px;
  background: rgba(5, 39, 58, 0.95);
  border: 1px solid rgba(0, 247, 255, 0.3);
  box-shadow: 0 0 20px rgba(0, 247, 255, 0.2);
  z-index: 100;
  color: #fff;
  pointer-events: auto;

  &::before {
    content: '';
    position: absolute;
    top: -1px;
    left: -1px;
    width: 10px;
    height: 10px;
    border-top: 2px solid #ffaa00;
    border-left: 2px solid #ffaa00;
  }

  &::after {
    content: '';
    position: absolute;
    top: -1px;
    right: -1px;
    width: 10px;
    height: 10px;
    border-top: 2px solid #ffaa00;
    border-right: 2px solid #ffaa00;
  }

  .popup-header {
    height: 44px;
    background: linear-gradient(90deg, rgba(0, 247, 255, 0.3) 0%, rgba(0, 247, 255, 0.05) 100%);
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 15px;
    border-bottom: 1px solid rgba(0, 247, 255, 0.2);

    .title-box {
      display: flex;
      align-items: center;
      gap: 10px;
      
      .title-arrow {
        color: #00f7ff;
        font-size: 18px;
        font-weight: bold;
      }
      
      .title-text {
        font-size: 20px;
        font-weight: bold;
        letter-spacing: 2px;
      }
    }

    .close-btn {
      font-size: 24px;
      cursor: pointer;
      color: rgba(255, 255, 255, 0.6);
      &:hover {
        color: #fff;
      }
    }
  }

  .popup-content {
    padding: 25px 30px;
    display: flex;
    flex-direction: column;
    gap: 15px;

    .info-item {
      display: flex;
      font-size: 16px;
      line-height: 1.5;

      .label {
        width: 140px;
        color: rgba(255, 255, 255, 0.7);
      }

      .value {
        flex: 1;
        color: #fff;
        font-weight: 500;
      }
    }
  }
}
</style>