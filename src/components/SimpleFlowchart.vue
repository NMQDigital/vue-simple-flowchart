<template>
  <div
    class="flowchart-container"
    @mousemove="handleMove"
    @mouseup="handleUp"
    @mousedown="handleDown"
  >
    <div class="node-add-btn" @click="paneControl = !paneControl">{{paneControl ? '-' : '+'}}</div>
    <div class="node-add-pnl" v-if="paneControl">
      <div
        class="node-category-box"
        v-for="(item, index) in nodeCategory"
        :key="index"
        @dblclick="addNode(item)"
      >{{item.text}}</div>
    </div>
    <svg width="100%" :height="`${height}px`">
      <flowchart-link
        v-bind.sync="link"
        v-for="(link, index) in lines"
        :key="`link${index}`"
        @deleteLink="linkDelete(link.id)"
        @editLink="linkEdit(link.id)"
      ></flowchart-link>
    </svg>
    <flowchart-node
      v-bind.sync="node"
      v-for="(node, index) in flowchartData.nodes"
      :key="`node${index}`"
      :options="nodeOptions"
      @linkingStart="linkingStart(node.id)"
      @linkingStop="linkingStop(node.id)"
      @nodeSelected="nodeSelected(node.id, $event)"
    ></flowchart-node>
    <div class="node-edit-wrapper" v-if="nodeValues.editModeNode">
      <div class="node-edit-panel">
        <div class="node-edit-panel-header">
          <h4>Edit Node Value</h4>
        </div>
        <div class="node-edit-panel-label">
          <input type="text" v-model="nodeValues.text">
        </div>
        <!-- <div class="node-edit-panel-color">
          <button
            v-for="(item, index) in nodeValues.colors"
            :style="{backgroundColor: item }"
            :class="nodeValues.colorIndex === index ? 'color-buttons-active' : 'color-buttons'"
            :key="index"
            @click="changeColor(item, index)"
          ></button>
        </div>-->
        <div class="flat-button" @click="acceptChanges">Accept Changes</div>
      </div>
    </div>
        <div class="node-edit-wrapper" v-if="nodeValues.editModeLink">
      <div class="node-edit-panel">
        <div class="node-edit-panel-header">
          <h4>Edit Link Value</h4>
        </div>
        <div class="node-edit-panel-label">
          <input type="text" v-model="nodeValues.text">
        </div>
        <!-- <div class="node-edit-panel-color">
          <button
            v-for="(item, index) in nodeValues.colors"
            :style="{backgroundColor: item }"
            :class="nodeValues.colorIndex === index ? 'color-buttons-active' : 'color-buttons'"
            :key="index"
            @click="changeColor(item, index)"
          ></button>
        </div>-->
        <div class="flat-button" @click="acceptChanges">Accept Changes</div>
      </div>
    </div>
  </div>
</template>

<script>
import FlowchartLink from "./FlowchartLink.vue";
import FlowchartNode from "./FlowchartNode.vue";
import { getMousePosition } from "../assets/utilty/position";

export default {
  name: "VueFlowchart",
  props: {
    categories: {
      type: Array,
      default() {
        return [];
      }
    },
    scene: {
      type: Object,
      default() {
        return {
          centerX: 1024,
          scale: 1,
          centerY: 140,
          nodes: [],
          links: []
        };
      }
    },
    height: {
      type: Number,
      default: 400
    }
  },
  data() {
    return {
      paneControl: false,
      action: {
        linking: false,
        dragging: false,
        scrolling: false,
        selected: 0
      },
      mouse: {
        x: 0,
        y: 0,
        lastX: 0,
        lastY: 0
      },
      draggingLink: null,
      rootDivOffset: {
        top: 0,
        left: 0
      },
      nodeValues: {
        id: 0,
        text: "",
        color: "",
        editModeNode: false,
        editModeLink: false,
        colors: [
          "red",
          "blue",
          "yellow",
          "orange",
          "black",
          "aqua",
          "green",
          "olive",
          "purple"
        ],
        colorIndex: -1
      },
      flowchartData: {"centerX":1308,"centerY":643,"scale":1,"nodes":[{"id":1555337469783,"x":-1294,"y":-632,"type":"Segment Member","label":"Test Members","color":"olive"},{"id":1555337475998,"x":-991,"y":-486,"type":"Wait","label":"2 Months","color":"navy"},{"id":1555337502295,"x":-1240,"y":-321,"type":"Email1","label":"","color":"green"},{"id":1555337535151,"x":-878,"y":-181,"type":"Decision","label":"Clicked Email1","color":"purple"},{"id":1555337563864,"x":-1011,"y":79,"type":"Email2","label":"","color":"black"},{"id":1555337584872,"x":-615,"y":29,"type":"Form","label":"","color":"blue"}],"links":[{"id":1,"from":1555337469783,"to":1555337475998,"text":"-"},{"id":2,"from":1555337475998,"to":1555337502295,"text":"-"},{"id":3,"from":1555337502295,"to":1555337535151,"text":"-"},{"id":4,"from":1555337535151,"to":1555337563864,"text":"True"},{"id":5,"from":1555337535151,"to":1555337584872,"text":"False"}]},
      nodeCategory: [
        { text: "Segment Member", color: "olive" },
        { text: "Email1", color: "green" },
        { text: "Email2", color: "black" },
        { text: "Form", color: "blue" },
        { text: "Landing Page", color: "red" },
        { text: "Decision", color: "purple" },
        { text: "Wait", color: "navy" },
      ]
    };
  },
  components: {
    FlowchartLink,
    FlowchartNode
  },
  computed: {
    nodeOptions() {
      return {
        centerY: this.flowchartData.centerY,
        centerX: this.flowchartData.centerX,
        scale: this.flowchartData.scale,
        offsetTop: this.rootDivOffset.top,
        offsetLeft: this.rootDivOffset.left,
        selected: this.action.selected
      };
    },
    lines() {

      const lines = this.flowchartData.links.map(link => {
        const fromNode = this.findNodeWithID(link.from);
        const toNode = this.findNodeWithID(link.to);
        let x, y, cy, cx, ex, ey;
        x = this.flowchartData.centerX + fromNode.x;
        y = this.flowchartData.centerY + fromNode.y;
        [cx, cy] = this.getPortPosition("bottom", x, y);
        x = this.flowchartData.centerX + toNode.x;
        y = this.flowchartData.centerY + toNode.y;
        [ex, ey] = this.getPortPosition("top", x, y);
        return {
          start: [cx, cy],
          end: [ex, ey],
          id: link.id,
          text: link.text
        };
      });
      if (this.draggingLink) {
        let x, y, cy, cx;
        const fromNode = this.findNodeWithID(this.draggingLink.from);
        x = this.flowchartData.centerX + fromNode.x;
        y = this.flowchartData.centerY + fromNode.y;
        [cx, cy] = this.getPortPosition("bottom", x, y);
        // push temp dragging link, mouse cursor postion = link end postion
        lines.push({
          start: [cx, cy],
          end: [this.draggingLink.mx, this.draggingLink.my]
        });
      }
      return lines;
    }
  },
  methods: {
    exportToJson() {
      return this.flowchartData.nodes;
    },
    findNodeWithID(id) {
      return this.flowchartData.nodes.find(item => {
        return id === item.id;
      });
    },
    getPortPosition(type, x, y) {
      if (type === "top") {
        return [x + 100, y]; // line connection top position nodeWidth / 2
      } else if (type === "bottom") {
        return [x + 100, y + 120]; // line connection bottom position nodeWidth / 2
      }
    },
    linkingStart(index) {
      this.action.linking = true;
      this.draggingLink = {
        from: index,
        mx: 0,
        my: 0
      };
    },
    linkingStop(index) {
      // add new Link
      if (this.draggingLink && this.draggingLink.from !== index) {
        // check link existence
        const existed = this.flowchartData.links.find(link => {
          return link.from === this.draggingLink.from && link.to === index;
        });
        if (!existed) {
          let maxID = Math.max(
            0,
            ...this.flowchartData.links.map(link => {
              return link.id;
            })
          );
          const newLink = {
            id: maxID + 1,
            from: this.draggingLink.from,
            to: index,
            text: '-' // add text option
          };
          this.flowchartData.links.push(newLink);
          this.$emit("linkAdded", newLink);
        }
      }
      this.draggingLink = null;
    },
    linkDelete(id) {
      const deletedLink = this.flowchartData.links.find(item => {
        return item.id === id;
      });
      if (deletedLink) {
        this.flowchartData.links = this.flowchartData.links.filter(item => {
          return item.id !== id;
        });
        this.$emit("linkBreak", deletedLink);
      }
    },
    nodeSelected(id, e) {
      this.action.dragging = id;
      this.action.selected = id;
      this.$emit("nodeClick", id);
      this.mouse.lastX =
        e.pageX || e.clientX + document.documentElement.scrollLeft;
      this.mouse.lastY =
        e.pageY || e.clientY + document.documentElement.scrollTop;
    },
    handleMove(e) {
      if (this.action.linking) {
        [this.mouse.x, this.mouse.y] = getMousePosition(this.$el, e);
        [this.draggingLink.mx, this.draggingLink.my] = [
          this.mouse.x,
          this.mouse.y
        ];
      }
      if (this.action.dragging) {
        this.mouse.x =
          e.pageX || e.clientX + document.documentElement.scrollLeft;
        this.mouse.y =
          e.pageY || e.clientY + document.documentElement.scrollTop;
        let diffX = this.mouse.x - this.mouse.lastX;
        let diffY = this.mouse.y - this.mouse.lastY;

        this.mouse.lastX = this.mouse.x;
        this.mouse.lastY = this.mouse.y;
        this.moveSelectedNode(diffX, diffY);
      }
      if (this.action.scrolling) {
        [this.mouse.x, this.mouse.y] = getMousePosition(this.$el, e);
        let diffX = this.mouse.x - this.mouse.lastX;
        let diffY = this.mouse.y - this.mouse.lastY;

        this.mouse.lastX = this.mouse.x;
        this.mouse.lastY = this.mouse.y;

        this.flowchartData.centerX += diffX;
        this.flowchartData.centerY += diffY;

        // this.hasDragged = true
      }
    },
    handleUp(e) {
      const target = e.target || e.srcElement;
      if (this.$el.contains(target)) {
        if (
          typeof target.className !== "string" ||
          target.className.indexOf("node-input") < 0
        ) {
          this.draggingLink = null;
        }
        if (
          typeof target.className === "string" &&
          target.className.indexOf("node-delete") > -1
        ) {
          // console.log('delete2', this.action.dragging);
          this.nodeDelete(this.action.dragging);
        }
        if (
          typeof target.className === "string" &&
          target.className.indexOf("node-edit") > -1
        ) {
          // console.log('delete2', this.action.dragging);
          this.nodeEdit(this.action.dragging);
        }
        // if (e.target.className.baseVal === 'link-change-btn') {
        //   // console.log('delete2', this.action.dragging);
        //   // console.log('delete');
        //   this.changeLinkText();
        // }
      }
      this.action.linking = false;
      this.action.dragging = null;
      this.action.scrolling = false;
      console.log(JSON.stringify(this.flowchartData), 'export');
    },
    handleDown(e) {
      const target = e.target || e.srcElement;
      // console.log('for scroll', target, e.keyCode, e.which)
      if (
        (target === this.$el || target.matches("svg, svg *")) &&
        e.which === 1
      ) {
        this.action.scrolling = true;
        [this.mouse.lastX, this.mouse.lastY] = getMousePosition(this.$el, e);
        this.action.selected = null; // deselectAll
      }
      this.$emit("canvasClick", e);
    },
    moveSelectedNode(dx, dy) {
      let index = this.flowchartData.nodes.findIndex(item => {
        return item.id === this.action.dragging;
      });
      let left =
        this.flowchartData.nodes[index].x + dx / this.flowchartData.scale;
      let top =
        this.flowchartData.nodes[index].y + dy / this.flowchartData.scale;
      this.$set(
        this.flowchartData.nodes,
        index,
        Object.assign(this.flowchartData.nodes[index], {
          x: left,
          y: top
        })
      );
    },
    nodeDelete(id) {
      this.flowchartData.nodes = this.flowchartData.nodes.filter(node => {
        return node.id !== id;
      });
      this.flowchartData.links = this.flowchartData.links.filter(link => {
        return link.from !== id && link.to !== id;
      });
      this.$emit("nodeDelete", id);
    },
    nodeEdit(id) {
      if (id) {
        const node = this.flowchartData.nodes.filter(x => x.id === id)[0];
        this.nodeValues.id = id;
        this.nodeValues.editModeNode = true;
        this.nodeValues.text = node.label;
        this.nodeValues.color = node.color;
        // this.flowchartData.nodes.forEach(x => {
        //   if (x.id === id) {
        //     x.color = 'black';
        //   }
        // });
      }
    },
    changeColor(color, index) {
      this.nodeValues.color = color;
      this.nodeValues.colorIndex = index;
    },
    acceptChanges() {
      if (this.nodeValues.editModeNode) {
        this.flowchartData.nodes.forEach(x => {
          if (x.id === this.nodeValues.id) {
            x.label = this.nodeValues.text;
            x.color = this.nodeValues.color;
          }
        });
      }
      if (this.nodeValues.editModeLink) {
        this.flowchartData.links.forEach(x => {
          if (x.id === this.nodeValues.id) {
            x.text = this.nodeValues.text;
          }
        });
      }
      this.nodeValues.editModeNode = false;
      this.nodeValues.editModeLink = false;
    },
    addNode(category) {
      const tmp = Date.now();
      this.flowchartData.nodes.push({
        id: tmp,
        x: -400,
        y: 50,
        type: category.text,
        label: "##value##",
        color: category.color
      });
      this.paneControl = false;
    },
    linkEdit(id) {
      if (id) {
        const link = this.flowchartData.links.filter(x => x.id === id)[0];
        this.nodeValues.id = id; // link id
        this.nodeValues.editModeLink = true;
        this.nodeValues.text = link.text;
        // this.flowchartData.nodes.forEach(x => {
        //   if (x.id === id) {
        //     x.color = 'black';
        //   }
        // });
      }
    }
  },
  mounted() {
    this.rootDivOffset.top = this.$el ? this.$el.offsetTop : 0;
    this.rootDivOffset.left = this.$el ? this.$el.offsetLeft : 0;
    if (this.scene.nodes.length > 0) {
      this.flowchartData = this.scene;
    }
    if (this.categories.length > 0) {
      this.nodeCategory.push(...this.categories);
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
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
    height: 240px;
    bottom: 0;
    right: 0;
    margin-bottom: 40px;
    margin-right: 100px;
    background-color: white;
    border: 3px solid rgba(128, 128, 128, 0.164);
    border-radius: 5px;
    z-index: 99999;
    transition: ease all 0.3;
    overflow-y: scroll;
    .node-category-box {
      position: relative;
      float: left;
      width: 80%;
      height: auto;
      line-height: 55px;
      margin: 2px;
      background-color: white;
      border: 3px solid rgba(128, 128, 128, 0.164);
      border-radius: 5px;
      &:hover {
        border-color: rgba(128, 128, 128, 0.527);
      }
    }
  }
.flowchart-container {
  margin: 0;
  background: #ddd;
  position: relative;
  overflow: hidden;
  svg {
    cursor: grab;
  }
}
.node-edit-wrapper {
  position: absolute;
  margin: auto;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: rgba(0, 0, 0, 0.548);
  width: 100%;
  height: 100%;
  z-index: 1;
  .node-edit-panel {
    position: absolute;
    margin: auto;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    width: 300px;
    height: 100px;
    background-color: whitesmoke;
    border: solid 5px white;
    border-radius: 45px;
    .node-edit-panel-header {
      position: relative;
      float: left;
      width: 100%;
      height: 15%;
      h4 {
        margin: 0;
      }
      // border-bottom: solid white 5px;
    }
    .node-edit-panel-label {
      position: relative;
      float: left;
      width: 100%;
      height: 25%;
      // border-bottom: solid white 5px;
      input {
        width: 80%;
        height: 25px;
        border-radius: 12.5px;
        outline: none;
        font-size: 20px;
        margin-top: 7%;
        text-align: center;
      }
    }
    // .node-edit-panel-color {
    //   position: relative;
    //   float: left;
    //   width: 100%;
    //   height: 25%;
    //   border-bottom: solid white 5px;
    //   .color-buttons,
    //   .color-buttons-active {
    //     width: 27px;
    //     height: 27px;
    //     border-radius: 50%;
    //     margin: 5px;
    //     outline: none;
    //   }
    //   .color-buttons-active {
    //     border: solid 2px rgb(60, 140, 231);
    //   }
    // }
  }
}
.flat-button {
  position: relative;
  width: 100px;
  height: 35px;
  background: rgb(60, 140, 231);
  margin: auto;
  bottom: -50%;
  overflow: hidden;
  z-index: 1;
  cursor: pointer;
  transition: color 0.3s;
  /* Typo */
  line-height: 35px;
  text-align: center;
  color: #fff;
  &:active {
    background: rgb(43, 108, 192);
  }
}
.flat-button:after {
  position: absolute;
  top: 90%;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgb(43, 108, 192);
  content: "";
  transition: transform 0.3s;
}
</style>
