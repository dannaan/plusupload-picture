<template>
  <div id="attachment">
    <div class="attachment_div" @click="showActionSheet()">
      +
    </div>
    <ul class="van-overflow-hidden">
      <li class="imgLi aaaa" v-for="(item, index) in uploadURLs">
        <img :src='item' class="upload_img"/>
        <span @click="delImg(index,item,id)"><span class="file-remove">+</span></span>
      </li>
      <p v-for="item in copyimgIDs" style="display: none;">
        {{item}}
      </p>
      <li class="imgLi  bbb" v-for="(item, index) in copyuploadURLs">
     <!--  <img :src='`URLPth`+item.pelativePath' class="upload_img"/>-->
        <img v-bind:src="URLPth+file+(item.pelativePath)" class="upload_img"/>
        <span @click="delImg1(index,item.id)"><span class="file-remove">+</span></span>
     </li>
    </ul>
    <div style="clear: both"></div>
    <p v-for="(item,index) in imgIDs" style="display: none">
      {{item}}
    </p>
  </div>
</template>
<script>
  import {Toast} from 'vant'

  export default {
    name: 'navbar',
    data() {
      return {
        uploadURLs: [],
        imageFile: [],
        imageNames: [],
        imgIDs: [],
        URLPth:'',
        file:'',
        copyimgIDs:[]
      }
    },

    methods: {
      showActionSheet() {
        let _this = this;
        var bts = [{
          title: "拍照"
        }, {
          title: "从相册选择"
        }];
        plus.nativeUI.actionSheet({
            cancel: "取消",
            buttons: bts
          },
          function (e) {
            if (e.index == 1) {
              _this.getImage();
            } else if (e.index == 2) {
              _this.galleryImgs();
            }
          }
        );
      },
      getImage() {
        let _this = this;
        var cmr = plus.camera.getCamera();
        cmr.captureImage(function (p) {
          plus.io.resolveLocalFileSystemURL(p, function (entry) {
            _this.compressImage(entry.toLocalURL(), entry.name);
          }, function (e) {
            plus.nativeUI.toast("读取拍照文件错误：" + e.message);
          });
        }, function (e) {
        }, {
          filter: 'image'
        });
      },
      galleryImgs() {
        let _this = this;
        plus.gallery.pick(function (event) {
          var files = event.files;
          for (var i = 0; i < files.length; ++i) {
            var file = files[i];
            _this.compressImage(file, file.substring(file.lastIndexOf('/') + 1, file.length));
          }
        }, function (e) {
        }, {
          filter: "image",
          multiple: true,
          maximum: 4,
          system: false
        });
      },
      showPics(url, name) {
        let _this = this;
        //根据路径读取到文件
        plus.io.resolveLocalFileSystemURL(url, function (entry) {
          entry.file(function (file) {
            let fileReader = new plus.io.FileReader();
            fileReader.readAsDataURL(file);
            fileReader.onloadend = function (e) {
              if (_this.uploadURLs.length > 3) {
                plus.nativeUI.toast("最多上传4张图片！");
                return false;
              } else {
                let picUrl = e.target.result.toString();

                _this.uploadURLs.push(picUrl);//这个是base64


                // _this.$emit('uploading',picUrl);

                let Token = localStorage.getItem('token');
                let URLPth= process.env.API_ROOT;
                alert(444);
                $.ajax({
                  url: URLPth+'/common/upload/images',
                  method: 'POST',
                  data: {
                    token:Token,
                    imageName:name,
                    imageStr:picUrl.split(",")[1]
                  },
                  contentType: 'application/x-www-form-urlencoded', // 注意这里应设为false
                  success: function (data) {
                    alert("正确的啦");

                    _this.imgIDs.push(data.result['id']);
                    _this.imgIDs.push(_this.copyimgIDs);

                    _this.$emit('attachmentsID', _this.imgIDs);
                  },
                  error: function (jqXHR) {
                    alert("错误啦");
                    console.log(JSON.stringify(jqXHR));
                  }
                })
              }
            }
          });
        });
      },
      compressImage(url, filename) {
        let _this = this;
        var name = "_doc/upload/" + filename;
        plus.zip.compressImage({
            src: url,
            dst: name,
            quality: 40,
            overwrite: true
          },
          function (zip) {
            //页面显示图片
            _this.showPics(zip.target, filename);
            _this.imageNames.push(name);

          }, function (error) {
            plus.nativeUI.toast("压缩图片失败，请稍候再试");
          });
      },
      delImg(index,id) {
        this.uploadURLs.splice(index, 1);
        this.imgIDs.splice(index, 1);
       },
      delImg1(index,id) {
         this.copyuploadURLs.splice(index, 1);
         this.copyimgIDs.splice(index, 1);
        this.$emit('attachmentsID', this.copyimgIDs);

       }
    },
    props:['copyuploadURLs','attachments'],
    created() {
      let URLPth= process.env.API_ROOT;
      this.URLPth = URLPth;
      this.file = '/file/';
    },
    watch: {
      attachments (val) {
        this.copyimgIDs = val.split(',');
        this.$emit('attachmentsID', this.copyimgIDs);
     }
    }
  }
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
  #attachment {
    .attachment_div {
      width: 120px;
      height: 120px;
      border: 10px solid #e2e2e2;
      line-height: 120px;
      text-align: center;
      font-size: 68px;
      color: #e2e2e2;
    }
    .upload_img {
      width: 130px;
      height: 130px;
      position: absolute;
      float: left;
      line-height: 130px;
      display: inline-block;
    }
    .file-remove {
      width: 35px;
      height: 35px;
      background: #666666;
      display: inline-block;
      text-align: center;
      line-height: 40px;
      color: white;
      border-radius: 50%;
      transform: rotate(45deg);
      font-size: 30px;
      position: absolute;
      left: 95px;
    }
    .imgLi {
      float: left;
      width: 140px;
      height: 140px;
      margin-right: 2px;
      position: relative;
      display: inline-block;
      margin-top: 20px;

      &:last-child{
        margin-right: 0;
      }
    }

  }
  ul{
    width: 570px;
  }
</style>
