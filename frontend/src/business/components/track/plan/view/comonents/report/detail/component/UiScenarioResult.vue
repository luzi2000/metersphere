<template>
  <el-container class="scenario-info">
    <ms-aside-container width="500px" :default-hidden-bottom-top="200" :enable-auto-height="true">
      <el-card>
        <el-scrollbar>
          <ms-table v-loading="result.loading"
                    :show-select-all="false"
                    :screen-height="null"
                    :enable-selection="false"
                    :highlight-current-row="true"
                    @refresh="getScenarioApiCase"
                    @handleRowClick="rowClick"
                    :data="scenarioCases">

            <ms-table-column
                :width="80"
                :label="$t('commons.id')"
                prop="customNum">
            </ms-table-column>
            <ms-table-column
                :label="$t('commons.name')"
                prop="name">
            </ms-table-column>
            <ms-table-column
                :label="$t('test_track.report.list.creator')"
                prop="creatorName"/>
            <ms-table-column
                :label="$t('test_track.case.priority')"
                :width="80">
              <template v-slot:default="scope">
                <priority-table-item :value="scope.row.level" ref="priority"/>
              </template>
            </ms-table-column>
            <ms-table-column
                :width="70"
                :label="$t('api_test.automation.step')"
                prop="stepTotal">
            </ms-table-column>
            <ms-table-column
                :width="80"
                :label="$t('test_track.plan_view.execute_result')"
                prop="lastResult">
              <template v-slot:default="{row}">
                <status-table-item v-if="row.lastResult === 'Success'" :value="'Pass'"/>
                <status-table-item v-else-if="row.lastResult === 'Error'" :value="'Failure'"/>
                <status-table-item v-else-if="row.lastResult === 'STOP'" :value="'STOP'"/>
                <status-table-item v-else-if="row.lastResult === 'Running'" :value="'Underway'"/>
                <status-table-item v-else-if="row.lastResult === 'Waiting'" :value="'Waiting'"/>
                <status-table-item v-else-if="row.lastResult === 'Timeout'" :value="'Timeout'"/>
                <el-tag size="mini" v-else>{{
                    $t('api_test.home_page.detail_card.unexecute')
                  }}
                </el-tag>
              </template>
            </ms-table-column>
          </ms-table>
        </el-scrollbar>
      </el-card>
    </ms-aside-container>
    <el-main>
      <ms-api-report
          v-if="showResponse"
          :is-plan="true"
          :share-id="shareId"
          :is-share="isShare"
          :template-report="response"
          :is-template="isTemplate"
          :infoDb="true"
          :report-id="reportId"
          @reportNotExist="showResponse = false"/>
      <div class="empty" v-else>{{ $t('test_track.plan.load_case.content_empty') }}</div>
    </el-main>
  </el-container>
</template>

<script>
import PriorityTableItem from "../../../../../../common/tableItems/planview/PriorityTableItem";
import TypeTableItem from "../../../../../../common/tableItems/planview/TypeTableItem";
import MethodTableItem from "../../../../../../common/tableItems/planview/MethodTableItem";
import StatusTableItem from "../../../../../../common/tableItems/planview/StatusTableItem";
import MsTable from "@/business/components/common/components/table/MsTable";
import MsTableColumn from "@/business/components/common/components/table/MsTableColumn";
import MsApiReport from "@/business/components/api/automation/report/ApiReportDetail";
import MsAsideContainer from "@/business/components/common/components/MsAsideContainer";
import MsMainContainer from "@/business/components/common/components/MsMainContainer";

export default {
  name: "UiScenarioResult",
  components: {
    MsMainContainer,
    MsAsideContainer,
    MsApiReport,
    MsTableColumn, MsTable, StatusTableItem, MethodTableItem, TypeTableItem, PriorityTableItem
  },
  props: {
    planId: String,
    isTemplate: Boolean,
    report: Object,
    isShare: Boolean,
    shareId: String,
    isAll: Boolean,
    isErrorReport: Boolean,
    isUnExecute: Boolean,
    isDb: Boolean,
    uiAllCases: Array,
    filterStatus: Array,
  },
  data() {
    return {
      scenarioCases: [],
      result: {},
      reportId: null,
      response: {},
      showResponse: false,
      resultMap: {
        'Success' : 'Pass',
        'Error' : 'Failure',
        'STOP' : 'STOP',
        'Running' : 'Running',
        'UnExecute' : 'Prepare',
      }
    }
  },
  mounted() {
    this.getScenarioApiCase();
  },
  watch: {
    scenarioCases() {
      if (this.scenarioCases) {
        this.$emit('setSize', this.scenarioCases.length);
      }
    },
    uiAllCases() {
      this.getScenarioApiCase();
    }
  },
  methods: {
    getScenarioApiCase() {
      this.scenarioCases = [];
      if (this.filterStatus && this.filterStatus.length > 0) {
        this.uiAllCases.forEach(item => {
          if (this.filterStatus.indexOf(item.lastResult) > -1) {
            this.scenarioCases.push(item);
          }
        });
      } else {
        this.scenarioCases = this.uiAllCases;
      }
    },
    rowClick(row) {
      this.showResponse = false;
      if (this.isTemplate) {
        if (row.response) {
          this.showResponse = true;
          this.response = row.response;
        }
      } else {
        if (row.reportId && row.lastResult !== "Running" && row.lastResult !== "Waiting") {
          this.showResponse = true;
          this.reportId = row.reportId;
        }
      }
    }
  }
}
</script>

<style scoped>

.padding-col {
  padding-right: 0px;
}

.el-card >>> .el-card__body {
  height: 588px;
}

/deep/ .api-report-content {
  height: auto;
}

.ms-aside-container {
  border: 0px;
  padding: 10px 0px 0px 10px;
}
</style>
