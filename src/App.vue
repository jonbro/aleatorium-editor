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
  const inputField = ref('')

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
    writer.write(encoder.encode(editor.value.getEditorContent()+"\x03"));
    writer.releaseLock();
  }
  function saveScript(filename)
  {
    if(port?.writable == null)
      {
        console.warn("no writable port");
        return;
      }
    const writer = port.writable.getWriter();
    writer.write(encoder.encode("^^S"+filename+"^"+editor.value.getEditorContent()+"\x03"));
    writer.releaseLock();
  }
  function loadScript(filename)
  {
    if(port?.writable == null)
      {
        console.warn("no writable port");
        return;
      }
    const writer = port.writable.getWriter();
    writer.write(encoder.encode("^^L"+filename+"\x03"));
    writer.releaseLock();
  }
  function rmScript(filename)
  {
    if(port?.writable == null)
      {
        console.warn("no writable port");
        return;
      }
    const writer = port.writable.getWriter();
    writer.write(encoder.encode("^^D"+filename+"\x03"));
    writer.releaseLock();
  }
  function AddToReturnLines(value)
  {
    const lines = value.split('\n');
    if(returnLines.value.length == 0)
    {
      returnLines.value.push("");
    }
    if(lines.length > 0)
    {
      returnLines.value[returnLines.value.length-1] += lines.shift();
    }
    lines.forEach(element => {
      returnLines.value.push(element);
    });
  }
  let commandSearch = "";
  let loading = false;
  let loadString = "";
  async function connectToSerial()
  {
    if ("serial" in navigator) {
      port = await navigator.serial.requestPort();
      await port.open({ baudRate: 115200, bufferSize: 131072 });
      const textDecoder = new TextDecoderStream();
      const readableStreamClosed = port.readable.pipeTo(textDecoder.writable);
      const reader = textDecoder.readable.getReader();
      // Listen to data coming from the serial device.
      while (true)
      {
        const { value, done } = await reader.read();
        if (done) {
          // Allow the serial port to be closed later. 
          console.log("got done");
          reader.releaseLock();
          break;
        }
        console.log(value);
        // continue;
        commandSearch += value;
        // look for the start of a command
        let i = 0;
        while(i<commandSearch.length)
        {
          let isCommand = commandSearch.substring(i, i+3);
          const regex = /\^\^[LE]/;
          // Alternative syntax using RegExp constructor
          // const regex = new RegExp('\\^\\^[LE]', '')
          let m;
          if ((m = regex.exec(isCommand)) !== null) {
            let command = m[0][2];
            console.log("command: "+ command);
            if(command == "L")
            {
              loading = true;
              loadString = "";
              // fast forward past the command
              i+=3;
            }
            if(command == "E" && loading)
            {
              editor.value.setEditorContent(loadString);
              loading = false;
              i+=3;
            }
          }
          else if(loading)
          {
            loadString += commandSearch.substring(i, i+1);
          }
          i++;
        }
        // truncate command search to the last three characters
        commandSearch = commandSearch.substring(commandSearch.length-3);
        AddToReturnLines(value);
      }
    } 
    else
    {
      console.log("serial not supported");
    }

  }
  function InsertParam(event)
  {
    editor.value.insertAtCursor("param."+event.target.innerText);
  }
  function InsertShape(event)
  {
    editor.value.insertAtCursor("shape."+event.target.innerText);
  }
  function onInputField()
  {
    var commandSplit = inputField.value.split(" ");
    if(commandSplit[0] == "save")
    {
      saveScript(commandSplit[1]);
    }
    else if(commandSplit[0] == "load")
    {
      loadScript(commandSplit[1]);
    }
    else if(commandSplit[0] == "rm" || commandSplit[0] == "del")
    {
      rmScript(commandSplit[1]);
    }
    inputField.value = "";
  }
  function clearInput()
  {
    inputField.value = "";
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
    <!-- TODO save using the last filename for the current buffer if available -->
    <!-- <button id="saveButton" type="button" @click="saveScript">save</button> -->
<!--         <div class="header-item" style="display: flex; flex-direction: row; align-items: center;">
      <div class="spinner" style="margin-left: -0.5em; margin-right: 0.5em;"></div>
      <p>MGLUE</p>
    </div> -->
  </header>
  <splitpanes class="default-theme" vertical style="width: 100vw; height:96vh">
    <pane><Editor ref="editor"/></pane>
    <pane>
      <splitpanes class="default-theme" horizontal>
        <pane>
          <h3>params</h3>
          <button @click="InsertParam">timbre</button>
          <button @click="InsertParam">color</button>
          <button @click="InsertParam">cutoff</button>
          <button @click="InsertParam">resonance</button>
          <button @click="InsertParam">volume</button>
          <button @click="InsertParam">pan</button>
          <button @click="InsertParam">attackTime</button>
          <button @click="InsertParam">decayTime</button>
          <button @click="InsertParam">attackTime2</button>
          <button @click="InsertParam">decayTime2</button>
          <button @click="InsertParam">LFORate</button>
          <button @click="InsertParam">LFODepth</button>
          <button @click="InsertParam">retriggerSpeed</button>
          <button @click="InsertParam">retriggerLength</button>
          <button @click="InsertParam">env1Target</button>
          <button @click="InsertParam">env1Depth</button>
          <button @click="InsertParam">env2Target</button>
          <button @click="InsertParam">env2Depth</button>
          <button @click="InsertParam">lfo1Target</button>
          <button @click="InsertParam">retriggerFade</button>
          <button @click="InsertParam">length</button>
          <button @click="InsertParam">conditionMode</button>
          <button @click="InsertParam">conditionData</button>
          <button @click="InsertParam">delaySend </button>
          <button @click="InsertParam">reverbSend</button>
          <button @click="InsertParam">shape</button>
          <h3>shapes</h3>
          <button @click="InsertShape">csaw</button>
          <button @click="InsertShape">morph</button>
          <button @click="InsertShape">saw_square</button>
          <button @click="InsertShape">sine_triangle</button>
          <button @click="InsertShape">buzz</button>
          <button @click="InsertShape">square_sub</button>
          <button @click="InsertShape">saw_sub</button>
          <button @click="InsertShape">square_sync</button>
          <button @click="InsertShape">saw_sync</button>
          <button @click="InsertShape">triple_saw</button>
          <button @click="InsertShape">triple_square</button>
          <button @click="InsertShape">triple_triangle</button>
          <button @click="InsertShape">triple_sine</button>
          <button @click="InsertShape">triple_ring_mod</button>
          <button @click="InsertShape">saw_swarm</button>
          <button @click="InsertShape">toy</button>
          <button @click="InsertShape">digital_filter_lp</button>
          <button @click="InsertShape">digital_filter_pk</button>
          <button @click="InsertShape">digital_filter_bp</button>
          <button @click="InsertShape">digital_filter_hp</button>
          <button @click="InsertShape">vosim</button>
          <button @click="InsertShape">vowel</button>
          <button @click="InsertShape">vowel_fof</button>
          <button @click="InsertShape">harmonics</button>
          <button @click="InsertShape">fm</button>
          <button @click="InsertShape">feedback_fm</button>
          <button @click="InsertShape">chaotic_feedback_fm</button>
          <button @click="InsertShape">struck_bell</button>
          <button @click="InsertShape">struck_drum</button>
          <button @click="InsertShape">kick</button>
          <button @click="InsertShape">cymbal</button>
          <button @click="InsertShape">snare</button>
          <button @click="InsertShape">wavetables</button>
          <button @click="InsertShape">wave_map</button>
          <button @click="InsertShape">wave_line</button>
          <button @click="InsertShape">wave_paraphonic</button>
          <button @click="InsertShape">filtered_noise</button>
          <button @click="InsertShape">twin_peaks_noise</button>
          <button @click="InsertShape">clocked_noise</button>
          <button @click="InsertShape">granular_cloud</button>
          <button @click="InsertShape">particle_noise</button>
          <button @click="InsertShape">digital_modulaton</button>
          <h3>envTargets</h3>
          <h3>params</h3>
          
        </pane>
        <pane style="flex-direction: column; display:flex;">
          <Console :dataItems="returnLines"/>
          <textarea style=" flex: 0 1 1.2em; width:100%;min-height: 1.2em;" v-model="inputField" @keydown.enter="onInputField"  @keyup.enter="clearInput"></textarea>
        </pane>
      </splitpanes>
    </pane>2
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
