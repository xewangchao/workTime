<template>
  <div class="font">
    <i-cell-group>
      <i-cell title="当前工地:">
        <input slot="footer" v-model="palceName" :disabled="true" />
      </i-cell>
      <i-cell title="开始时间:" v-if="placeData.payrollSystem == 1">
        <picker slot="footer" mode="time" :value="placeData.startTime" @change="bindStartDateChange">
          <input v-model="placeData.startTime" :disabled="true" />
        </picker>
      </i-cell>
      <i-cell title="结束时间:" v-if="placeData.payrollSystem == 1">
        <picker slot="footer" mode="time" :value="placeData.endTime" @change="bindEndDateChange">
          <input v-model="placeData.endTime" :disabled="true" />
        </picker>
      </i-cell>
      <i-cell title="工作天数:" v-if="placeData.payrollSystem == 2">
        <radio-group slot="footer" class="radio-group" @change="handleFruitChange">
          <label class="radio" v-for="(item, index) in radioArray" :key="item.code">
            <radio :value="item.code" /> {{item.name}}
          </label>
        </radio-group>
      </i-cell>
    </i-cell-group>

    <div class="btn">
      <button @click="save">保存</button>
    </div>
    <i-message id="message" />
    <!-- <VueTabBar></VueTabBar> -->
  </div>
</template>

<script>
  import VueTabBar from '../../components/tabBar/tabBar.vue'
  const {
    $Message
  } = require('../../../static/iview/base/index.js');
  import {
    formatTime
  } from '../../utils/index.js'
  export default {
    data() {
      return {
        palceName: '',
        placeData: {
          startTime: '',
          endTime: '',
          hour: '',
          payrollSystem: '',
          workId: ''
        },
        radioArray: [{
            code: 1,
            name: '1天'
          },
          {
            code: 0.5,
            name: '0.5天'
          }
        ]
      }
    },

    components: {
      VueTabBar
    },

    methods: {
      getPlaceById() {
        this.$http.get(`/work/address/queryByPk/${this.placeData.workId}/`).then(res => {
          this.palceName = res.info.address
          this.placeData.payrollSystem = res.info.payrollSystem
        })
      },
      save() {
        wx.showLoading({
          mask: true
        })
        this.$http.post(`/work/log/save`, this.placeData).then(res => {
          if (res.success) {
            for (let i in this.placeData) {
              this.placeData[i] = ''
            }
            wx.hideLoading()
            wx.switchTab({
              url: `/pages/workPlace/main`,
              success: function (res) {
                $Message({
                  content: '保存成功',
                  // type: 'warning'
                });
              }
            })
          } else {
            $Message({
              content: res.message,
              type: 'warning'
            });
          }
        })
      },
      bindStartDateChange(data) {
        this.placeData.startTime = formatTime(new Date()) + ' ' + data.target.value + ':00'
      },
      bindEndDateChange(data) {
        this.placeData.endTime = formatTime(new Date()) + ' ' + data.target.value + ':00'
      },
      handleFruitChange(data) {
        this.placeData.hour = data.target.value
      }
    },
    onShow() {
      this.placeData.workId = this.$mp.page.options.id ? this.$mp.page.options.id : ''
      this.getPlaceById()
    }
  }
</script>

<style lang="stylus" scoped>
  @import '../../styles/common.styl';

  .bottom-btn {
    position absolute bottom 0 width 100%
  }
</style>