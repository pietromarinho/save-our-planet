<template>
  <div id="app">
    <div class="header">
      <header class="text-center">
        <nav class="navbar navbar-expand-lg">
          <a class="navbar-brand text-white" href="#">TELLUS</a>
          <button
            class="navbar-toggler"
            type="button"
            data-toggle="collapse"
            data-target="#navbarText"
            aria-controls="navbarText"
            aria-expanded="false"
            aria-label="Toggle navigation"
          >
            <span class="navbar-toggler-icon"></span>
          </button>
          <div class="collapse navbar-collapse" id="navbarText">
            <ul class="navbar-nav mr-auto">
              <li class="nav-item active">
                <a class="nav-link text-white" href="#">
                  Globe
                  <span class="sr-only">(current)</span>
                </a>
              </li>
              <li class="nav-item">
                <a class="nav-link text-white" href="#">Graphics</a>
              </li>
              <li class="nav-item">
                <a class="nav-link text-white" href="#">Who we are?</a>
              </li>
            </ul>
            <span class="navbar-text">Register occurrence</span>
          </div>
        </nav>
      </header>
    </div>

    <div class="container pt-2">
      <div class="row content-center">
        <h6 class="text-center text-white">Select a layer to view</h6>
        <select name id v-model="layerSelect" @change="selectLayer($event)" class="form-control">
          <option value="1">Earth Surface Temperature (LST)</option>
          <option value="2">Vegetation Index</option>
        </select>
        <h5 class="text-white text-center pt-4">View occurrences around Planet Earth</h5>
      </div>
    </div>

    <div class="container-fluid container-nasa pt-2">
      <div class="globo">
        <canvas id="canvasOne" width="600" height="800">Your browser does not support HTML5 Canvas.</canvas>
      </div>

      <div class="card border-white" style="background-color: black" v-show="this.clickMap">
        <div class="card-header text-white" style="background-color: #7A04D4">
          <h4>Details</h4>
        </div>
        <img src="../public/imgs/desmatamento.png" class="card-img-top" alt="#" />
        <div class="card-body">
          <div v-for="infor in informations" :key="infor.id">
            <div class="card-titulo">
              <h1>Amazonas</h1>
            </div>
            <div class="info">
              <label>Current temperature</label>
              <h3>
                {{infor.temperature}}
                <hr />
              </h3>
            </div>

            <div class="info">
              <label>Temperature changes</label>
              <div v-for="inforTemp in infor.temperature_historic" :key="inforTemp.id">
                <h3>{{inforTemp.value}}° in {{inforTemp.data_create}}</h3>
              </div>
              <hr />
            </div>

            <div class="info">
              <label>Current Vegetation Rate</label>
              <h3>
                {{infor.vegetation_rate}}%
                <hr />
              </h3>
            </div>

            <div class="info">
              <label>Vegetation rate changes</label>
              <div v-for="inforVeg in infor.vegetation_rate_historic" :key="inforVeg.id">
                <h3>{{inforVeg.value}}% in {{inforVeg.data_create}}</h3>
              </div>
              <hr />
            </div>

            <div class="info">
              <label>Current population incidence rate</label>
              <h3>
                {{infor.population_incident}}%
                <hr />
              </h3>
            </div>

            <div class="info">
              <label>Changes in population incidence rate</label>
              <div v-for="inforInc in infor.population_incident_historic" :key="inforInc.id">
                <h3>{{inforInc.value}}% in {{inforInc.data_create}}</h3>
              </div>
              <hr />
            </div>

            <h3>Occurrences</h3>
            <div v-for="inforOcc in infor.occurrences" :key="inforOcc.id">
                <h3>{{inforOcc.description}} in {{inforOcc.data_create}}</h3>
              </div>
          </div>
        </div>
      </div>

      <div class="card-nasa" v-show="false"></div>
    </div>
  </div>
</template>

<script>
import worldwind from "@nasaworldwind/worldwind";
import $ from "jquery";
import axios from "axios";

export default {
  name: "app",
  components: {},
  data() {
    return {
      clickMap: false,
      wwd: null,
      layerSelect: null,
      informations: null
    };
  },

  beforeCreate() {},

  created() {
    this.getApi();
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

    this.markerDefault(this.wwd);

    this.clickMarker(this.wwd);
  },

  methods: {
    getApi() {
      axios
        .get(`https://api.jsonbin.io/b/5dabe6f645e66e5713fb45a8`)
        .then(response => {
          this.informations = response.data;
          console.log(JSON.stringify(this.informations));
        });
    },

    markerDefault(wwd) {
      var placemarkLayer = new WorldWind.RenderableLayer("Placemark");
      wwd.addLayer(placemarkLayer);
      var placemarkAttributes = new WorldWind.PlacemarkAttributes(null);

      placemarkAttributes.imageOffset = new WorldWind.Offset(
        WorldWind.OFFSET_FRACTION,
        0.3,
        WorldWind.OFFSET_FRACTION,
        0.0
      );

      placemarkAttributes.labelAttributes.color = WorldWind.Color.YELLOW;
      placemarkAttributes.labelAttributes.offset = new WorldWind.Offset(
        WorldWind.OFFSET_FRACTION,
        0.5,
        WorldWind.OFFSET_FRACTION,
        1.0
      );

      placemarkAttributes.imageSource = require("./assets/images/marker.png");

      var position = new WorldWind.Position(
        -3.092002,
        -59.945043000000005,
        100.0
      );
      var placemark = new WorldWind.Placemark(
        position,
        false,
        placemarkAttributes
      );

      placemark.label =
        "Amazonas\n" +
        "Lat " +
        placemark.position.latitude.toPrecision(4).toString() +
        "\n" +
        "Lon " +
        placemark.position.longitude.toPrecision(5).toString();
      placemark.alwaysOnTop = true;

      placemarkLayer.addRenderable(placemark);
    },

    clickMarker(wwd) {
      var vm = this;

      wwd.addEventListener("mousemove", function(event) {
        let clickRecognizer = new worldwind.ClickRecognizer(wwd, function(
          recognizer
        ) {
          let x = recognizer.clientX;
          let y = recognizer.clientY;

          let pickList = wwd.pick(wwd.canvasCoordinates(x, y));

          var mapPosition = pickList.objects[0].position;

          if (
            vm.distance(mapPosition.longitude, mapPosition.latitude) == false &&
            vm.clickMap == false
          ) {
            wwd.goTo(
              new WorldWind.Location(
                mapPosition.latitude,
                mapPosition.longitude
              )
            );
          } else if (vm.distance(mapPosition.longitude, mapPosition.latitude)) {
            wwd.goTo(
              new WorldWind.Location(
                mapPosition.latitude,
                mapPosition.longitude
              )
            );
          }

          if (vm.distance(mapPosition.longitude, mapPosition.latitude)) {
            vm.buildMarker(true);
          } else {
            vm.buildMarker(false);
          }

          vm.getApiTemperature(mapPosition.longitude, mapPosition.latitude);

          if (pickList.objects.length == 1 && pickList.objects[0].isTerrain) {
            var mapPosition = pickList.objects[0].position;
          }
        });
      });
    },

    eventTab(event) {
      e.preventDefault();
      $(this).tab("show");
    },

    buildMarker(valor) {
      this.clickMap = valor;
    },

    distance(logitude, latitude) {
      let distance = Math.acos(
        Math.cos(this.radians(-3.092002)) *
          Math.cos(this.radians(latitude)) *
          Math.cos(this.radians(-59.945043000000005) - this.radians(logitude)) +
          Math.sin(this.radians(-3.092002)) * Math.sin(this.radians(latitude))
      );

      if (distance <= 0.03118488184143756) {
        return true;
      }

      return false;
    },

    radians(value) {
      return value * (Math.PI / 180);
    },

    selectLayer(event) {
      var vm = this;
      var serviceAddress =
        "https://neo.sci.gsfc.nasa.gov/wms/wms?SERVICE=WMS&REQUEST=GetCapabilities&VERSION=1.3.0";
      var layer = this.layerSelect;

      if (layer == "1") {
        var layerName = "MOD_LSTD_M";

        $.get(serviceAddress)
          .done(function(xmlDom) {
            var wms = new worldwind.WmsCapabilities(xmlDom);
            var wmsLayerCapabilities = wms.getNamedLayer(layerName);
            var wmsConfig = WorldWind.WmsLayer.formLayerConfiguration(
              wmsLayerCapabilities
            );
            var wmsLayer = new worldwind.WmsLayer(wmsConfig);
            vm.wwd.addLayer(wmsLayer);
          })
          .fail(function() {});
      } else if (layer == "2") {
        var layerName = "MOD_NDVI_M";

        $.get(serviceAddress)
          .done(function(xmlDom) {
            var wms = new WorldWind.WmsCapabilities(xmlDom);
            var wmsLayerCapabilities = wms.getNamedLayer(layerName);
            var wmsConfig = WorldWind.WmsLayer.formLayerConfiguration(
              wmsLayerCapabilities
            );
            var wmsLayer = new WorldWind.WmsLayer(wmsConfig);
            vm.wwd.addLayer(wmsLayer);
          })
          .fail(function() {});
      } else {
        vm.wwd.addLayer(null);
      }
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
  background-color: #7a04d4;
  color: #ffffff;
}

.header * {
  margin: 0;
}

.container-fluid {
  margin-bottom: 0 !important;
}

.container-nasa {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  display: flex;
  /* align-items: center; */
  justify-content: space-between;
  margin-bottom: 20%;
}

.row {
  padding: 0 37%;
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

.card img {
  padding: 0;
}

.card hr {
  padding: 0;
  margin: 3px 0;
  border-color: #ffffff;
}

.card .card-titulo {
  text-align: center;
  background-color: #7a04d4;
}

.card-titulo * {
  margin: 0;
}

.card-header {
  background-color: #7a04d4;
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

.card-body {
  overflow-y: scroll;
  padding: 0 !important;
}
.card-body label {
  padding: 0 5px;
  margin: 0;
}

@media only screen and (max-width: 600px) {
  .container-fluid {
    flex-direction: column;
  }

  .row {
    padding: 5px 50px !important;
  }

  .row h5 {
    padding: 1rem !important;
  }

  .globo {
    max-width: 100%;
    width: 100%;
  }
  #canvasOne {
    max-width: 100%;
    width: 100%;
  }
  .card {
    width: 100%;
  }
}
</style>
