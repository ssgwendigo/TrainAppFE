<!--
Could extend this to work for searching bus data as well
would require a bit of retooling in regards to how we setup the table and fetch data
-->

<template>
  <v-form>
        <v-container>
          <v-row>
            <v-col
                cols="12"
                md="4"
            >
              <v-select
                  label="Select your station"
                  :items=stationList
                  item-title="stationName"
                  item-value="stationCode"
                  v-model="this.selectedStation"
                  return-object
              ></v-select>
            </v-col>

            <v-col
                cols="12"
                md="4"
            >
              <v-btn block v-on:click="getStationTimes">Search</v-btn>
            </v-col>

            <v-col
                cols="12"
                md="4"
            >

            </v-col>
          </v-row>
        </v-container>
      </v-form>
<!--  We could probably separate the table component from the station picker for more re-usability -->
  <v-container>
    <v-row>
      <v-col cols="12" md="6">
        <div v-if="stationTimesError">No station data could be found.  Please search another station or try again later.</div>
        <v-table  v-if="stationTimesLoaded">
          <thead>
          <tr>
            <th class="text-left">
              Day
            </th>
            <th class="text-left">
              Opening Time
            </th>
            <th class="text-left">
              First Trains
            </th>
            <th class="text-left">
              Last Trains
            </th>
          </tr>
          </thead>
          <tbody>
            <tr
                v-for="day in this.stationTimes"
            >
              <td>{{ day.day }}</td>
              <td>{{ day.OpeningTime }}</td>
<!--
We're assuming we got a value back for first trains and last trains and should probably be checking
if that's the case or formatting this before we get to the table
if day.FirstTrains.length == 0 display blank string or an error 'no train data available'
-->
              <td>{{ day.FirstTrains.Time }} to station {{ day.FirstTrains.DestinationStation }}</td>
              <td>{{ day.LastTrains.Time }} to station {{ day.LastTrains.DestinationStation }}</td>
            </tr>
          </tbody>
        </v-table>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      stationList: [ ],
      selectedStation: { stationName: 'Metro Center', abbr: 'A01' },
      stationTimes: {},
      stationTimesLoaded: false,
      stationTimesError: false,
      loading: false,
    }
  },
  created() {
    // this.stationList = [{ stationName: 'Florida', abbr: 'stationCode' },
    //   { stationName: 'Georgia', abbr: 'stationCode' },
    //   { stationName: 'Nebraska', abbr: 'stationCode' },
    //   { stationName: 'California', abbr: 'stationCode' },
    //   { stationName: 'New York', abbr: 'stationCode' },];
    this.getStationList()
  },
  methods: {
    getStationList() {
      axios.get('http://localhost:8000/stationList')
      .then(response => {
        console.log(response.data.Stations);
        let stationList = []
        //we just want the station name and station code so we're going to make
        // a new array with just the needed data
        // we also probably could've sorted this on the backend but I'd rather the data come whole
        // and we let FE deal with arranging it how it needs it instead of assuming a
        // general use case at the API level
        for (let i = 0; i < response.data.Stations.length; i++) {
          let code = response.data.Stations[i].Code;
          let name = response.data.Stations[i].Name;
          stationList[i] = {stationName: name, abbr: code};

        }
        //it would also be nice if we alphabetized this to be more user friendly if this were live
        this.stationList = stationList;
      }).catch(error => {
        console.log(error);
      })
    },
    getStationTimes() {
      console.log(this.selectedStation.abbr);
      axios.get('http://localhost:8000/stationTimes/' + this.selectedStation.abbr)
      .then(response => {
        let times = response.data.StationTimes;
        delete times[0].Code;
        delete times[0].StationName;

        Object.keys(times[0]).forEach(key => {
          console.log(key, times[0][key]);
          times[0][key]['day'] = key;
          this.stationList[key] = times[0][key];
        })

        console.log(times);
        //there's probably a prettier way to organize these by day
        this.stationTimes[0] = times[0]['Monday'];
        this.stationTimes[1] = times[0]['Tuesday'];
        this.stationTimes[2] = times[0]['Wednesday'];
        this.stationTimes[3] = times[0]['Thursday'];
        this.stationTimes[4] = times[0]['Friday'];
        this.stationTimes[5] = times[0]['Saturday'];
        this.stationTimes[6] = times[0]['Sunday'];

        console.log(this.stationTimes);
        this.stationTimesLoaded = true;
      })
      .catch(error => {
        console.log(error);
      })
    }
  }
}
</script>

<!--<script setup>-->
<!--import { beforeCreate } from 'vue';-->


<!--beforeCreate(() => {-->

<!--})-->

<!--</script>-->

<!--<template>-->
<!--  <v-form v-model="valid">-->
<!--    <v-container>-->
<!--      <v-row>-->
<!--        <v-col-->
<!--            cols="12"-->
<!--            md="4"-->
<!--        >-->
<!--          <v-select-->
<!--              label="Select your station"-->
<!--              :items=stationList-->
<!--          ></v-select>-->
<!--        </v-col>-->

<!--        <v-col-->
<!--            cols="12"-->
<!--            md="4"-->
<!--        >-->
<!--          <v-text-field-->
<!--              v-model="lastname"-->
<!--              :rules="nameRules"-->
<!--              :counter="10"-->
<!--              label="Last name"-->
<!--              hide-details-->
<!--              required-->
<!--          ></v-text-field>-->
<!--        </v-col>-->

<!--        <v-col-->
<!--            cols="12"-->
<!--            md="4"-->
<!--        >-->
<!--          <v-text-field-->
<!--              v-model="email"-->
<!--              :rules="emailRules"-->
<!--              label="E-mail"-->
<!--              hide-details-->
<!--              required-->
<!--          ></v-text-field>-->
<!--        </v-col>-->
<!--      </v-row>-->
<!--    </v-container>-->
<!--  </v-form>-->
<!--</template>-->
