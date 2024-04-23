<script setup>
import { ref, onMounted } from 'vue'

const list = ref([
 "hello",
 "hi"
])

const itemRefs = ref([])
const props = defineProps(['dataItems'])

</script>
<template>
    <div id="console">
        <ul>
            <li v-for="item in dataItems">
            {{ item }}
            </li>
        </ul>
    </div>
</template>

<style scoped>
#console{
    height: 100%;
    overflow: auto;
}
</style>
