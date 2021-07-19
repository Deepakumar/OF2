<template>
  <div class="container">
    <div class="section">
      <div id="savedResultView" class="row">
        <div class="col s10 offset-s1 center-align">
          <div>
            <v-card-title>
              <router-link
                :to="{
                  name: 'input',
                }"
                >New Query</router-link
              >
              <v-spacer></v-spacer>
              <v-text-field
                v-model="search"
                append-icon="mdi-magnify"
                label="Enter Search Term Here"
                single-line
                hide-details
              ></v-text-field>
            </v-card-title>
            <v-data-table
              :headers="headers"
              :items="desserts"
              :page.sync="page"
              :items-per-page="itemsPerPage"
              :search="search"
              hide-default-footer
              class="elevation-1"
              @page-count="pageCount = $event"
            >
              <template v-slot:item.analysisIdNav="{ item }">
                <div v-if="item.status != 'START_PROCESSING'">
                  <router-link
                    :to="{
                      name: 'result',
                      params: { analysisId: item.analysisIdNav },
                    }"
                  >
                    <v-icon large>mdi-chart-bar</v-icon>
                  </router-link>
                </div>
              </template>
              <template v-slot:item.status="{ item }">
                <v-tooltip top v-if="item.status == 'START_PROCESSING'">
                  <template v-slot:activator="{ on, attrs }">
                    <v-progress-circular
                      :width="5"
                      color="teal"
                      indeterminate
                      v-bind="attrs"
                      v-on="on"
                    ></v-progress-circular>
                  </template>
                  <span>Processing</span>
                </v-tooltip>

                <v-tooltip
                  top
                  v-else-if="item.status == 'COMPLETED'"
                  color="teal"
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-progress-circular
                      :rotate="360"
                      :width="5"
                      :value="100"
                      color="teal"
                      v-bind="attrs"
                      v-on="on"
                      ><span style="font-size: 0.5em"
                        >100%</span
                      ></v-progress-circular
                    >
                  </template>
                  <span>Completed</span>
                </v-tooltip>

                <v-tooltip top v-else-if="item.status == 'PENDING'">
                  <template v-slot:activator="{ on, attrs }">
                    <v-progress-circular
                      :rotate="360"
                      :width="5"
                      :value="0"
                      color="teal"
                      v-bind="attrs"
                      v-on="on"
                      ><span style="font-size: 0.5em"
                        >100%</span
                      ></v-progress-circular
                    >
                  </template>
                  <span>Top tooltip</span>
                </v-tooltip>

                <v-tooltip top v-else-if="item.status == 'ERRORED'">
                  <template v-slot:activator="{ on, attrs }">
                    <v-icon
                      aria-hidden="false"
                      color="red"
                      style="font-size: 2em"
                      v-bind="attrs"
                      v-on="on"
                    >
                      info
                    </v-icon>
                  </template>
                  <span>Error</span>
                </v-tooltip>

                
                <v-icon
                  v-else-if="item.status == 'CANCELLED'"
                  aria-hidden="false"
                  style="font-size: 2em"
                  v-bind="attrs"
                  v-on="on"
                >
                  cancel
                </v-icon>

                <v-btn
                  v-if="item.status == 'PENDING'"
                  class="ms-6"
                  @click="cancelAnalyse(item.analysisId)"
                  >Cancel</v-btn
                >
              </template>
            </v-data-table>
            <v-row>
              <v-col cols="1">
                <v-combobox
                  :value="itemsPerPage"
                  :items="[5, 10, 15, 20, 25, 30]"
                  label="Items per page"
                  type="number"
                  @input="itemsPerPage = parseInt($event, 5)"
                ></v-combobox>
              </v-col>
              <v-col cols="11">
                <v-pagination v-model="page" :length="pageCount"></v-pagination>
              </v-col>
            </v-row>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import analysisAPI from "../api/analysis";
import moment from "moment";
import _ from "lodash";

export default {
  data() {
    return {
      search: "",
      page: 1,
      pageCount: 0,
      itemsPerPage: 10,
      headers: [
        { text: "Date", align: "start", sortable: true, value: "date" },
        { text: "Analysis ID", value: "analysisId", sortable: true },
        { text: "Email", value: "email", sortable: true },
        { text: "Organism", value: "organism", sortable: true },
        { text: "Genes", value: "genes", sortable: true },
        { text: "Status", value: "status", sortable: true },
        { text: "", value: "analysisIdNav", sortable: true },
      ],
      desserts: [],
    };
  },
  mounted() {
    const that = this;
    analysisAPI.getAll().then((response) => {
      console.log(response);
      response.data.forEach((element) => {
        that.desserts.push({
          date: moment(element.analysisDate).format("YYYY-MM-DD"),
          analysisId: element.analysisId,
          email: element.email,
          organism: element.organism,
          genes: element.numberOfGenes,
          analysisIdNav: element.analysisId,
          status: element.status,
        });
      });
    });
  },
  methods: {
    cancelAnalyse(analysisId) {
      let that = this;
      if (analysisId != null) {
        analysisAPI.cancelAnalyse(analysisId).then((response) => {
          let cancelItem = _.find(that.desserts, { analysisId: analysisId });
          if (cancelItem != null) {
            cancelItem.status = "CANCELLED";
          }
        });
      }
    },
  },
};
</script>

<style></style>
