<template>
    <v-container>
        <v-file-input outlined chips label="Selecciona el archivo" accept=".txt" prepend-icon="mdi-file-code"
            v-model="file" @change="readChange">
        </v-file-input>
        <v-btn color="success" @click="read">Leer doc</v-btn>
        <CodeEditor v-model="data" :read_only="false" :display_language="false" width="1080px"></CodeEditor>
        <div>
            codigo sin comentarios
            <p v-for="linea in code">{{ linea }}</p>
        </div>

    </v-container>
</template>

<script>
//import editor from '@/components/editor.vue'
import CodeEditor from 'simple-code-editor'

const exp = new RegExp("/(?=/)", "");//expresion para eliminar comentarios
const coment = new RegExp("\r\n", "")//expresion para cortar el doc
export default {
    components: {
        CodeEditor
    },
    data() {
        return {
            file: null,//es el documento
            data: '',//es la data del doc
            cut: null,//es el codigo cortado
            code: null,//codigo puro sin comentarios
            armado: null
        }
    }, methods: {
        read() {
            let reader = new FileReader()

            reader.readAsText(this.file)

            reader.onload = () => {
                this.code = reader.result
            }

        },
        async readChange(value) {
            let reader = new FileReader()//es el objeto para leer
            reader.readAsText(value)//le paso el doc para leer

            reader.onload = () => { //funcion que devuelve el contenido al terminar de codificar
                this.data = reader.result//guardo el texto puro

                this.cut = this.data.split(coment)//corta el texto en los saltos 
                this.code = this.cut.filter(text => !exp.test(text))//quita los comentarios
                //this.armado = this.cut.join("\r\n")//esta funcion puede servir para ensamblar un nuevo codigo
            }
        }
    },
}
</script>