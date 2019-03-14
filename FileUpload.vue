<template>
    <div class="card">

        <div class="card-header">

            <label v-if="!forUpload" class="btn btn-primary bgicon" for='file'>
                Selecionar arquivo
                <input @change="handleFileUpload" id="file" ref="file" type='file' style="display:none">
            </label>
              
            <a v-if="forUpload" @click="cancelFile()" class="btn btn-danger bgicon" style="color: white">
                <i class="fa fa-undo"></i> Cancelar
            </a>
            <a v-if="forUpload" @click="createFile()" class="btn btn-primary bgicon" style="color: white">
                <i class="fa fa-cloud-upload"></i> Upload
            </a>

            <span v-if="file.name">
                {{ file.name }}
            </span>
            <span v-if="error" style="color: red">
                {{error}}
            </span>

            <div  v-if="forUpload" class="progress" style="padding-top: 3px">
                <div class="progress-bar" role="progressbar" :style="{'width' : width + '%'}" aria-valuenow="0" aria-valuemin="0" aria-valuemax="100"></div>
            </div>
            
        </div>

        <div v-if="this.uploaded.length" class="card-body"> 
            <div class="row">
                <div class="col-sm-12">
                    <table class="table table-hover">
                        <thead>
                            <tr>
                                <th class="col-sm-2">#</th>
                                <th class="col-sm-8">Nome/link</th>
                                <th class="col-sm-2">Ações</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="(u , index) in uploaded">
                                <td>{{index}}</td>
                                <td>
                                    {{ u.name}}
                                </td>
                                <td>
                                    <div class="btn-group" role="group" aria-label="First group">
                                        <a type="button" @click="showFile(u.id)" target="_black" class="btn btn-primary" style="color: white">
                                            <i class="fa fa-eye"></i> Ver
                                        </a>
                                        <a type="button" @click="removeFile(u.id)" class="btn btn-danger" style="color: white">
                                            <i class="fa fa-trash"></i> Apagar
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
      props: ['action','ext','params'],
      data() {
          return {
              file: '',
              uploaded: [],
              forUpload: false,
              error: false,
              headers: {
                'Content-Type': 'multipart/form-data'
              },
              width: 0,
          }
      },
      // depois de montado
      beforeMount(){
          this.listFile(); 
      },
      methods: {
        // antes de ser feito o upload
        handleFileUpload(){
          this.file = this.$refs.file.files[0];
          let filetype = this.file.type.split('/')[1];

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
        createFile(){
            let urlCreate = this.getBaseUrl() + this.action;
            console.log(urlCreate);
            
            let formData = new FormData();
            formData.append('file', this.file);
            axios.post( urlCreate,formData,{headers: this.headers})
            .then(this.progress())
            .then((response) =>{
                this.listFile()//chama list para atualizar 
            })
            .catch((error) => {
                console.log(error)
                this.erro = "Erro ao enviar arquivo"
            });
        },
        progress(){
            setTimeout(()=>{ 
                    if(this.width < 100) {
                        this.width += 1; 
                        this.progress();
                    }else{
                       this.cancelFile(); 
                    }
                }, 25);
            
        },
        // listagem dos arquivos existentes
        listFile(){
            let urlIndex = this.getBaseUrl() + this.action;
            axios
            .get(urlIndex)
            .then((response) => (this.uploaded = response.data))
            .catch(error => console.log(error));
        },
        showFile(id){
            let urlIndex = this.getBaseUrl() + this.action + '/' + id;
            window.open(urlIndex, "_blank")
        },
        // apagar arquivo
        removeFile(id){
            let urlDelete = this.getBaseUrl() + this.action + '/' + id;

            axios
            .delete(urlDelete)
            .then((response) => this.listFile())//chama list para atualizar
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

<style scope>
.bgicon{
    color: white;
}
</style>