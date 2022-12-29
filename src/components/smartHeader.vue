<template>
  <div>
    <el-container class="home-container">
      <el-header>
        <div class="header-logo">
          <div class="logo">
            <div>
              <img :src="logoImg"
                   alt="" />
            </div>
            <div class="logo-txt">
              <div class="txt-big">
                <span>智慧小区</span>
              </div>
              <!-- <div class="txt-small">
                <span>Beta版本</span>
              </div> -->
            </div>
          </div>
          <!-- 头部导航 -->
          <div class="header-menu">
            <el-menu class="el-menu-demo"
                     :default-active="activeIndex"
                     mode="horizontal"
                     @select="handleSelect"
                     router>
              <el-menu-item :index="item.path"
                            :disabled='item.disabled'
                            v-for="item in headMenu"
                            :key="item.path">{{ item.label }}</el-menu-item>
              <i class="el-icon-message-solid notice" @click="noticeDrawer"></i>
                            <span>
                              <el-select size="mini" @change='changeSet' v-model="value" style="paddingTop:15px;position:fixed;right: 100px; ">
                                <el-option
                                  v-for="item in options"
                                  :key="item.value"
                                  :label="item.label"
                                  :value="item.value">
                                </el-option>
                              </el-select>
                            </span>
                            <el-button type="warning" class="exit" @click="exit">退出登录</el-button>
            </el-menu>
          </div>
          <div></div>
        </div>
      </el-header>
      <el-container style="height:100%">
        <router-view></router-view>
      </el-container>
      <div class='control'>
        <div class='control_box'
             @click="toAdmin">
          <span>综合服务平台</span>
        </div>
      </div>
    </el-container>
    <el-drawer
        title="消息列表"
        :visible.sync="drawer"
        :direction="direction"
        :before-close="handleClose">

        <div style="marginLeft:30px;marginBottom:10px">
             <el-select v-model="sendType" placeholder="请选择消息类型" size="mini">
                <el-option
                  v-for="item in Typeoptions"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value">
                </el-option>
              </el-select>
             <el-input placeholder="请输入内容"
                        v-model="sendContent"
                        size="mini"
                        style="width:300px;marginLeft:30px"
                        clearable>
            </el-input>
        </div>
        <div v-for="(item,index) in list" :key="index">
          <el-card shadow="never" style="height:105px;fontSize:17px">
            <el-row :gutter="20">
              <el-col :span="2">
                <img src="../assets/images/提示.png" alt="" class="noticeType" v-if="item.type!=7">
                <img src="../assets/images/订单.png" alt="" class="noticeType" v-else>
              </el-col>
              <el-col :span="18">
                <div>{{item.content}}</div>
                <div style="color:#999999">{{item.sendDate}}</div>
              </el-col>
              <el-col :span="4">
                <img :src="item.sendStatusPic" v-if="item.sendStatusPic" title="标记已读"
                @mouseenter="green(index)" @mouseleave="gray(index)" @click="read(item.id)"
                 style="weight:24px;height:24px;marginTop:17px;marginLeft:20px">
                <div  style="color:#999999;marginTop:17px;marginLeft:17px" v-else>已读</div>
              </el-col>
            </el-row>
          </el-card>
        </div>
        <el-card shadow="hover" style="height:60px;fontSize:18px;textAlign:center;">
          <div style="display:flex;">
            <span style="flex:1;cursor: pointer;" @click="getList()" :class="{'disabled':pageNum==1}">首页</span>
            <span style="flex:1;cursor: pointer;" @click="getList(-1)" :class="{'disabled':pageNum==1}">上一页</span>
            <span style="flex:1;cursor: pointer;" @click="getList(1)" :class="{'disabled':pageNum==pageCount}">下一页</span>
          </div>
        </el-card>
        <el-card shadow="hover" 
        style="height:65px;fontSize:16px;textAlign:center;color:#999999;cursor: pointer;"
        @click.native="allread()"
        >
          一键已读
        </el-card>
      </el-drawer>
  </div>
</template>

<script>
import {manageSysMessageList,manageSysMessageRead,manageSysMessageAllRead} from '../api/basic'
export default {
  inject:['reload'],
  data () {
    return {
      value:1,
      options:[{
        label:'2020年账套',
        value:1
      },{
        label:'2021年账套',
        value:2
      }],
      sendType:null,
      Typeoptions:[
        {
          label:'报事报修',
          value:1
        },{
          label:'装修',
          value:2
        },{
          label:'绿化任务',
          value:3
        },{
          label:'卫生任务',
          value:4
        },{
          label:'家政服务',
          value:5
        },{
          label:'建议咨询',
          value:6
        },{
          label:'订单',
          value:7
        }
      ],
      sendContent:null,
      list:[
        
      ],
      pageNum:null,
      pageCount:null,//总页数
      rowCount:null,//总页数
      drawer: false,
      direction: 'rtl',
      logoImg: require('../../src/assets/images/logo.png'),
      activeIndex: this.$route.path,
      headMenu: [
        {
          path: '/overview',
          label: '概览',
        },
        
        {
          // path: '/Butler service',
          path: '/butler',
          label: '管家服务',
        },
        {
          // path: '/Operation Management',
          path: '/operation',
          label: '运营管理',
        },
        {
          // path: '/Commodity Center',
          path: '/commodity',
          label: '商品中心',
        },
        // {
        //   path: '/system',
        //   label: '系统设置',
        //   disabled: true,
        // },
      ],
    }
  },
  mounted () {
    if (localStorage.getItem('ACset') == 1) {
      this.value = '2020年账套'
    }else if (localStorage.getItem('ACset') == 2){this.value = '2021年账套'}
    
    this.getPath()
    this.pageNum = 1
    let resData ={
      pageNum:this.pageNum,
      size:5
    }
    manageSysMessageList(resData).then(res=>{
      console.log(res);
      this.rowCount=res.rowCount
      this.pageCount = res.pageCount
      res.tableList.forEach(item => {
        if(item.sendStatus==1){
          let obj = {
          content:item.content,
          sendDate:item.sendDate,
          sendStatusPic:require("../assets/images/未读.png"),
          type:item.type, // 消息类型（1.报事报修，2.装修，3.绿化任务，4.卫生任务,5.家政服务,6.建议咨询，7.订单）
          id:item.id
        }
        this.list.push(obj)
        }else{
          let obj = {
          content:item.content,
          sendDate:item.sendDate,
          type:item.type, // 消息类型（1.报事报修，2.装修，3.绿化任务，4.卫生任务,5.家政服务,6.建议咨询，7.订单）
          id:item.id
        }
        this.list.push(obj)
        }
        // console.log(this.list);
      });
    })
  },
  methods: {
    changeSet(val){
      console.log(val);
      localStorage.setItem('ACset',val)
      this.reload()
      // ACset
    },
    noticeDrawer(){
      this.drawer = true
    },
    // 动态改变图片
    green(i){
      console.log(i);
      this.list[i].sendStatusPic = require("../assets/images/已读.png")
    },
    gray(i){
      console.log(i);
      this.list[i].sendStatusPic = require("../assets/images/未读.png")
    },
    //标记已读
    read(id){
      console.log(id);
      manageSysMessageRead({manageSysMessageId:id}).then(res=>{
        console.log(res);
        if(res.status){
          this.$message({
            type:'success',
            message:res.message
          })
          // this.reload()
          this.getList()
        }
      })
    },
    //一键已读
    allread(){
      manageSysMessageAllRead().then(res=>{
        if(res.status){
          this.$message({
            type:'success',
            message:res.message
          })
          this.reload()
        }
      })
    },
    handleClose(){
      this.drawer = false
    },
    getList(num){
      this.list = []
      // 判断 如果是首页没有传参则跳转至第一页
      if(num == undefined){
        console.log('首页');
        this.pageNum =1
      }
      else{
        this.pageNum +=num
      }
      console.log(this.pageNum);
      let resData ={
      pageNum:this.pageNum,
      size:5
    }
    manageSysMessageList(resData).then(res=>{
      console.log(res);
      res.tableList.forEach(item => {
        if(item.sendStatus==1){
          let obj = {
          content:item.content,
          sendDate:item.sendDate,
          sendStatusPic:require("../assets/images/未读.png"),
          type:item.type, // 消息类型（1.报事报修，2.装修，3.绿化任务，4.卫生任务,5.家政服务,6.建议咨询，7.订单）
          id:item.id
        }
        this.list.push(obj)
        }else{
          let obj = {
          content:item.content,
          sendDate:item.sendDate,
          type:item.type, // 消息类型（1.报事报修，2.装修，3.绿化任务，4.卫生任务,5.家政服务,6.建议咨询，7.订单）
          id:item.id
        }
        this.list.push(obj)
        }
        console.log(this.list);
      });
    })
    },
    exit(){
      window.sessionStorage.clear(),
      this.$router.push('/login')
    },
    complex () {
      window.location.href = ' http://testmanage.kaidalai.cn/manage/#/headKf'
    },
    handleSelect (key, keyPath) { },
    // 路由过滤
    getPath () {
      var str = `${this.$route.path}`
      if (str.lastIndexOf('/') > 0) {
        this.activeIndex = str.substring(
          str.indexOf('/'),
          str.lastIndexOf('/')
        )
      } else {
        this.activeIndex = this.$route.path
      }
    },
    toAdmin () {
      this.$router.push('/company')
    },
  },
  watch: {
    // '$route': 'getPath',
    $route: {
      handler: function (val, oldVal) {
        this.getPath()
      },
      // 深度观察监听
      deep: true,
    },
    sendType:{
      handler(newV){
        this.list = []
        let resData={
          type:newV,
          size:5,
          pageNum:this.pageNum
        }
        manageSysMessageList(resData).then(res=>{
          res.tableList.forEach(item => {
        if(item.sendStatus==1){
          let obj = {
          content:item.content,
          sendDate:item.sendDate,
          sendStatusPic:require("../assets/images/未读.png"),
          type:item.type, // 消息类型（1.报事报修，2.装修，3.绿化任务，4.卫生任务,5.家政服务,6.建议咨询，7.订单）
          id:item.id
        }
        this.list.push(obj)
        }else{
          let obj = {
          content:item.content,
          sendDate:item.sendDate,
          type:item.type, // 消息类型（1.报事报修，2.装修，3.绿化任务，4.卫生任务,5.家政服务,6.建议咨询，7.订单）
          id:item.id
        }
        this.list.push(obj)
        }
        // console.log(this.list);
      });
        })
      },
       deep: true,
    },
    sendContent:{
      handler(newV){
        this.list = []
        let resData={
          content:newV,
          size:5,
          pageNum:this.pageNum
        }
        manageSysMessageList(resData).then(res=>{
          res.tableList.forEach(item => {
        if(item.sendStatus==1){
          let obj = {
          content:item.content,
          sendDate:item.sendDate,
          sendStatusPic:require("../assets/images/未读.png"),
          type:item.type, // 消息类型（1.报事报修，2.装修，3.绿化任务，4.卫生任务,5.家政服务,6.建议咨询，7.订单）
          id:item.id
        }
        this.list.push(obj)
        }else{
          let obj = {
          content:item.content,
          sendDate:item.sendDate,
          type:item.type, // 消息类型（1.报事报修，2.装修，3.绿化任务，4.卫生任务,5.家政服务,6.建议咨询，7.订单）
          id:item.id
        }
        this.list.push(obj)
        }
        // console.log(this.list);
      });
        })
      },
       deep: true,
    }
  },
}
</script>

<style scoped>
.exit{
    position: fixed !important;
    right: 300px; 
    top: 10px;
}
.notice{
  position: fixed !important;
  color:black;
    z-index: 999;
    font-size: 20px;
    right: 500px; 
    top: 20px;
    cursor: pointer;
}
.noticeType{
  height: 22px;
  width: 22px;
}
.disabled{
    pointer-events: none;
    cursor: default;
    opacity: 0.6;
}
.el-container {
    position: relative;
    overflow: hidden;
}
.control_box {
    margin-left: 20px;
    color: #ffffff;
    opacity: 0.6;
}
.control_box:hover {
    opacity: 1;
}
.control {
    cursor: pointer;
    width: 20px;
    height: 40px;
    line-height: 40px;
    background: rgba(0, 0, 0, 0.4);
    border-radius: 100px 0px 0px 100px;
    bottom: 20%;
    z-index: 1000;
    position: absolute;
    right: 0;
}
.control span {
    font-size: 16px;
    font-family: PingFangSC-Regular, PingFang SC;
    font-weight: 400;
}
.control:hover {
    width: 120px;
}
.home-container {
    height: 100vh;
}
.el-header {
    background-color: rgba(255, 255, 255, 1);
    color: #333;
    text-align: center;
    display: flex;
    align-items: center;
    box-shadow: 0px 2px 4px 0px rgba(224, 224, 224, 0.5);
}

.header-logo {
    display: flex;
}
.header-logo > .logo {
    display: flex;
    align-items: center;
}
/* .logo span{
    font-size: 20px;
    font-weight: 700;
    padding: 0 20px;
  } */
.logo img {
    width: 30px;
    height: 21px;
    margin-right: 20px;
    margin-left: 8px;
}
.el-menu.el-menu--horizontal {
    border-bottom: none;
}
.logo-txt {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-right: 30px;
}
.logo-txt .txt-big span {
    font-size: 20px;
    font-family: LiHeiPro;
    line-height: 20px;
}
.logo-txt .txt-small span {
    font-size: 12px;
}
.logo-txt .txt-small {
    margin: 0 9px;
    padding: 2px;
    border: 1px solid #e2e2e9;
    border-radius: 3px;
}
.el-menu--horizontal > .el-menu-item.is-active {
    border-bottom: 3px solid rgba(251, 71, 2, 1);
    color: #303133;
}
</style>
