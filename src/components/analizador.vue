<template>
  <v-container>
    <v-form>
      <v-row>
        <v-card-text>
          There are many variations of passages of Lorem Ipsum available, but
          the majority have suffered alteration in some form, by injected
          humour, or randomised words which don't look even slightly believable.
          If you are going to use a passage of Lorem Ipsum, you need to be sure
          there isn't anything embarrassing hidden in the middle of text. All
          the Lorem Ipsum generators on the Internet tend to repeat predefined
          chunks as necessary, making this the first true generator on the
          Internet.
          <v-file-input
            outlined
            chips
            label="Selecciona el archivo"
            accept=".txt"
            prepend-icon="mdi-file-code"
            @change="read"
          />
        </v-card-text>
      </v-row>

      <v-card>
        <v-row>
          <v-col>
            <v-container grid-list-xl>
              <CodeEditor
                v-model="data"
                height="500px"
                :languages="[['javascript', 'LaxB']]"
                :copy_code="false"
              />
            </v-container>
          </v-col>
          <v-col>
            <v-card-actions>
              <v-btn
                color="black"
                x-large
                dark
                :disabled="data == ''"
                outlined
                @click="analyze_code"
              >
                <v-icon x-large> mdi-console </v-icon>
                Analizar
              </v-btn>
            </v-card-actions>
            <v-container grid-list-xs>
              <CodeEditor
                v-model="result"
                :read_only="true"
                height="200px"
                :languages="[['powershell', 'PowerShell 7.3.0']]"
                :copy_code="false"
              />
            </v-container>
          </v-col>
        </v-row>
      </v-card>
    </v-form>
  </v-container>
</template>

<script>
import CodeEditor from 'simple-code-editor';

const coment = /[/](?=[/])/; // expresion para eliminar comentarios
const cut = /\r\n/; // expresion para cortar el doc
const cutEdit = /\n/; // corta el codigo cuando se edita en el navegador
const endLine = /[|]$/; // Revisa que tenga el fin de linea
const charactInv = /[^\w\s|/\n':*+=<>,-]/; // Busca los caracteres que no esten en el grupo

export default {
  name: 'LexiCode',
  components: {
    CodeEditor,
  },
  data() {
    return {
      data: '//Escribre tu codigo o\n//busca un archivo .txt en tu ordenador', // es la data del doc
      cut: null, // es el codigo cortado
      // code: null,//codigo puro sin comentarios
      // armado: '',
      errors: [],
      result: '',
      Structura: ['Prog', 'Fprog', 'Usar', 'In', 'Fin'],
    };
  },
  methods: {
    read(value) {
      if (!value) return;
      const reader = new FileReader(); // es el objeto para leer
      reader.readAsText(value); // le paso el doc para leer
      reader.onload = () => {
        // funcion que devuelve el contenido al terminar de codificar
        this.data = reader.result; // guardo el texto puro
        this.cut = this.data.split(cut); // corta el texto en los saltos
        this.data = this.cut.join('\n'); // se arma de nuevo para imprimirlo en el editor
        this.analyze_code();
      };
    },
    analyze_code() {
      this.errors = []; // limpia los errores
      this.Structura = ['Prog', 'Fprog', 'Usar', 'In', 'Fin']; // llenar de nuevo
      this.cut = this.data.split(cutEdit); // corta el texto en los saltos
      // this.code = this.cut.filter(text => !coment.test(text) | text == "")//quita los comentarios
      // this.armado = this.code.join("\n")
      this.analizeLine();
      if (this.errors.length === 0)
        this.result = 'No se encontraron errores de lexico, felicidades!!!';
    },

    analizeLine() {
      this.cut.forEach((line, index) => {
        if (coment.test(line) || line === '') return; // console.log('no evaluo coments ni espacios')
        if (this.searchStructure(line)) return; // busca la estructura
        const res = charactInv.exec(line);
        if (res)
          this.listErrors(
            `Error en la linea ${index + 1}, posicion ${
              res.index + 1
            }: caracter invalido "${res[0]}"`
          );
        if (!endLine.test(line))
          this.listErrors(
            `Error en la linea ${index + 1}, hace falta el fin de linea "|"`
          );
      });
      this.Structura.forEach((item) =>
        this.listErrors(`No se encontro el la pralabra reservada: "${item}"`)
      );
      this.result = this.errors.join('\r\n');
    },
    searchStructure(line) {
      const res = /((Prog)|(Fprog)|(Usar)|(In)|(Fin))($)/.exec(line);
      if (res) {
        const index = this.Structura.indexOf(res[0]); // Busca la pos palabra que encontro
        this.Structura.splice(index, 1); // la quita del array
        return true; // informa que la linea es parte de la estructura
      }
      return false;
    },
    listErrors(error) {
      this.errors.push(error);
    },
  },
};
// Pruebas para ignorar las comillas:
// /[^\w\s|/\n':*+=<>,-](?=(?:[^']*'[^']*')*[^']*$)/
// (?:[^']*) Crea un grupo pasivo de todo lo que no sean comillas
// '[^']*' Busca entre comillas todo lo que no sean comillas
// (?:[^']*'[^']*')* Crea un grupo pasivo de todo lo que no este entre comillas
// [^']*$) Termina con cualquier caracter que no sea comilla
</script>
