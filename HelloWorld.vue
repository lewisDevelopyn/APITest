<template>
<div class="bigcontainer">
  <head></head>
  <div id="formcontainer" class="container">
    <form style="margin-top:10px; margin-left:10px;">
      <span>
        <strong>Input Langitude and Longitude:</strong>
      </span>
      <span>
        <label>Latitude:</label>
        <input type="number" id="Latitude" v-model="latitude" required />
        <label>Longitude:</label>
        <input
          style="margin-right: 10px;"
          type="number"
          id="Longitude"
          v-model="longitude"
          required
        />
        <button @click="userInput()" type="button">Submit</button>
      </span>
    </form>
  </div>
  <table cellpadding="0" cellspacing="0" border="0">
    <thead class="tbl-header">
      <tr>
        <th>Countries</th>
        <th>Cities</th>
        <th>Measurements</th>
      </tr>
    </thead>
    <tbody class="tbl-content">
      <tr>
        <td valign="top">
          <ul>
            <li v-for="country in listofcountries" :key="country.name">
              {{country.name}}
              <button @click="code1(country.code)">Display Cities</button>
            </li>
          </ul>
        </td>
        <td valign="top">
          <ul>
            <li v-for="city in cities" :key="city.city">
              {{city.city}}
              <button @click="code2(city.city)">Display Measurements</button>
            </li>
          </ul>
        </td>
        <td valign="top">
          <ul>
            <li>{{errormessage}}</li>
          </ul>
          <ul>
            <li v-for="measurement in measurements1" :key="measurement.city">
              <br />
              Latitude:{{measurement.coordinates.latitude}}
              <br />
              Longitude: {{measurement.coordinates.longitude}}
              <br />
              Parameter: {{measurement.parameter}}
              <br />
              Value: {{measurement.value}} {{measurement.unit}}
              <br />
              Date: UTC:{{measurement.date.utc}}, Local:{{measurement.date.local}}
              <br />
            </li>
          </ul>
          {{citylocation}}
          <ul>
            <li v-for="(data, index) in cityLookup" :key="index">
              <br />
              Parameter: {{data.parameter}}
              <br />
              Value: {{data.value}} {{data.unit}}
              <br />
              Date: UTC:{{data.date.utc}}, Local:{{data.date.local}}
              <br />
            </li>
          </ul>
        </td>
      </tr>
    </tbody>
  </table>
</div>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      listofcountries: [],
      listofcities: [],
      seen: null,
      cities: [],
      listofmeasurements: [],
      measurements1: [],
      latitude: "",
      longitude: "",
      cityLookup: null,
      citylocation: null,
      errormessage: null
    };
  },
  mounted() {
    const axios = require("axios");
    var self = this;
    // Make a request for a user with a given ID
    axios
      .get("https://api.openaq.org/v1/cities?limit=10000")
      .then(function(response) {
        // handle success
        console.log(response.data.results);
        self.listofcities = response.data.results;
      })
      .catch(function(error) {
        // handle error
        console.log(error);
      })
      .finally(function() {
        // always executed
      });

    axios
      .get("https://api.openaq.org/v1/countries?limit=10000")
      .then(function(response) {
        // handle success
        console.log(response.data.results);
        self.listofcountries = response.data.results;
      })
      .catch(function(error) {
        // handle error
        console.log(error);
      })
      .finally(function() {
        // always executed
      });
    axios
      .get("https://api.openaq.org/v1/measurements?limit=10000&has_geo=true")
      .then(function(response) {
        // handle success
        console.log(response.data.results);
        self.listofmeasurements = response.data.results;
      })
      .catch(function(error) {
        // handle error
        console.log(error);
      })
      .finally(function() {
        // always executed
      });
  },
  methods: {
    code1(code) {
      console.log(code);
      var cities = this.listofcities.filter(city => city.country === code);
      this.cities = cities;
      console.log(cities);
    },
    code2(x) {
      console.log(x);
      var measurements = this.listofmeasurements.filter(
        measurement => measurement.city === x
      );
      this.measurements1 = measurements;
      this.cityLookup = null;
      this.citylocation = null;
      if (this.measurements1.length === 0) {
        this.errormessage = "No Data Available";
      }
      if (this.measurements1.length > 0) {
        this.errormessage = null;
      }
    },
    userInput() {
      const axios = require("axios");
      var self = this;
      axios
        .get("https://api.openaq.org/v1/locations?has_geo=true", {
          params: {
            coordinates: `${this.latitude},${this.longitude}`,
            radius: 200000,
            order_by: "distance"
          }
        })
        .then(function(response) {
          // handle success
          console.log(response);
          self.lookupMeasurements(response.data.results[0].city);
        })
        .catch(function(error) {
          // handle error
          console.log(error);
        })
        .finally(function() {
          // always executed
        });
    },
    lookupMeasurements: function(city) {
      const axios = require("axios");
      let self = this;
      axios
        .get(`https://api.openaq.org/v1/measurements?city=${encodeURI(city)}`)
        .then(function(response) {
          // handle success
          console.log(response.data.results);
          self.cityLookup = response.data.results;
          self.citylocation = response.data.results[0].city;
          self.measurements1 = null;
        })
        .catch(function(error) {
          // handle error
          console.log(error);
        })
        .finally(function() {
          // always executed
        });
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
ul {
  list-style-type: none;
}
table,
th,
td,
tr {
  border: 1px solid black;
}

li {
  text-align: center;
  margin: 10px;
}

label {
  padding-left: 20px;
  padding-right: 10px;
}

.container {
  display: flex;
  margin-bottom: 20px;
  color: #fff;
}

.bigcontainer {
  background: -webkit-linear-gradient(left, #25c481, #25b7c4);
  background: linear-gradient(to right, #25c481, #25b7c4);
  font-family: "Roboto", sans-serif;
}
td {
  padding: 15px;
  text-align: center;
  font-weight: 300;
  font-size: 18px;
  color: #fff;
  border-bottom: solid 1px rgba(255, 255, 255, 0.1);
}

table {
  width: 100%;
  table-layout: fixed;
}
.tbl-header {
  background-color: rgba(255, 255, 255, 0.3);
}
.tbl-content {
  height: 300px;
  overflow-x: auto;
  margin-top: 0px;
  border: 1px solid rgba(255, 255, 255, 0.3);
}
th {
  padding: 20px 15px;
  text-align: left;
  font-weight: 500;
  font-size: 18px;
  color: #fff;
  text-transform: uppercase;
}

button {
  color: #fff;
  font-size: 18px;
  background-color: #36454f;
  border: none;
  border-radius: 8px;
}
</style>
