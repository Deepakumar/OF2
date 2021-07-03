<template>
  <div class="container">
    <div class="section">
      <div id="savedResultView" class="row">
        <div class="col s10 offset-s1 center-align">
          <div>
            <v-card-title>
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
              @page-count="pageCount = $event">
              <template v-slot:item.analysisIdNav="{ item }">
                <router-link :to="{ name: 'result', params: { analysisId: item.analysisIdNav }}">
                   <v-icon   large>mdi-chart-bar</v-icon>
                </router-link>
               
              </template>
            </v-data-table>
            <div class="text-center pt-2">
              <v-pagination v-model="page" :length="pageCount"></v-pagination>
              <v-text-field
                :value="itemsPerPage"
                label="Items per page"
                type="number"
                min="-1"
                max="15"
                @input="itemsPerPage = parseInt($event, 10)"
              ></v-text-field>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import analysisAPI from "../api/analysis";
import moment from 'moment'

export default {
  data() {
    return {
      page: 1,
      pageCount: 0,
      itemsPerPage: 10,
      headers: [
        { text: "Date", align: "start", sortable: true, value: "date" },
        { text: "Analysis ID", value: "analysisId", sortable: true },
        { text: "Email", value: "email", sortable: true },
        { text: "Organism", value: "organism", sortable: true },
        { text: "Genes", value: "genes", sortable: true },
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
          analysisIdNav:element.analysisId
        });
      });
    });
  },
};
</script>

<style></style>
