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
    <ul>
        <li v-for="item in dataItems">
        {{ item }}
        </li>
    </ul>
</template>
