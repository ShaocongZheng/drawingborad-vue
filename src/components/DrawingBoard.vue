<template>
  <div class="container">
    <canvas id="drawing-board" ref="drawingboard"></canvas>
    <!--  <div class="bg-btn"></div>
    <div class="color-group" id="bgGroup">
        <h3>选择背景颜色:</h3>
        <ul class="clearfix">
            <li class="bgcolor-item" style="background-color: blue;"></li>
            <li class="bgcolor-item" style="background-color: black;"></li>
            <li class="bgcolor-item" style="background-color: #FF3333;"></li>
            <li class="bgcolor-item" style="background-color: #0066FF;"></li>
            <li class="bgcolor-item" style="background-color: #FFFF33;"></li>
            <li class="bgcolor-item" style="background-color: #33CC66;"></li>
            <li class="bgcolor-item" style="background-color: gray;"></li>
            <li class="bgcolor-item" style="background-color: #F34334;"></li>
            <li class="bgcolor-item" style="background-color: #fff;box-shadow: 0 1px 2px 0 rgba(32,33,36,0.28);"></li>
            <li class="bgcolor-item" style="background-color: #9B27AC;"></li>
            <li class="bgcolor-item" style="background-color: #4CB050;"></li>
            <li class="bgcolor-item" style="background-color: #029688;"></li>
        </ul>
        <i class="closeBtn"></i>
    </div> -->
    <div class="tools">
      <div class="container">
        <button class="save" id="save" title="保存"></button>
        <button
          class="brush"
          :class="{ active: brushActive }"
          id="brush"
          title="画笔"
          @click="
            (brushActive = true),
              (eraserEnabled = false),
              (brushDetailActive = true)
          "
        ></button>
        <button
          class="eraser"
          :class="{ active: eraserEnabled }"
          id="eraser"
          title="橡皮擦"
          @click="(brushActive = false), (eraserEnabled = true)"
        ></button>
        <button class="clear" id="clear" title="清屏"></button>
        <button class="undo" id="undo" title="撤销"></button>
        <button class="redo" id="redo" title="再做"></button>
      </div>
    </div>
    <div
      class="pen-detail"
      :class="{ active: brushDetailActive }"
      id="penDetail"
      ref="penDetail"
    >
      <i class="closeBtn"></i>
      <!-- <p>笔类型</p>
        <ul class="pen-type clearfix">
            <li class="pen"></li>
            <li class="line"></li>
            <li class="circle"></li>
            <li class="rect"></li>
        </ul> -->
      <p>笔大小</p>
      <span class="circle-box"><i id="thickness"></i></span>
      <input type="range" id="range1" min="1" max="10" value="1" />
      <p>笔颜色</p>
      <ul class="pen-color clearfix">
        <li class="color-item active" style="background-color: black"></li>
        <li class="color-item" style="background-color: #ff3333"></li>
        <li class="color-item" style="background-color: #99cc00"></li>
        <li class="color-item" style="background-color: #0066ff"></li>
        <li class="color-item" style="background-color: #ffff33"></li>
        <li class="color-item" style="background-color: #33cc66"></li>
      </ul>
      <p>不透明度</p>
      <i class="showOpacity"></i>
      <input type="range" id="range2" min="1" max="10" value="1" />
    </div>
  </div>
</template>
<script>
export default {
  name: "DrawingBoard",
  data() {
    return {
      canvas: {},
      context: {},
      painting: false, // 定义画笔的初始状态
      brushActive: true,
      eraserEnabled: false,
      undoActive: false,
      brushDetailActive: false,

      radius: 5,
      lineWidth: 2,
      canvasHistory: [],
      step: -1,
    };
  },
  created() {},
  mounted() {
    this.canvas = document.getElementById("drawing-board");
    console.log(this.canvas);
    this.context = this.canvas.getContext("2d");

    this.resizeCanvas();
    this.mouseListner();
    window.dispatchEvent(new Event("resize")); // 触发resize
  },
  methods: {
    mouseListner() {
      // 记录画笔最后一次的位置
      let lastPoint = {
        x: 0,
        y: 0,
      };
      if (document.body.ontouchstart !== undefined) {
        this.canvas.ontouchstart = (e) => {
          this.painting = true;
          let x1 = e.touches[0].clientX;
          let y1 = e.touches[0].clientY;
          lastPoint = { x: x1, y: y1 };
          if (this.eraserEnabled) {
            //要使用eraser
            this.context.save();
            this.context.globalCompositeOperation = "destination-out";
            this.context.beginPath();
            this.radius = this.lineWidth / 2 > 5 ? this.lineWidth / 2 : 5;
            this.context.arc(x1, y1, this.radius, 0, 2 * Math.PI);
            this.context.clip();
            this.context.clearRect(0, 0, this.canvas.width, this.canvas.height);
            this.context.restore();
          }
        };
        this.canvas.ontouchmove = (e) => {
          let x1 = lastPoint["x"];
          let y1 = lastPoint["y"];
          let x2 = e.touches[0].clientX;
          let y2 = e.touches[0].clientY;
          if (!this.painting) {
            return;
          }
          if (this.eraserEnabled) {
            this.moveHandler(x1, y1, x2, y2);
            //记录最后坐标
            lastPoint["x"] = x2;
            lastPoint["y"] = y2;
          } else {
            let newPoint = { x: x2, y: y2 };
            this.drawLine(lastPoint.x, lastPoint.y, newPoint.x, newPoint.y);
            lastPoint = newPoint;
          }
        };
        this.canvas.ontouchend = () => {
          this.painting = false;
          this.canvasDraw();
        };
      } else {
        // 鼠标按下事件
        this.canvas.onmousedown = (e) => {
          this.painting = true;
          let x1 = e.clientX;
          let y1 = e.clientY;
          console.log("mousedown", this.painting);
          if (this.eraserEnabled) {
            //要使用eraser
            //鼠标第一次点下的时候擦除一个圆形区域，同时记录第一个坐标点
            this.context.save();
            this.context.globalCompositeOperation = "destination-out";
            this.context.beginPath();
            this.radius = this.lWidth / 2 > 5 ? this.lWidth / 2 : 5;
            this.context.arc(x1, y1, this.radius, 0, 2 * Math.PI);
            this.context.clip();
            this.context.clearRect(0, 0, this.canvas.width, this.canvas.height);
            this.context.restore();
            lastPoint = { x: x1, y: y1 };
          } else {
            lastPoint = { x: x1, y: y1 };
          }
        };

        // 鼠标移动事件
        this.canvas.onmousemove = (e) => {
          let x1 = lastPoint["x"];
          let y1 = lastPoint["y"];
          let x2 = e.clientX;
          let y2 = e.clientY;
          if (!this.painting) {
            return;
          }
          if (this.eraserEnabled) {
            this.moveHandler(x1, y1, x2, y2);
            //记录最后坐标
            lastPoint["x"] = x2;
            lastPoint["y"] = y2;
          } else {
            let newPoint = { x: x2, y: y2 };

            this.drawLine(lastPoint.x, lastPoint.y, newPoint.x, newPoint.y);
            lastPoint = newPoint;
          }
        };

        // 鼠标松开事件
        this.canvas.onmouseup = () => {
          this.painting = false;
          this.canvasDraw();
        };
      }
    },
    resizeCanvas() {
      window.onresize = () => {
        let imgData = this.context.getImageData(
          0,
          0,
          this.canvas.width,
          this.canvas.height
        );
        let pageWidth = document.documentElement.clientWidth;
        let pageHeight = document.documentElement.clientHeight;

        this.canvas.width = pageWidth;
        this.canvas.height = pageHeight;
        this.context.putImageData(imgData, 0, 0);
      };
    },
    drawLine(x1, y1, x2, y2) {
      this.context.beginPath();
      this.context.lineWidth = this.lineWidth;
      this.context.lineCap = "round"; // 设置末端样式
      this.context.lineJoin = "round"; //设置线条与线条结合处样式
      this.context.moveTo(x1, y1);
      this.context.lineTo(x2, y2);
      this.context.stroke();
      this.context.closePath();
    },
    moveHandler(x1, y1, x2, y2) {
      let canvasRect = this.$refs.drawingboard.getBoundingClientRect();
      x1 = x2 - canvasRect.left;
      y1 = y2 - canvasRect.top;
      console.log(x2, y2, x2 - canvasRect.left, y2 - canvasRect.top);
      this.context.save();
      this.context.beginPath();
      this.context.globalCompositeOperation = "destination-out";
      this.radius = this.lineWidth / 2 > 5 ? this.lineWidth / 2 : 5;
      this.context.arc(x1, y1, this.radius, 0, 2 * Math.PI);
      this.context.clip();
      this.context.clearRect(0, 0, this.canvas.width, this.canvas.height);
      this.context.restore();
    },
    canvasDraw() {
      this.step++;
      // if (this.step < this.canvasHistory.length) {
      //   this.canvasHistory.length = this.step // 截断数组
      // }
      // 添加新的绘制到历史记录
      this.canvasHistory.push(this.canvas.toDataURL());
      // console.log(this.canvasHistory);
      if (this.step > 0) {
        this.undoActive = true;
      }
    },
    handleBrushClick() {
      this.$refs.penDetail.cla;
    },
  },
};
</script>
<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
*::before {
  box-sizing: border-box;
}
*::after {
  box-sizing: border-box;
}
ul {
  padding: 0;
}
li {
  list-style: none;
}
.clearfix::after {
  content: "";
  display: block;
  clear: both;
}
.bg-btn {
  position: fixed;
  top: 25px;
  left: 25px;
  width: 50px;
  height: 50px;
  background: url("../assets/image/ic_droplet1.png") center center no-repeat;
  background-size: 30px 30px;
  border-radius: 50%;
  box-shadow: 0 1px 2px 0 rgba(32, 33, 36, 0.28);
  cursor: pointer;
  background-color: #fff;
}
.bg-btn.active {
  box-shadow: 0 1px 8px 0 rgba(32, 33, 36, 0.28);
}
.tools {
  position: fixed;
  left: 0;
  bottom: 30px;
  width: 100%;
  height: 50px;
  display: flex;
  justify-content: center;
  text-align: center;
}
.tools .container {
  /* height: 70px; */
  padding: 8px 20px;
  border-radius: 40px;
  box-shadow: 0 1px 2px 0 rgba(32, 33, 36, 0.28);
  background: #fff;
}
.tools button {
  width: 30px;
  height: 30px;
  border: none;
  outline: none;
  background-size: 20px 20px;
  background-position: center center;
  background-repeat: no-repeat;
  background-color: #fff;
  margin: 0 6px;
  transition: 0.3s;
  cursor: pointer;
  border: 2px solid transparent;
}
.tools button.save {
  background-image: url("../assets/image/save.png");
}
.tools button.brush {
  background-image: url("../assets/image/pen.png");
}
.tools button.eraser {
  background-image: url("../assets/image/eraser.png");
}
.tools button.clear {
  background-image: url("../assets/image/clear.png");
}
.tools button.undo {
  background-image: url("../assets/image/undo_sel.png");
}
.tools button.redo {
  background-image: url("../assets/image/redo_sel.png");
}
.tools button.undo.active {
  background-image: url("../assets/image/undo.png");
  border-color: transparent;
}
.tools button.redo.active {
  background-image: url("../assets/image/redo.png");
  border-color: transparent;
}
.tools button.active {
  border-radius: 5px;
  border-color: #1398e6;
}

.color-group {
  display: none;
  position: absolute;
  top: 50px;
  left: 50%;
  margin-left: -161px;
  width: 322px;
  padding: 16px;
  border-radius: 5px;
  box-shadow: 0 1px 4px 0 rgba(32, 33, 36, 0.28);
  background-color: #fff;
}
.color-group .closeBtn,
.pen-detail .closeBtn {
  position: absolute;
  top: 6px;
  right: 8px;
  width: 32px;
  height: 32px;
  background: url("../assets/image/close.png") center center no-repeat;
  cursor: pointer;
}
.color-group.active {
  display: block;
}
.color-group h3 {
  font-weight: 500;
  margin-bottom: 8px;
  font-size: 18px;
}
.color-group li {
  position: relative;
  float: left;
  list-style: none;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  margin: 4px;
  cursor: pointer;
}
.color-group li.active::before {
  position: absolute;
  left: 4px;
  top: 4px;
  content: "";
  display: block;
  width: 32px;
  height: 32px;
  background: transparent;
  border: 3px solid #fff;
  border-radius: 50%;
}
.pen-detail {
  display: none;
  position: fixed;
  left: 50%;
  margin-left: -140px;
  bottom: 90px;
  width: 280px;
  height: 210px;
  padding: 20px 24px;
  border: 1px solid #81a4bd;
  border-radius: 5px;
  color: #808fa2;
  font-style: 18px;
  background: #fff;
  font-size: 14px;
}
.pen-detail.active {
  display: block;
}
.pen-detail p {
  margin-top: 2px;
  margin-bottom: 4px;
}
.pen-detail .pen-type li {
  float: left;
  width: 30px;
  height: 30px;
  background-size: 28px 28px;
  background-position: center center;
  background-repeat: no-repeat;
  cursor: pointer;
}
.pen-detail .pen-type li.pen {
  background-image: url("../assets/image/pen_sel.png");
}
.pen-detail .pen-type li.pen.active {
  background-image: url("../assets/image/pen.png");
}
.pen-detail .pen-type li.line {
  background-image: url("../assets/image/line_sel.png");
}
.pen-detail .pen-type li.line.active {
  background-image: url("../assets/image/line.png");
}
.pen-detail .pen-type li.circle {
  border-radius: 50%;
  border: 2px solid #cdcdcd;
}
.pen-detail .pen-type li.circle.active,
.pen-detail .pen-type li.rect.active {
  border-color: #1f1f1f;
}
.pen-detail .pen-type li.rect {
  border-radius: 4px;
  border: 2px solid #cdcdcd;
}
.pen-detail .pen-type li + li {
  margin-left: 28px;
}

.circle-box {
  line-height: 24px;
}

.circle-box {
  position: relative;
  width: 24px;
  height: 24px;
  display: inline-block;
  text-align: center;
  margin-right: 8px;
}
#thickness {
  position: absolute;
  top: 50%;
  left: 50%;
  margin-left: -1px;
  margin-top: -1px;
  background: #000;
  border-radius: 50%;
  transform-origin: center;
  width: 2px;
  height: 2px;
}
input[type="range"] {
  -webkit-appearance: none;
  width: 180px;
  height: 24px;
  outline: none;
}
input[type="range"]::-webkit-slider-runnable-track {
  background-color: #dbdbdb;
  height: 4px;
  border-radius: 5px;
}
input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  /* border: 5px solid #fff; */
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background: #ff4081;
  cursor: pointer;
  margin-top: -4px;
}
.pen-color li {
  position: relative;
  float: left;
  list-style: none;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  margin: 4px;
  cursor: pointer;
}
.pen-color li.active::before {
  position: absolute;
  left: 3px;
  top: 3px;
  content: "";
  display: block;
  width: 24px;
  height: 24px;
  background: transparent;
  border: 2px solid #fff;
  border-radius: 50%;
}
.pen-detail i.showOpacity {
  display: inline-block;
  width: 26px;
  height: 26px;
  background: #000;
  border-radius: 50%;
  margin-right: 4px;
  margin-left: 4px;
}
canvas {
  display: block;
  background: #fff;
}

@media screen and (max-width: 768px) {
  .color-group {
    top: 80px;
  }
  .tools {
    bottom: 15px;
  }
  .pen-detail {
    bottom: 80px;
  }
}
.container {
  height: 100%;
}
</style>