<template>
  <div id="app">
    <div class="header">
      <header>
        <h3>Nome da aplicação</h3>
      </header>
    </div>

    <div class="container">
      <div class="globo">
        <canvas id="canvasOne" width="600" height="800">Your browser does not support HTML5 Canvas.</canvas>
      </div>

      <div class="card" v-show="this.clickMap">
        <div class="card-titulo">
          <h1>Titulo</h1>
        </div>
        <div class="info">
          <label>Temperatura atual</label>
          <h3>
            teste
            <hr />
          </h3>
        </div>

        <div class="info">
          <label>Mudanças na temperatura</label>
          <h3>30° em 2010</h3>
          <h3>30° em 2010</h3>
          <h3>30° em 2010</h3>
          <h3>30° em 2010</h3>
          <hr />
        </div>

        <div class="info">
          <label>Taxa de vegetação atual</label>
          <h3>
            teste
            <hr />
          </h3>
        </div>

        <div class="info">
          <label>Mudanças na taxa da vegetação</label>
          <h3>30% em 2010</h3>
          <h3>30% em 2010</h3>
          <h3>30% em 2010</h3>
          <h3>30% em 2010</h3>
          <hr />
        </div>

        <div class="info">
          <label>Taxa de incidência populacional atual</label>
          <h3>
            teste
            <hr />
          </h3>
        </div>

        <div class="info">
          <label>Mudanças na taxa de incidência populacional</label>
          <h3>30% em 2010</h3>
          <h3>30% em 2010</h3>
          <h3>30% em 2010</h3>
          <h3>30% em 2010</h3>
          <hr />
        </div>

        <h3>Ocorrencias</h3>
        <ul>
          <li>EX1</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import worldwind from "@nasaworldwind/worldwind";

export default {
  name: "app",
  components: {},
  data() {
    return {
      clickMap: false,
      wwd: null
    };
  },

  created() {},

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
              100.0
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

            wwd.goTo(
              new WorldWind.Location(
                mapPosition.latitude,
                mapPosition.longitude
              )
            );
            vm.buildMarker();
          }
        });
      });
    },

    buildMarker() {
      this.clickMap = !this.clickMap;
    }
  }
};
</script>

<style>
#app {
  background-color: #000;
  max-height: 100vh;
}

.header {
  background-color: #7f66d4;
  color: #ffffff;
}

.header * {
  margin: 0;
}

.container {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.globo {
  width: calc(100vw - 30%);
}

#canvasOne {
  max-width: calc(100vw - 45%);
  width: calc(100vw - 46%);
  max-height: calc(100vh - 30px);
}

.card {
  max-height: calc(100vh - 30px);
  width: calc(100vw - 70%);
  background-color: transparent;
  margin-right: 10px;
  overflow-y: auto;
  border-style: solid;
  border-width: 3px;
  border-color: #a2a2a2f8;
}

.card * {
  color: #ffffff;
  padding: 10px 5px;
}

.card hr {
  padding: 0;
  margin: 3px 0;
}

.card .card-titulo {
  text-align: center;
  background-color: #7f66d4;
}

.card-titulo * {
  margin: 0;
}

h3 {
  margin: 0;
  padding: 0px 8px;
}

ul {
  margin: 0;
  padding: 0 5px !important;
}

ul *li {
  padding: 0 5px;
}
</style>
