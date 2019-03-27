<template>
  <div id="app">
    <!-- <h1> simple flowchart</h1>
    <div class="tool-wrapper">
      <select v-model="newNodeType">
        <option v-for="(item, index) in nodeCategory" :key="index" :value="index">{{item}}</option>
      </select>
      <input type="text" v-model="newNodeLabel" placeholder="Input node label">
      <button @click="addNode">ADD</button>
    </div>-->
    <!-- <div class="node-add-btn" @click="paneControl = !paneControl">{{paneControl ? 'x' : '+'}}</div>
    <div class="node-add-pnl" v-if="paneControl">
      <div
        class="node-category-box"
        v-for="(item, index) in nodeCategory"
        :key="index"
        @dblclick="addNode(item)"
      >{{item.text}}</div>
    </div> -->

    <simple-flowchart
      ref="flowchart"
      @nodeClick="nodeClick"
      @nodeDelete="nodeDelete"
      @linkBreak="linkBreak"
      @linkAdded="linkAdded"
      @canvasClick="canvasClick"
      :height="800"
    />
  </div>
</template>

<script>
import SimpleFlowchart from "./components/SimpleFlowchart.vue";

export default {
  name: "app",
  components: {
    SimpleFlowchart
  },
  data() {
    return {
      scene: {
        centerX: 1024,
        centerY: 140,
        scale: 1,
        nodes: [
          {
            id: 2,
            x: -700,
            y: -69,
            type: "Action",
            label: "test1",
            color: "red"
          },
          {
            id: 4,
            x: -357,
            y: 80,
            type: "Script",
            label: "test2",
            color: "olive"
          },
          {
            id: 6,
            x: -557,
            y: 80,
            type: "Rule",
            label: "test3",
            color: "green"
          }
        ],
        links: [
          {
            id: 3,
            from: 2, // node id the link start
            to: 4 // node id the link end
          }
        ]
      },
      newNodeType: 0,
      newNodeLabel: "",
      nodeCategory: [
        { text: "type1", color: "orange" },
        { text: "type2", color: "red" },
        { text: "type3", color: "olive" },
        { text: "type4", color: "green" },
        { text: "type5", color: "blue" }
      ],
      paneControl: false
    };
  },
  methods: {
    exportToJson() {
      // console.log(this.$refs.flowchart.exportToJson());
    },
    canvasClick(e) {
      // console.log("canvas Click, event:", e);
    },
    importAsJson(val) {
      console.log(val);
    },
    addNode(category) {
      // let maxID = Math.max(0, ...this.scene.nodes.map((link) => {
      //   return link.id
      // }))
      const tmp = Date.now();
      this.scene.nodes.push({
        id: tmp,
        x: -400,
        y: 50,
        type: category.text,
        label: "##value##",
        color: category.color
      });
    },
    nodeClick(id) {
      this.exportToJson();
      // console.log("node click", id);
    },
    nodeDelete(id) {
      // console.log("node delete", id);
    },
    linkBreak(id) {
      // console.log("link break", id);
    },
    linkAdded(link) {
      // console.log("new link added:", link);
    }
  }
};
</script>

<style lang="scss">
#app {
  user-select: none;
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 0;
  overflow: hidden;
  height: 800px;
  .tool-wrapper {
    position: relative;
  }
  .node-add-btn {
    cursor: pointer;
    position: fixed;
    margin: auto;
    width: 50px;
    height: 50px;
    line-height: 50px;
    font-size: 24px;
    font-weight: bold;
    bottom: 0;
    right: 0;
    margin-bottom: 40px;
    margin-right: 40px;
    border-radius: 50%;
    color: white;
    background: rgb(60, 140, 231);
    border: 5px solid rgb(43, 108, 192);
    z-index: 99999;
    &:active {
      background-color: rgb(43, 108, 192);
    }
  }
  .node-add-pnl {
    cursor: pointer;
    position: fixed;
    margin: auto;
    width: 200px;
    height: 140px;
    bottom: 0;
    right: 0;
    margin-bottom: 40px;
    margin-right: 100px;
    background-color: white;
    border: 3px solid rgba(128, 128, 128, 0.164);
    border-radius: 20px;
    z-index: 99999;
    transition: ease all 0.3;
    .node-category-box {
      position: relative;
      float: left;
      width: 55px;
      height: 55px;
      line-height: 55px;
      margin: 2px;
      background-color: white;
      border: 3px solid rgba(128, 128, 128, 0.164);
      border-radius: 20px;
      &:hover {
        border-color: rgba(128, 128, 128, 0.527);
      }
    }
  }
}
</style>
