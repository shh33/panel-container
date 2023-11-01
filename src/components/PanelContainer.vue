<template>
  <div
    class="panel-container"
    ref="panelBox"
    @mousedown.prevent="onmousedownHandle"
  >
    <div class="map-container" ref="map" @mouseout.stop :style="transformStyle">
      <template v-if="!loading">
        <slot></slot>
      </template>
    </div>
    <hcc-loading v-if="loading" />
    <img
      v-else
      class="img-container"
      :src="imageUrl"
      :style="transformStyle"
      alt=""
    />
  </div>
</template>

<script>
function _debounce(fn, delay = 200) {
  var timer;
  return function () {
    var th = this;
    var args = arguments;
    if (timer) {
      clearTimeout(timer);
    }
    timer = setTimeout(function () {
      timer = null;
      fn.apply(th, args);
    }, delay);
  };
}
// function _throttle(fun, time = 200) {
//   let t1 = 0; //初始时间
//   return function () {
//     let t2 = new Date(); //当前时间
//     if (t2 - t1 > time) {
//       fun.apply(this, arguments);
//       t1 = t2;
//     }
//   };
// }

export default {
  name: "PanelContainer",
  props: {
    imageUrl: {}
  },
  data() {
    return {
      boxW: 0,
      boxH: 0,
      imgW: 0,
      imgH: 0,
      top: 0,
      left: 0,
      scale: 1,
      size: 0,
      mousewheelevt: null,
      endPosition: { x: 0, y: 0 },
      orginalInfo: {},
      loading: false,
      ismove: false
    };
  },
  computed: {
    transformStyle() {
      return {
        position: "absolute",
        transformOrigin: "top left",
        width: this.imgW + "px",
        height: this.imgH + "px",
        transform: `translate(${this.left}px, ${this.top}px) scale(${this.scale})`
      };
    },
    originalScale() {
      return this.imgW / this.orginalInfo.orginalWidth;
    },
    fixedScale() {
      return 1 / this.scale;
    },
    maxMoveLeft() {
      return this.boxW - this.imgW * this.scale;
    },
    maxMoveTop() {
      return this.boxH - this.imgH * this.scale;
    }
  },
  mounted() {
    this.initImage();

    this.mousewheelevt = /Firefox/i.test(navigator.userAgent)
      ? "DOMMouseScroll"
      : "mousewheel";
    this.$refs.map.addEventListener(this.mousewheelevt, this.wheelHandle, {
      passive: true
    });
    // window.addEventListener('resize', this.resize);
    this.$once("hook:beforeDestroy", () => {
      // window.removeEventListener('resize', this.resize);
      this.$refs.map.removeEventListener(this.mousewheelevt, this.wheelHandle, {
        passive: true
      });
    });
  },
  methods: {
    /**
     * 获取图片的大小
     */
    getImgSize(url) {
      return new Promise((resolve, reject) => {
        let imgObj = new Image();
        imgObj.src = url;
        imgObj.onload = () => {
          resolve({
            width: imgObj.width,
            height: imgObj.height
          });
        };
        imgObj.onerror = () => {
          reject({});
        };
      });
    },

    /**
     * 初始化图片
     */
    initImage: _debounce(async function () {
      if (!this.imageUrl) {
        return;
      }
      this.loading = true;
      this.boxW = 0;
      this.boxH = 0;
      this.imgW = 0;
      this.imgH = 0;
      this.top = 0;
      this.left = 0;
      this.scale = 1;
      this.size = 0;
      let { width, height } = await this.getImgSize(this.imageUrl);
      if (!width || !height) {
        this.$emit("onload", {
          msg: "图片加载失败！",
          sucess: false
        });
        this.loading = false;
        return;
      }
      // 设置原始图片的大小
      this.orginalInfo = {
        orginalWidth: width,
        orginalHeight: height
      };
      this.resize();
      this.$emit("onload", {
        msg: "图片加载成功！",
        sucess: true
      });
    }),

    /**
     * 初始化渲染
     */
    resize: _debounce(function () {
      if (!this.imageUrl) {
        return;
      }
      this.boxW = 0;
      this.boxH = 0;
      this.imgW = 0;
      this.imgH = 0;
      this.top = 0;
      this.left = 0;
      this.scale = 1;
      this.size = 0;
      let { orginalWidth: width, orginalHeight: height } = this.orginalInfo;
      if (!width || !height) {
        return;
      }
      // 设置原始图片的大小
      let realWidth = width;
      let realHeight = height;
      // 获取高宽比例
      const whRatio = realWidth / realHeight;
      const hwRatio = realHeight / realWidth;
      //获取盒子的大小
      const boxW = this.$refs.panelBox.clientWidth;
      const boxH = this.$refs.panelBox.clientHeight;
      if (realWidth >= realHeight) {
        this.imgH = hwRatio * boxW;
        const nih = this.imgH;
        if (nih > boxH) {
          this.imgH = boxH;
          this.imgW = whRatio * boxH;
        } else {
          this.imgW = boxW;
        }
        this.top = (boxH - this.imgH) / 2;
        this.left = (boxW - this.imgW) / 2;
      } else {
        this.imgW = (boxH / realHeight) * realWidth;
        this.imgH = boxH;
        this.left = (boxW - this.imgW) / 2;
      }
      this.boxW = boxW;
      this.boxH = boxH;
      this.endPosition.x = this.left;
      this.endPosition.y = this.top;
      this.orginalInfo = {
        left: this.left,
        top: this.top,
        direction: realWidth >= realHeight ? "horizontal" : "vertical",
        orginalWidth: realWidth,
        orginalHeight: realHeight
      };
      this.loading = false;
    }),

    /**
     * 鼠标滚动 实现放大缩小
     */
    wheelHandle(e) {
      const ev = e || window.event; // 兼容性处理 => 火狐浏览器判断滚轮的方向是属性 detail，谷歌和ie浏览器判断滚轮滚动的方向是属性 wheelDelta
      const dir = ev.detail ? ev.detail * -120 : ev.wheelDelta;
      //滚动的数值 / 2000 => 表示滚动的比例，用此比例作为图片缩放的比例
      let zoom = dir / 2000;
      this.size += zoom;
      if (this.size < -0.5) {
        this.size = -0.5;
      }
      if (this.size > 4) {
        this.size = 4;
      }
      let lastScale = this.scale;
      let nexScale = 1 + this.size;
      let ox = e.offsetX;
      let oy = e.offsetY;
      if (e.target !== this.$refs.map) {
        let { x, y } = this.$refs.map.getBoundingClientRect();
        if (e.target.dataset["type"] == "fixed") {
          let cx = e.pageX + e.target.offsetWidth / 2 - ox;
          let cy = e.pageY + e.target.offsetHeight / 2 - oy;
          ox = (cx - x) * this.fixedScale;
          oy = (cy - y) * this.fixedScale;
        } else {
          ox = (e.pageX - x) / lastScale;
          oy = (e.pageY - y) / lastScale;
        }
      }
      let translateX = this.imgW * (ox / this.imgW) * (nexScale - lastScale);
      let translateY = this.imgH * (oy / this.imgH) * (nexScale - lastScale);
      this.changeXY((this.left -= translateX), (this.top -= translateY));
      this.endPosition.x = this.left;
      this.endPosition.y = this.top;
      this.scale = nexScale;
    },

    /**
     * 处理图片拖动
     */
    onmousedownHandle(e) {
      let dom = document;
      const that = this;
      let startPosition = { x: e.x, y: e.y };
      dom.onmousemove = function (el) {
        that.ismove = true;
        const ev = el || window.event; // 阻止默认事件
        let moveX = el.clientX - startPosition.x;
        let moveY = el.clientY - startPosition.y;
        ev.preventDefault();
        let updateX = moveX + that.endPosition.x;
        let updateY = moveY + that.endPosition.y;
        that.changeXY(updateX, updateY);
      };
      dom.onmouseup = function () {
        that.endPosition.x = that.left;
        that.endPosition.y = that.top;
        this.onmousemove = null;
        this.onmouseup = null;
        this.onmouseout = null;
        setTimeout(() => {
          if (that.ismove) that.ismove = false;
        }, 0);
      };
      if (e.preventDefault) {
        e.preventDefault();
      } else {
        return false;
      }
    },

    /**
     * 更新位置 (锁边逻辑暂时注释)
     */
    changeXY(x, y) {
      let updateX = x;
      let updateY = y;
      // if (this.maxMoveLeft > 0) {
      //   if (updateX > this.maxMoveLeft) {
      //     updateX = this.maxMoveLeft;
      //   }
      //   if (updateX < 0) {
      //     updateX = 0;
      //   }
      // } else if (this.maxMoveLeft < 0) {
      //   if (updateX < this.maxMoveLeft) {
      //     updateX = this.maxMoveLeft;
      //   }
      //   if (updateX > 0) {
      //     updateX = 0;
      //   }
      // } else {
      //   updateX = 0;
      // }
      // if (this.maxMoveTop > 0) {
      //   if (updateY > this.maxMoveTop) {
      //     updateY = this.maxMoveTop;
      //   }
      //   if (updateY < 0) {
      //     updateY = 0;
      //   }
      // } else if (this.maxMoveTop < 0) {
      //   if (updateY < this.maxMoveTop) {
      //     updateY = this.maxMoveTop;
      //   }
      //   if (updateY > 0) {
      //     updateY = 0;
      //   }
      // } else {
      //   updateY = 0;
      // }
      this.left = updateX;
      this.top = updateY;
    }
  }
};
</script>

<style lang="scss" scoped>
.panel-container {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;
  text-align: left;
  .img-container {
    z-index: 1;
  }
  .map-container {
    z-index: 2;
    svg {
      z-index: 2;
    }
    & > div,
    .fixed-icon {
      z-index: 3;
    }
  }
}
</style>
