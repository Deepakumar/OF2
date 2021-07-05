<template>
  <v-container>
    <v-form id="input_form" @submit="saveData">
      <v-row>
        <v-col cols="4" offset-s1>
          <v-file-input
            placeholder="Upload your documents"
            label="Upload File"
            multiple
            prepend-icon="mdi-paperclip"
          >
            <template v-slot:selection="{ text }">
              <v-chip small label color="primary">
                {{ text }}
              </v-chip>
            </template>
          </v-file-input>
        </v-col>
        <v-col cols="2">
          <v-radio-group v-model="from.accessionType" mandatory>
            <v-radio label="Protein" value="protein"></v-radio>
            <v-radio label="Gene" value="nucleotide"></v-radio>
          </v-radio-group>
        </v-col>
        <v-col cols="6">
          <v-text-field
            label="NCBI or Uniprot Accession(s) seperated by comma"
            v-model="from.ncbiAccessionInput"
            @blur="validate"
          ></v-text-field>
          <label v-if="errors.invalidAccession" style="color: red"
            >Invalid Accession(s)- {{ errors.invalidAccessionMsg }}</label
          >
          <label style="color: red" v-if="$v.from.ncbiAccessionInput.$dirty && !$v.from.ncbiAccessionInput.required">Accession is required.</label>
          <v-chip class="ma-n3 float-right" x-small>
            {{ from.ncbiAccessionInput.length }}/100
          </v-chip>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12">
          <v-textarea
            label="Gene Sequence"
            v-model="from.sequence"
            :rules="textAreaRules"
          ></v-textarea>
          <v-chip class="ma-n3 float-right" x-small>
            {{ from.sequence.length }}/5000
          </v-chip>
          <div style="color: red" v-if="$v.from.sequence.$dirty && !$v.from.sequence.required">Sequence is required.</div>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12">
          <v-autocomplete
            v-model="from.organismName"
            :items="organismList"
            label="Organism"
            item-text="name"
            item-value="name"
          >
            <template v-slot:item="data">
              <v-list-item-avatar>
                <img :src="data.item.img" />
              </v-list-item-avatar>
              <v-list-item-content>
                <v-list-item-title v-html="data.item.name"></v-list-item-title>
              </v-list-item-content>
            </template>
          </v-autocomplete>
          <div style="color: red" v-if="$v.from.organismName.$dirty && !$v.from.organismName.required">Organis is required.</div>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="s6" offset-s1>
          <v-container fluid class="d-flex align-center pl-0">
            <p class="mb-0 mr-2">Try out by:</p>
            <label class="d-flex align-center">
              sequence
              <v-switch
                true-value="true"
                false-value="false"
                class="ml-2"
                v-model="from.exampleMethod"
                @change="clearAccessionSequenceAndOrganism"
              >
              </v-switch>
              <span class="d-inline lever">accessions</span>
            </label>
          </v-container>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="s12" class="d-flex align-center">
          <v-expansion-panels flat class="pa-0">
            <v-expansion-panel class="pa-0">
              <v-expansion-panel-header class="pa-0">
                <v-spacer />
                <v-col cols="2"> Advanced parameters: </v-col>
              </v-expansion-panel-header>
              <v-expansion-panel-content class="pa-0">
                <v-card flat color="transparent" class="pa-0">
                  <v-subheader class="pa-0"
                    >Maximum
                    <a
                      class="mx-2"
                      href="https://blast.ncbi.nlm.nih.gov/Blast.cgi?CMD=Web&PAGE_TYPE=BlastDocs&DOC_TYPE=FAQ"
                      target="_blank"
                      >E-value</a
                    >
                    for BLAST(e-10):</v-subheader
                  >
                  <v-slider
                    max="10"
                    min="1"
                    thumb-label
                    v-model="from.maxEvalue"
                    ticks
                    color="teal"
                  ></v-slider>
                  <v-subheader class="pa-0"
                    >Maximum target sequences for BLAST:</v-subheader
                  >
                  <v-slider
                    max="1000"
                    min="100"
                    v-model="from.maxTargetSequence"
                    thumb-label
                    ticks
                    color="teal"
                  ></v-slider>
                  <v-subheader class="pa-0">Identity:</v-subheader>
                  <v-slider
                    max="100"
                    min="60"
                    v-model="from.identity"
                    thumb-label
                    ticks
                    color="teal"
                  ></v-slider>
                </v-card>
              </v-expansion-panel-content>
            </v-expansion-panel>
          </v-expansion-panels>
        </v-col>
      </v-row>
      <v-row align="left" justify="space-around">
        <v-col cols="6">
          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-icon
                size="72"
                color="green darken-2"
                class="icon icon-species icon-ecoli pa-4"
                v-on:click="loadExampleData('Escherichia coli(562)')"
                v-bind="attrs"
                v-on="on"
              ></v-icon>
            </template>
            <span>Escherichia coli</span>
          </v-tooltip>

          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-icon
                size="72"
                color="green darken-2"
                class="icon icon-species icon-fly pa-4"
                v-on:click="loadExampleData('Drosophila melanogaster(7227)')"
                v-bind="attrs"
                v-on="on"
              >
              </v-icon>
            </template>
            <span>Drosophila melanogaster</span>
          </v-tooltip>

          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-icon
                size="72"
                color="green darken-2"
                class="icon icon-species icon-human pa-4"
                v-on:click="loadExampleData('Homo sapiens(9606)')"
                v-bind="attrs"
                v-on="on"
              >
              </v-icon>
            </template>
            <span>Homo sapiens</span>
          </v-tooltip>

          <v-tooltip bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-icon
                size="72"
                color="green darken-2"
                class="icon icon-species icon-brassica pa-4"
                v-on:click="loadExampleData('Arabidopsis thaliana(3702)')"
                v-bind="attrs"
                v-on="on"
              >
              </v-icon>
            </template>
            <span>Arabidopsis thaliana</span>
          </v-tooltip>
        </v-col>
        <v-spacer />
      </v-row>
      <v-row>
        <v-spacer />
        <v-col cols="2" offset-10>
          <v-btn @click="analysData" :disabled="($v.$invalid && $v.$dirty) || this.errors.invalidAccession" color="teal">
            <span style="color:white;">Submit</span>
            <v-icon right dark class="" text color="white">
              mdi-send
            </v-icon>
          </v-btn>
        </v-col>
      </v-row>
    </v-form>
    <v-dialog
      v-model="isLoading"
      width="800"
      height="200"
      overlay-color="#506c87"
    >
      <v-card color="#f5f7f9">
        <v-card-title class="text-h5">
          ORFanID in Progress....
          <v-progress-linear indeterminate color="teal"></v-progress-linear>
        </v-card-title>
        <v-card-text class="justify-center">
          <center>
            <v-img
              src="../assets/images/loading4.gif"
              width="200"
              height="75"
            />
          </center>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-container>
</template>
<script>
import $ from "jquery";
import analysis from "@/api/analysis";
import { maxLength, required, requiredIf } from "vuelidate/lib/validators";

export default {
  name: "Home",
  methods: {
    loadExampleData(_exampleName) {
      this.clearAccessionSequenceAndOrganism();
      this.from.exampleName = _exampleName;
      let url = "";
      if (this.from.exampleMethod == "true") {
        if (this.from.accessionType === "protein") {
          this.from.ncbiAccessionInput =
            this.exampleProteinDataValues[this.from.exampleName];
        } else {
          this.from.ncbiAccessionInput =
            this.exampleNucliotideDataValues[this.from.exampleName];
        }
        this.from.sequence = "";
      } else {
        url = `/data/example-${this.from.accessionType}-${this.from.exampleName}.fasta`;
        let that = this;
        $.get(url, (data) => {
          that.from.sequence = this.remove_linebreaks(data);
        });
        this.from.ncbiAccessionInput = "";
      }
      this.from.organismName = this.from.exampleName;
    },
    clearAccessionSequenceAndOrganism() {
      this.from.ncbiAccessionInput = "";
      this.from.sequence = "";
      this.from.organismName = "";
    },
    analysData() {
      this.$v.$touch();
      if (this.$v.$invalid== false) {
        console.log(this.from);
        var requestInfo = {
          accessionType: this.from.accessionType,
          identity: this.from.identity,
          maxEvalue: this.from.maxEvalue,
          maxTargetSequence: this.from.maxTargetSequence,
          organismName: this.from.organismName,
          sequence: this.from.sequence,
        };

        if (this.from.exampleMethod == "true") {
          requestInfo.accession = this.from.ncbiAccessionInput;
        } else {
          requestInfo.sequence = this.from.sequence;
        }
        this.isLoading = true;
        analysis
          .analyse(requestInfo)
          .then((response) => {
            this.analyseResult.session = response.data;
            this.isLoading = false;
            this.$router.push("result?analysisId" + this.analyseResult.session);
          })
          .catch((error) => {
            this.isLoading = false;
            console.log(error);
          });
      }
    },
    validate() {
      analysis
        .validate({
          accessions: this.from.ncbiAccessionInput,
          accessionType: this.from.accessionType,
        })
        .then((response) => {
          if (response.data.isValid == false) {
            this.errors.invalidAccessionMsg =
              response.data.invalidAccessions.join(", ");
            this.errors.invalidAccession = true;
          } else {
            this.errors.invalidAccession = false;
          }
        });
    },
    onCancel() {
      console.log("a");
    },
    remove_linebreaks(str) {
    return str.replace( /[\r]+/gm, "" );
    }
  },
  mounted() {
    analysis.getOrganismList().then((response) => {
      Object.entries(response.data).forEach((element) => {
        this.organismList.push({
          name: element[0],
          img: element[1],
        });
      });
    });
  },
  data() {
    return {
      isLoading: false,
      fullPage: true,
      toggleAdvanceparameters: false,
      organismList: [],
      from: {
        sequenceGroup: 1,
        analysisId: new Date().toJSON().replace(/-/g, "/"),
        exampleName: "",
        accessionType: "protein",
        ncbiAccessionInput: "",
        exampleMethod: "false",
        sequence: "",
        organismName: "",
        maxEvalue: 3,
        maxTargetSequence: 550,
        identity: 60
      },
      analyseResult: {
        session: "",
      },
      errors: {
        invalidAccession: false,
        invalidAccessionMsg: "",
      },
      textAreaRules: [
        (value) => (value || "").length <= 5000 || "Max 5000 characters",
      ],
      exampleProteinDataValues: {
        "Escherichia coli(562)": "NP_415100.1,YP_002791247.1,NP_414542.1",
        "Drosophila melanogaster(7227)": "NP_524859.2",
        "Homo sapiens(9606)": "NP_001119584.1",
        "Arabidopsis thaliana(3702)": "NP_187663.1",
      },
      exampleNucliotideDataValues: {
        "Escherichia coli(562)": "NZ_JAACYZ010000241.1,X86971.1",
        "Drosophila melanogaster(7227)": "NM_080120.3",
        "Homo sapiens(9606)": "NM_001126112.2",
        "Arabidopsis thaliana(3702)": "NM_111887.3",
      },
    };
  },
  validations: {
    from: {
      sequence: {
        maxLength: maxLength(5000),
      },
      organismName: {
        required,
      },
      ncbiAccessionInput: {
        required: function ncbiAccessionRequired(val) {
          if(this.from.exampleMethod == "true" && val  == "") {
            return false
          }else {
            return true
          }
        }
      },
      sequence: {
        required: function sequenceRequired(val) {
          if(this.from.exampleMethod == "false" && val == "") {
            return false
          } else {
            return true
          }
        }
      },
    },
  },
};
</script>
<style scoped>
.v-fade {
  display: inherit !important; /* override v-show display: none */
  transition: opacity 0.5s;
}

.v-fade[style*="display: none;"] {
  opacity: 0;
  pointer-events: none; /* disable user interaction */
  user-select: none; /* disable user selection */
  height: 0px !important;
}

.v-expansion-panels {
  box-shadow: none;
}

.v-icon {
  cursor: pointer;
}
</style>
<!--export default class Home extends Vue {}-->
