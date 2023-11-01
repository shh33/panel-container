<template>
  <div id="app">
    <PanelContainer ref="panel" :imageUrl="imageUrl" @onload="onloadSucess">
      <template v-if="flag">
        <div
          class="fixed-icon"
          :style="{
            position: 'absolute',
            width: '30px',
            height: '30px',
            top: '41.4%',
            left: '57.3%',
            background: 'rgba(0,0,0,.5)',
            transform: 'translate(-50%, -50%) scale(' + fixedScale + ')'
          }"
          @click="ddd('以图片背景鼠标对应点，为缩放位置')"
        ></div>
        <div
          class="fixed-icon"
          data-type="fixed"
          :style="{
            position: 'absolute',
            width: '30px',
            height: '30px',
            top: '38.7%',
            left: '78.8%',
            background: 'rgba(0,0,0,.5)',
            transform: 'translate(-50%, -50%) scale(' + fixedScale + ')'
          }"
          @click="ddd('以固定像素盒子的中心，为缩放位置')"
        ></div>
        <svg
          xmlns="http://www.w3.org/2000/svg"
          version="1.1"
          :style="{
            position: 'relative',
            width: '100%',
            height: '100%'
          }"
        >
          <g
            v-for="item in svgPoints"
            :style="{
              transform: 'scale(' + originalScale + ')'
            }"
            :key="item.name"
          >
            <polygon
              class="polygon"
              :points="item.coordinates"
              @click="ddd(item.name)"
            />
          </g>
        </svg>
      </template>
    </PanelContainer>
  </div>
</template>

<script>
import PanelContainer from "./components/PanelContainer.vue";
export default {
  name: "App",
  data() {
    return {
      imageUrl: require("./assets/bg.jpeg"),
      svgPoints: [
        {
          coordinates: [
            "1790.9931740614334,2978.5166240409208",
            "2698,2840.4092071611253",
            "2808.4982935153585,3438.8746803069052",
            "1892.283276450512,3609.2071611253195"
          ],
          name: "1号楼"
        },
        {
          coordinates: [
            "943.8395904436861,1325.831202046036",
            "1349,1312.0204603580564",
            "1376.6245733788396,1896.6751918158568",
            "994.4846416382252,1924.296675191816"
          ],
          name: "2号楼"
        }
      ],
      flag: false
    };
  },
  components: {
    PanelContainer
  },
  computed: {
    fixedScale() {
      return this.$refs?.panel?.fixedScale;
    },
    originalScale() {
      return this.$refs?.panel?.originalScale;
    },
    orginalHeight() {
      return this.$refs?.panel?.orginalInfo?.orginalHeight;
    },
    orginalWidth() {
      return this.$refs?.panel?.orginalInfo?.orginalWidth;
    },
    ismove() {
      return this.$refs?.panel?.ismove;
    }
  },
  methods: {
    ddd(num) {
      if (this.ismove) return;
      alert(num);
    },
    onloadSucess(data) {
      this.flag = data.sucess;
    }
  }
};
</script>

<style>
html,
body,
#app {
  position: relative;
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  box-sizing: border-box;
}
div {
  box-sizing: border-box;
}
</style>
