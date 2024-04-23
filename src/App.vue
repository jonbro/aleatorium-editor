<div id="app">
</div>

<script setup>
  import { Splitpanes, Pane } from "splitpanes"
  import 'splitpanes/dist/splitpanes.css'
  import Editor from './components/Editor.vue'
  import Console from "./components/Console.vue"
  import { createApp, ref, onMounted, onUnmounted } from 'vue'

  const con = ref(null);
  const editor = ref(null);
  createApp({
    components: { Splitpanes, Pane },
  }).mount('#app')

  const returnLines = ref([])
  var port = undefined;
  const encoder = new TextEncoder();

  var keydownHandler = function(e) {
    if (e.ctrlKey && e.which == 80) {
      runScript();
      e.preventDefault();
      returnLines.value.push("hello")
    }
  };
  function runScript()
  {
    if(port?.writable == null)
      {
        console.warn("no writable port");
        return;
      }
    const writer = port.writable.getWriter();
    console.log(editor.value.getEditorContent());
    writer.write(encoder.encode(editor.value.getEditorContent()+"\x03"));
    writer.releaseLock();
  }
  async function connectToSerial()
  {
    if ("serial" in navigator) {
      port = await navigator.serial.requestPort();
      await port.open({ baudRate: 115200 });
      const textDecoder = new TextDecoderStream();
      const readableStreamClosed = port.readable.pipeTo(textDecoder.writable);
      const reader = textDecoder.readable.getReader();
      // Listen to data coming from the serial device.
      while (true)
      {
        const { value, done } = await reader.read();
        if (done) {
          // Allow the serial port to be closed later.
          reader.releaseLock();
          break;
        }
        returnLines.value.push(value);
        // document.getElementById("console").append(value);
      }
    } 
    else
    {
      console.log("serial not supported");
    }

  }

  onMounted(()=>{
    window.addEventListener('keydown', keydownHandler);
  });
  onUnmounted(()=>{
    window.removeEventListener('keydown', keydownHandler);
  });

</script>
<template>
  <header style="height:3vh">
    <button id="connectButton" type="button" @click="connectToSerial">connect to serial</button>
    <button id="runButton" type="button" @click="runScript">run (ctrl+p)</button>
<!--         <div class="header-item" style="display: flex; flex-direction: row; align-items: center;">
      <div class="spinner" style="margin-left: -0.5em; margin-right: 0.5em;"></div>
      <p>MGLUE</p>
    </div> -->
  </header>
  <splitpanes class="default-theme" vertical style="width: 100vw; height:96vh">
    <pane><Editor ref="editor"/></pane>
    <pane>
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
  overflow: clip;
  margin: 0;
  padding: 0;
  max-width: 100%;
}
</style>
