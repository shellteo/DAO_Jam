<template>
  <div class="user-page">
    <g-header />
    <div v-if="userInfo.banner" class="banner user-page-banner">
      <img :src="userInfo.banner" alt="banner">
    </div>
    <div v-else class="default-banner user-page-banner">
      <img src="@/assets/img/user_banner.png" alt="banner">
    </div>
    <div class="user-info user-page-info">
      <div class="user-info-center">
        <div class="fl ac jc token-avatar">
          <avatar :src="userInfo.avatar" size="120px" class="avatar" />
          <img
            v-if="tokenUser"
            class="token-link"
            src="@/assets/img/token_link.png"
            alt="token-link"
          >
          <tokenAvatar v-if="tokenUser" :token="tokenData" />
        </div>
        <div>
          <h1 class="username">
            {{ userInfo.name }}
            <!-- {{ userInfo }} -->
            <!-- <el-tooltip
            v-if="seedUser"
            class="item"
            effect="dark"
            content="种子用户"
            placement="top"
          >
            <svg-icon
              class="seeduser"
              icon-class="seeduser"
            />
            </el-tooltip>-->

            <el-tooltip
              v-if="tokenUser"
              class="tooltip"
              effect="dark"
              content="发行了Fan票的用户"
              placement="top"
            >
              <svg-icon class="tokens" icon-class="token" />
            </el-tooltip>
          </h1>
        </div>

        <p class="profile">
          {{ userInfo.introduction || $t('not') }}
        </p>
        <div class="fl jc">
          <router-link
            :to="{name: 'user-id-follow', params: {id: $route.params.id}}"
            class="fl fdc ac follow-block"
          >
            <span class="follow-number">{{ userInfo.follows || 0 }}</span>
            <span class="follow-title">{{ $t('follow') }}</span>
          </router-link>

          <router-link
            :to="{name: 'user-id-fan', params: {id: $route.params.id}}"
            class="fl fdc ac follow-block"
          >
            <span class="follow-number">{{ userInfo.fans || 0 }}</span>
            <span class="follow-title">{{ $t('fans') }}</span>
          </router-link>
        </div>
        <el-tooltip v-if="isMe(Number($route.params.id))" class="item" effect="dark" content="上传背景图" placement="top">
          <bannerUpload
            :img-upload-done="imgUploadDone"
            :update-type="'banner'"
            @doneImageUpload="doneImageUpload"
            class="follow3"
          >
            <el-button slot="uploadButton" size="small" class="ibutton" type="info" icon="el-icon-s-open" />
          </bannerUpload>
        </el-tooltip>
        <followBtn
          v-if="!isMe(Number($route.params.id))"
          :id="Number($route.params.id)"
          class="follow"
        />
        <router-link v-else :to="{name: 'setting'}">
          <el-button size="small" class="follow edit">
            {{ $t('user.editProfile') }}
          </el-button>
        </router-link>
        <el-button @click="shareModalShow = true" size="small" class="follow2">
          <svg-icon icon-class="share_new" />
          {{ $t('share') }}
        </el-button>
      </div>
    </div>
    <div class="user-page-content">
      <userPageNav />
      <div class="article">
        <slot name="list" />
      </div>
    </div>
    <Share
      :share-modal-show="shareModalShow"
      :minetoken-user="{nickname: userInfo.name}"
      :page-type="1"
      :img="userInfo.avatar"
      @input="val => shareModalShow = val"
    />
  </div>
</template>

<script>
import { mapState, mapActions, mapGetters } from 'vuex'
import userPageNav from './user_page_nav'
import tokenAvatar from './token_avatar.vue'
import avatar from '@/components/avatar/index'
import followBtn from '@/components/follow_btn'
import Share from '@/components/token/token_share.vue'

import bannerUpload from '@/components/bannerUpload/index.vue'

export default {
  components: {
    avatar,
    followBtn,
    userPageNav,
    tokenAvatar,
    Share,
    bannerUpload
  },
  data() {
    return {
      token: false,
      seedUser: false,
      tokenUser: false,
      tokenData: Object.create(null),
      shareModalShow: false,
      imgUploadDone: 0 // 图片是否上传完成
    }
  },
  computed: {
    ...mapState({
      userInfo: state => state.user.userInfo
    }),
    ...mapGetters(['isMe'])
  },
  created() {
    if (!this.$route.params.id) this.$router.go(-1)
    // this.getMyUserData()
  },
  mounted() {
    this.refreshUser({ id: this.$route.params.id })
    this.tokenUserId(this.$route.params.id)
  },
  methods: {
    ...mapActions('user', ['refreshUser', 'followOrUnfollowUser']),
    async getMyUserData() {
      const res = await this.$API
        .getMyUserData()
        .then(res => {
          const statusToken =
            res.data.status & this.$userStatus.hasMineTokenPermission
          const statusSeedUser = res.data.status & this.$userStatus.isSeedUser
          if (res.code === 0 && statusToken) this.token = true
          if (res.code === 0 && statusSeedUser) this.seedUser = true
        })
        .catch(err => {
          console.log(err)
        })
    },
    async tokenUserId(id) {
      await this.$API
        .tokenUserId(id)
        .then(res => {
          if (res.code === 0 && res.data.id > 0) {
            this.tokenUser = true
            this.tokenData = res.data
          }
        })
        .catch(err => console.log('get token user error', err))
    },
    doneImageUpload(res) {
      this.imgUploadDone += Date.now()
      this.refreshUser({ id: this.$route.params.id })
    }
  }
}
</script>

<style lang="less" scoped>
.user-page {
  .minHeight();
}
.banner {
  height: 290px;
  background-color: #ffffff;
  box-sizing: border-box;
  text-align: center;
  overflow: hidden;
  padding: 0px;
  img {
    max-width: 3840px;
    height: 100%;
    display: block;
    object-fit: cover;
    margin: 0 auto;
    padding: 0;
    width: 100%;
    height: 100%;
  }
}
.default-banner {
  height: 290px;
  background-color: #C8D7FF;
  box-sizing: border-box;
  img {
    height: 100%;
    display: block;
    margin: 0 auto;
  }
}

.user-info {
  width: 100%;
  height: 240px;
  background-color: #fff;
  box-sizing: border-box;
  .token-avatar {
    margin-top: -62px;
    margin-bottom: 20px;
  }
  &-center {
    width: 766px;
    height: 100%;
    margin: 0 auto;
    padding: 1px 10px 0;
    text-align: center;
    position: relative;
    box-sizing: border-box;
  }
  .avatar {
    border: 6px solid #fff;
    display: inline-block;
    background: #fff;
    box-sizing: border-box;
  }
  .username {
    font-size: 24px;
    font-weight: bold;
    color: #000;
    line-height: 1;
    padding: 0;
    margin: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    display: inline-block;
    position: relative;
    .tooltip {
      position: absolute;
      right: -28px;
    }
  }
  .token-link {
    width: 50px;
    height: 20px;
    margin: 0 20px;
  }
  .seeduser {
    font-size: 36px;
    background: #fff;
    margin-left: 4px;
  }
  .tokens {
    cursor: pointer;
    width: 26px;
    margin-left: 4px;
  }
  .profile {
    font-size: 16px;
    font-weight: 400;
    color: #000;
    line-height: 22px;
    padding: 0;
    margin: 10px 0 20px;
  }
  .follow-block {
    margin: 0 34px;
  }
  .follow-number {
    font-size: 24px;
    font-weight: bold;
    color: rgba(84, 45, 224, 1);
    line-height: 28px;
    padding: 0;
    margin: 0 0 8px;
  }
  .follow-title {
    font-size: 14px;
    font-weight: 400;
    color: rgba(0, 0, 0, 1);
    line-height: 20px;
    padding: 0;
    margin: 0;
  }
  .follow2 {
    position: absolute;
    right: 10px;
    bottom: 30px;
  }
  .follow3 {
    position: absolute;
    right: 10px;
    bottom: 250px;
    .ibutton {
      font-size: 18px;
      padding: 5px;
      background: #9093998c;
      border-color: #00000000;
      &:hover {
        background: #909399;
      }
    }
  }
  .follow {
    position: absolute;
    right: 10px;
    bottom: 70px;
    &.edit {
      padding-left: 11px;
      padding-right: 11px;
    }
  }
}

.article {
  width: 766px;
  height: 100%;
  margin: 0 auto;
  padding: 0 10px;
  box-sizing: border-box;
}
</style>
