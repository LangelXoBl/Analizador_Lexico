<template>
    <v-container>
        <v-row>
            <v-file-input outlined chips label="Selecciona el archivo" accept=".txt" prepend-icon="mdi-file-code"
                v-model="file" @change="readChange">
            </v-file-input>
            <v-btn color="success" @click="analyze_code" right>Analizar</v-btn>
        </v-row>
        <v-row>
            <v-col>
                codigo puro:{{ modif }}
                <CodeEditor v-model="data" width="540px" height="550px" :language_selector="true"
                    :languages="[['javascript', 'JS'], ['plaintext', 'TXT']]">
                </CodeEditor>
            </v-col>
            <v-col>
                Codigo limpio
                <CodeEditor v-model="result" :read_only="true" width="540px" height="550px" :display_language="false">
                </CodeEditor>
            </v-col>
        </v-row>

        Resultado:
        <CodeEditor v-model="armado" :read_only="true" width="1130px" :display_language="false">
        </CodeEditor>
        {{ data }}
        {{ cut }}
    </v-container>
</template>

<script>
//import editor from '@/components/editor.vue'
import CodeEditor from 'simple-code-editor'

const coment = new RegExp("/(?=/)", "");//expresion para eliminar comentarios
const cut = new RegExp("\r\n", "")//expresion para cortar el doc
const cut_edit = new RegExp("\n", "")//corta el codigo cuando se edita en el navegador
const space = new RegExp("", "")
const charactInv = new RegExp("[^\w\s|/\n]", "g");
const flags = /[^\w\s|/\n':*+=<>,-]/
const endLine = /[|]$/

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
            armado: '',
            proces: '',
            errors: [],
            result: '',
            modif: false,
            Structura: ['Prog', 'Fprog', 'Usar', 'In', 'Fin']
        }
    }, methods: {
        analyze_code() {
            this.errors = []//limpia los errores
            //this.result = ''
            this.cut = this.data.split(cut_edit)//corta el texto en los saltos
            this.proces = 'procesando...'
            console.log(this.cut)
            this.code = this.cut.filter(text => !coment.test(text))//quita los comentarios
            this.code = this.code.filter(text => text != "")//quita espacios en blanco
            this.armado = this.code.join("\n")
            this.proces = 'quitando comentarios y saltos de linea...'
            console.log(this.code)
            console.log(flags.exec(this.data))
            this.analizeLine();
            console.log()
        },
        readChange(value) {
            let reader = new FileReader()//es el objeto para leer
            reader.readAsText(value)//le paso el doc para leer

            reader.onload = () => { //funcion que devuelve el contenido al terminar de codificar
                this.proces = 'Leyendo...'
                this.data = reader.result//guardo el texto puro
                //this.analyze_code();
                this.cut = this.data.split(cut)//corta el texto en los saltos
                this.proces = 'procesando...'
                console.log(this.cut)
                this.code = this.cut.filter(text => !coment.test(text))//quita los comentarios
                this.code = this.code.filter(text => text != "")
                this.proces = 'quitando comentarios y saltos de linea...'
                this.armado = this.code.join("\r\n")//esta funcion puede servir para ensamblar un nuevo codigo
                //console.log(flags.exec(this.data)['index'])
                this.analizeLine();//analiza linea por linea
            }
        },
        analizeLine() {
            //console.log('structure', /((Prog)|(Fprog)|(usar)|(In)|(Fin))($)/gm.exec(this.data))
            //console.log(typeof this.cut[0])
            //console.table(this.cut[1])
            this.cut.forEach((line, index) => {
                console.log(line)
                if (coment.test(line) || line == '') return console.log('no evaluo coments ni espacios')
                let res = flags.exec(line)
                if (this.searchStructure(line)) return//busca la estructura
                //console.log('no permitido', flags.test(line))
                //console.log(res)
                if (res) this.listErrors(`Error en la linea ${index + 1}, posicion ${res['index'] + 1}: caracter invalido "${res[0]}"`)
                if (!endLine.test(line)) this.listErrors(`Error en la linea ${index + 1}, hace falta el fin de linea "|"`)
            });
            this.Structura.forEach(item => this.listErrors(`No se encontro el delimitador ${item}`))
            this.result = this.errors.join("\r\n")
        },
        listErrors(error) {
            this.errors.push(error)

        },
        searchStructure(line) {
            const res = /((Prog)|(Fprog)|(Usar)|(In)|(Fin))($)/.exec(line)
            if (res) {
                const index = this.Structura.indexOf(res[0])
                this.Structura.splice(index, 1)
                return true
            }
            return false
        }
    },
}
//:language_selector="true"
//:languages="[['javascript', 'JS'], ['plaintext', 'TXT']]"
</script>