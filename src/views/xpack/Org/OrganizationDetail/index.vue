<template>
  <GenericDetailPage :object.sync="Organization" :active-menu.sync="config.activeMenu" v-bind="config" v-on="$listeners">
    <keep-alive>
      <component :is="config.activeMenu" :object="Organization" />
    </keep-alive>
  </GenericDetailPage>
</template>

<script>
import { GenericDetailPage, TabPage } from '@/layout/components'
import OrganizationDetail from './OrganizationDetail'
import OrganizationMembershipList from './OrganizationMembershipList'
import { getApiPath } from '@/utils/common'
const performDelete = function() {
  const url = getApiPath(this)
  return this.$axios.delete(url)
}
export default {
  components: {
    GenericDetailPage,
    OrganizationDetail,
    OrganizationMembershipList,
    TabPage
  },
  data() {
    return {
      Organization: {},
      config: {
        activeMenu: 'OrganizationDetail',
        submenu: [
          {
            title: this.$t('xpack.Organization.OrganizationDetail'),
            name: 'OrganizationDetail'
          },
          {
            title: this.$t('xpack.Organization.OrganizationMembership'),
            name: 'OrganizationMembershipList'
          }
        ],
        actions: {
          deleteCallback: function() {
            const msg = this.$t('xpack.Organization.DeleteOrgMsg')
            const title = this.$t('xpack.Organization.DeleteOrgTitle')
            this.$alert(msg, title, {
              type: 'warning',
              confirmButtonClass: 'el-button--danger',
              showCancelButton: true,
              beforeClose: async(action, instance, done) => {
                if (action !== 'confirm') return done()
                instance.confirmButtonLoading = true
                try {
                  await performDelete.bind(this)()
                  this.$message.success(this.$t('common.deleteSuccessMsg'))
                  return done()
                } finally {
                  instance.confirmButtonLoading = false
                  this.$router.push('/xpack/orgs/') // 返回列表
                }
              }
            }).catch(() => {
              /* 取消*/
            })
          }
        }
      }
    }
  }
}
</script>

<style scoped>

</style>
