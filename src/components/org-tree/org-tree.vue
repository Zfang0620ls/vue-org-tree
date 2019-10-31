<template>
<div>
  <div class="org-tree__download-btn__wrap">
  <div class="org-tree__download-btn" v-if="this.export" @click="this._handleExport">导出图片</div>
  </div>

  <a class="download" download="w3logo.png"></a>
  <div id="org-tree__wrap" class="org-tree__wrap">
  <div class="org-tree-container" ref="otContainer">
    <div class="org-tree" :class="{horizontal, collapsable}">
      <org-tree-node
        :data="data"
        :props="props"
        :horizontal="horizontal"
        :label-width="labelWidth"
        :collapsable="collapsable"
        :render-content="renderContent"
        :label-class-name="labelClassName"
        @on-drop="this.handleDrop"
        @on-dragstart="this.handleDragStart"
        @on-expand="this.onExpand"
        @on-node-click="(e, data) => {$emit('on-node-click', e, data)}"
      />
    </div>
    <div>

    </div>
  </div>
  </div>
    <div class="export-content"   >
    <div class="org-tree" :class="{horizontal}" ref="exportContainer">
      <org-tree-node
        :data="data"
        :props="props"
        :horizontal="horizontal"
        :label-width="labelWidth"
        :render-content="renderContent"
        :label-class-name="labelClassName"
      />
    </div>
    <div>

    </div>
  </div>
</div>
</template>

<script>
import render from './node'
import html2canvas from 'html2canvas'
export default {
  name: 'Vue2OrgTree',
  components: {
    OrgTreeNode: {
      render,
      functional: true
    }
  },
  data(){
    return {
      chart:{},
      dragstartNode:{},
      dragstartData:{}
    }
  },
  props: {
    data: {
      type: Object,
      required: true
    },
    props: {
      type: Object,
      default: () => ({
        label: 'label',
        expand: 'expand',
        children: 'children'
      })
    },
    horizontal: Boolean,
    selectedKey: String,
    collapsable: Boolean,
    renderContent: Function,
    labelWidth: [String, Number],
    labelClassName: [Function, String],
    selectedClassName: [Function, String],
    zoom:Boolean,
    export:Boolean,
    draggable:Boolean,
    canMove:Boolean,
    isShow:false
  },
  created( ) {
  },
  mounted () {
    const otContainer = this.$refs.otContainer;
    this.chart = otContainer;
    if(this.zoom){
     otContainer.addEventListener('wheel', this._onWheeling.bind(this));
    }

    if(this.canMove){
      document.getElementById('org-tree__wrap').style.overflow = 'hidden';
      otContainer.addEventListener('mousedown', this._onPanStart.bind(this));
      otContainer.addEventListener('touchstart', this._onPanStart.bind(this));
      otContainer.addEventListener('mouseup', this._onPanEnd.bind(this));
      document.addEventListener('touchend', this._onPanEnd.bind(this));
    }
  },
  methods:{
  _onWheeling(event) {
    event.preventDefault();

    let newScale = event.deltaY > 0 ? 0.8 : 1.2;

    this._setChartScale(this.chart, newScale);
  },
    _setChartScale(chart, newScale) {
    let lastTf = window.getComputedStyle(chart).transform;

    if (lastTf === 'none') {
      chart.style.transform = 'scale(' + newScale + ',' + newScale + ')';
    } else {
      let matrix = lastTf.split(',');

      if (!lastTf.includes('3d')) {
        matrix[0] = 'matrix(' + newScale;
        matrix[3] = newScale;
        chart.style.transform = lastTf + ' scale(' + newScale + ',' + newScale + ')';
      } else {
        chart.style.transform = lastTf + ' scale3d(' + newScale + ',' + newScale + ', 1)';
      }
    }
    chart.dataset.scale = newScale;
  },
  _handleExport(){
    const sourceChart = this.$refs.exportContainer;
    this.$nextTick(()=>{
    html2canvas(sourceChart, {
      'width': (!this.horizontal ? sourceChart.clientHeight : sourceChart.clientWidth)+200,
      'height': (!this.horizontal ? sourceChart.clientWidth : sourceChart.clientHeight)+200,
    })
    .then((canvas) => {
      let downloadBtn = document.querySelector('.download');

      downloadBtn.setAttribute('href', canvas.toDataURL());
      downloadBtn.click();
    })
    .catch((err) => {
      console.error('导出图像失败!', err);
    })
    })
  },
  handleDrop(e,root, node, data){

  },
  handleDragStart(root, node, data){
  },
  _onPanStart(event) {
    let chart = event.currentTarget;

    // if (this._closest(event.target, (el) => el.classList && el.classList.contains('node')) ||
    //   (event.touches && event.touches.length > 1)) {
    //   chart.dataset.panning = false;
    //   return;
    // }
    chart.style.cursor = 'move';
    chart.dataset.panning = true;

    let lastX = 0,
      lastY = 0,
      lastTf = window.getComputedStyle(chart).transform;

    if (lastTf !== 'none') {
      let temp = lastTf.split(',');

      if (!lastTf.includes('3d')) {
        lastX = Number.parseInt(temp[4], 10);
        lastY = Number.parseInt(temp[5], 10);
      } else {
        lastX = Number.parseInt(temp[12], 10);
        lastY = Number.parseInt(temp[13], 10);
      }
    }
    let startX = 0,
      startY = 0;

    if (!event.targetTouches) { // pan on desktop
      startX = event.pageX - lastX;
      startY = event.pageY - lastY;
    } else if (event.targetTouches.length === 1) { // pan on mobile device
      startX = event.targetTouches[0].pageX - lastX;
      startY = event.targetTouches[0].pageY - lastY;
    } else if (event.targetTouches.length > 1) {
      return;
    }
    chart.dataset.panStart = JSON.stringify({ 'startX': startX, 'startY': startY });
    chart.addEventListener('mousemove', this._onPanning.bind(this));
    chart.addEventListener('touchmove', this._onPanning.bind(this));
  },
  _onPanning(event) {
    let chart = event.currentTarget;

    if (chart.dataset.panning === 'false') {
      return;
    }
    let newX = 0,
      newY = 0,
      panStart = JSON.parse(chart.dataset.panStart),
      startX = panStart.startX,
      startY = panStart.startY;

    if (!event.targetTouches) { // pand on desktop
      newX = event.pageX - startX;
      newY = event.pageY - startY;
    } else if (event.targetTouches.length === 1) { // pan on mobile device
      newX = event.targetTouches[0].pageX - startX;
      newY = event.targetTouches[0].pageY - startY;
    } else if (event.targetTouches.length > 1) {
      return;
    }
    let lastTf = window.getComputedStyle(chart).transform;

    if (lastTf === 'none') {
      if (!lastTf.includes('3d')) {
        chart.style.transform = 'matrix(1, 0, 0, 1, ' + newX + ', ' + newY + ')';
      } else {
        chart.style.transform = 'matrix3d(1, 0, 0, 0, 0, 1, 0, 0, 0, 0, 1, 0, ' + newX + ', ' + newY + ', 0, 1)';
      }
    } else {
      let matrix = lastTf.split(',');

      if (!lastTf.includes('3d')) {
        matrix[4] = newX;
        matrix[5] = newY + ')';
      } else {
        matrix[12] = newX;
        matrix[13] = newY;
      }
      chart.style.transform = matrix.join(',');
    }
  },
  _onPanEnd(event) {
    let chart = event.currentTarget;

    if (chart.dataset.panning === 'true') {
      chart.dataset.panning = false;
      chart.style.cursor = 'default';
      document.body.removeEventListener('mousemove', this._onPanning);
      document.body.removeEventListener('touchmove', this._onPanning);
    }
  },
  onExpand(e,data) {
      if ("expand" in data) {
        data.expand = !data.expand;
        if (!data.expand && data.children) {
          this.collapse(data.children);
        }
      } else {
        this.$set(data, "expand", true);
      }
  },
  collapse(list) {
      var _this = this;
      list.forEach(function(child) {
        if (child.expand) {
          child.expand = false;
        }
        child.children && _this.collapse(child.children);
      });
  },
}
}
</script>

<style lang="less">
@import '../../styles/org-tree';
.org-tree__download-btn {
  padding: 5px 10px;
  font-size:15px;
  color:white;
  background: #19be6b;
  display:inline-block;
  cursor: pointer;
  border-radius:4px;
}

.export-content {
  position: absolute;
  height: 0 !important;
  top: 0;
  left: 0;
  overflow: hidden;
}
.org-tree__wrap,.org-tree-container{
  width: 100%;
  height: 100%;
}
.org-tree__download-btn__wrap{
  display: flex;
  align-items: center;
  justify-content: flex-end;

}
</style>
