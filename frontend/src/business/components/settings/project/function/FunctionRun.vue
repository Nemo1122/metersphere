<template>
  <span></span>
</template>
<script>
import {getCurrentProjectID, strMapToObj} from "@/common/js/utils";
import {TYPE_TO_C} from "@/business/components/api/automation/scenario/Setting";
import TestPlan from "@/business/components/api/definition/components/jmeter/components/test-plan";
import ThreadGroup from "@/business/components/api/definition/components/jmeter/components/thread-group";

export default {
  name: 'FunctionRun',
  components: {},
  props: {
    environment: Object,
    debug: Boolean,
    reportId: String,
    runData: Array,
    type: String,
    envMap: Map,
    isStop: Boolean,
  },
  data() {
    return {
      result: {},
      loading: false,
      runId: "",
      reqNumber: 0,
      websocket: {}
    }
  },

  watch: {
    // 初始化
    reportId() {
      this.run()
    },
    isStop() {
      if (!this.isStop && this.websocket && this.websocket.close instanceof Function) {
        this.websocket.close();
      }
    }
  },
  methods: {
    initWebSocket() {
      let protocol = "ws://";
      if (window.location.protocol === 'https:') {
        protocol = "wss://";
      }
      const uri = protocol + window.location.host + "/api/definition/run/report/" + this.runId + "/debug";
      this.websocket = new WebSocket(uri);
      this.websocket.onmessage = this.onMessage;
    },
    onMessage(e) {
      if (e.data) {
        let data = JSON.parse(e.data);
        this.websocket.close();
        this.$emit('runRefresh', data);
      }
    },
    sort(stepArray) {
      if (stepArray) {
        for (let i in stepArray) {
          if (!stepArray[i].clazzName) {
            stepArray[i].clazzName = TYPE_TO_C.get(stepArray[i].type);
          }
          if (stepArray[i].hashTree && stepArray[i].hashTree.length > 0) {
            this.sort(stepArray[i].hashTree);
          }
        }
      }
    },
    run() {
      let projectId = getCurrentProjectID();

      let testPlan = new TestPlan();
      testPlan.clazzName = TYPE_TO_C.get(testPlan.type);
      let threadGroup = new ThreadGroup();
      threadGroup.clazzName = TYPE_TO_C.get(threadGroup.type);
      threadGroup.hashTree = [];
      testPlan.hashTree = [threadGroup];
      this.runData.forEach(item => {
        item.projectId = projectId;
        if (!item.clazzName) {
          item.clazzName = TYPE_TO_C.get(item.type);
        }
        threadGroup.hashTree.push(item);
      })
      this.sort(testPlan.hashTree);
      let reqObj = {id: this.reportId, testElement: testPlan, type: this.type, clazzName: this.clazzName ? this.clazzName : TYPE_TO_C.get(this.type), projectId: projectId, environmentMap: strMapToObj(this.envMap)};
      let url = "/custom/func/run";
      reqObj.reportId = this.reportId;
      this.$post(url, reqObj, res => {
        this.runId = res.data;
        this.initWebSocket();
      }, () => {
        this.$emit('errorRefresh', {});
      })
    }
  }
}
</script>
