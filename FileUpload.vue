<template>
    <div class="card">

        <div class="card-header">
            <label v-if="!forUpload" class="btn btn-primary" style="color: white" for='file'>
                Selecionar arquivo
                <input id="file" ref="file" type='file' style="display:none" @change="handleFileUpload">
            </label>
            <span v-if="file.name">
                {{ file.name }}
            </span>
            <span v-if="error" style="color: red">
                {{error}}
            </span>  
            <a v-if="forUpload" class="btn btn-primary" @click="create()" style="color: white">
                <i class="fa fa-cloud-upload"></i> Upload
            </a>
            <a v-if="forUpload" class="btn btn-danger" @click="cancelFile()" style="color: white">
                <i class="fa fa-undo"></i> Cancelar
            </a>
        </div>

        <div v-if="this.uploaded.length" class="card-body"> 
            <div class="row">
                <div class="col-sm-12">
                    <table class="table table-hover table-striped">
                        <thead>
                            <tr>
                                <th scope="col">#</th>
                                <th scope="col">Nome/link</th>
                                <th scope="col">Ações</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="(u , index) in uploaded">
                                <th scope="row">{{index}}</th>
                                <td>
                                    {{ u.filename}}
                                </td>
                                <td>
                                    <div class="btn-group" role="group" aria-label="First group">
                                        <a type="button" class="btn btn-primary" :href="u.url" target="_black">
                                            <i class="fa fa-eye"></i>Ver
                                        </a>
                                        <a type="button" class="btn btn-danger" @click="remove(u.id)">
                                            <i class="fa fa-trash"></i>Apagar
                                        </a>
                                    </div>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>    
        </div>
    </div>

</template>

<script>
    export default {
      props: ['action','ext','obs','apagados','params'],
      data() {
          return {
              file: '',
              uploaded: [],
              forUpload: false,
              error: false,
              headers: {
                'Content-Type': 'multipart/form-data'
              },
          }
      },
      // depois de montado
      beforeMount(){
          this.list(); 
          this.getBaseUrl();
      },
      methods: {
        // antes de ser feito o upload
        handleFileUpload(){
          this.file = this.$refs.file.files[0];
          let filetype = this.file.type.split('/')[1];
          console.log(filetype)

          if (!this.ext.includes(filetype)) {
            this.forUpload = false;
            this.error = 'Extenção inválida! deve ser: ' + this.ext.join(', ');
            this.file = '';
          }else{
            this.error = false;
            this.forUpload = true;
          }
          
        },
        // retornar ao inicial
        cancelFile(){
          this.file ='';
          this.forUpload = false;
        },
        // envio do arquivo
        create(){
            let urlCreate = this.getBaseUrl() + this.action;
            console.log(urlCreate);
            
            let formData = new FormData();
            formData.append('file', this.file);
            axios.post( urlCreate,formData,{headers: this.headers})
            .then((response) => (this.uploaded = response.data))
            .catch(error => console.log(error));
        },
        // listagem dos arquivos existentes
        list(){
            let urlIndex = this.getBaseUrl() + this.action;
            axios
            .get(urlIndex)
            .then((response) => (this.uploaded = response.data))
            .catch(error => console.log(error));
        },
        // apagar arquivo
        remove(id){
            let urlDelete = this.getBaseUrl() + this.action + '/' + id;

            axios
            .delete(urlDelete)
            .then((response) => this.list())//chama list para atualizar
            .catch(error => console.log(error));
        },
        getBaseUrl(){
           let getUrl = window.location; 
           let params = this.params ? this.params + '/' : '';
           let baseUrl = getUrl.protocol + "//" + getUrl.host + "/" + getUrl.pathname.split('/')[1]+"/"+params;
           
           return baseUrl;
        }
      }
    }
</script>
<style>
 
</style>