<template>
  <userLayout>
    <template slot="main">
      <user-nav nav-list-url="account" />
      <userNavType nav-list-url="setting" />
      <template v-if="viewStatus === 0">
        <assets :assets="assets" @toggleWithdraw="status => viewStatus = status" type="ONT" class="assets-margin" />
        <div v-loading="loading" class="card-container">
          <no-content-prompt :list="articleCardData.articles">
            <assetCard v-for="(item, index) in articleCardData.articles" :key="index" :asset="item" />
          </no-content-prompt>
        </div>
        <user-pagination
          v-show="!loading"
          :current-page="currentPage"
          :params="articleCardData.params"
          :api-url="articleCardData.apiUrl"
          :page-size="9"
          :total="total"
          :need-access-token="true"
          @paginationData="paginationData"
          @togglePage="togglePage"
          class="pagination"
        />
      </template>
      <template v-else>
        <withdraw @toggleWithdraw="status => viewStatus = status" @withdrawDone="viewStatus = 0" type="ONT" class="withdraw" />
      </template>
    </template>
    <template slot="nav">
      <myAccountNav />
    </template>
  </userLayout>
</template>

<script>
import userLayout from '@/components/user/user_layout.vue'
import myAccountNav from '@/components/my_account/my_account_nav.vue'
import userNav from '@/components/user/user_nav.vue'
import userNavType from '@/components/user/user_nav_type.vue'
import userPagination from '@/components/user/user_pagination.vue'
import assetCard from '@/components/asset_card/index.vue'
import assets from '@/components/user/assets.vue'
import withdraw from '@/components/user/withdraw.vue'
export default {
  components: {
    userLayout,
    myAccountNav,
    userNav,
    userNavType,
    userPagination,
    assetCard,
    assets,
    withdraw
  },
  data() {
    return {
      articleCardData: {
        params: {
          symbol: (this.$route.query.type || 'ONT').toUpperCase(),
          pagesize: 9
        },
        apiUrl: 'assetList',
        articles: []
      },
      currentPage: Number(this.$route.query.page) || 1,
      loading: false, // 加载数据
      total: 0,
      assets: {},
      viewStatus: 0 // 0 1
    }
  },
  methods: {
    paginationData(res) {
      // console.log(res)
      this.articleCardData.articles = res.data.logs
      this.assets = res.data
      this.total = res.data.count || 0
      this.loading = false
    },
    togglePage(i) {
      this.loading = true
      this.articleCardData.articles = []
      this.currentPage = i
      this.$router.push({
        query: {
          page: i
        }
      })
    }
  }
}
</script>
