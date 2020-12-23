<template>
  <div class="hello">

    <el-upload
      class="upload-demo"
      drag
      accept="image/jpeg,image/png"
      action="#"
      :show-file-list="false"
      :before-upload="beforeUpload"
      :on-change="onChange"
      >
      <i class="el-icon-upload"></i>
      <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
      <div class="el-upload__tip" slot="tip">若批量检测，请选择文件夹上传</div>

    </el-upload>

    <div class="btns">
      <form name="uploadFilesBox" id="form">
        <a href="javascript:;" class="input-file input-fileup" >
          <input ref="file" class="fileUploaderClass" type='file' id="uploadFilesBtn" name="uploadFilesBtn" webkitdirectory style="display: none" @change.stop="uploadFiles"/>
          <el-button type="primary" onclick="document.uploadFilesBox.uploadFilesBtn.click()">选择文件夹<i class="el-icon-upload el-icon--right"></i></el-button>
        </a>
      </form>

      <el-button id="check-btn" type="success" round @click="test" v-loading.fullscreen.lock="fullscreenLoading">检测</el-button>

      <el-button type="warning" round @click="saveFiles" v-if="showSave">下载</el-button>

    </div>

    <div class="ID_pic_wrap" style="width: 100%;height: 100%;padding: 0"  v-for="(item,i) in imgs">
      <img :src="item" alt="" v-show="showOld" style="" ref="myImg">
      <canvas :id="forId(i)" width="0" height="0" v-show="!showOld"></canvas>
    </div>


<!--    <input type="button" @click="test" value="检测" v-loading.fullscreen.lock="fullscreenLoading">-->

  </div>
</template>

<script>
  import axios from 'axios'
  import JSZip from 'jszip'
  import FileSaver from 'file-saver'
  import index from "../router";
export default {
  data() {
    return {
      src1:'',
      showOld: true,
      fullscreenLoading: false,
      imgs: [],
      showSave: false
    };
  },
  methods: {
    forId:function(index){
      return "myCanvas_" +index
    },
    uploadFiles () {
      // console.log(this.$refs.file.files);
      var filesList = []
      this.imgs = []
      this.src1 = ''
      this.showOld = true
      this.showSave = false

      filesList = this.$refs.file.files
      // console.log(filesList)
      var uploadFilesLength = this.$refs.file.files.length
      var _this = this

      for(let i=0;i<uploadFilesLength;i++){
        if(filesList[i].type == "image/jpeg" || filesList[i].type == "image/png"){
          var reader = new FileReader();
          reader.readAsDataURL(filesList[i]);
          reader.onload = function(e) {
            _this.imgs.push(e.target.result) //将图片路径赋值给src
          }
        }
      }
    },
    beforeUpload(file){
      const isJPG = file.type === 'image/jpeg';
      const isPNG = file.type === 'image/png';
      const isLt2M = file.size / 1024 / 1024 < 2;
      if (!isJPG && !isPNG) {
        this.$message.error('图片只能是JPG或PNG格式!');
      }
      if (!isLt2M) {
        this.$message.error('图片大小不能超过 2MB!');
      }
      return isJPG && isLt2M;
    },
    onChange(file,fileList) {
      var _this = this;
      var reader = new FileReader();

      this.showOld = true
      this.showSave = false
      this.imgs = []

      //转base64
      reader.readAsDataURL(file.raw);
      reader.onload = function(e) {
        // _this.src1 = e.target.result //将图片路径赋值给src
        if(!_this.imgs.includes(e.target.result))
          _this.imgs.push(e.target.result)
      }
      //
      // console.log(this.src1)
      // console.log(this.imgs.length)
    },
    draw(positions,index) {
      var c = document.getElementById("myCanvas_"+index);
      var cxt =c.getContext("2d");

      var _this = this;
      // console.log(positions)

      // if(count == 0){
      //   var img = new Image();
      //   img.src = this.imgs[index]
      //   c.width = img.width
      //   c.height = img.height
      //   cxt.drawImage(img, 0, 0, img.width, img.height);
      //   this.showOld = false
      // }

      var img = new Image();

      var promise = new Promise(                         // make a promise
        function(resolve, reject) {
          img.onload = function() {
            cxt.drawImage(img, 0, 0, img.width, img.height);
            resolve();     // keep the promise -- lets the "then" proceed
          };

          img.src = _this.imgs[index]
          c.width = img.width
          c.height = img.height
        }
      );

      promise.then( function() {
        cxt.lineWidth = 3;
        cxt.strokeStyle = "rgb(128, 0, 128)"; //"rgb(0, 0, 0)";

        cxt.fillStyle = "rgb(255,255,255)"
        cxt.font = '3px'
        cxt.textBaseline = 'top'

        for(var i=0;i<positions.length;i++)
        {
          cxt.strokeRect(positions[i].left, positions[i].top, positions[i].width, positions[i].height);

          var str = positions[i].orientation + " "+positions[i].score.toString()
          cxt.fillText(str,positions[i].left+2,positions[i].top+2)
          // console.log(positions[i])
        }
        _this.showOld = false
        console.log("画完第"+index)
      });


      // cxt.lineWidth = 3;
      // cxt.strokeStyle = "rgb(128, 0, 128)"; //"rgb(0, 0, 0)";
      // cxt.strokeRect(x, y, width, height);
      //
      // console.log("画完第"+index)
    },
    test() {
      var _this = this
      let access_token = '24.df4c811258167f341199b22fe0720186.2592000.1611054910.282335-22834848'
      let url = 'https://aip.baidubce.com/rest/2.0/image-classify/v1/body_attr?access_token='

      if(!this.imgs.length){
        this.$message.warning('未选择图片')
      }
      else
        this.showSave = true

      for(var num = 0;num < this.imgs.length; num++){
        var imgUrl = this.imgs[num];
        var img = imgUrl.replace(/^data:image\/\w+;base64,/, "");
        const bodyFormData = new FormData();

        bodyFormData.append('image',img);

        this.src1 = imgUrl;

        _this.timer(url,access_token,bodyFormData,num)
        // this.check(url+access_token,bodyFormData,num)

      }
    },
    timer(url,access_token,bodyFormData,num) {
      setTimeout( () =>{this.check(url+access_token,bodyFormData,num)},num*2000)
    },
    check(url,bodyFormData,num){
      console.log(num)
      var _this = this
      axios.post(url,bodyFormData,
        {headers :
            {'content-type': 'application/x-www-form-urlencoded'},

        })
        .then( (res) => {
          console.log(res)

          var person_num = res.data.person_num;

          if(res.data.error_msg == 'empty image'){
            this.$message.error('请选择图片');
            return 0
          }
          if(res.status != 200){
            this.$message.error('网络连接错误');
            return 0
          }

          if(person_num){
            let positions = []
            for(let i = 0;i<person_num;i++){

              // console.log(i+'----')
              var person = res.data.person_info[i]
              // console.log(`第${i+1}个人的位置`)
              // console.log(person.location)
              let height = person.location.height
              let left = person.location.left
              let top = person.location.top
              let width = person.location.width

              let orientation = person.attributes.orientation.name
              let score  = person.attributes.orientation.score.toFixed(3)

              console.log('正在画第'+num)
              var position = {height:height,left:left,top:top,width:width,orientation:orientation,score:score}
              positions.push(position)
              // _this.openFullScreen1(left,top,width,height,i,num)
            }
            _this.openFullScreen1(positions,num)
          }
          else {
            this.$message.error('识别不到人体');
            return 0
          }
        } )
    },
    openFullScreen1(positions,index) {
      var _this = this
      this.fullscreenLoading = true;
      setTimeout(() => {
        this.fullscreenLoading = false;
        this.draw(positions,index)
        // setTimeout(() => {
        //   _this.draw(positions,index)
        // },1000)

      }, 1000);
    },
    saveFiles(){
      if(this.imgs.length === 1){
        var strDataURI = document.getElementById("myCanvas_0").toDataURL("image/png",1);
        var dlLink = document.createElement('a');
        dlLink.download = "检测结果";
        dlLink.href = strDataURI;
        // dlLink.dataset.downloadurl = [MIME_TYPE, dlLink.download, dlLink.href].join(':');
        document.body.appendChild(dlLink);
        dlLink.click();
        document.body.removeChild(dlLink);
        return 0;
        // document.getElementById("download").setAttribute("href",strDataURI);
      }


      var blogTitle = '检测结果'
      var zip = new JSZip()
      var imgs = zip.folder(blogTitle)
      var baseList = []
      var imgNameList = [];

      for(var i=0;i<this.imgs.length;i++){
        let id = "myCanvas_"+i.toString()
        let img_name = 'res_'+i.toString()
        imgNameList.push(img_name)
        let url = document.getElementById(id).toDataURL("image/png",1)
        baseList.push(url.substring(22))
        if(baseList.length === this.imgs.length && baseList.length>0){
          for(let j=0;j<baseList.length;j++){
            imgs.file(imgNameList[j]+'.png',baseList[j],{base64:true})
          }
          zip.generateAsync({type:'blob'}).then(function (content){
            FileSaver.saveAs(content,blogTitle+'.zip')
          })
        }
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

.upload-demo {
  /*border: 1px solid #2c3e50;*/
}

.btns {
  /*border: 1px solid #2c3e50;*/
}
#form {
  margin-top: 10px;
}

#check-btn {
  margin: 10px 0;
}
</style>
