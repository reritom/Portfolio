<template>
  <div class="canvas-wrapper">
    <canvas id="canvas" :width="windowWidth" :height="windowHeight"></canvas>
  </div>
</template>

<script>
export default {
  name: 'CanvasFrame',
  data: function() {
    return {
      'windowWidth': window.innerWidth,
      'windowHeight': window.innerHeight
    }
  },
  computed: {
  },
  methods: {
    draw: function() {
      var canvas = document.getElementById("canvas");
      var context = canvas.getContext("2d");
      context.fillStyle = "red";
      context.font = "bold 100px Arial";
      context.fillText("Zibri", (this.windowWidth / 2) - 17, (this.windowHeight / 2) + 8);

      // Determine the pixels that have been added
      let dummyContext = this.createContext(canvas.width, canvas.height);
      const dummyContextOriginalPixels = this.getCanvasPixels(dummyContext);
      dummyContext.font = "bold 100px Arial";
      dummyContext.fillText("Zibri", (this.windowWidth / 2) - 17, (this.windowHeight / 2) + 8);
      const dummyContextModifiedPixels = this.getCanvasPixels(dummyContext);
      const changedPixels = this.getChangedPixels(dummyContextOriginalPixels, dummyContextModifiedPixels, canvas.width);
      for (let cp of changedPixels) {
        this.colourPixel(context, cp[0], cp[1], "rgb(1,1,1)");
      }

    },
    getCanvasPixels: function(context) {
      // Return a flat list containing all of the pixels [r,g,b,a]
      const imageData = context.getImageData(0, 0, context.canvas.width, context.canvas.height).data;
      let pixels = [];
      for (var i = 0; i < imageData.length; i+=4) {
          pixels.push([imageData[i], imageData[i+1], imageData[i+2], imageData[i+3]]);
        }
      return pixels
    },
    colourPixel(context, x, y, fillstyle) {
      context.fillStyle = fillstyle;
      context.fillRect(x, y, 1, 1);
    },
    getChangedPixels: function(originalPixels, modifiedPixels, canvasWidth) {
      // Return a flat list of [x, y] coordinates of changed pixels
      let changedPixels = [];
      for (let i = 0; i < originalPixels.length; i++) {
        if (!this.arraysAreEqual(originalPixels[i], modifiedPixels[i])) {
          //console.log(`${originalPixels[i]} doesnt match ${modifiedPixels[i]}`);
          changedPixels.push([i % canvasWidth, Math.floor(i / canvasWidth)]);
        }
      }
      console.log(`There are ${changedPixels.length} changed pixels, first at ${changedPixels[0]}`);
      return changedPixels
    },
    arraysAreEqual: function(array1, array2) {
      return array1.length === array2.length && array1.every((value, index) => value === array2[index])
    },
    createContext: function(width, height) {
      // Create an empty dummy canvas context
      var canvas = document.createElement('canvas');
      canvas.width = width;
      canvas.height = height;
      return canvas.getContext("2d");
    }
  },
  created() {
    window.addEventListener('resize', () => {
      //console.log(`resizing ${window.innerWidth} ${window.innerHeight}`);
      this.windowWidth = window.innerWidth;
      this.windowHeight = window.innerHeight;
      this.$nextTick(() => this.draw());
    })
  },
  mounted() {
    this.draw();
  }
}
</script>
