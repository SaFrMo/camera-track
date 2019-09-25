<template>
    <main class="camera-path">
        <vue-three-wrap :start="start" :update="update" />
    </main>
</template>

<script>
import * as THREE from 'three'
import { loadObjects } from 'vue-three-wrap/extras/load-gltf/'
import VueThreeWrap from 'vue-three-wrap'

export default {
    components: {
        'vue-three-wrap': VueThreeWrap
    },
    methods: {
        async start({ scene, camera, renderer }) {
            const objects = await loadObjects('/camera-track.glb')

            objects.forEach(obj => {
                scene.add(obj)
            })

            // add some lighting
            this.prepLighting(scene)

            // place and rotate the camera
            camera.position.set(-4, 4, 4)
            camera.lookAt(new THREE.Vector3(0, 0, 0))
        },
        update() {},
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
