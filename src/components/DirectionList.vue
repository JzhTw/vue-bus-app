<template>
  <div>
    <!-- Modal -->
    <div
      class="modal fade"
      id="exampleModal"
      tabindex="-1"
      aria-labelledby="exampleModalLabel"
      aria-hidden="true"
    >
      <div class="modal-dialog modal-fullscreen">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="exampleModalLabel">去程 & 返程</h5>
            <button
              type="button"
              class="btn-close"
              data-bs-dismiss="modal"
              aria-label="Close"
            ></button>
          </div>
          <div class="modal-body">
            <div class="mb-3">
              <button class="btn btn-primary" @click="route_type = 0">
                往 - {{ route_nm[0] }}
              </button>
              <button class="btn btn-primary ms-2" @click="route_type = 1">
                往 - {{ route_nm[1] }}
              </button>
            </div>

            <ul class="list-group">
              <li
                class="list-group-item"
                v-for="(item, index) in filter_Route"
                :key="index"
              >
                <span
                  class="badge bg-primary btn-lg"
                  v-if="
                    item.EstimateTime !== null && item.EstimateTime !== '進站中'
                  "
                >
                  {{ item.EstimateTime }}分鐘
                </span>
                <span
                  class="badge bg-warning btn-lg"
                  v-else-if="item.EstimateTime === '進站中'"
                >
                  {{ item.EstimateTime }}
                </span>
                <span class="badge bg-secondary btn-lg" v-else> 無資訊 </span>
                {{ item.StopName.Zh_tw }}
              </li>
            </ul>
          </div>
          <div class="modal-footer">
            <button
              type="button"
              class="btn btn-secondary"
              data-bs-dismiss="modal"
            >
              Close
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: ["route_data", "route_nm"],
  data() {
    return {
      route_type: 0,
      //   to:this.route_data.filter((item) => item.Direction === 0)
    };
  },
  computed: {
    filter_Route: function () {
      const vm = this;
      return vm.route_data.filter((item) => item.Direction === vm.route_type);
    },
  },
};
</script>

<style>
</style>