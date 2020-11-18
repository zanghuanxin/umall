<template>
  <div class="add">
    <el-dialog :title="info.title" :visible.sync="info.isshow" @closed="closed">
      <el-form :model="list" :rules="rules">
        <el-form-item label="标题" label-width="120px" prop="title">
          <el-input v-model="list.title" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图片" label-width="120px" v-if="list.pid!==0" prop="img">>
          <!-- 1.原生js上传图片 -->
          <!-- 1.绘制html +css  -->
          <!-- 如果添加成功，此时，input上的文件应该清掉，所以直接将input节点清除 -->
          <!-- <div class="myupload">
            <h3>+</h3>
            <img class="img" v-if="imgUrl" :src="imgUrl" alt="">
           
            <input v-if="info.isshow" type="file" class="ipt" @change="changeFile">
          </div>-->

          <!-- 2.element-ui 上传文件 -->
          <el-upload
            class="avatar-uploader"
            action="#"
            :show-file-list="false"
            :on-change="changeFile2"
            :before-upload="beforeUpload"
           
          >
            <img v-if="imgUrl" :src="imgUrl" class="avatar" />
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
        </el-form-item>
        <el-form-item label="状态" label-width="120px">
          <el-switch v-model="list.status" :active-value="1" :inactive-value="2"></el-switch>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
      <el-button type="primary" v-if="info.title=='添加轮播图'" @click="add">添 加</el-button>
        <el-button type="primary" v-else @click="update">修 改</el-button>

      </div>
        </el-dialog>
  </div>
</template>

<script>
import { errAlert, successAlert } from "../../../utils/alert";
import path from "path";
import { mapGetters, mapActions } from "vuex";
import { reqbannerList,reqbannerAdd, reqbannerDetail, reqbannerUpdate } from "../../../utils/http";
export default {
  props: ["info"],
    data() {
    return {
      list: {
        title:"",
        img: null,
        status: 1
      },
        imgUrl: "",
         rules: {
        title: [{ required: true, message: "请输入分类名称", trigger: "blur" }], 
      },
    };
  },
  computed:{
...mapGetters({
    cateList: "banner/list"
})
  },
  methods: {
    cancel() {
      this.info.isshow = false;
    },
    empty() {
    this.list = {
       title:"",
        img: null,
        status: 1
      }
      this.imgUrl=""
    },
      check() {
      return new Promise((resolve, reject) => {
        //验证
        if (this.list.title === "") {
        errAlert("标题不能为空");
          return;
        }
        if (!this.list.img) {
          errAlert("请选择图片");
          return;
        }
        resolve();
      });
    },
    beforeUpload(file){
      let extname = path.extname(file.name);
       let arr = [".jpg", ".jpeg", ".png", ".gif"];
       if (file.size > 2 * 1024 * 1024) {
        errAlert("文件大小不能超过2M");
        return false;
      } else if(!arr.includes(extname) ){
         errAlert("请上传正确的图片格式！");
        return false;
      } 
    },
    changeFile2(e){
      let  file=e.raw;
      this.imgUrl=URL.createObjectURL(file);
      this.list.img=file;
    },
    add() {
       this.check().then(() => {
        //添加逻辑
         reqbannerAdd(this.list).then(res => {
        if (res.data.code == 200) {
          successAlert("添加成功"),
            this.cancel()
            this.empty()
            this.reqList()
        }
      });
      });
     
    },
    ...mapActions({
      reqList:"banner/reqList"
    }),
    getOne(id) {
      reqbannerDetail(id).then(res => {
        this.list = res.data.list;


        this.imgUrl=this.$imgPre+ this.list.img;
        this.list.id=id;
      });
    },
    update() {
       this.check().then(() => {
        //添加逻辑
          reqbannerUpdate(this.list).then(res=>{
          if(res.data.code==200){
              successAlert("修改成功"),
              this.cancel();
              this.empty();
              this.reqList()
          }
      })
      });
    
    },
    closed(){
        if(this.info.title==='编辑轮播图'){
            this.empty()
        }
    }
  },

  mounted() {
    reqbannerList().then(res => {
      if (res.data.code === 200) {
          this.bannerList = res.data.list;
      }
    });
  },
};
</script>

<style scoped lang="stylus">
.myupload {
  width: 100px;
  height: 100px;
  border-radius: 5px;
  border: 1px dashed #ccc;
  position: relative;
}
.myupload h3 {
  width: 100%;
  height: 100px;
  font-size: 30px;
  text-align: center;
  line-height: 100px;
  color: #666;
  font-weight: 100;
}
.myupload .ipt {
  width: 100px;
  height: 100px;
  position: absolute;
  left: 0;
  top: 0;
  opacity: 0;
}
.myupload .img {
  width: 100px;
  height: 100px;
  position: absolute;
  left: 0;
  top: 0;
}

  // 穿透
 .add >>> .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
  .avatar-uploader .el-upload:hover {
    border-color: #409EFF;
  }
  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
  }
  .avatar {
    width: 178px;
    height: 178px;
    display: block;
  }

</style>