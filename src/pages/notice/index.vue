<template>
  <div class='container'>
     <login :visible='visible' v-on:modalClose='closeModalEvent'></login>
     <div class='body'>
       <div class='tabs'>
         <div :class='{selected:tab==="unread"}' @click.stop='changeTab($event)' data-tab='unread'>未读消息:{{unread.length}}</div>
         <div :class='{selected:tab==="read"}' @click.stop='changeTab($event)' data-tab='read'>已读消息</div>
       </div>
       <div>
         <button  v-show="tab==='unread'" class='read-all' @click.stop='readAll'>一键已读</button>
        <div class='notice' v-for='item in currentData' :key='item.id' :data-id='item.id'>
          <div>
            {{item.author.loginname}}在<span style='color:$color;' @click.stop='goDetail($event)' :data-topicid='item.topic.id'>{{item.topic.title}}</span>回复了你
          </div>
          <div class='content'>
            {{item.reply.content}}
          </div>
          <div>
            {{item.reply.create_at}}
          </div>
        </div>
       </div>
     </div>
  </div>
</template>
<script>
import { api } from "../../const";
import login from "../../components/login";
import { passTime } from "../../utils/index";
export default {
  components: {
    login
  },
  data() {
    return {
      read: [],
      unread: [],
      visible: false,
      tab: "read" // read
    };
  },
  computed: {
    currentData() {
      return this.tab === "unread"
        ? this.formatTimeUnread
        : this.formatTimeRead;
    },
    formatTimeRead() {
      return this.read.map(_ => {
        return {
          ..._,
          reply: {
            ..._.reply,
            create_at: passTime(_.reply.create_at)
          }
        };
      });
    },
    formatTimeUnread() {
      return this.unread.map(_ => {
        return {
          ..._,
          reply: {
            ..._.reply,
            create_at: passTime(_.reply.create_at)
          }
        };
      });
    }
  },
  onShow() {
    const accesstoken = wx.getStorageSync("accesstoken");
    if (accesstoken) {
      this.getData();
    } else {
      this.visible = true;
    }
  },
  methods: {
    async readAll() {
      const accesstoken = wx.getStorageSync("accesstoken");
      const res = await this.$http.post(`${api}/message/mark_all`, {
        accesstoken
      });
      if (res.data.success) {
        wx.showToast({
          title: "全部已读成功",
          icon: "none",
          duration: 2000
        });
      }
    },
    closeModalEvent() {
      this.visible = false;
      this.getData();
    },
    goDetail(e) {
      // wx.setStorageSync("topicid",e.target.dataset.topicid);
      wx.navigateTo({
        url: `../detail/main?topicid=${e.target.dataset.topicid}`
      });
    },
    changeTab(e) {
      this.tab = e.target.dataset.tab;
    },
    async getData() {
      const accesstoken = wx.getStorageSync("accesstoken");
      if (accesstoken) {
        const res = await this.$http.get(`${api}/messages`, {
          accesstoken,
          mdrender: false
        });
        if (res.data.success) {
          this.read = res.data.data.has_read_messages;
          this.unread = res.data.data.hasnot_read_messages;
        }
      }
    }
  }
};
</script>

<style lang='scss' scoped>
.container {
  background-color: rgb(245, 245, 249);
  width: 100vw;
  .body {
    display: flex;
    flex-direction: column;
    .tabs {
      display: flex;
      background-color: white;
      & > div {
        width: 50%;
        text-align: center;
      }
    }
    .notice {
      background-color: white;
      margin-bottom: 20rpx;
      padding: 30rpx;
      .content {
        border-left: 4rpx solid $color;
        padding-left: 10rpx;
        margin: 10rpx 0;
        overflow: hidden;
        text-overflow: ellipsis;
      }
    }
    .read-all {
      color: $color;
      border: 2rpx solid $color;
      font-size: 20rpx;
      width: 99rpx;
      border-radius: 20rpx;
      height: 46rpx;
      line-height: 46rpx;
      text-align: center;
      padding: 0;
      margin: 10rpx;
    }
  }
}
.selected {
  color: $color;
  border-bottom: 2rpx solid $color;
}
</style>

