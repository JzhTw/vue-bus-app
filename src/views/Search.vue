<template>
  <div id="search">
    <Nav />
    <div class="p-4">
      <span class="fs-2">查詢{{ position_zhtw }}的公車</span>
      <div class="row g-0" style="height: 500px; overflow: auto">
        <div class="col-sm-12 col-md-10">
          <form>
            <div class="input-group mb-3">
              <input
                type="text"
                class="form-control"
                placeholder="請輸入號碼(ex:242 or 不輸入->全部搜尋)"
                aria-label="Recipient's username"
                aria-describedby="button-addon2"
                v-model="route_name"
                @keyup.enter="searchdata"
              />
              <button
                class="btn btn-secondary"
                type="submit"
                @click="searchdata"
              >
                搜尋
              </button>
            </div>
          </form>
          <!-- icon -->
          <div>
            <svg
              width="20"
              height="20"
              viewBox="0 0 12 13"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                d="M0 2.5C0 1.39543 0.895431 0.5 2 0.5H6C9.31371 0.5 12 3.18629 12 6.5C12 9.81371 9.31371 12.5 6 12.5H2C0.895431 12.5 0 11.6046 0 10.5V2.5Z"
                fill="#61A68A"
              />
            </svg>
            <span class="fs-4">查詢結果({{ position_zhtw }})</span>
          </div>
          <!-- icon-end -->
          <!-- Search  -->
          <div class="row">
            <!-- V-for -->
            <div
              class="col-sm-12 col-md-4"
              v-for="item in route_data"
              :key="item.RouteUID"
            >
              <div class="card mb-2 route__card">
                <div class="card-body">
                  <div class="route__card__body">
                    <p>
                      {{ item.RouteName.Zh_tw }}
                      <br />
                      {{ item.DepartureStopNameZh }} -
                      {{ item.DestinationStopNameZh }}
                    </p>
                    <div
                      class="route__card__icon"
                      style="display: flex; cursor: pointer"
                      @click="favorite(item.RouteUID, false)"
                    >
                      <i
                        class="fa fa-heart"
                        :style="{ color: item.type ? '#f3166e' : '' }"
                        aria-hidden="true"
                      ></i>
                    </div>
                  </div>
                  <div class="float-end clearfix">
                    <button
                      class="btn btn-primary btn-sm"
                      @click="searchdata_list(item.RouteName.Zh_tw)"
                    >
                      詳細資料
                    </button>
                  </div>
                </div>
              </div>
            </div>
            <!-- End For -->
          </div>

          <!-- Search End -->
          <!-- icon -->
          <div>
            <svg
              width="20"
              height="20"
              viewBox="0 0 12 13"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                d="M0 2.5C0 1.39543 0.895431 0.5 2 0.5H6C9.31371 0.5 12 3.18629 12 6.5C12 9.81371 9.31371 12.5 6 12.5H2C0.895431 12.5 0 11.6046 0 10.5V2.5Z"
                fill="#61A68A"
              />
            </svg>
            <span class="fs-4">在{{ position_zhtw }}已儲存的路線</span>
          </div>
          <!-- icon-end -->

          <div class="row">
            <!-- V-for -->
            <div
              class="col-sm-12 col-md-4"
              v-for="item in local_fv_route_data[0].fav_route"
              :key="item.RouteUID"
            >
              <div class="card mb-2 route__card">
                <div class="card-body">
                  <div class="route__card__body">
                    <p>
                      {{ item.RouteName.Zh_tw }}
                      <br />
                      {{ item.DepartureStopNameZh }} -
                      {{ item.DestinationStopNameZh }}
                    </p>
                    <div
                      class="route__card__icon"
                      style="display: flex; cursor: pointer"
                    >
                      <i
                        class="fa fa-heart"
                        :style="{ color: item.type ? '#f3166e' : '' }"
                        aria-hidden="true"
                        @click="favorite(item.RouteUID, true)"
                      ></i>
                    </div>
                  </div>
                  <div class="float-end clearfix">
                    <button
                      class="btn btn-primary btn-sm"
                      @click="searchdata_list(item.RouteName.Zh_tw)"
                    >
                      詳細資料
                    </button>
                  </div>
                </div>
              </div>
            </div>

            <!-- End For -->
          </div>
        </div>
      </div>
    </div>
    <DirectionList :route_data="route_sub_data" :route_nm="route_sub_name" />
    <Footer />
  </div>
</template>

<script>
import Nav from "@/components/Nav.vue";
import Footer from "@/components/Footer.vue";
import DirectionList from "@/components/DirectionList.vue";

import axios from "axios";
export default {
  components: {
    Nav,
    Footer,
    DirectionList,
  },
  data() {
    return {
      position: "",
      position_zhtw: "",
      route_name: "",
      route_data: [],
      local_fv_route_data: [],
      route_sub_data: [],
      route_sub_name: ["", ""],
    };
  },
  created() {
    const vm = this;
    vm.position = vm.$route.params.position;
    // Check city
    switch (vm.position) {
      case "Taipei":
        vm.position_zhtw = "台北市";
        break;
      case "NewTaipei":
        vm.position_zhtw = "新北市";
        break;
      case "Taoyuan":
        vm.position_zhtw = "桃園市";
        break;
      case "Taichung":
        vm.position_zhtw = "台中市";
        break;
      case "Tainan":
        vm.position_zhtw = "台南市";
        break;
      case "Kaohsiung":
        vm.position_zhtw = "高雄市";
        break;
      default:
        break;
    }
    // Check fav
    if (localStorage.getItem("fv_route_data") !== null) {
      let tmp_route_data = JSON.parse(localStorage.getItem("fv_route_data"));
      vm.local_fv_route_data = tmp_route_data.filter((item) => {
        return item.City === vm.position;
      });
      if (vm.local_fv_route_data.length === 0) {
        vm.local_fv_route_data.push({
          City: vm.position,
          fav_route: [],
        });
        tmp_route_data.push({
          City: vm.position,
          fav_route: [],
        });
        localStorage.setItem("fv_route_data", JSON.stringify(tmp_route_data));
      }
    } else {
      let tmp_route_data = [
        {
          City: vm.position,
          fav_route: [],
        },
      ];
      localStorage.setItem("fv_route_data", JSON.stringify(tmp_route_data));
    }
  },
  methods: {
    searchdata() {
      const vm = this;
      const AppID = "9a683d2b67d1404c9af3b36c096a1518";
      const AppKey = "U7RkTz42_KNQ-A1LDCsdiH6u6Co";
      var GMTString = new Date().toGMTString();
      var ShaObj = new window.jsSHA("SHA-1", "TEXT");
      ShaObj.setHMACKey(AppKey, "TEXT");
      ShaObj.update("x-date: " + GMTString);
      var HMAC = ShaObj.getHMAC("B64");
      //GET請求
      axios({
        method: "get",
        url: `https://ptx.transportdata.tw/MOTC/v2/Bus/Route/City/${vm.position}/${vm.route_name}`,
        headers: {
          Authorization:
            'hmac username="' +
            AppID +
            '", algorithm="hmac-sha1", headers="x-date", signature="' +
            HMAC +
            '"',
          "X-Date": GMTString,
        },
        params: {},
      })
        .then((response) => {
          if (response.data !== undefined) {
            let tmp_route_data = Object.assign([], response.data);
            vm.route_data = tmp_route_data.map((item) => {
              return { ...item, type: false };
            });
          }
        })
        .catch((error) => console.log(error));
    },
    searchdata_list(route_name) {
      const vm = this;
      const AppID = "9a683d2b67d1404c9af3b36c096a1518";
      const AppKey = "U7RkTz42_KNQ-A1LDCsdiH6u6Co";
      const GMTString = new Date().toGMTString();
      const ShaObj = new window.jsSHA("SHA-1", "TEXT");
      ShaObj.setHMACKey(AppKey, "TEXT");
      ShaObj.update("x-date: " + GMTString);
      const HMAC = ShaObj.getHMAC("B64");
      const myModal = new window.bootstrap.Modal(
        document.getElementById("exampleModal"),
        {
          keyboard: false,
        }
      );

      //GET請求
      axios({
        method: "get",
        url: `https://ptx.transportdata.tw/MOTC/v2/Bus//EstimatedTimeOfArrival/City/${vm.position}/${route_name}`,
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
          $filter: `RouteName/Zh_tw eq '${route_name}'`,
          $orderby: `StopSequence,Direction`,
          $format: `JSON`,
        },
      })
        .then((response) => {
          let tmp_time;
          vm.route_sub_data = response.data;
          vm.route_sub_data = vm.route_sub_data.map((item) => {
            if (
              item.StopStatus === 2 ||
              item.StopStatus === 3 ||
              item.StopStatus === 4
            ) {
              tmp_time = null;
            } else {
              if (item.EstimateTime) {
                tmp_time = Math.floor(item.EstimateTime / 60);
                if (tmp_time < 3) {
                  tmp_time = "進站中";
                }
              }
            }

            return { ...item, EstimateTime: tmp_time };
          });

          let tmp_outward = vm.route_sub_data.filter(
            (item) => item.Direction === 0
          );
          let tmp_return = vm.route_sub_data.filter(
            (item) => item.Direction === 1
          );
          if (tmp_outward.length === 0 || tmp_return.length === 0) {
            console.log('255未知')
          } else {
            vm.route_sub_name[0] = tmp_outward.slice(-2, -1)[0].StopName.Zh_tw;
            vm.route_sub_name[1] = tmp_return.slice(-2, -1)[0].StopName.Zh_tw;
          }

          myModal.show();
        })
        .catch((error) => console.log(error));
    },
    favorite(route_uid, isFav) {
      const vm = this;
      vm.route_data = vm.route_data.map((item) => {
        if (item.RouteUID === route_uid) {
          return { ...item, type: !item.type };
        } else {
          return { ...item };
        }
      });
      if (isFav !== true) {
        vm.tmp_local_fv_route_data = vm.route_data.filter((item) => {
          return item.type === true;
        });
        vm.local_fv_route_data[0].fav_route = vm.tmp_local_fv_route_data;
        vm.local_fv_route_data = vm.local_fv_route_data.map((item) => {
          return item;
        });
        let tmp_route_data = JSON.parse(localStorage.getItem("fv_route_data"));
        tmp_route_data = tmp_route_data.map((item) => {
          return item.City === vm.position
            ? { ...item, fav_route: vm.local_fv_route_data[0].fav_route }
            : { ...item };
        });
        localStorage.setItem("fv_route_data", JSON.stringify(tmp_route_data));
      } else {
        let fav_index = vm.local_fv_route_data[0].fav_route.find((item) => {
          return item.RouteUID === route_uid;
        });
        vm.local_fv_route_data[0].fav_route.splice(fav_index, 1);
        let tmp_route_data = JSON.parse(localStorage.getItem("fv_route_data"));
        tmp_route_data = tmp_route_data.map((item) => {
          return item.City === vm.position
            ? { ...item, fav_route: vm.local_fv_route_data[0].fav_route }
            : { ...item };
        });
        localStorage.setItem("fv_route_data", JSON.stringify(tmp_route_data));
      }
    },
  },
};
</script>

<style scoped>
.route__card .route__card__body {
  display: flex;
  justify-content: space-between;
}

.route__card .route__card__body :nth-child(2) {
  display: flex;
  font-size: 30px;
  justify-content: flex-end;
  align-items: center;
}
</style>