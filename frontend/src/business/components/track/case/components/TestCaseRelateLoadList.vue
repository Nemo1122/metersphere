<template>
  <div>

      <el-input :placeholder="$t('commons.search_by_name_or_id')" @blur="initTable"
                @keyup.enter.native="initTable" class="search-input" size="small" v-model="condition.name"/>
      <ms-table-adv-search-bar :condition.sync="condition" class="adv-search-bar"
                               v-if="condition.components !== undefined && condition.components.length > 0"
                               @search="initTable"/>
      <ms-table v-loading="result.loading" :data="tableData" :condition="condition" :page-size="pageSize"
                :total="total"
                :showSelectAll="false"
                :screenHeight="screenHeight"
                @refresh="initTable"
                ref="table">

        <ms-table-column
          prop="num"
          label="ID"
          width="100px"
          sortable=true>
        </ms-table-column>

        <ms-table-column
          prop="name"
          :label="$t('commons.name')"/>

        <ms-table-column
          prop="status"
          :label="$t('commons.status')"
          min-width="80">
          <template v-slot:default="{row}">
            <ms-performance-test-status :row="row"/>
          </template>
        </ms-table-column>
        <ms-table-column
          prop="updateTime"
          :label="$t('commons.update_time')"
          min-width="150px">
          <template v-slot:default="scope">
            <span>{{ scope.row.updateTime | timestampFormatDate }}</span>
          </template>
        </ms-table-column>
        <ms-table-column prop="createTime"
                         :label="$t('commons.create_time')"
                         min-width="150px">
          <template v-slot:default="scope">
            <span>{{ scope.row.createTime | timestampFormatDate }}</span>
          </template>
        </ms-table-column >

      </ms-table>
      <ms-table-pagination :change="initTable" :current-page.sync="currentPage" :page-size.sync="pageSize"
                           :total="total"/>

    <table-select-count-bar :count="selectRows.size"/>

  </div>

</template>

<script>

import MsTable from "@/business/components/common/components/table/MsTable";
import MsTableColumn from "@/business/components/common/components/table/MsTableColumn";
import MsTablePagination from "@/business/components/common/pagination/TablePagination";
import TableSelectCountBar from "@/business/components/api/automation/scenario/api/TableSelectCountBar";
import MsPerformanceTestStatus from "@/business/components/performance/test/PerformanceTestStatus";
import MsTableAdvSearchBar from "@/business/components/common/components/search/MsTableAdvSearchBar";
import {TEST_CASE_RELEVANCE_LOAD_CASE} from "@/business/components/common/components/search/search-components";

export default {
  name: "TestCaseRelateLoadList",
  components: {
    MsPerformanceTestStatus,
    TableSelectCountBar,
    MsTablePagination,
    MsTable,
    MsTableColumn,
    MsTableAdvSearchBar,
  },
  data() {
    return {
      condition: {
        components: TEST_CASE_RELEVANCE_LOAD_CASE
      },
      result: {},
      screenHeight: '600px',//屏幕高度
      tableData: [],
      currentPage: 1,
      pageSize: 10,
      total: 0,
    }
  },
  props: {
    projectId: String,
  },
  created: function () {
    this.initTable();
  },
  watch: {
    projectId() {
      this.initTable();
    }
  },
  computed: {
    selectRows() {
      if (this.$refs.table) {
        return this.$refs.table.getSelectRows();
      } else {
        return new Set();
      }
    }
  },
  methods: {
    initTable(projectId) {
      this.condition.status = "";
      if (projectId != null && typeof projectId === 'string') {
        this.condition.projectId = projectId;
      } else if (this.projectId != null) {
        this.condition.projectId = this.projectId;
      }
      let url = '/test/case/relevance/load/list/';
      this.result = this.$post(this.buildPagePath(url), this.condition, response => {
        this.total = response.data.itemCount;
        this.tableData = response.data.listObject;
      });
    },
    clear() {
      if (this.$refs.table) {
        this.$refs.table.clear();
      }
    },
    buildPagePath(path) {
      return path + "/" + this.currentPage + "/" + this.pageSize;
    },
    getSelectIds() {
      return this.$refs.table.selectIds;
    },
    clearSelection() {
      if (this.$refs.table) {
        this.$refs.table.clearSelectRows();
      }
    },
  },
}
</script>

<style scoped>
.search-input {
  float: right;
  width: 300px;
  margin-right: 20px;
}
.adv-search-bar {
  float: right;
  margin-top: 5px;
  margin-right: 10px;
}
</style>
