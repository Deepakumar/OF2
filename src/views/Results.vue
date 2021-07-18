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
                <v-progress-circular
                  v-if="item.status == 'START_PROCESSING'"
                  :width="5"
                  color="teal"
                  indeterminate
                ></v-progress-circular>
                <v-progress-circular
                  v-else-if="item.status == 'COMPLETED'"
                  :rotate="360"
                  :width="5"
                  :value="100"
                  color="teal"
                  ><span style="font-size: 0.5em"
                    >100%</span
                  ></v-progress-circular
                >
                <v-progress-circular
                  v-else-if="item.status == 'PENDING'"
                  :rotate="360"
                  :width="5"
                  :value="0"
                  color="teal"
                  ><span style="font-size: 0.5em"
                    >100%</span
                  ></v-progress-circular
                >
                <v-icon
                  v-else-if="item.status == 'ERROR'"
                  aria-hidden="false"
                  color="red"
                  style="font-size: 2em"
                >
                  info
                </v-icon>
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
};
</script>

<style></style>
