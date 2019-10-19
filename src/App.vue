<template>
  <div id="app">
    <div class="globo">
      <canvas id="canvasOne" width="600" height="800">Your browser does not support HTML5 Canvas.</canvas>
    </div>

    <div class="card" v-show="this.clickMap">teste</div>
  </div>
</template>

<script>
import worldwind from "@nasaworldwind/worldwind"

export default {
  name: "app",
  components: {},
  data() {
    return {
      clickMap: false,
      wwd: null
    };
  },

  created() {
    
  },

  mounted() {
    this.wwd = new worldwind.WorldWindow("canvasOne");

    var layers = [
      // Imagery layers.
      { layer: new WorldWind.BMNGOneImageLayer(), enabled: true },
      { layer: new WorldWind.BMNGLandsatLayer(), enabled: true },
      { layer: new WorldWind.BMNGLayer(), enabled: true },
      { layer: new WorldWind.BingAerialWithLabelsLayer(null), enabled: true },
      // Add atmosphere layer on top of all base layers.
      { layer: new WorldWind.AtmosphereLayer(), enabled: true },
      // WorldWindow UI layers.
      { layer: new WorldWind.CompassLayer(), enabled: true },
      { layer: new WorldWind.CoordinatesDisplayLayer(this.wwd), enabled: true },
      { layer: new WorldWind.ViewControlsLayer(this.wwd), enabled: true }
    ];

    for (var l = 0; l < layers.length; l++) {
      layers[l].layer.enabled = layers[l].enabled;
      this.wwd.addLayer(layers[l].layer);
    }

    this.clickMarker(this.wwd);
  },

  methods: {
    clickMarker(wwd) {

      var vm = this;

      wwd.addEventListener("mousemove", function(event) {
        let clickRecognizer = new worldwind.ClickRecognizer(wwd, function(
          recognizer
        ) {
          let x = recognizer.clientX;
          let y = recognizer.clientY;

          let pickList = wwd.pick(wwd.canvasCoordinates(x, y));

          if (pickList.objects.length == 1 && pickList.objects[0].isTerrain) {
            var mapPosition = pickList.objects[0].position;

            var placemarkLayer = new worldwind.RenderableLayer("Placemark");
            wwd.addLayer(placemarkLayer);

            var placemarkAttributes = new worldwind.PlacemarkAttributes(null);

            placemarkAttributes.imageOffset = new worldwind.Offset(
              WorldWind.OFFSET_FRACTION,
              0.3,
              WorldWind.OFFSET_FRACTION,
              0.0
            );

            placemarkAttributes.labelAttributes.color = WorldWind.Color.YELLOW;
            placemarkAttributes.labelAttributes.offset = new worldwind.Offset(
              WorldWind.OFFSET_FRACTION,
              0.5,
              WorldWind.OFFSET_FRACTION,
              1.0
            );

            placemarkAttributes.imageSource =
              WorldWind.configuration.baseUrl + "images/pushpins/plain-red.png";

            var position = new worldwind.Position(
              mapPosition.latitude,
              mapPosition.longitude,
              50.0
            );
            var placemark = new worldwind.Placemark(
              position,
              false,
              placemarkAttributes
            );

            placemark.label =
              "+\n" +
              "Lat " +
              placemark.position.latitude.toPrecision(4).toString() +
              "\n" +
              "Lon " +
              placemark.position.longitude.toPrecision(5).toString();
            placemark.alwaysOnTop = true;

            placemarkLayer.addRenderable(placemark);

            wwd.goTo(new WorldWind.Location(mapPosition.latitude, mapPosition.longitude));
            vm.buildMarker();

          }
        });
      });
    },

    buildMarker() {
      this.clickMap = !this.clickMap    
    }
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.globo {
  width: calc(100vw - 30%);
}

.card {
  background-color: #000;
  height: 100vh;
  width: calc(100vw - 70%);
}
</style>
