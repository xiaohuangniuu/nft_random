<!-- Please remove this file from your project -->
<template>
    <div class="container mx-auto">
      <div class="my-6">
        <div class="overflow-x-auto">
          <table class="table w-full">
            <thead>
            <tr>
              <th></th>
              <th>文件名称</th>
              <th>权重</th>
              <th>操作</th>
            </tr>
            </thead>
            <tr v-if="!files.length">
              <td colspan="9">
                <div class="text-center p-5">
                  <h4>拖拽文件到此进行上传<br/></h4>
                  <label :for="name" class="btn btn-primary">选择文件支持多选</label>
                </div>
              </td>
            </tr>

            <tbody v-if="files.length">
            <tr>
              <td colspan="4">
                <div class="alert alert-warning">
                  <div class="flex-1">
                    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" class="w-6 h-6 mx-2 stroke-current">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path>
                    </svg>
                    <label>等待上传</label>
                  </div>
                </div>
              </td>
            </tr>
            <tr v-for="(file,index) in files" :key="file.id">
              <th>{{index+1}}</th>
              <td>
                <div class="flex items-center space-x-3">
                  <div class="avatar">
                    <div class="w-12 h-12 mask mask-squircle">
                      <img v-if="file.thumb" :src="file.thumb">
                    </div>
                  </div>
                  <div>
                    <div class="font-bold">
                      {{file.name}}
                    </div>
                  </div>
                </div>
              </td>
              <td><input type="text" placeholder="权重"  class="input input-bordered" disabled></td>
              <td>
                <button class="btn btn-secondary" @click="$refs.upload.remove(file)">删除</button>
              </td>
            </tr>
            </tbody>
            <tbody v-for="(filesInfo,name) in uploadedFile" :key="name">

              <tr>
                <td colspan="4">
                  <div class="alert alert-info">
                    <div class="flex-1">
                      <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" class="w-6 h-6 mx-2 stroke-current">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path>
                      </svg>
                      <label>{{name}}</label>
                    </div>
                  </div>
                </td>
              </tr>
              <tr v-for="(file,index) in filesInfo" :key="file.id">
                <th>{{index+1}}</th>
                <td>
                  <div class="flex items-center space-x-3">
                    <div class="avatar">
                      <div class="w-12 h-12 mask mask-squircle">
                        <img v-if="file.thumb" :src="file.thumb">
                      </div>
                    </div>
                    <div>
                      <div class="font-bold">
                        {{file.name}}
                      </div>
                    </div>
                  </div>
                </td>
                <td><input type="number" placeholder="权重" class="input input-bordered" @blur="watchWeight({'dir_name':dir_name,'category_name':name,'file_name':file.name},$event)"></td>
                <td>
                  <button  class="btn btn-secondary" @click="removeFile({'dir_name':dir_name,'category_name':name,'file_name':file.name})">删除远程文件</button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="example-foorer" >
          <div class="btn-group">
            <file-upload
              class="btn btn-primary"
              :post-action="postAction"
              :put-action="putAction"
              :extensions="extensions"
              :accept="accept"
              :multiple="multiple"
              :size="size || 0"
              :thread="thread < 1 ? 1 : (thread > 5 ? 5 : thread)"
              v-model="files"
              :data="{dir_name: '',category_name:''}"
              @input-filter="inputFilter"
              @input-file="inputFile"
              ref="upload">
              <div style="margin-top: 15px;">选择文件</div>
            </file-upload>

            <button type="button" class="btn btn-success" v-if="!$refs.upload || !$refs.upload.active" @click="openModal">
              <i class="fa fa-arrow-up" aria-hidden="true"></i>
              开始上传
            </button>
            <button type="button" class="btn btn-danger"  v-else @click.prevent="$refs.upload.active = false">
              <i class="fa fa-stop" aria-hidden="true"></i>
              结束上传
            </button>
            <div>
              <button type="button" class="btn btn-danger" @click="generateNFT">
                生成nft
              </button>
            </div>
          </div>
        </div>
      </div>


      <div v-bind:class="{ 'modal-open': isOpen }" class="modal">
        <div class="modal-box">
          <p>请输入素材类型名称(如头部,脚部等)</p>
          <input type="text" placeholder="名称" v-model="ElementName" class="input input-bordered" ><br/>
          <p>出现几率(百分比如100%,80%)</p>
          <input type="text" placeholder="出现几率(百分比如100%,80%)" v-model="percentage" class="input input-bordered" >%<br/>
          <p>图层层级从小到大(大的覆盖在小的上面)</p>
          <input type="text" placeholder="图层层级从小到大" v-model="index" class="input input-bordered" >

          <div class="modal-action">
            <button class="btn btn-primary" @click="defineModal">确定</button>
            <button class="btn btn-warning" @click="closeModal">关闭</button>
          </div>
        </div>
      </div>
    </div>



</template>
<script>
import { v4 as uuidv4 } from 'uuid';
import FileUpload from 'vue-upload-component'
export default {
  name: 'NuxtTutorial',
  props: {

  },
  components: {
    FileUpload,
  },
  data() {
    return {
      uploadedFile:{},
      categoryInfo:{},
      files:[],
      accept: 'image/png,image/gif,image/jpeg,image/webp',
      extensions: 'gif,jpg,jpeg,png,webp',
      multiple: true,
      drop: true,
      minSize: 1024,
      size: 1024 * 1024 * 10,
      upload_uuid:"",
      name: 'file',
      images: [],
      uploadFileAction: 'http://localhost:8080/upload_file',
      removeFileAction: 'http://localhost:8080/remove_file',
      generateNFTAction: 'http://localhost:8080/generate_nft',
      putAction: '',
      postAction:'',
      isOpen:false,
      ElementName:"",
      percentage:0,
      index:0,
      thread: 3,
      dir_name:uuidv4(),
      data:{},
    }
  },
  setup(props) {
    console.log(props)
  },
  updated: function () {
    this.$nextTick(function () {
      if (this.$refs.upload && this.$refs.upload.uploaded) {
        this.$refs.upload.clear();
      }
    })
  },
  methods: {
    inputFilter(newFile,oldFile,prevent) {
      if (newFile && !oldFile) {
        if (/(\/|^)(Thumbs\.db|desktop\.ini|\..+)$/.test(newFile.name)) {
          return prevent()
        }
        if (/\.(php5?|html?|jsx?)$/i.test(newFile.name)) {
          return prevent()
        }
      }

      if (newFile && newFile.error === "" && newFile.file && (!oldFile || newFile.file !== oldFile.file)) {
        // Create a blob field
        // 创建 blob 字段
        newFile.blob = ''
        let URL = (window.URL || window.webkitURL)
        if (URL) {
          newFile.blob = URL.createObjectURL(newFile.file)
        }
        // Thumbnails
        // 缩略图
        newFile.thumb = ''
        if (newFile.blob && newFile.type.substr(0, 6) === 'image/') {
          newFile.thumb = newFile.blob
        }
      }
    },
    inputFile(newFile,oldFile) {
      if (newFile && oldFile) {
        // update
        if (newFile.error && !oldFile.error) {
          alert("上传失败,请稍后尝试");
        }
      }

      // 删除文件
      if (!newFile && oldFile) {
        // remove
        if (oldFile.success && oldFile.response.id) {
          // post id
          console.log("删除文件成功")
        }
      }

      // 自动上传
      if (Boolean(newFile) !== Boolean(oldFile) || oldFile.error !== newFile.error) {
        if (this.uploadAuto && !this.$refs.upload.active) {
          this.$refs.upload.active = true
        }
      }
    },
    watchWeight(params,event) {
      if (params.category_name in this.uploadedFile) {
        for (let i=0;i<this.uploadedFile[params.category_name].length;i++) {
          if (this.uploadedFile[params.category_name][i].name === params.file_name) {
            this.uploadedFile[params.category_name][i].data = {'weight':event.valueOf()}
          }
        }
      }
    },
    openModal() {
      this.isOpen = true;
    },
    closeModal() {
      this.isOpen = false;
    },
    //上传文件
    async defineModal() {
      if (this.ElementName === "" ) {
        alert("元素名称不能为空");
        return;
      }
      if (this.files.length === 0 ) {
        alert("还未选择文件");
        return;
      }
      this.isOpen = false

      //获取名称 修改PostAction
      for (let i = 0; i<this.files.length;i++) {
        let formData = new FormData();
        formData.append("file", this.files[i].file);
        formData.append("dir_name", this.dir_name);
        formData.append("category_name", this.ElementName);

        const requestOptions = {
          method: "POST",
          body: formData,
        }

        //https://www.cnblogs.com/liufei1983/p/8735031.html
        const response = await fetch(this.uploadFileAction, requestOptions)
        const responseJson = await response.json();
        if (responseJson.code === 1000) {
          if (this.ElementName in this.uploadedFile) {
            this.uploadedFile[this.ElementName].push(this.files[i])
          }else {
            this.uploadedFile[this.ElementName] = [];
            this.uploadedFile[this.ElementName].push(this.files[i])
          }
        }

      }
      this.categoryInfo[this.ElementName] = {"index":this.index,"percentage":this.percentage}
      console.log("categoryInfo",this.categoryInfo)
      this.ElementName = "";
      this.index = 0;
      this.percentage = 0;
      this.$refs.upload.clear();
      this.$forceUpdate()
    },
    async generateNFT() {
      if (this.uploadedFile.length === 0 ){
        alert("未上传文件");
        return;
      }

      let categoryList = [];
      for (let category_name in this.uploadedFile) {
        let categoryInfo = {'category_name':category_name,"index":1,"percentage":0,"files":[]};

        if (category_name in this.categoryInfo) {
          categoryInfo["index"] = this.categoryInfo[category_name]["index"]
          categoryInfo["percentage"] = this.categoryInfo[category_name]["percentage"]
        }

        for (let i = 0;i<this.uploadedFile[category_name].length;i++) {
          let fileInfo = {
            "file_name":this.uploadedFile[category_name][i].name,
            "weight":this.uploadedFile[category_name][i].data['weight']
          }
          categoryInfo["files"].push(fileInfo);
        }
        categoryList.push(categoryInfo)
      }

      console.log("categorylist",categoryList)
      const requestOptions = {
        method: "POST",
        body: JSON.stringify({
          "uuid":this.dir_name,
          "category":categoryList,})
      }
      const response = await fetch(this.generateNFTAction, requestOptions)
      const responseJson = await response.json();
      console.log(responseJson)
      // if (responseJson.code === 1000) {
      //   if (this.ElementName in this.uploadedFile) {
      //     this.uploadedFile[this.ElementName].push(this.files[i])
      //   }else {
      //     this.uploadedFile[this.ElementName] = [];
      //     this.uploadedFile[this.ElementName].push(this.files[i])
      //   }
      // }
    },
    // 删除文件
    removeFile(params) {
      let category_name = params.category_name;
      let file_name = params.file_name;
      const requestOptions = {
        method: "POST",
        body: JSON.stringify({
          "dir_name":params.dir_name,
          "category_name":category_name,
          "file_name":file_name  })
      }
      fetch(this.removeFileAction,requestOptions).then(response => response.text()).then((data)=> {
        let dataJson = JSON.parse(data)
        if (dataJson.code === 1000) {
          for (category_name in this.uploadedFile) {
            let delIndex = -1
            this.uploadedFile[category_name].forEach(function (element,index) {
              if (element.name === file_name) {
                delIndex = index
              }
            })

            if (delIndex > -1) {
              this.uploadedFile[category_name].splice(delIndex,1)
            }
          this.$forceUpdate()
          }
          alert("删除成功");
        }else{
          alert(dataJson.msg);
        }
      })
    }
  }
}
</script>
