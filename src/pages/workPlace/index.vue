<template>
  <div class="background font">
    <div v-if="dataArr.length == 0" class="noContant">
      暂无工地
    </div>
    <div>
      <i-cell-group>
        <i-cell @click="toPage('placeView',item.id)" v-for="(item,index) in dataArr" :key="index">
          <div class="place">
            <span class="placeHead">{{item.firstLetter}}</span>
            <div>
              <p style="font-weight:900;font-size:16px;">{{item.address}}</p>
              <p>
                <span>{{item.workTypeName}} |</span>
                <span> {{item.statusName}}</span>
              </p>
              <p>{{item.beginDate}}入场</p>
            </div>
                    <div class="box">
          <div class="text">{{item.statusName}}</div>
        </div>
          </div>
          <!-- <p>{{item.workTypeName}}</p>
          <p>{{item.beginDate}}</p>
          <p>{{item.statusName}}</p> -->
          <i-icon v-if="item.status == 1" @click="selectedItem = item; actionVisible = true" slot="footer" type="other"
            size="28" color="#80848f" />
        </i-cell>
      </i-cell-group>
    </div>
    <div>
      <i-action-sheet :visible="actionVisible" :actions="actions" @cancel="handleClickCancel" @clickItem="handleClickItem" />
    </div>
    <i-modal title="结账" :visible="visible" @ok.stop="handleOk" @cancel.stop="handleClose">
    </i-modal>
    <i-modal title="删除确认" :visible="deleteVisible" @cancel.stop="handleClose" @ok.stop="deleteOk">
      <view>删除后无法恢复哦</view>
    </i-modal>
    <i-message id="message" />
  </div>
</template>

<script>
  import VueTabBar from '../../components/tabBar/tabBar.vue'
  const {
    $Message
  } = require('../../../static/iview/base/index.js');
  export default {
    data() {
      return {
        searchData: {
          currentPage: 1,
          pageSize: 10000,
          status: ''
        },
        deleteVisible: false,
        actionVisible: false,
        selectedItem: {},
        realMoney: '',
        dataArr: [],
        visible: false,
        actions: [{
            name: '编辑',
          },
          {
            name: '记工'
          },
          {
            name: '收款'
          },
          {
            name: '结账',
          },
          {
            name: '删除',
            color: '#ed3f14'
          }
        ]
      }
    },

    computed: {

    },

    components: {
      VueTabBar
    },

    methods: {
      deleteOk() {
        this.$http.post('/work/address/openOrClose', {
          active: 0,
          ids: [this.selectedItem.workId]
        }).then(res => {
          if (res.success) {
            this.getAdderssList(this.searchData)
            this.deleteVisible = false
            this.actionVisible = false
            $Message({
              content: '删除成功',
              // type: 'warning'
            });
          } else {
            $Message({
              content: res.message,
              type: 'warning'
            });
          }
        })
      },
      handleClickCancel() {
        this.actionVisible = false
      },
      // 修改了static/iview/action-sheet文件
      handleClickItem(event) {
        switch (event.target.name) {
          case '编辑':
            this.toPage('newPlace', this.selectedItem.workId)
            this.actionVisible = false
            break
          case '记工':
            if (this.selectedItem.payrollSystem == 3) {
              $Message({
                content: '包工制无需记工',
                type: 'warning'
              });
            } else {
              this.toPage('remarkTime', this.selectedItem.workId)
              this.actionVisible = false
            }
            break
          case '收款':
            this.toPage('actualPay', this.selectedItem.workId)
            this.actionVisible = false
            break
          case '结账':
            this.visible = true
            break
          case '删除':
            this.deleteVisible = true
            break
        }
      },
      getAdderssList(pamars) {
          wx.showLoading({
          mask: true
        })
        this.$http.post('/work/address/list', pamars).then(res => {
          if (res.success) {
             wx.hideLoading()
            this.dataArr = res.info.list
            this.dataArr.forEach(e => {
              e.firstLetter = e.address.charAt(0)
            });
          }
        })
      },
      toPage(to, data) {
        wx.navigateTo({
          url: `/pages/${to}/main?id=${data}`,
          success: function (res) {
            console.log("success")
          },
        })
      },
      handleClose() {
        this.visible = false
      },
      // 结账
      handleOk() {
        this.$http.post('/receivable/save', {
          workId: this.selectedItem.workId
        }).then(res => {
          if (res.success) {
            this.visible = false
            this.actionVisible = false
            $Message({
              content: '结账成功',
            });
          } else {
            $Message({
              content: res.message,
              type: 'warning'
            });
          }
        })
      }
    },
    onShow() {
      var status = wx.getStorageSync("status")
       this.searchData.status = status ? status : ''
      this.getAdderssList(this.searchData)
    },
    onHide(){
      wx.removeStorageSync('status')
    }
  }
</script>

<style lang="stylus" scoped>
  @import '../../styles/common.styl'

  .place {
    display flex;
    align-items center;

    .placeHead {
      font-size: 30px;
      border: 1px solid #ddd;
      width: 50px;
      height: 50px;
      line-height: 50px;
      text-align: center;
      border-radius: 50%;
      margin-right: 20px;

    }
        .box {
      width: 160rpx;
      height: 160rpx;
      background-color: $theme;
      color: #fff;
      transform: rotate(45deg);
      -ms-transform: rotate(45deg);
      -moz-transform: rotate(45deg);
      -webkit-transform: rotate(45deg);
      -o-transform: rotate(45deg);
      position: absolute;
      right: -100rpx;
      top: -100rpx;
      text-align: center;

    }

    .text {
      position: absolute;
      bottom: 0;
      font-size 10px;
      left:25px;
    }
  }
</style>