<template>
  <div style="text-align:center">
    <video id="video" width="640" height="480" autoplay hidden></video>

    <span v-bind:style="{ 'background-color': textColor }">Color Select For replace</span>

    <div id="color-picker" >
      <div class="wrapper-dropdown">
        <span @click="toggleDropdown()" v-html="selector"></span>
        <ul class="dropdown" v-show="active">
          <li v-for="color in colors" @click="setColor(color.hex, color.name)" :key="color.hex">
            <span :style="{background: color.hex}"></span>
            {{color.name}}
          </li>
        </ul>
      </div>
    </div>
    <br />
    <canvas id="canvas" ref="canvas" width="640" height="480" @click="clickCanvas"></canvas>
  </div>
</template>

<script>
export default {
  data() {
    return {
      IM: "",
      textColor: "#b66c8e",
      oldRGBRed: 90,
      oldRGBGreen: 90,
      oldRGBBlue: 90,
      newRGBRed: 255,
      newRGBGreen: 0,
      newRGBBlue: 0,
      rangPixcel: 40,
      active: false,
      selectedColor: "",
      selectedColorName: "",
      colors: [
        {
          hex: "#FF2400",
          name: "Scarlet"
        },
        {
          hex: "#FFA07A",
          name: "Light Salmon"
        },
        {
          hex: "#CD5C5C",
          name: "Indian Red"
        },
        {
          hex: "#DC143C",
          name: "Crimson"
        },
        {
          hex: "#20B2AA",
          name: "Light Sea Green"
        },
        {
          hex: "#008B8B",
          name: "Dark Cyan"
        },
        {
          hex: "#4682B4",
          name: "Steel Blue"
        }
      ]
    };
  },
  methods: {
    showIM: function() {
      // Grab elements, create settings, etc.
      let video = document.getElementById("video");
      let context = this.$refs.canvas.getContext("2d");
      // Get access to the camera!
      if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
        // Not adding `{ audio: true }` since we only want video now
        navigator.mediaDevices
          .getUserMedia({ video: true })
          .then(function(stream) {
            //video.src = window.URL.createObjectURL(stream);
            video.srcObject = stream;
            video.play();
            //this.drawX(video, this.context, 640, 480);
          });
      }
      // context.drawImage(video, 0, 0, w, h);
      let self = this;
      video.addEventListener(
        "play",
        function() {
          self.drawX(this, context, 640, 480);

          //context.drawImage(video, 0, 0, width, height);
          //setTimeout(this.drawX, 10, width, height);
        },
        false
      );
      //let context = canvas.getContext('2d');
    },
    drawX: function(video, context, width, height) {
      let self = this;
      context.drawImage(video, 0, 0, width, height);

      let w = 640;
      let h = 480;
      //   let oldRed = 74;
      //   let oldGreen = 74;
      //   let oldBlue = 74;

      //   let newRed = 255;
      //   let newGreen = 0;
      //   let newBlue = 0;
      context.drawImage(video, 0, 0, w, h);

      // pull the entire image into an array of pixel data
      let imageData = context.getImageData(0, 0, w, h);
      //context.fillStyle = "#FF0000";
      //context.fillRect(20, 20, w, h);

      // examine every pixel,
      // change any old rgb to the new-rgb
      for (var i = 0; i < imageData.data.length; i += 4) {
        // is this pixel the old rgb?
        if (
          imageData.data[i] >= this.oldRGBRed - this.rangPixcel &&
          imageData.data[i] <= this.oldRGBRed + this.rangPixcel &&
          (imageData.data[i + 1] >= this.oldRGBGreen - this.rangPixcel &&
            imageData.data[i + 1] <= this.oldRGBGreen + this.rangPixcel) &&
          (imageData.data[i + 2] >= this.oldRGBBlue - this.rangPixcel &&
            imageData.data[i + 2] <= this.oldRGBBlue + this.rangPixcel)
          //   imageData.data[i] <= this.oldRGBRed &&
          //   imageData.data[i + 1] <= this.oldRGBGreen &&
          //   imageData.data[i + 2] <= this.oldRGBBlue
        ) {
          // change to your new rgb
          imageData.data[i] = this.newRGBRed;
          imageData.data[i + 1] = this.newRGBGreen;
          imageData.data[i + 2] = this.newRGBBlue;
        }
      }
      context.putImageData(imageData, 0, 0);
      setTimeout(self.drawX, 10, video, context, width, height);
    },
    rgbToHex: function(r, g, b) {
      if (r > 255 || g > 255 || b > 255) throw "Invalid color component";
      return ((r << 16) | (g << 8) | b).toString(16);
    },
    hexToRgb: function(hex) {
      // Expand shorthand form (e.g. "03F") to full form (e.g. "0033FF")
      var shorthandRegex = /^#?([a-f\d])([a-f\d])([a-f\d])$/i;
      hex = hex.replace(shorthandRegex, function(m, r, g, b) {
        return r + r + g + g + b + b;
      });

      var result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
      return result
        ? {
            r: parseInt(result[1], 16),
            g: parseInt(result[2], 16),
            b: parseInt(result[3], 16)
          }
        : null;
    },

    clickCanvas: function(event) {
      console.log(event.clientX);
      console.log(event.clientY);

      let context = this.$refs.canvas.getContext("2d");
      var p = context.getImageData(event.clientX, event.clientY, 1, 1).data;
      var hex = "#" + ("000000" + this.rgbToHex(p[0], p[1], p[2])).slice(-6);

      this.oldRGBRed = p[0];
      this.oldRGBGreen = p[1];
      this.oldRGBBlue = p[2];

      //   this.newRGBRed = 255;
      //   this.newRGBGreen = 0;
      //   this.newRGBBlue = 0;

      this.textColor = hex;
      console.log(hex);
    },
    setColor: function(color, colorName) {
      this.selectedColor = color;
      this.selectedColorName = colorName;
      let cr = this.hexToRgb(color);
      this.newRGBRed = cr.r;
      this.newRGBGreen = cr.g;
      this.newRGBBlue = cr.b;

      console.log(this.hexToRgb(color));
      this.active = false;
    },
    toggleDropdown: function() {
      this.active = !this.active;
    }
  },
  mounted: function() {
    this.showIM();
  },
  computed: {
    selector: function() {
      if (!this.selectedColor) {
        return "Color";
      } else {
        return (
          '<span style="background: ' +
          this.selectedColor +
          '"></span> ' +
          this.selectedColorName
        );
      }
    }
  }
};
</script>

<style >
* {
  box-sizing: border-box;
  font-family: "Arial";
}
.wrapper-dropdown {
  position: relative;
  width: 200px;
  background: #fff;
  color: #2e2e2e;
  outline: none;
  cursor: pointer;
}
.wrapper-dropdown > span {
  width: 100%;
  display: block;
  border: 1px solid #ababab;
  padding: 5px;
}
.wrapper-dropdown > span > span {
  padding: 0 12px;
  margin-right: 5px;
}
.wrapper-dropdown > span:after {
  content: "";
  width: 0;
  height: 0;
  position: absolute;
  right: 16px;
  top: calc(50% + 4px);
  margin-top: -6px;
  border-width: 6px 6px 0 6px;
  border-style: solid;
  border-color: #2e2e2e transparent;
}

.wrapper-dropdown .dropdown {
  position: absolute;
  z-index: 10;
  top: 100%;
  left: 0;
  right: 0;
  background: #fff;
  font-weight: normal;
  list-style-type: none;
  padding-left: 0;
  margin: 0;
  border: 1px solid #ababab;
  border-top: 0;
}

.wrapper-dropdown .dropdown li {
  display: block;
  text-decoration: none;
  color: #2e2e2e;
  padding: 5px;
  cursor: pointer;
}

.wrapper-dropdown .dropdown li > span {
  padding: 0 12px;
  margin-right: 5px;
}

.wrapper-dropdown .dropdown li:hover {
  background: #eee;
  cursor: pointer;
}
</style>