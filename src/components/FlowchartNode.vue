<template>
  <div
    class="flowchart-node"
    :style="nodeStyle"
    @mousedown="handleMousedown"
    @mouseover="handleMouseOver"
    @mouseleave="handleMouseLeave"
    @mouseup="handleUp"
    v-bind:class="{selected: options.selected === id}"
  >
    <div class="node-port node-input" @mousedown="inputMouseDown" @mouseup="inputMouseUp"></div>
    <div class="node-main">
      <div v-text="type" :style="{ backgroundColor: color }" class="node-type"></div>
      <div v-text="label" class="node-label"></div>
    </div>
    <div class="node-port node-output" @mousedown="outputMouseDown"></div>
    <div v-show="show.delete" class="node-options-pane">
      <button class="node-edit">Edit</button>
      <button class="node-delete">&times;</button>
    </div>
  </div>
</template>

<script>
export default {
  name: "FlowchartNode",
  props: {
    id: {
      type: Number,
      default: 1000,
      validator(val) {
        return typeof val === "number";
      }
    },
    color: {
      type: String,
      default: "black",
      validator(val) {
        return typeof val === "string";
      }
    },
    x: {
      type: Number,
      default: 0,
      validator(val) {
        return typeof val === "number";
      }
    },
    y: {
      type: Number,
      default: 0,
      validator(val) {
        return typeof val === "number";
      }
    },
    type: {
      type: String,
      default: "Default"
    },
    label: {
      type: String,
      default: "input name"
    },
    options: {
      type: Object,
      default() {
        return {
          centerX: 1920,
          scale: 1,
          centerY: 140
        };
      }
    }
  },
  data() {
    return {
      show: {
        delete: false,
        editMode: false,
        colorMode: false,
      }
    };
  },
  mounted() {},
  computed: {
    nodeStyle() {
      return {
        top: this.options.centerY + this.y * this.options.scale + "px", // remove: this.options.offsetTop +
        left: this.options.centerX + this.x * this.options.scale + "px", // remove: this.options.offsetLeft +
        transform: `scale(${this.options.scale})`
      };
    }
  },
  methods: {
    handleMousedown(e) {
      const target = e.target || e.srcElement;
      // console.log(target);
      if (
        target.className.indexOf("node-input") < 0 &&
        target.className.indexOf("node-output") < 0
      ) {
        this.$emit("nodeSelected", e);
      }
      e.preventDefault();
    },
    handleMouseOver() {
      this.show.delete = true;
    },
    handleMouseLeave() {
      this.show.delete = false;
    },
    outputMouseDown(e) {
      this.$emit("linkingStart");
      e.preventDefault();
    },
    inputMouseDown(e) {
      e.preventDefault();
    },
    inputMouseUp(e) {
      this.$emit("linkingStop");
      e.preventDefault();
    },
    handleUp(e) {
      // const target = e.target || e.srcElement;
      // if (this.$el.contains(target)) {
      //   // if (
      //   //   typeof target.className === "string" &&
      //   //   target.className.indexOf("node-edit") > -1
      //   // ) {
      //   //   // console.log('delete2', this.action.dragging);
      //   //   this.show.editMode = true;
      //   //   this.header = this.nodeHeader;
      //   // }
      //   // if (
      //   //   typeof target.className === "string" &&
      //   //   target.className.indexOf("node-color") > -1
      //   // ) {
      //   //   // console.log('delete2', this.action.dragging);
      //   //   this.show.colorMode = true;
      //   // }
      //   // if (
      //   //   typeof target.className === "string" &&
      //   //   target.className.indexOf("new-node-val") > -1
      //   // ) {
      //   //   // console.log('delete2', this.action.dragging);
      //   //   e.target.focus();
      //   // }
      // }
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
$themeColor: rgb(1, 1, 1);
// $themeColor: rgb(255, 136, 85);
$portSize: 20;

.flowchart-node {
  user-select: none;
  margin: 0;
  width: 200px;
  height: 120px;
  position: absolute;
  box-sizing: border-box;
  border: none;
  background: white;
  z-index: 1;
  opacity: 0.9;
  cursor: move;
  transform-origin: top left;
  overflow-y: hidden;
  overflow-x: hidden;
  .node-main {
    text-align: center;
    .node-type {
      color: white;
      font-size: 18px;
      padding: 6px;
      text-align: left;
    }
    .node-label {
      margin: 10px;
      font-size: 20px;
      // background: black;
      height: auto;
    }
  }
  .node-port {
    position: absolute;
    width: #{$portSize}px;
    height: #{$portSize}px;
    left: 50%;
    transform: translate(-50%);
    border: 1px solid #ccc;
    border-radius: 100px;
    background: white;
    z-index: 99999;
    &:hover {
      background: $themeColor;
      border: 1px solid $themeColor;
    }
  }
  .node-input {
    top: #{-2 + $portSize/-2}px;
  }
  .node-output {
    bottom: #{-2 + $portSize/-2}px;
  }
  .node-options-pane {
    position: absolute;
    width: 100%;
    height: 100%;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    margin: auto;
    background-color: rgba(1, 1, 1, 0.8);
    transition: ease-in all 0.2s;
    .node-delete {
      position: absolute;
      width: auto;
      height: 20%;
      right: 0;
      margin: 4px;
      transform-origin: center;
      border-radius: 45px;
      color: white;
      background: #E74C3C;
      border: 2px solid #C0392B;
      cursor: pointer;
      &:active {
        background: #C0392B;
      }
    }
    .node-edit {
      position: absolute;
      width: 100px;
      height: 20%;
      right: 0;
      left: 0;
      top: 50%;
      margin: auto;
      transform-origin: center;
      border-radius: 45px;
      cursor: pointer;
    }
  }
}
.selected {
  box-shadow: 0 0 0 2px $themeColor;
}
</style>
