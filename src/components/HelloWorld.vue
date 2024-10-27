<template>
  <div class="container">
  <div class="container__left">
    <img
      width="100px"
      height="100px"
      v-for="item in list"
      :src="item.src"
      @click="handleClick(item.src)"
      :key="item.id"/>
  </div>
  <div class="container__right">
    <div class="tool">
      <button @click="switchRect">方形</button>
      <button @click="switchPaint">画笔</button>
      <button @click="app.setTool(selectTool)">选择</button>
      <button @click="removeSelect">删除当前选中</button>
      <button @click="upZindex">zindex上移</button>
      <button @click="downZindex">zindex下移</button>
      <button @click="copy">复制当前选中</button>
      <button @click="undo">撤销</button>
      <button @click="redo">重做</button>
      <button @click="clear">清空</button>
      <button @click="exportPNG">导出为png</button>
      <button @click="exportJSON">导出为json</button>
      <input type="file" ref="file" id="fileInput" style="visibility: hidden;" @change="getData">
      <button @click="importJSON">导入json</button>
    </div>
    <div class="canves" ref="canvasRef"></div>
  </div>
  </div>
</template>

<script>
import img1 from '@/assets/test1.png';
import img2 from '@/assets/test2.png';
import img3 from '@/assets/test5.png';
import img4 from '@/assets/xiaolin.png';
import img5 from '@/assets/xiao.png';
import img6 from '@/assets/bg.png';
import { App } from '@pictode/core';
import {
  // ArrowTool,
  // DrawingTool,
  // EllipseTool,
  // EraserTool,
  // IframeTool,
  ImageTool,
  // LineTool,
  RectTool,
  // RegularPolygonTool,
  SelectTool,
  // TextTool,
} from '@pictode/tools';
import AlignmentPlugin from '@pictode/plugin-alignment';
import HistoryPlugin from '@pictode/plugin-history';
import RulerPlugin from '@pictode/plugin-ruler';
import SelectorPlugin from '@pictode/plugin-selector';
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
      app: null,
      image1: null,
      rectTool: null,
      selectTool: null,
      currentTool: null,
      canvesDom: null,
      selected: null,
      list: [
        {
          src: img1,
          id: 1
        },
        {
          src: img2,
          id: 2
        },
        {
          src: img3,
          id: 3
        },
        {
          src: img4,
          id: 4
        },
        {
          src: img5,
          id: 5
        },
        {
          src: img6,
          id: 6
        }
      ]
    }
  },
  mounted() {
    this.canvesDom = this.$refs.canvasRef;
    this.initPitcode();
  },
  methods: {
    initPitcode() {
      const app = new App({
        background: {
          enabled: false,
          shape: 'circle',
          color: '#000',
          padding: 40,
          size: 2,
        },
      });

      const historyPlugin = new HistoryPlugin({
        enabled: true,
        stackSize: 50,
      });

      const selectorPlugin = new SelectorPlugin({
        enabled: true,
        multipleSelect: true,
      });

      const alignmentPlugin = new AlignmentPlugin({
        enabled: true,
      });

      const rulerPlugin = new RulerPlugin({
        enabled: false,
        axis: ['x', 'y'],
      });

      app.use(rulerPlugin);
      app.use(historyPlugin);
      app.use(selectorPlugin);
      app.use(alignmentPlugin);

      app.on('selected:changed', this.onSelectedChanged);
      app.on('tool:changed', this.onToolChanged);
      app.on('canvas:zoom:end', this.onZoomEnd);
      app.on('mouse:down', this.onMouseDown);
      app.on('mouse:up', this.onMouseUp);

      app.mount(this.canvesDom);
      this.app = app;
      this.selectTool = new SelectTool({
        hooks: {
          onActive() {
            selectorPlugin.enable();
          },
          onInactive() {
            selectorPlugin.disable();
          },
        },
      });
      this.app.setTool(this.selectTool)
      console.log(app);
    },
    onSelectedChanged({selected}) {
      this.selected = selected;
    },
    onToolChanged() {

    },
    onZoomEnd() {

    },
    onMouseDown({ event }) {
      if (event.evt.button === 1) {
        this.app.triggerPanning(true);
      }
    },
    onMouseUp({ event }) {
      if (event.evt.button === 1) {
        this.app.triggerPanning(false);
      }
    },
    switchPaint() {

    },
    switchRect() {
      const rectTool = new RectTool({
        config: {
          stroke: "black",
          strokeWidth: 2,
          fill: "gray",
          cornerRadius: 0,
          opacity: 1,
        },
        hooks: {
          onActive(app) {
            app.containerElement.style.cursor = 'crosshair';
            app.cancelSelect();
          },
          onInactive(app) {
            app.containerElement.style.cursor = `default`;
          },
          onCompleteDrawing: (app, node) => {
            this.app.setTool(this.selectTool);
            this.$nextTick(() => app.select(node));
          },
        },
      });
      this.app.setTool(rectTool);
    },
    handleClick(src) {
      const image = new Image();
      image.src = src;
      const image1 = new ImageTool({
        config: {
          image
        },
        hooks: {
          onActive(app) {
            app.cancelSelect();
          },
          onInactive(app) {
            app.containerElement.style.cursor = `default`;
          },
          onCompleteDrawing: (app, node) => {
            this.app.setTool(this.selectTool);
            this.$nextTick(() => app.select(node));
          },
        },
      });
      this.app.setTool(image1);
    },
    removeSelect() {
      if (this.selected) {
        this.app.remove(...this.selected);
      }
    },
    upZindex() {
      this.app.moveUp(...this.selected);
    },
    downZindex() {
      this.app.moveDown(...this.selected);
    },
    copy() {
      const newData = this.selected.map((node)=> {
        const copy = node.clone();
        // 使新节点产生偏移
        copy.setAttrs({
          x: copy.x() + copy.width() / 2,
          y: copy.y() + copy.height() / 2,
          id: '',
        });
        return copy;
      });
      this.app.add(...newData);
      this.app.setTool(this.selectTool);
      this.$nextTick(() => this.app.select(...newData));
    },
    clear() {
      this.app.clear();
    },
    undo() {
      this.app.undo();
    },
    redo() {
      this.app.redo();
    },
    async exportPNG() {
      const { dataURL } = await this.app.toDataURL();
      this.download(dataURL, '测试导出图片.png')
    },
    async exportJSON() {
      const jsonStr = await this.app.toJSON();
      var blob = new Blob([jsonStr]); //  创建 blob 对象
      this.download(URL.createObjectURL(blob), '导出配置.json');
    },
    download(link, filename){
      let a = document.createElement('a')
      a.href = link
      a.download = filename || 'default.png'
      a.dispatchEvent(new MouseEvent('click'))
    },
    async importJSON() {
      this.$refs.file.click();
    },
    getData(event) {
      const file  = event.target.files[0]
      const reader = new FileReader();
      reader.onload = () => {
        this.app.fromJSON(reader.result);
      };
      reader.readAsText(file);
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.container {
  width: 100vw;
  height: 100vh;
  display: flex;
}
.container__left {
  flex-basis: 100px;
  height: 100%;
  outline: 1px solid #d8dcde;
}
.container__right {
  flex-grow: 1;
  height: 100%;
  display: flex;
  flex-direction: column;
  outline: 1px solid #d8dcde
}
.tool {
  width: 100%;
  height: 50px;
  outline: 1px solid #d8dcde
}
</style>
