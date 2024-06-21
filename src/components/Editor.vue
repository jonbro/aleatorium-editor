<script setup>
import { ref, onMounted } from 'vue';
import { VAceEditor } from 'vue3-ace-editor';
import 'ace-builds/src-noconflict/theme-monokai';
import 'ace-builds/src-noconflict/mode-lua';

const content = ref("function hello() end");
const aceRef = ref(null);

onMounted(() => {
    var editor = aceRef.value.getAceInstance();
    if("ctrl-p" in editor.keyBinding.$handlers[0].commandKeyBinding)
        delete editor.keyBinding.$handlers[0].commandKeyBinding["ctrl-p"];
})
function getEditorContent() {
    var editor = aceRef.value.getAceInstance();
    return editor.getValue();
}    
function setEditorContent(v) {
    var editor = aceRef.value.getAceInstance();
    return editor.setValue(v);
}    
function insertAtCursor(v){
    var editor = aceRef.value.getAceInstance();
    editor.session.insert(editor.getCursorPosition(), v);
    editor.focus();
}
defineExpose({
    getEditorContent,
    setEditorContent,
    insertAtCursor
});

</script>

<template>
    <v-ace-editor
        ref="aceRef" 
        style="height: 100%"
        v-model:value="content"
        lang="lua"
        theme="monokai" />
</template>