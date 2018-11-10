<template>
	<div id="slider"
		:style="{cursor:cursor}"
		@mouseover="displayCarrets()"
		@mouseleave="hideCarrets()"
		>
		<transition name="fade" mode="out-in">
			<img id="slide" :src="images[currentImg]" 
				width="styleWidth"
				:key="images[currentImg]"
				@click="nextImg()" />
		</transition>
		<div v-if="images.length>1">
			<font-awesome-icon icon="angle-left" class="icon icon-left" 
				:style="{top: carretTopPos, display: carretLeftDisplay}"
				@click="prevImg()">
			</font-awesome-icon>
			<font-awesome-icon icon="angle-right" class="icon icon-right"
				:style="{top: carretTopPos, display: carretRightDisplay}"
				@click="nextImg()">
			</font-awesome-icon>
		</div>
	</div>
</template>

<script>
import Hammer from "hammerjs";
import { library } from "@fortawesome/fontawesome-svg-core";
import { faAngleRight, faAngleLeft } from "@fortawesome/free-solid-svg-icons";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";

library.add(faAngleRight, faAngleLeft);

export default {
  props: {
    images: Array,
    width: {
      type: String,
      default: null
    },
    height: {
      type: String,
      default: null
    }
  },
  components: {
    FontAwesomeIcon
  },
  data() {
    return {
      currentImg: 0,
      carretTopPos: "0px",
      carretLeftDisplay: "none",
      carretRightDisplay: "none",
      styleWidth: "100%",
      styleHeight: "auto",
      cursor: "default"
    };
  },
  methods: {
    displayCarrets() {
      this.carretRightDisplay = "block";
      if (this.currentImg !== 0) {
        this.carretLeftDisplay = "block";
      }
    },
    hideCarrets() {
      this.carretRightDisplay = "none";
      this.carretLeftDisplay = "none";
    },
    nextImg() {
      let total = this.images.length;
      if (this.currentImg == total - 1) {
        this.currentImg = 0;
      } else if (this.currentImg < total - 1) {
        this.currentImg++;
      }
      this.displayCarrets();
    },
    prevImg() {
      let total = this.images.length;
      if (this.currentImg > 0) {
        this.currentImg--;
      } else if (this.currentImg == 0) {
        this.currentImg = total - 1;
      }
      this.displayCarrets();
    },
    init() {
      let i = 0;
      for (let src of this.images) {
        let img = new Image();
        let numHeight = null;
        img.onload = function() {
          numHeight = this.height;
        };
        // preload
        img.src = src;
        let self = this;
        function loaded() {
          // set height
          if (self.height !== null) {
            // set height if a value has been provided
            self.styleHeight = self.height;
          } else {
            // set height from the image
            self.styleHeight = numHeight.toString() + "px";
          }
          // set width
          if (self.width !== null) {
            // set width if a value has been provided
            // or keep the default "100%"
            self.styleWidth = self.width;
          }
          // set the carret position
          self.carretTopPos = "-" + Math.round(numHeight / 2).toString() + "px";
        }
        if (i === 0) {
          if (img.complete) {
            loaded();
          } else {
            img.addEventListener("load", loaded);
            img.addEventListener("error", function() {
              console.log("Error loading image");
            });
          }
        }
        i++;
      }
      if (this.images.length > 1) {
        this.cursor = "pointer";
      }
    }
  },
  created() {
    this.init();
  },
  mounted() {
    let slider = document.getElementById("slider");
    let mc = new Hammer(slider);
    let self = this;
    mc.on("swipeleft", function() {
      self.prevImg();
    });
    mc.on("swiperight", function() {
      self.nextImg();
    });
    document.onkeydown = function(evt) {
      evt = evt || window.event;
      switch (evt.keyCode) {
        case 37:
          self.prevImg();
          break;
        case 39:
          self.nextImg();
          break;
      }
    };
  }
};
</script>

<style scoped>
#slider {
  text-align: center;
  width: 100%;
}
#slide {
  max-width: 100%;
}
.fade-enter-active {
  transition: opacity 0.4s;
}
.fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}
.icon {
  font-size: 64pt;
  color: lightslategray;
  opacity: 0.4;
  position: relative;
}
.icon-left {
  float: left;
}
.icon-right {
  float: right;
}
</style>