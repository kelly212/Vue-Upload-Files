<template xmlns:v-slot="http://www.w3.org/1999/XSL/Transform">
			<div>
						<v-card class="drag-drop-container" :theme="theme">
									<div id="drop-area" class="drop-area"
														:class="{ 'highlight': isDragging }"
														@dragenter.prevent="handleDragEnter"
														@dragover.prevent="handleDragOver"
														@dragleave.prevent="handleDragLeave"
														@drop.prevent="handleDrop"
									>
												<span style="color: gray">Arraste e solte um arquivo aqui ou clique para selecionar.</span>
												<div>
															<v-row>
																		<v-col :cols="multiple? getColSize(files.length):12" v-for="(file, index) in files" :key="index">
																					<v-card style="margin:2%; height: 150px" class="preview-area">
																								<v-card-text>
																											<v-list-item style="text-align: left">
																														<v-list-item-title style="white-space: pre-wrap;">
																																	<p class="">{{ cortarString(file.name, multiple?30:100) }}</p>
																														</v-list-item-title>
																														<v-list-item-subtitle>{{ formatFileSize(file.size) }}</v-list-item-subtitle>
																														<template v-slot:prepend>
																																	<v-avatar>
																																				<v-icon>{{ getFileIcon(file.name) }}</v-icon>
																																	</v-avatar>
																														</template>
																														
																														<template v-slot:append>
																																	<v-btn @click="removeFile(index)" color="red" variant="text" icon size="small">
																																				<v-icon>mdi-delete</v-icon>
																																	</v-btn>
																														</template>
																											</v-list-item>
																								</v-card-text>
																					</v-card>
																		</v-col>
																		<v-col v-if="multiple ? true : files.length <= 0" :cols="multiple? getColSize(files.length):12">
																					<v-card class="drop-message" @click="openFileDialog">
																								<div>
																											<v-icon :size="icone_size">{{icone}}</v-icon>
																											<div v-html="text_add_html"></div>
																											<input type="file" id="fileInput" class="file-input" @change="handleFileSelect"
																																		ref="fileInput" :multiple="multiple">
																								</div>
																					</v-card>
																		
																		</v-col>
															</v-row>
												</div>
									</div>
									<v-card-actions v-if="show_enviar  && files.length > 0">
												<v-btn block variant="flat" size="small" @click="send" color="green">Enviar Arquivos</v-btn>
									</v-card-actions>
						</v-card>
			</div>
</template>

<script>
   /* eslint-disable */
   export default {
      name: 'VUploadFiles',
      components: {},
      props: {
         text_add_html: {default: '<h5>Clique para selecionar.</h5>'},
         icone: {default: 'mdi-upload'},
         theme: {default: 'light'},
         icone_size: {default: '50'},
         multiple: {default: true},
         show_enviar: {default: true},
      },
      data: () => ({
         isDragging: false,
         files: [],
         atualizacao: [],
      }),
      methods: {
         cortarString(str, tam) {
            if (this.validarCampo(str)) {
               if (str.length > tam) {
                  str = str.slice(0, tam) + '...'
               }
            }
            return str
         },

         getFileIcon(filename) {
            const extension = filename.split('.').pop().toLowerCase();

            const iconMap = {
               pdf: 'mdi-file-pdf-box',
               doc: 'mdi-file-word',
               docx: 'mdi-file-word',
               xls: 'mdi-file-excel',
               xlsx: 'mdi-file-excel',
               ppt: 'mdi-file-powerpoint',
               pptx: 'mdi-file-powerpoint',
               txt: 'mdi-file-document-outline',
               jpg: 'mdi-file-image',
               jpeg: 'mdi-file-image',
               png: 'mdi-file-image',
               gif: 'mdi-file-image',
               mp3: 'mdi-file-music',
               wav: 'mdi-file-music',
               mp4: 'mdi-file-video',
               avi: 'mdi-file-video',
               zip: 'mdi-folder-zip',
               rar: 'mdi-folder-zip',
               csv: 'mdi-file-delimited',
               html: 'mdi-language-html5',
               css: 'mdi-language-css3',
               js: 'mdi-language-javascript',
               json: 'mdi-code-json',
            };

            return iconMap[extension] || 'mdi-file'; // ícone padrão
         },
         getColSize(length) {
            if (length === 0) return 12;
            if (length === 1) return 6;
            if (length === 2) return 4;
            if (length >= 3) return 3;
            return 3; // fallback para 4 itens
         },

         validarCampo(campo) {
            return (campo !== undefined && campo !== null && campo !== '')
         },
         send() {
            this.$emit('send', true)
         },

         openFileDialog() {
            this.$refs.fileInput.click()
         },
         uploadFile(event) {
            let self = this
            var files = event[0]

            if (files !== null && files !== undefined) {
               const reader = new FileReader()
               reader.readAsDataURL(files)
               reader.onload = e => {

                  self.atualizacao.push({
                     src: e.target.result,
                     name: files.name
                  })

                  this.$emit('upload', self.atualizacao)
               }
            }
         },
         handleDragEnter() {
            this.isDragging = true;
         },
         handleDragOver() {
            this.isDragging = true;
         },
         handleDragLeave() {
            this.isDragging = false;
         },
         handleDrop(e) {
            this.isDragging = false;
            const droppedFiles = e.dataTransfer.files;
            if (droppedFiles.length) {
               this.uploadFile(droppedFiles)
               this.files = [...this.files, ...Array.from(droppedFiles)];
            }
         },
         handleFileSelect(e) {
            const selectedFiles = e.target.files;
            if (selectedFiles.length) {
               this.uploadFile(selectedFiles)
               this.files = [...this.files, ...Array.from(selectedFiles)];
            }
         },
         removeFile(index) {
            this.files.splice(index, 1);
            this.atualizacao.splice(index, 1);
            this.$emit('upload', this.atualizacao)
         },
         formatFileSize(bytes) {
            if (bytes === 0) return '0 Bytes';

            const k = 1024;
            const sizes = ['Bytes', 'KB', 'MB', 'GB'];
            const i = Math.floor(Math.log(bytes) / Math.log(k));
            return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
         },
      },
   }
</script>
<style scoped>
			
			.drag-drop-container {
						width: 100%;
						margin: 0 auto;
			}
			
			.drop-area {
						border: 2px dashed #ccc;
						border-radius: 8px;
						text-align: center;
						transition: all 0.3s ease;
						height: auto;
			}
			
			.drop-area.highlight {
						border-color: #178c13;
						background-color: #daffe1;
			}
			
			.drop-message {
						border: 1px dashed #ccc;
						height: 150px;
						padding: 30px;
						margin: 2%;
						color: gray;
			}
			
			.upload-icon {
						width: 50px;
						height: 50px;
						fill: #777;
						margin-bottom: 15px;
			}
			
			.file-input {
						display: none;
			}
			
			.preview-area {
						border: 1px dashed #ccc;
			}
			
			.preview-info {
						text-align: left;
						flex-grow: 1;
			}
			
			.preview-icon {
						width: 30px;
						height: 30px;
						margin-right: 10px;
						fill: #555;
			}
			
			.preview-name {
						font-weight: bold;
						white-space: nowrap;
						overflow: hidden;
						text-overflow: ellipsis;
						max-width: 300px;
			}
			
			.preview-size {
						font-size: 0.8em;
						color: #777;
			}


</style>
