<template>
  <el-dialog title="快递范围" :visible.sync="visible" width="600px" :before-close="dialogCloseConfirm">
    <el-card>
      <div class="tree" style="height:500px">
        <el-tree ref="regionTree" :data="data" show-checkbox node-key="regionId"
                 @node-contextmenu="checkedChildren" check-strictly :default-expanded-keys="['0']"
                 :props="defaultProps">
        </el-tree>
      </div>
    </el-card>
    <div slot="footer" class="dialog-footer">
      <el-button type="primary" @click.native="submit">提交</el-button>
    </div>
  </el-dialog>
</template>
<script>
  import {RegionApi} from '@/modules/admin';
  import {ExpressRegionApi} from './api';

  export default {
    name: 'Region',
    data() {
      return {
        select: [],
        data: [],
        visible: false,
        current: {},
        defaultProps: {
          children: 'children',
          label: 'regionName'
        },
      };
    },
    methods: {
      checkedChildren(e, data, node) {
        this.recursionChildren(data, !node.checked);
      },
      recursionChildren(data, checked) {
        this.$refs.regionTree.setChecked(data, checked);
        if (data.children) {
          data.children.forEach(child => this.recursionChildren(child, checked));
        }
      },
      show(warehouse) {
        this.select = [];
        this.current = warehouse;
        if (this.data.length === 0) {
          RegionApi.listSimple().then(data => {
            this.data = data;
            this.visible = true;
            this.loadExpressRegion();
          });
        } else {
          this.visible = true;
          this.loadExpressRegion();
        }
      },
      loadExpressRegion() {
        ExpressRegionApi.listSimple({expressId: this.current.expressId}).then(response => {
          response.forEach(item => {
            this.select.push(item.regionId);
          });
          this.$refs.regionTree.setCheckedKeys(this.select);
        });
      },
      submit() {
        let selects = this.$refs.regionTree.getCheckedNodes();
        if (selects == null || selects.length === 0) {
          this.$message.warning('请选择仓库范围');
          return;
        }
        let expressRegions = [];
        selects.forEach(item => {
          let region = {
            expressId: this.current.expressId,
            expressName: this.current.expressName,
            regionId: item.regionId,
            regionName: item.regionName,
            level: item.level
          };
          expressRegions.push(region);
        });
        ExpressRegionApi.saveExpressRegion(this.current.expressId, expressRegions).then(() => {
          this.$message.success('配置仓库范围成功！');
          this.visible = false;
        });
      }
    }
  };
</script>