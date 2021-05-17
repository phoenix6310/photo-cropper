<template>
  <div class="photo">
    <div class="select-photo">
      <div class="tip">拍照</div>
      <input
        type="file"
        accept="image/*"
        ref="selectPhoto"
        placeholder="拍照"
      />
    </div>
    <div class="cropper-wrap" v-show="showCroperWrap">
      <div class="cropper-container">
        <vueCropper
          ref="cropper"
          :img="option.img"
          :outputSize="option.size"
          :outputType="option.outputType"
          :info="true"
          :full="option.full"
          :canMove="option.canMove"
          :canMoveBox="option.canMoveBox"
          :fixedBox="option.fixedBox"
          :original="option.original"
          :autoCrop="option.autoCrop"
          :autoCropWidth="option.autoCropWidth"
          :autoCropHeight="option.autoCropHeight"
          :centerBox="option.centerBox"
          :high="option.high"
          :infoTrue="option.infoTrue"
          :maxImgSize="option.maxImgSize"
          @imgLoad="imgLoad"
          @cropMoving="cropMoving"
          :enlarge="option.enlarge"
          :mode="option.mode"
          :limitMinSize="option.limitMinSize"
        ></vueCropper>
      </div>
      <div class="control-btns">
        <button @click="uploadOrigin" class="upload-btn">上传原图</button>
        <button @click="upload" class="upload-btn">上传截图</button>
      </div>
    </div>
    <!-- 显示数据 -->
    <div class="formula-list" ref="formulaList">
      <div
        v-for="item in latexResults"
        :key="item.source.article_id"
        class="formula-item"
      >
        <div class="item-text">
          <a :href="item.source.article_id" target="_blank">{{
          item.source.article_title
        }}</a>
        </div>
        <div class="item-text">
          {{ item.source.math_jax }}
        </div>
        <div class="item-text">
          {{ item.source.authors_name }}|{{ item.source.publish_year }}|{{
            item.source.similarity
          }}
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import { VueCropper } from "vue-cropper";
import axios from "axios";
export default {
  data() {
    return {
      showCroperWrap: false,
      option: {
        img: "",
        size: 1,
        full: false,
        outputType: "png",
        canMove: true,
        fixedBox: false,
        original: false,
        canMoveBox: true,
        autoCrop: true,
        // 只有自动截图开启 宽度高度才生效
        autoCropWidth: 200,
        autoCropHeight: 150,
        centerBox: false,
        high: false,
        cropData: {},
        enlarge: 1,
        mode: "contain",
        maxImgSize: 3000,
        limitMinSize: [100, 120],
      },
      originFile: null,
      latexResults: [],
    };
  },
  components: {
    VueCropper,
  },
  mounted() {
    let that = this;
    let fileTag = that.$refs.selectPhoto;
    fileTag.onchange = function () {
      let file = fileTag.files[0];
      console.log(file, "1");
      let fileReader = new FileReader();
      that.originFile = file;

      fileReader.onload = (e) => {
        let data;
        if (typeof e.target.result === "object") {
          data = window.URL.createObjectURL(new Blob([e.target.result]));
        }
        that.showCroperWrap = true;
        that.option.img = data;
      };

      fileReader.readAsArrayBuffer(file);
    };
  },
  methods: {
    uploadOrigin() {
      if (this.originFile) {
        this.uploadFn(this.originFile);
      }
    },
    cropMoving(data) {
      this.option.cropData = data;
    },
    imgLoad(msg) {
      console.log(msg);
    },
    upload() {
      this.$refs.cropper.getCropBlob((data) => {
        let blob = window.URL.createObjectURL(data);
        let name = Math.floor(Math.random() * 1000000);
        let file = new window.File([blob], name, { type: "image/png" });

        console.log(file, "2");

        this.uploadFn(file);
      });
    },
    uploadFn(file) {
      let formData = new FormData();
      formData.append("file", file);
      axios({
        method: "post",
        // url: "/api/latexsearch/API/ImageSearch",
        url: "https://shuyuan.apabi.com/latexsearch/API/ImageSearch",
        data: formData,
      }).then((res) => {
        console.log(res);
        this.latexResults = res.data.hits;

        this.$nextTick(() => {
          MathJax.Hub.Typeset(this.$refs.formulaList);
        });
      });
    },
  },
};
</script>

<style lang="scss">
.photo {
  font-size: 18px;
  .select-photo {
    background-color: #feae1b;
    color: #fff;
    width: 60px;
    height: 60px;
    overflow: hidden;
    position: relative;
    border-radius: 30px;
    margin: 30px auto 10px;
    input {
      display: block;
      font-size: 100px;
      outline-style: none;
      opacity: 0;
      position: absolute;
      cursor: pointer;
    }
    .tip {
      position: absolute;
      width: 60px;
      height: 60px;
      left: 0;
      top: 0;
      display: flex;
      justify-content: center;
      align-items: center;
    }
  }

  .cropper-wrap {
    .cropper-container {
      height: 500px;
    }
    .control-btns {
      margin: 0 auto;
      display: flex;
      width: 88%;
      justify-content: space-around;
      .upload-btn {
        margin-top: 20px;
        text-decoration: none;
        text-align: center;
        display: inline-block;
        appearance: none;
        cursor: pointer;
        border: none;
        font-size: 16px;
        height: 50px;
        line-height: 50px;
        padding: 0 30px;
        border-radius: 200px;
        background-color: #feae1b;
        color: #fff;
      }
    }
  }
  .formula-list {
    width: 88%;
    margin: 12px auto 0;
    .formula-item {
      display: flex;
      justify-content: space-around;
      border-bottom: 1px solid #eee;
      margin-bottom: 10px;
      padding: 6px 0;
      .item-text{
        width: 33.3%;
        text-align: left;
      }
    }
  }
}
</style>