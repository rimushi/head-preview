<template>
    <div class="moreLink">         
          <el-form label-position="top"  class="dialog-owner">             
                <el-form-item label="" class="upload-item" prop="file">
                    <div class="picture-outer">
                      <div class="picture-inner">
                          <i class="el-icon-plus avatar-uploader-icon" v-if="noPicture"> </i>  
                          <img v-else :src="pictureSrc" id="J_picture" :class="pictureClass"> 
                          <input type="file" class="picture-input" id="J_upload" accept="image/*" capture="camera" @change="previewFile">
                      </div>
                    </div>
               </el-form-item>
               <div class="divTip">点击图片可更换,适用于各种终端的头像预览</div>
          </el-form>             
      </div>
</template>
<script>
  import EXIF from "exif-js"
  import lrz from "lrz" 
  export default {
    data () {
      return {         
          noPicture: true,
          picturSource: '',
          pictureSrc: '',
          pictureClass: ''
      }
    },
    created(){
        //手机端页面自适应解决方案—rem布局
        (function (doc, win) {
            var docEl = doc.documentElement,
            resizeEvt = 'orientationchange' in window ? 'orientationchange' : 'resize',
            recalc = function () {
                var clientWidth = docEl.clientWidth;
                if (!clientWidth) return;
                if(clientWidth>=640){
                    docEl.style.fontSize = '100px';
                }else{
                    docEl.style.fontSize = 100 * (clientWidth / 640) + 'px';
                }
            };

            if (!doc.addEventListener) return;
            win.addEventListener(resizeEvt, recalc, false);
            doc.addEventListener('DOMContentLoaded', recalc, false);
      })(document, window); 
    },
    methods: {     
      previewFile(e) {  
          var self = this;
          var v, Orientation, orient, picWidth, picHeight;
          var file = e.target.files[0];
          // console.log(file.size/1024);

          //配置类似localResizeIMG最大宽或最大高
          var config = {
              width: document.getElementById("J_upload").offsetWidth*4,
              height: document.getElementById("J_upload").offsetHeight*4,
              quality: 0.8
          };  

          //如果是IOS需要判断旋转方向
          var ua = navigator.userAgent.toLowerCase(); 
          if (/iphone|ipad|ipod/.test(ua)) {
            EXIF.getData(file, function() {
                Orientation = EXIF.getTag(this, 'Orientation');
                if(typeof Orientation != 'undefined'){
                  orient = Orientation;
                }               
            }); 
          } 

          lrz(file,config)
          .then(function (rst) {
              //根据旋转后的预览图计算高度和宽度的比值，然后设置水平或竖直居中
              if(window.FileReader) {
                var fr = new FileReader();                       
                fr.readAsDataURL(file);                            
                fr.onloadend = function(e) {
                    //这里的e是fileReader对象   
                    self.picturSource = e.target.result;
                    var img = new Image();
                    img.src = e.target.result;
                    img.onload = function() { 
                        //根据旋转度取出图片的长宽比 
                        //因为6和8需要旋转正副90度                       
                        if(orient == 6 || orient == 8){                                               
                          picWidth = img.height;
                          picHeight = img.width;
                        } else {
                          picWidth = img.width;
                          picHeight = img.height;
                        } 
                        v = picHeight/picWidth; 

                        self.noPicture = false;
                        self.pictureSrc = rst.base64;

                        if(v>1) {      
                          //宽度固定，高度竖向居中                     
                          self.pictureClass = "picture-img-width";                
                        } else {
                          //高度固定，宽度竖向居中 
                          self.pictureClass = "picture-img-height";
                        }    
                    }
                }
              }
          })
          .catch(function (err) {
              // console.log(err)
              // console.log('压缩失败')
          })
          .always(function () {
              // 清空文件上传控件的值
              e.target.value = null
          });         
      }      
    }    
  }


</script>
<style>
.moreLink{ 
  color: #6fa7c6;
  font-size: 14px;
}
.moreLink .avatar-uploader .el-upload {
  border: 1px solid #6fa7c6;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  margin-left: 20%;
}

.moreLink .avatar-uploader-icon {
  font-size: 28px;
  color: #6fa7c6;
  width: 3rem;
  height: 3rem;
  line-height: 3rem;
  text-align: center;
}
.picture-outer {
  text-align: center;
}
.picture-inner {
  position: relative;
  margin: 0 auto;
  width: 3rem;
  height: 3rem;
  border: 1px solid #6fa7c6;
  border-radius: .1rem;
  overflow: hidden;
  /*display: flex;*/
}
.picture-img {
  /*max-width: 3rem;
  max-height: 3rem; */
  /*margin: auto;*/
}

.picture-img-width {
  /*max-width: 3rem;
  max-height: 3rem;*/ 
  /*margin: auto;*/  
  width: 3rem;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
.picture-img-height {
  height: 3rem;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
/*重置上传按钮是透明色*/
.picture-input { 
  width: 3rem;
  height: 3rem;   
  cursor: pointer;
  font-size: .75rem;
  outline: medium none;
  position: absolute;
  filter: alpha(opacity=0);
  -moz-opacity: 0;
  opacity: 0; 
  left: 0;
  top: 0;
}
</style>