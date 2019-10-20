<template>
  <div id="app">
    <div class="header">
      <header class="text-center">
        <nav class="navbar navbar-expand-lg">
          <a class="navbar-brand text-white" href="#">TELLUS</a>
          <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarText" aria-controls="navbarText" aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
          </button>
          <div class="collapse navbar-collapse" id="navbarText">
            <ul class="navbar-nav mr-auto">
              <li class="nav-item active">
                <a class="nav-link text-white" href="#">Globo<span class="sr-only">(current)</span></a>
              </li>
              <li class="nav-item">
                <a class="nav-link text-white" href="#">Gráficos</a>
              </li>
              <li class="nav-item">
                <a class="nav-link text-white" href="#">Quem somos?</a>
              </li>
            </ul>
            <span class="navbar-text">
              Cadastrar ocorrência
            </span>
          </div>
        </nav>
       
      </header>
    </div>

    <div class="container pt-2">
      <div class="row content-center">
        <div class="col-3"></div>
        <div class="col-6">
          <h6 class="text-center text-white">Selecione uma camada para visualização</h6>
          <select name id v-model="layerSelect" @change="selectLayer($event)" class="form-control">
            <option value="1">Temperatura da superfície da terra (LST)</option>
            <option value="2">Índice de vegetação</option>
          </select>
          <h5 class="text-white text-center pt-4">Visualize ocorrências ao redor do Planeta Terra</h5>
        </div>
        <div class="col-3"></div>
      </div>
    </div>
    

    <div class="container-fluid container-nasa pt-2">
      <div class="globo">
        <canvas id="canvasOne" width="600" height="800">Your browser does not support HTML5 Canvas.</canvas>
      </div>



      <div class="card border-white" style="background-color: black">
        <div class="card-header text-white" style="background-color: #7A04D4" >
          <h4>Detalhes</h4>
        </div>
        <img src="../public/imgs/desmatamento.png" class="card-img-top" alt="#">
        <div class="card-body">
           <h5 class="text-white">Informações</h5>
           <hr style="background-color:white;">
           <p class="text-white">
             Temperatura:<br>
             Índice de vegetação:<br>
             Data:
           </p>
           <h5 class="text-white">Ocorrências</h5>
           <hr style="background-color:white;">
        </div>
      </div>

      <div class="card-nasa" v-show="this.clickMap">
        <div v-for="infor in informations" :key="infor.id">
          <div class="card-titulo">
            <h1>{{infor.location_name}}</h1>
          </div>
          <div class="info">
            <label>Temperatura atual</label>
            <h3>
              {{infor.temperature}}
              <hr />
            </h3>
          </div>

          <div class="info">
            <label>Mudanças na temperatura</label>
            <div v-for="inforTemp in infor.temperature_historic" :key="inforTemp.id">
              <h3>{{inforTemp.value}}° em {{inforTemp.data_create}}</h3>
            </div>
            <hr />
          </div>

          <div class="info">
            <label>Taxa de vegetação atual</label>
            <h3>
              {{infor.vegetation_rate}}%
              <hr />
            </h3>
          </div>

          <div class="info">
            <label>Mudanças na taxa da vegetação</label>
            <div v-for="inforVeg in infor.vegetation_rate_historic" :key="inforVeg.id">
              <h3>{{inforVeg.value}}% em {{inforVeg.data_create}}</h3>
            </div>
            <hr />
          </div>

          <div class="info">
            <label>Taxa de incidência populacional atual</label>
            <h3>
              {{infor.population_incident}}%
              <hr />
            </h3>
          </div>

          <div class="info">
            <label>Mudanças na taxa de incidência populacional</label>
            <div v-for="inforInc in infor.population_incident_historic" :key="inforInc.id">
              <h3>{{inforInc.value}}% em {{inforInc.data_create}}</h3>
            </div>
            <hr />
          </div>

          <h3>Ocorrencias</h3>
          <ul>
            <li>EX1</li>
          </ul>
        </div>
      </div>
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

    this.markerDefault(this.wwd)

    this.clickMarker(this.wwd)
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

      var position = new WorldWind.Position(-3.092002, -59.945043000000005, 100.0);
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

          if (pickList.objects.length == 1 && pickList.objects[0].isTerrain) {
            var mapPosition = pickList.objects[0].position;

            wwd.goTo(
              new WorldWind.Location(
                mapPosition.latitude,
                mapPosition.longitude
              )
            )

            vm.buildMarker();
          }
        });
      });
    },

    eventTab(event){
     
      e.preventDefault()
      $(this).tab('show')
 
    },

    buildMarker() {
      this.clickMap = !this.clickMap;
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
  background-color: #7A04D4;
  color: #ffffff;
}

.header * {
  margin: 0;
}

.container-nasa {
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

.card-nasa {
  max-height: calc(100vh - 30px);
  width: calc(100vw - 70%);
  background-color: transparent;
  margin-right: 10px;
  overflow-y: auto;
  border-style: solid;
  border-width: 3px;
  border-color: #a2a2a2f8;
}

.card-nasa * {
  color: #ffffff;
  padding: 10px 5px;
}

.card-nasa hr {
  padding: 0;
  margin: 3px 0;
}

.card-nasa .card-titulo {
  text-align: center;
  background-color: #7A04D4;
}

.card-titulo * {
  margin: 0;
}

.card{
  width: 25rem;
}

.card-header{
  background-color:#7A04D4;
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
