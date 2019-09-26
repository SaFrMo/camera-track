<template>
    <main class="camera-path">
        <vue-three-wrap :start="start" :update="update" />
    </main>
</template>

<script>
import * as THREE from 'three'
import loadScene from 'vue-three-wrap/extras/load-gltf/'
import VueThreeWrap from 'vue-three-wrap'

const ref = {}

export default {
    components: {
        'vue-three-wrap': VueThreeWrap
    },
    methods: {
        async start({ scene, camera, renderer }) {
            const loadedScene = await loadScene('/camera-track.glb')
            const objects = Array.from(loadedScene.scene.children)

            objects.forEach(obj => {
                scene.add(obj)
            })

            ref.mixer = new THREE.AnimationMixer(objects[1])
            loadedScene.animations.forEach(clip => {
                ref.mixer.clipAction(clip).play()
            })

            ref.cameraPos = objects[1]

            // add some lighting
            this.prepLighting(scene)

            // place and rotate the camera
            camera.position.set(-10, 4, 4)
            camera.lookAt(new THREE.Vector3(0, 0, 0))
        },
        update({ camera }) {
            if (ref.mixer) {
                ref.mixer.update(0.0166 * 0.1)
                camera.position.copy(ref.cameraPos.position)
                camera.lookAt(new THREE.Vector3(0, 0, 0))
            }
        },
        prepLighting(scene) {
            // some nice lighting
            const output = {}

            // Ambient light
            output.light = new THREE.AmbientLight(0xffffff, 0.5)

            // Shadow light
            output.shadowLight = new THREE.DirectionalLight(0xffffff, 0.5)
            output.shadowLight.position.set(200, 200, 200)
            output.shadowLight.castShadow = true
            output.shadowLight.shadow.mapSize.width = output.shadowLight.shadow.mapSize.height = 2048

            // Backlight
            output.backLight = new THREE.DirectionalLight(0xffffff, 0.2)
            output.backLight.position.set(-100, 200, 50)
            // output.backLight.castShadow = true

            // Add lights
            scene.add(output.backLight)
            scene.add(output.light)
            scene.add(output.shadowLight)

            // in case we want to do anything with these later
            return output
        }
    }
}
</script>

<style lang="scss">
.camera-path,
.vue-three-wrap {
    position: fixed;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
}
</style>
