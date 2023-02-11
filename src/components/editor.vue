<template>
  <CodeEditor
    v-model="code"
    :read_only="false"
    :display_language="false"
  ></CodeEditor>
</template>
<script>
import CodeEditor from 'simple-code-editor';

export default {
  name: 'EditorComponent',
  components: {
    CodeEditor,
  },
  data() {
    return {
      code: 'Hola mundo',
    };
  },
};
</script>
