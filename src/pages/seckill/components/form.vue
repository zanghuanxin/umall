<template>
  <div class="add">
    <el-dialog :title="info.title" :visible.sync="info.isshow" @closed="closed">
      <el-form :model="list">
        <el-form-item label="活动名称" label-width="120px">
          <el-input v-model="list.title" autocomplete="off"></el-input>
        </el-form-item>

          <div class="block">
    <!-- <span class="demonstration">起始日期时刻为 12:00:00，结束日期时刻为 08:00:00</span> -->
      <el-form-item label="活动期限" label-width="120px">
  <el-card class="box-card">
     
  <el-date-picker 
    v-model="dateTime" 
    type="datetimerange" 
    range-separator="至" 
    start-placeholder="开始日期" 
    end-placeholder="结束日期" 
    size="medium">
  </el-date-picker>

    </el-card>
     </el-form-item>

  </div>

        <el-form-item label="一级分类" label-width="120px" prop="first_cateid">
          <el-select v-model="list.first_cateid" placeholder="请选择分类" @change="changeFirst">
            <el-option :value="0" label="顶级分类"></el-option>
            <el-option
              v-for="item in  cateList"
              :key="item.id"
              :label="item.catename"
              :value="item.id"
            ></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="二级分类" label-width="120px" prop="second_cateid">
          <el-select v-model="list.second_cateid" placeholder="请选择分类" @change="changeSecond">
            <el-option
              v-for="item in secondCateList"
              :key="item.id"
              :value="item.id"
              :label="item.catename"
            ></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="商品" label-width="120px" prop="goodsid">
          <el-select v-model="list.goodsid" placeholder="请选择商品" @change="getGoods">
            <el-option
              v-for="item in list1"
              :key="item.id"
              :value="item.id"
              :label="item.goodsname"
            ></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="状态" label-width="120px">
          <el-switch v-model="list.status" :active-value="1" :inactive-value="2"></el-switch>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancel">取 消</el-button>
        <el-button type="primary" v-if="info.title=='活动添加'" @click="add">添 加</el-button>
        <el-button type="primary" v-else @click="update">修 改</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { errAlert, successAlert } from "../../../utils/alert";
import path from "path";
import { mapGetters, mapActions } from "vuex";
import {
  reqcateList,
  reqgoodsList,
  reqseckList,
  reqseckAdd,
  reqseckDetail,
  reqseckUpdata,
} from "../../../utils/http";
export default {
  props: ["info"],
  data() {
    return {
      list: {
        title: "",
        begintime: null,
        endtime: null,
        first_cateid: "",
        second_cateid: "",
        goodsid: [],
        status: 1    
      },
      list1: [],
      secondCateList: [],
    };
  },
  computed: {
    ...mapGetters({
      cateList: "cate/list",

      goodlist: "goods/list",
    }),
     dateTime: {
        get() {
      if (this.list.begintime && this.list.endTime) {
         return [this.list.begintime, this.list.endTime]
       } else {
         return []
      }
    },
    set() {
      // console.log(v)
      this.list.begintime = null
      this.list.endTime = null
    }
  }
  },
  methods: {
    cancel() {
      this.info.isshow = false;
    },
    empty() {
      this.list = {
        title: "",
       begintime: null,
        endtime: null,
        first_cateid: "",
        second_cateid: "",
        goodsid: [],
        status: 1,
      };

      secondCateList: [];
    },
    add() {
      reqseckAdd(this.list).then((res) => {
        console.log(this.cateList);

        if (res.data.code == 200) {
          successAlert("添加成功"), this.cancel();
          this.empty();
          this.reqList();
          this.reqSeckList();
        }
      });
    },
    ...mapActions({
      reqSeckList: "seck/reqList",
      reqCateList: "cate/reqList",
      reqGoodsList: "goods/reqList",
    }),
    changeFirst() {
      //二级分类的id重置
      this.list.second_cateid = "";
      this.list.goodsid = "";
      this.getSecondList();
    },
    getSecondList() {
      //获取二级分类list
      reqcateList({ pid: this.list.first_cateid }).then((res) => {
        this.secondCateList = res.data.list;
      });
    },
    changeSecond() {
      this.list.goodsid = "";

      let fid = this.list.first_cateid;
      let sid = this.list.second_cateid;

      this.getGoods({ fid, sid });
    },
    getGoods(p) {
      reqgoodsList(p).then((res) => {
        this.list1 = res.data.list;
      });
    },

    getOne(id) {
      reqseckDetail(id).then((res) => {
        this.list = res.data.list;
        this.list.id = id;
      });
    },
    update() {
      reqseckUpdata(this.list).then((res) => {
        if (res.data.code == 200) {
          successAlert("更新成功"), this.cancel();
          this.empty();
          reqSeckList: "seck/reqList";
        }
      });
    },
    closed() {
      if (this.info.title === "活动修改") {
        this.empty();
      }
    },
  },

  mounted() {
    this.reqCateList();
    this.getSecondList();
    this.getGoods();
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