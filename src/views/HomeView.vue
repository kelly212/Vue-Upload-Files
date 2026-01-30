<template>
  <div>
    <v-container>
      <br>
      <div style="text-align: center">
        <h1>Vue Upload File</h1>
        <br>
      </div>
      <v-row justify="center">
        <v-col cols="12">
											<h2>Arquivo único</h2>
          <v-upload-files @send="enviar" @upload="updateFiles" :multiple="false"></v-upload-files>
        </v-col>
									<v-col cols="12">
												<h2>Multiplos arquivos</h2>
          <v-upload-files @send="enviar" @upload="updateFiles" :multiple="true"></v-upload-files>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import { defineComponent } from 'vue';

// Components
import VUploadFiles from '../components/VUploadFiles.vue';

export default defineComponent({
  name: 'HomeView',
  data:()=>({
					files: []
 
  
  }),
  components: {
    VUploadFiles,
  },

   methods: {
      updateFiles(files){
         console.log('updateFiles:',files);
      },
      enviar(){
         console.log('enviar');
      }
   },
});
</script>
