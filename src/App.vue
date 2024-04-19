<div id="app">
</div>

<script setup>
  import { Splitpanes, Pane } from "splitpanes"
  import 'splitpanes/dist/splitpanes.css'
  import Editor from './components/Editor.vue'
  import Console from "./components/Console.vue"
  import { createApp, ref, onMounted, onUnmounted } from 'vue'

  const con = ref(null);

  createApp({
    components: { Splitpanes, Pane },
  }).mount('#app')

  const returnLines = ref([])
  
  var keydownHandler = function(e) {
    if (e.ctrlKey && e.which == 80) {
        console.log("we here");
        // const writer = port.writable.getWriter();
        // writer.write(encoder.encode(editor.getValue()+"\x03"));
        // writer.releaseLock();
        e.preventDefault();
        returnLines.value.push("hello")
    }
  };

  onMounted(()=>{
    window.addEventListener('keydown', keydownHandler);
  });
  onUnmounted(()=>{
    window.removeEventListener('keydown', keydownHandler);
  });

</script>
<template>
  <splitpanes class="default-theme" vertical style="width: 100vw">
    <pane><Editor/></pane>
    <pane style="height:100vh">
      <splitpanes class="default-theme" horizontal>
        <pane>"docs"</pane>
        <pane>
          <Console :dataItems="returnLines"/>
          <textarea></textarea>
        </pane>
      </splitpanes>
    </pane>
  </splitpanes>
</template>

<style scoped>
#app{
  margin: 0;
  padding: 0;
  max-width: 100%;
}
</style>
