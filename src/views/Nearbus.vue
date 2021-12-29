<template>
  <div id="nearbus">
    <Nav />
    <div class="p-4">
      <span class="fs-2">附近的公車站牌</span>
      <div class="row">
        <div class="col-sm-12 col-md-6 p-0 m-0">
          <div id="map" ref="map" style="width: 100%; height: 450px"></div>
          <div id="popup" ref="popup"></div>
        </div>
        <div class="col-sm-12 col-md-6" style="overflow: auto; height: 450px">
          <div
            class="col-sm-12 col-md-10"
            v-for="item in route"
            :key="item.StopUID"
          >
            <div class="card mb-2 route__card">
              <div class="card-body route__card__body">
                <span>
                  <svg
                    width="25"
                    height="25"
                    viewBox="0 0 12 13"
                    fill="none"
                    xmlns="http://www.w3.org/2000/svg"
                  >
                    <path
                      d="M0 2.5C0 1.39543 0.895431 0.5 2 0.5H6C9.31371 0.5 12 3.18629 12 6.5C12 9.81371 9.31371 12.5 6 12.5H2C0.895431 12.5 0 11.6046 0 10.5V2.5Z"
                      fill="#61A68A"
                    />
                  </svg>

                  {{ item.StationID }} -
                  {{ item.StationName.Zh_tw }}
                </span>
                <button
                  class="btn dark__btn btn-sm"
                  @click="
                    flyto([
                      item.StationPosition.PositionLon,
                      item.StationPosition.PositionLat,
                    ])
                  "
                >
                  前往站牌
                </button>
                <p>
                  <!-- {{ item.txt }} -->
                </p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <Footer />
  </div>
</template>

<script>
import Nav from "@/components/Nav.vue";
import Footer from "@/components/Footer.vue";
import axios from "axios";
import { Map, View, Feature } from "ol";
import { OSM, Vector as VectorSource } from "ol/source";
import { Vector as VectorLayer, Tile as TileLayer } from "ol/layer";
import Overlay from "ol/Overlay";
import Point from "ol/geom/Point";
import { Icon, Style } from "ol/style";
import "ol/ol.css";
export default {
  components: {
    Nav,
    Footer,
  },
  data() {
    return {
      position: "雙北市",
      route: [],
      iconFeatures: [],
      map: {},
      layer: {},
    };
  },
  mounted() {
    const vm = this;
    const AppID = "9a683d2b67d1404c9af3b36c096a1518";
    const AppKey = "U7RkTz42_KNQ-A1LDCsdiH6u6Co";
    var GMTString = new Date().toGMTString();
    var ShaObj = new window.jsSHA("SHA-1", "TEXT");
    ShaObj.setHMACKey(AppKey, "TEXT");
    ShaObj.update("x-date: " + GMTString);
    var HMAC = ShaObj.getHMAC("B64");
    if ("geolocation" in navigator) {
      navigator.geolocation.getCurrentPosition(function (position) {
        vm.initMap();
        let lng = position.coords.longitude;
        let lat = position.coords.latitude;
        axios({
          method: "get",
          url: `https://ptx.transportdata.tw/MOTC/v2/Bus/Station/NearBy`,
          headers: {
            Authorization:
              'hmac username="' +
              AppID +
              '", algorithm="hmac-sha1", headers="x-date", signature="' +
              HMAC +
              '"',
            "X-Date": GMTString,
          },
          params: {
            $top: 20,
            $spatialFilter: `nearby(${lat},${lng}, 500)`,
          },
        })
          .then((response) => {
            vm.route = response.data;
            vm.view.setZoom(15);
            vm.view.setCenter([lng, lat]);
            response.data.forEach((item) => {
              const iconFeature = new Feature({
                geometry: new Point([
                  item.StationPosition.PositionLon,
                  item.StationPosition.PositionLat,
                ]),
                name: "Marker",
                population: 4000,
                rainfall: 500,
                data: {
                  ...item,
                },
              });
              const iconStyle = new Style({
                image: new Icon({
                  anchor: [0.5, 46],
                  anchorXUnits: "fraction",
                  anchorYUnits: "pixels",
                  src: "https://openlayers.org/en/latest/examples/data/icon.png",
                }),
              });
              iconFeature.setStyle(iconStyle);

              let vectorSource = new VectorSource({
                features: [iconFeature],
              });

              let markerVectorLayer = new VectorLayer({
                source: vectorSource,
              });

              vm.map.addLayer(markerVectorLayer);
            });
          })
          .catch((error) => console.log(error));
      }, vm.error);
    }
    //GET請求
  },
  methods: {
    initMap() {
      const vm = this;

      vm.view = new View({
        projection: "EPSG:4326",
        center: [121, 23.5],
        zoom: 11,
      });
      vm.map = new Map({
        target: "map",
        layers: [
          new TileLayer({
            source: new OSM()
          }),
        ],
        view: vm.view,
      });

      vm.popel = vm.$refs.popup;
      vm.popelement = new window.bootstrap.Popover(vm.popel, {
        placement: "top",
        html: true,
        content: "",
      });
      const popup = new Overlay({
        element: vm.popel,
        positioning: "bottom-center",
        stopEvent: false,
      });
      vm.map.addOverlay(popup);
      // vm.popelement.show();
      // display popup on click
      vm.map.on("click", function (evt) {
        const feature = vm.map.forEachFeatureAtPixel(
          evt.pixel,
          function (feature) {
            return feature;
          }
        );

        if (feature) {
          popup.setPosition(evt.coordinate);
          vm.popelement._config.content = `${feature.values_.data.StationName.Zh_tw}`;
          vm.popelement.setContent();
          vm.popelement.show();
        } else {
          vm.popelement.hide();
        }
      });

      // Close the popup when the map is moved
      vm.map.on("movestart", function () {
        vm.popelement.hide();
      });
    },
    flyto(location) {
      const vm = this;
      const duration = 2000;
      const zoom = vm.view.getZoom();
      let parts = 2;
      let called = false;
      function callback(complete) {
        --parts;
        if (called) {
          return;
        }
        if (parts === 0 || !complete) {
          called = true;
          console.log("ok");
        }
      }
      vm.view.animate(
        {
          center: location,
          duration: duration,
        },
        callback
      );
      vm.view.animate(
        {
          zoom: zoom - 1,
          duration: duration / 2,
        },
        {
          zoom: zoom + 3,
          duration: duration / 2,
        },
        callback
      );
    },
    error() {
      console.log("請接受取得位址 謝謝！");
    },
  },
  beforeDestroy(){
    const vm = this;
    vm.popelement.dispose();
    console.log(vm)
  }
};
</script>

<style scoped>
.google__img {
  width: 500px;
  height: 500px;
}
#map {
  height: 100%;
  width: 100%;
  margin: 0;
}
.dark__btn {
  background-color: #b96c58;
  color: white;
}

@media only screen and (min-width: 0px) and (max-width: 540px) {
  .google__img {
    width: 100%;
    height: auto !important;
  }
}
</style>