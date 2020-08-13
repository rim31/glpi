<!--https://riptutorial.com/javascript/example/1260/filtering-object-arrays -->
<!-- https://codepen.io/kevjose/pen/YzXrobv?editors=1010 -->
<template>
  <div id="app">
    <div class="panel panel-sm">
      <div class="panel-body">
        <div class="form-group">
          <label for="csv_file" class="control-label col-sm-3 text-center">CSV file to import</label>
          <div class="col-sm-12 text-center">
            <input
              type="file"
              id="csv_file"
              name="csv_file"
              class="buttonInput btn is-info pb-2"
              @change="loadCSV($event)"
            />
          </div>
        </div>
        <div class="tableCSV">
          <div v-if="parse_csv.length > 0" class="mb-2 mr-1 ml-1">
            <!-- <p v-if="filterData.length > 0">{{filterData.length}} article(s)</p> -->
            <p v-if="finalData.length > 0">{{finalData.length}} article(s)</p>
            <div class="field control d-flex justify-content-between">
              <input v-model="keyword" class="form-control" placeholder="Filter users by word" />
              <!-- <label for="position">Entrez un produit</label>
              <input
                placeholder="windows par exemple"
                type="text"
                id="product"
                class="input"
                v-model="product"
                v-on:keypress="searchProduct"
              />-->
              <!-- <label for="position">Entrez un prix</label> -->
              <input
                placeholder="tarif €"
                type="number"
                id="price"
                class="form-control w-25"
                v-model="price"
                v-on:keypress="addPrice"
              />
              <span :style="{display:hidden}">
                {{price
                |
                positiveNumber
                }}
              </span>
              <button v-on:click="changeHeader" class="btn btn-primary">Price</button>
            </div>
            <h1>{{ filterData.length * price | amout}} € HT/mois</h1>
          </div>
          <!-- <table v-if="parse_csv" class="fixed_headerO">
            <thead class="headerTable">
              <tr v-for="(items, index) in parse_csv" :key="index">
                <template v-if="index == 0">
                  <td v-for="(item, key) in items" :key="key">{{ key }}</td>
                </template>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(items, index) in filterData" :key="index" :keyword="keyword">
                <td v-for="(item, key) in items" :key="key">{{ items[key] }}</td>
              </tr>
            </tbody>
          </table>-->
          <table v-if="parse_csv">
            <thead>
              <tr>
                <th
                  v-for="(key, indexHeader) in parse_header"
                  :class="{ active: sortKey == key }"
                  :key="indexHeader"
                  @click="sorting(key)"
                >
                  <!-- @click="sortBy(key)" -->
                  {{ key | capitalize }} &#8645;
                  <span
                    class="arrow"
                    :class="sortOrders[key] > 0 ? 'asc' : 'dsc'"
                  ></span>
                </th>
              </tr>
            </thead>
            <tr v-for="(csv, indexTab) in filterData" :key="indexTab">
              <td v-for="(key, index) in parse_header" :key="index">{{csv[key]}}</td>
            </tr>
          </table>
        </div>
      </div>
    </div>
    <!-- <Lol v-if="result.length > 0" v-bind:result="result" /> -->
  </div>
</template>

<script>
import { bus } from "../main";
import _ from "lodash";
// import Lol from "./Lol";

export default {
  name: "HelloWorld",
  props: {
    msg: String
  },
  beforeMount() {
    this.parse_csv = [];
  },
  // components: { Lol },
  data() {
    return {
      myBill: [{ products: "", prices: 0 }],
      product: "",
      price: 0,
      bill: 0,
      parse_header: [],
      parse_csv: [],
      finalData: [],
      sortOrders: {},
      sortKey: "",
      keyword: "",
      order: 1,
      result: [],
      keys: []
    };
  },
  methods: {
    sorting(key) {
      this.order *= -1;
      this.parse_csv = this.parse_csv.sort((a, b) =>
        (this.order === 1 ? a[key] > b[key] : a[key] < b[key]) ? 1 : -1
      );
    },
    sortBy(key) {
      var vm = this;
      vm.sortKey = key;
      vm.sortOrders[key] = vm.sortOrders[key] * -1;
    },
    csvJSON(csv) {
      var vm = this;
      var lines = csv.split("\n");
      var result = [];
      var headers = lines[0]
        .split(";")
        .join(",")
        .split(",");
      vm.parse_header = lines[0]
        .split(";")
        .join(",")
        .split(",");
      lines[0]
        .split(";")
        .join(",")
        .split(",")
        .forEach(function(key) {
          vm.sortOrders[key] = 1;
        });

      lines.map(function(line, indexLine) {
        if (indexLine < 1) return; // Jump header line

        var obj = {};
        var currentline = line
          .split(";")
          .join(",")
          .split(",");

        headers.map(function(header, indexHeader) {
          obj[header] = currentline[indexHeader];
        });

        result.push(obj);
      });

      result.pop(); // remove the last item because undefined values

      return result; // JavaScript object
    },
    loadCSV(e) {
      var vm = this;
      if (window.FileReader) {
        var reader = new FileReader();
        reader.readAsText(e.target.files[0]);
        var filename = e.target.files[0].name;
        var extension = e.target.files[0].type;
        if (extension !== "text/csv") {
          console.log(filename, extension);
          alert("error file upload");
          return;
        }
        // Handle errors load
        reader.onload = function(event) {
          var csv = event.target.result;
          vm.parse_csv = vm.csvJSON(csv.replace(/"/g, ""));
          // console.log(vm.parse_csv);
        };
        reader.onerror = function(evt) {
          if (evt.target.error.name == "NotReadableError") {
            alert("Canno't read file !");
          }
        };
      } else {
        alert("FileReader are not supported in this browser.");
      }
    },
    changeHeader() {
      bus.$emit("changeHeader", "mon Nouveau Header");
    },
    searchProduct() {
      console.log(this.product);
      bus.$emit("searchProduct", "product");
    },
    addPrice() {
      console.log(this.price);
      bus.$emit("addPrice", "price");
    }
  },
  computed: {
    // filter par mot clé : array of obj:
    // chercher le mot clé dans chacune des lignes du tableau le mot clé et retourner la ligne si trouvé
    filterData() {
      // this.finalData = ["filterData"];
      return this.keyword
        ? this.parse_csv.filter(obj => {
            var flag = false;
            Object.values(obj).forEach(val => {
              if (
                String(val ? val.toLowerCase() : val).indexOf(
                  this.keyword.toLowerCase()
                ) > -1
              ) {
                flag = true;
                return;
              }
            });
            if (flag) return obj;
          })
        : this.parse_csv;
      // return this.finalData;
    }
  },
  watch: {
    filterData: {
      deep: true,
      handler: function(newVal) {
        this.finalData = newVal;
        this.result = [];
        if (newVal.length > 0) {
          this.keys = Object.keys(newVal[0]);
          // console.log(keys);
          for (let index = 0; index < this.keys.length; index++) {
            this.result.push(_.groupBy(newVal, this.keys[index]));
          }
        }
        console.log(newVal);
        console.log(this.result);
      }
    }
  },
  filters: {
    capitalize: function(str) {
      return str.charAt(0).toUpperCase() + str.slice(1);
    },
    positiveNumber: function(value) {
      if (value < 0) return 0;
      return value;
    },
    amout: function(value) {
      if (value < 0) return 0;
      return value;
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 4px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
html,
body {
  margin: 0;
  padding: 0;
}
body {
  /* margin: 32px auto; */
}
.panel {
  border: 2px solid #dfdfdf;
  box-shadow: rgba(0, 0, 0, 0.15) 0 1px 0 0;
  height: auto;
  overflow: hidden;
  /* margin: 10px; */
}
.panel.panel-sm {
  /* max-width: 95%;
  margin: 10px auto; */
}
.panel-heading {
  border-bottom: 2px solid #dfdfdf;
}
.panel-heading h1,
.panel-heading h2,
.panel-heading h3,
.panel-heading h4,
.panel-heading h5,
.panel-heading h6 {
  margin: 0;
  padding: 0;
}
.panel-body .checkbox-inline {
  /* padding: 15px 20px; */
}
table {
  font-family: arial, sans-serif;
  border-collapse: collapse;
  width: 100%;
}

td,
th {
  border: 1px solid #dddddd;
  text-align: left;
  padding: 8px;
}

tr:nth-child(even) {
  background-color: #dddddd;
}

.buttonInput {
  height: 30px;
  background-color: #42b983;
  border-radius: 5px;
}

.tableCSV {
  overflow-x: auto;
}

.headerTable {
  font-weight: bold;
  color: red;
}

.fixed_header {
  width: 100%;
  table-layout: fixed;
  border-collapse: collapse;
}

.fixed_header tbody {
  display: block;
  width: 100%;
  overflow: auto;
  height: 90vh;
}

.fixed_header thead tr {
  display: block;
}

.fixed_header thead {
  background: black;
  color: #fff;
}
.tableCSV {
  /* background-color: grey; */
}
</style>
