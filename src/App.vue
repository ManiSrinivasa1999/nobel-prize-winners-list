<template>
  <!-- App.vue -->
  <v-app>
    <v-app-bar
      fixed
      height="64px"
      color="indigo"
      dark
      elevate-on-scroll
      scroll-threshold="100"
      app
    >
      <h1 class="ma-auto text-h3 font-weight-bold heading">
        Nobel Prize Winners
      </h1>
    </v-app-bar>

    <!-- Sizes your content based upon application components -->
    <v-main>
      <v-conatiner>
        <v-row align="center" class="flex flex-row">
          <v-col cols="12" class="mx-4 mt-4">
            <div class="text-h4">Filters</div>
          </v-col>
        </v-row>
        <v-data-iterator
          :items="modifyFormatOfData"
          :items-per-page.sync="modifyFormatOfData.length"
          :search="search"
          :sort-desc="sortByCategories[sortDesc]"
          :sort-by="sortBy.toLowerCase()"
          :loading="loading"
          light
          hide-default-footer
        >
          <template v-slot:header>
            <v-row class="flex flex-row justify-space-between ma-2">
              <v-col
                cols="12"
                md="2"
                sm="12"
                xs="12"
                class="flex justify-center"
              >
                <v-select
                  :items="years"
                  label="Select Year"
                  v-model="selectedYear"
                  outlined
                  @change="changeYear()"
                ></v-select>
              </v-col>
              <v-col cols="12" md="2" sm="12" xs="12">
                <v-select
                  :items="Object.keys(categories)"
                  label="Select Category"
                  v-model="selectedCategory"
                  outlined
                  @change="changeCategory()"
                ></v-select>
              </v-col>
              <v-col cols="12" md="2" sm="12" xs="12">
                <v-select
                  :items="Object.keys(sortByCategories)"
                  label="Sort By"
                  v-model="sortBy"
                  outlined
                  @change="changeCategory()"
                ></v-select>
              </v-col>
              <v-col cols="12" md="2" sm="12" xs="12">
                <div class="pb-6">
                  <v-btn
                    class="ma-2 white--text"
                    :color="moreThanOneTimeWinners ? 'indigo' : 'red'"
                    x-large
                    outlined
                    @click="displayMoreThanOneTimeWinner()"
                  >
                    Multiple Time Winners
                  </v-btn>
                </div>
              </v-col>
              <v-col cols="12" md="3" sm="12" xs="12">
                <v-toolbar color="transparent" flat class="mb-6">
                  <v-text-field
                    v-model="search"
                    clearable
                    hide-details
                    solo
                    background-color=""
                    append-icon="mdi-magnify"
                    label="Search"
                  ></v-text-field>
                </v-toolbar>
              </v-col>
              <v-col cols="12" md="1" sm="12" xs="12">
                <div class="pb-6">
                  <v-btn
                    class="my-4 white--text"
                    color="red"
                    large
                    @click="clear()"
                  >
                    Clear
                  </v-btn>
                </div>
              </v-col>
            </v-row>
          </template>
          <template v-slot:default="props">
            <v-row>
              <v-col
                cols="12"
                xl="3"
                lg="3"
                md="4"
                sm="10"
                v-for="(prize, i) in props.items"
                :key="i"
              >
                <v-container fluid>
                  <v-card color="indigo">
                    <v-card-text
                      class="mt-4 text-center text-h6 white--text truncate"
                    >
                      <v-tooltip top>
                        <template v-slot:activator="{ on, attrs }">
                          <div v-bind="attrs" v-on="on" class="truncate">
                            {{ prize.name }}
                          </div>
                        </template>
                        <span>{{ prize.name }}</span>
                      </v-tooltip>
                    </v-card-text>
                    <v-divider light></v-divider>
                    <v-card-text class="text-h6 mt-2 text-center white--text">
                      {{
                        prize.category[0].toUpperCase() +
                        prize.category.slice(1)
                      }}
                      -
                      {{ prize.year }}
                      <br />
                      <v-tooltip bottom>
                        <template v-slot:activator="{ on, attrs }">
                          <div v-bind="attrs" v-on="on" class="truncate">
                            {{ prize.motivation }}
                          </div>
                        </template>
                        <span>{{ prize.motivation }}</span>
                      </v-tooltip>
                    </v-card-text>
                  </v-card>
                </v-container>
              </v-col>
            </v-row>
          </template>
          <template v-slot:loading>
            <div class="text-center text-h4">Loading Nobel Prizes List....</div>
          </template>
        </v-data-iterator>
      </v-conatiner>
    </v-main>

    <v-footer app>
      <!-- -->
    </v-footer>
  </v-app>
</template>

<script>
import axios from "axios";
export default {
  name: "App",
  components: {},
  data() {
    return {
      search: "",
      sortDesc: false,
      sortBy: "Name",
      list: [],
      loading: false,
      error: false,
      years: [],
      selectedYear: "",
      isMutlipleTimeWinners: false,
      sortByCategories: { Name: "name", Year: "year" },
      categories: {
        Chemistry: "chemistry",
        Economics: "economics",
        Medicine: "medicine",
        Literature: "literature",
        Physics: "physics",
        Peace: "peace",
      },
      selectedCategory: "",
      moreThanOneTimeWinners: false,
    };
  },
  mounted() {
    this.setYearsValues();
    this.getDataFromAPI();
  },
  methods: {
    getDataFromAPI() {
      this.loading = true;
      axios
        .get("http://api.nobelprize.org/v1/prize.json")
        .then((response) => {
          this.list = response.data.prizes;
        })
        .catch((error) => {
          console.log(error);
          this.error = true;
        })
        .finally(() => (this.loading = false));
    },
    setYearsValues() {
      for (let year = 2022; year > 1900; year -= 1) {
        this.years.push(year);
      }
    },
    changeCategory() {
      this.moreThanOneTimeWinners = false;
    },
    changeYear() {
      this.moreThanOneTimeWinners = false;
    },
    displayMoreThanOneTimeWinner() {
      this.moreThanOneTimeWinners = !this.moreThanOneTimeWinners;
      this.selectedYear = "";
      this.selectedCategory = "";
    },
    clear() {
      this.selectedCategory = "";
      this.selectedYear = "";
      this.search = "";
      this.displayMoreThanOneTimeWinner = false;
    },
  },
  computed: {
    capitalizeString(string) {
      console.log(string);
      return string[0].toUpperCase() + string.slice(1);
    },
    modifyFormatOfData() {
      let res = [];
      this.list.forEach((item) => {
        if (item["laureates"]) {
          item.laureates.forEach((obj) => {
            let newObj = {
              name:
                (obj.surname ? obj.surname : "") +
                " " +
                (obj.firstname ? obj.firstname : ""),
              motivation: obj.motivation,
              year: item.year,
              category: item.category,
            };
            res.push(newObj);
          });
        }
      });
      // more than one time winners
      if (this.moreThanOneTimeWinners) {
        let names = [];
        res.forEach((prize) => {
          let count = res.filter((item) => item.name == prize.name).length;
          if (count > 1) {
            names.push(prize);
          }
        });
        return names;
      } else {
        // filter by year
        let filteredDataByYear = [];
        if (this.selectedYear) {
          filteredDataByYear = res.filter(
            (item) => item.year == this.selectedYear
          );
          // filter by category
          let filteredByCategory = [];
          if (this.selectedCategory) {
            filteredByCategory = filteredDataByYear.filter(
              (item) => item.category == this.categories[this.selectedCategory]
            );
            return filteredByCategory;
          }
          return filteredDataByYear;
        } else {
          // filter by category when year not selected
          let filteredByCategory = [];
          if (this.selectedCategory) {
            filteredByCategory = res.filter(
              (item) => item.category == this.categories[this.selectedCategory]
            );
            return filteredByCategory;
          }
        }
      }
      return res;
    },
  },
};
</script>
<style>
.truncate {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
/* .truncate:hover {
  overflow: visible;
  white-space: normal;
} */
</style>
