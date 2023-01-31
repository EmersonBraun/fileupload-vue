<template>
    <div class="card">
        <div v-if="title" class="card-header">
            {{ title }}
        </div>
        <div v-if="!only" class="card-body">

            <label v-if="!forUpload" class="btn btn-primary bgicon" for='file'>
                Select file
                <input @change="verifyType" id="file" ref="file" type='file' style="display:none">
            </label>

            <a v-if="forUpload" @click="cancelFile()" class="btn btn-danger bgicon" style="color: white">
                <i class="fa fa-undo"></i> Cancel
            </a>
            <a v-if="forUpload" @click="createFile()" class="btn btn-primary bgicon" style="color: white">
                <i class="fa fa-cloud-upload"></i> Upload
            </a>

            <span v-if="file.name">
                {{ file.name }}
            </span>
            <span v-if="error" style="color: red">
                {{ error }}
            </span>

            <div v-if="progressBar" class="progress" style="padding-top: 3px">
                <div class="progress-bar" role="progressbar" :style="{ 'width': width + '%' }" :aria-valuenow="width"
                    aria-valuemin="0" aria-valuemax="100"></div>
            </div>

        </div>

        <div v-if="uploaded.length" class="card-footer">
            <div class="row">
                <div class="col-sm-12">
                    <table class="table table-hover">
                        <thead>
                            <tr>
                                <th v-if="!only" class="col-sm-2">#</th>
                                <th class="col-sm-8">Name/link</th>
                                <th class="col-sm-2">Actions</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="(upload, index) in uploaded" :key="index">
                                <td v-if="!only">{{ counter = upload.id }}</td>
                                <td>
                                    {{ upload.name }}
                                </td>
                                <td>
                                    <div class="btn-group" role="group" aria-label="First group">
                                        <a type="button" @click="showFile(upload.id)" target="_black"
                                            class="btn btn-primary" style="color: white">
                                            <i class="fa fa-eye"></i> Show
                                        </a>
                                        <a type="button" @click="removeFile(upload.id)" class="btn btn-danger"
                                            style="color: white">
                                            <i class="fa fa-trash"></i> Delete
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
import axios from 'axios';
export default {
    props: ['title', 'action', 'ext', 'params', 'unique'],
    data() {
        return {
            file: '',
            uploaded: [],
            forUpload: false,
            error: false,
            progressBar: false,
            width: 0,
            only: false,
            headers: {
                'Content-Type': 'multipart/form-data'
            },
            counter: 0
        }
    },
    beforeMount() {
        if (!this.getBaseUrl()) {
            console.error('Base url not found');
            return
        }
        this.listFile();
    },
    watch: {
        counter() {
            this.verifyOnly();
        }
    },
    methods: {
        async createFile() {
            let urlCreate = `${this.getBaseUrl()}/${this.action}`;

            let formData = new FormData();
            formData.append('file', this.file);
            try {
                const response = axios.post(urlCreate, formData, { headers: this.headers })
                if (response.status == 200) {
                    this.progress();
                    this.listFile();
                    return
                }
            } catch (error) {
                console.error(error);
                this.error = "Error to send file"
            }
        },

        async listFile() {
            let urlIndex = `${this.getBaseUrl()}/${this.action}`;
            try {
                const data = await axios.get(urlIndex)
                if (data) {
                    this.uploaded = response.data
                    this.counter = response.data.length
                    return
                }
            } catch (error) {
                console.error(error);
            }
        },
        showFile(id) {
            let urlIndex = `${this.getBaseUrl()}/${this.action}/${id}`;
            window.open(urlIndex, "_blank")
        },
        async removeFile(id) {
            let urlDelete = `${this.getBaseUrl()}/${this.action}/${id}`;
            try {
                const response = await axios.delete(urlDelete)
                if (response.status == 200) {
                    this.listFile();
                    return
                }
            } catch (error) {
                console.error(error);
            }
        },
        verifyType() {
            this.file = this.$refs.file.files[0];
            let filetype = this.file.type.split('/')[1];

            if (!this.ext.includes(filetype)) {
                this.forUpload = false;
                this.error = 'Invalid extension: ' + this.ext.join(', ');
                this.file = '';
                return
            }
            this.error = false;
            this.forUpload = true;

        },
        verifyOnly() {
            if (this.unique == true && this.counter > 0) {
                this.only = true;
                return
            }
            this.only = false;
        },
        cancelFile() {
            this.file = '';
            this.forUpload = false;
        },
        getBaseUrl() {
            const { protocol, host, pathname  } = window.location;
            let params = this.params ? this.params : '';
            let baseUrl = `${protocol}//${host}/${pathname.split('/')[1]}/${params}`;

            return baseUrl;
        },
        progress() {
            this.progressBar = true;
            setTimeout(() => {
                if (this.width < 100) {
                    this.width += 1;
                    this.progress();
                } else {
                    this.width = 0;
                    this.listFile();
                    this.cancelFile();
                    this.progressBar = false;
                }
            }, 25);
        },
    }
}
</script>

<style scope>
.bgicon {
    color: white;
}
</style>