<template>
  <el-row :gutter="20">
    <el-col :md="14" :sm="24">
      <ListTable ref="listTable" :table-config="tableConfig" :header-actions="headerActions" />
    </el-col>
    <el-col :md="10" :sm="24">
      <AssetRelationCard type="primary" v-bind="assetRelationConfig" />
      <RelationCard type="info" style="margin-top: 15px" v-bind="nodeRelationConfig" />
    </el-col>
  </el-row>
</template>

<script>
import ListTable from '@/components/ListTable/index'
import RelationCard from '@/components/RelationCard/index'
import AssetRelationCard from '@/components/AssetRelationCard'
import { DeleteActionFormatter } from '@/components/ListTable/formatters'

export default {
  name: 'ChangeAuthPlanAsset',
  components: {
    ListTable, RelationCard, AssetRelationCard
  },
  props: {
    object: {
      type: Object,
      required: true,
      default: () => ({})
    }
  },
  data() {
    return {
      tableConfig: {
        url: `/api/v1/xpack/change-auth-plan/plan/${this.object.id}/assets/`,
        columns: [
          'hostname', 'ip', 'delete_action'
        ],
        columnsMeta: {
          delete_action: {
            prop: 'id',
            label: this.$t('common.Actions'),
            align: 'center',
            width: 150,
            objects: this.object.assets,
            formatter: DeleteActionFormatter,
            onDelete: function(col, row, cellValue, reload) {
              this.$axios.patch(
                `/api/v1/xpack/change-auth-plan/plan/${this.object.id}/asset/remove/`,
                { assets: [row.id] }
              ).then(res => {
                this.$message.success(this.$t('common.deleteSuccessMsg'))
                reload()
              }).catch(error => {
                this.$message.error(this.$t('common.deleteErrorMsg' + ' ' + error))
              })
            }.bind(this)
          }
        },
        tableAttrs: {
          border: false
        }
      },
      headerActions: {
        hasSearch: true,
        hasRefresh: true,
        hasLeftActions: true,
        hasRightActions: true,
        hasExport: false,
        hasImport: false,
        hasCreate: false,
        hasBulkDelete: false,
        hasBulkUpdate: false
      },
      assetRelationConfig: {
        icon: 'fa-edit',
        title: this.$t('xpack.ChangeAuthPlan.AddAsset'),
        canSelect: (row, index) => {
          return this.object.assets.indexOf(row.id) === -1
        },
        performAdd: (items, that) => {
          const relationUrl = `/api/v1/xpack/change-auth-plan/plan/${this.object.id}/asset/add/`
          const data = {
            assets: items
          }
          return this.$axios.patch(relationUrl, data)
        },
        onAddSuccess: (items, that) => {
          this.$log.debug('AssetSelect value', that.assets)
          this.$message.success(this.$t('common.updateSuccessMsg'))
          this.$refs.listTable.reloadTable()
          that.$refs.assetSelect.$refs.select2.clearSelected()
        }
      },
      nodeRelationConfig: {
        icon: 'fa-edit',
        title: this.$t('xpack.ChangeAuthPlan.AddNode'),
        objectsAjax: {
          url: `/api/v1/assets/nodes/`,
          transformOption: (item) => {
            return { label: item.full_value, value: item.id }
          }
        },
        hasObjectsId: this.object.nodes,
        performAdd: (items, that) => {
          const relationUrl = `/api/v1/xpack/change-auth-plan/plan/${this.object.id}/`
          const nodes = items.map(v => v.value)
          const iHasObjects = that.iHasObjects.map(v => v.value)
          const data = {
            nodes: Array.from(new Set([...iHasObjects, ...nodes]))
          }
          return this.$axios.patch(relationUrl, data)
        },
        onAddSuccess: (objects, that) => {
          that.iHasObjects = [...that.iHasObjects, ...objects]
          that.$refs.select2.clearSelected()
          this.$message.success(this.$t('common.updateSuccessMsg'))
          this.$refs.listTable.reloadTable()
        },
        performDelete: (item) => {
          const nodes = this.object.nodes
          const deleteNode = item.value
          nodes.splice(nodes.indexOf(deleteNode), 1)
          const data = {
            nodes: nodes
          }
          const relationUrl = `/api/v1/xpack/change-auth-plan/plan/${this.object.id}/`
          return this.$axios.patch(relationUrl, data)
        },
        onDeleteSuccess: (obj, that) => {
          const theRemoveIndex = that.iHasObjects.findIndex((v) => v.value === obj.value)
          that.iHasObjects.splice(theRemoveIndex, 1)
          while (that.select2.disabledValues.indexOf(obj.value) !== -1) {
            const i = that.select2.disabledValues.indexOf(obj.value)
            that.select2.disabledValues.splice(i, 1)
          }
          this.$message.success(this.$t('common.deleteSuccessMsg'))
          this.$refs.listTable.reloadTable()
        }
      }
    }
  }
}
</script>

<style scoped>

</style>
