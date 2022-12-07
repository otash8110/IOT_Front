<!-- eslint-disable vue/no-side-effects-in-computed-properties -->
<template>
  <v-container>
    <v-row justify="center" class="flex-column">
      <v-col>
        <v-card v-if="enterHistory">
            <template v-slot:title>
              Last Visitor
            </template>
            <template v-slot:text>
              Last entered: {{LastPersonEntered}}
            </template>
        </v-card>
      </v-col>
      <v-col>
        <v-row >
          <v-col cols="12">
            <v-card class="mx-auto"
            >
              <v-card-item>
                <v-icon size="x-large" color="red">mdi-home-thermometer-outline</v-icon>
                <v-card-title class="text-xs-h6 text-md-h5 text-lg-h4">
                  Temperature
                </v-card-title>
              </v-card-item>
              <v-card-text>
                <Line
                  id="my-chart-temp"
                  :options="chartOptions"
                  :data="ChartDataTemp"
                  ref="bar"
                />
              </v-card-text>
            </v-card>
          </v-col>
          
          <v-col>
            <v-card>
              <v-card-item>
                <v-icon size="x-large" color="blue">mdi-water-percent</v-icon>
                <v-card-title class=" text-xs-h6 text-md-h5 text-lg-h4">
                  Humidity
                </v-card-title>
              </v-card-item>
              <v-card-text>
                <Line
                  id="my-chart-hum"
                  :options="chartOptions"
                  :data="ChartDataHum"
                  ref="bar"
                />
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </v-col>
      <v-col>
        <v-row>
          <v-col cols="7">
            <v-card>
              <v-card-item>
                <v-icon size="x-large" color="black">mdi-lock-open</v-icon>
                <v-card-title>
                  Door lock
                </v-card-title>
              </v-card-item>
              <v-card-text>
                <v-switch
                v-model="lockState"
                @change="HandleLock"
                color="green"
                :label="DoorSwitch" inset></v-switch>
              </v-card-text>
            </v-card>
          </v-col>
          <v-col cols="5">
            <v-card>
              <v-card-item>
                <v-icon size="x-large" color="brown">mdi-door-open</v-icon>
                <v-card-title>
                  Door lock
                </v-card-title>
              </v-card-item>
              <v-card-text class="text-center">
                <v-btn stacked
                color="green"
                prepend-icon="mdi-lock-open" variant="flat" v-if="doorState">
                  Door Open
                </v-btn>
                <v-btn stacked
                color="red"
                prepend-icon="mdi-lock" variant="flat" v-if="!doorState">
                  Door Closed
                </v-btn>
              </v-card-text>
            </v-card>
          </v-col>
          <v-col cols="12">
            <v-card>
              <v-card-item>
                <v-icon size="x-large" color="yellow">mdi-lightbulb-on</v-icon>
                <v-card-title>
                  Kitchen Light
                </v-card-title>
              </v-card-item>
              <v-card-text class="text-center">
                <v-switch
                v-model="ledState"
                @change="HandleLed"
                color="green"
                :label="LedState" inset></v-switch>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </v-col>
      <v-col>
        <v-table theme="dark">
        <thead>
          <tr>
            <th class="text-left">
              Id
            </th>
            <th class="text-left">
              Member
            </th>
            <th class="text-left">
              Time
            </th>
          </tr>
        </thead>
        <tbody>
          <tr
          v-for="item in ArrayOfEnteredCards"
          :key="item.Time">
            <td>{{item.Id}}</td>
            <td>{{item.Member}}</td>
            <td>{{item.Time}}</td>
          </tr>
        </tbody>
      </v-table>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from 'axios';
import { Line } from "vue-chartjs";
import { Chart as ChartJS, Title, Tooltip, Legend, LineElement, CategoryScale, LinearScale, PointElement } from 'chart.js';

ChartJS.register(Title, Tooltip, Legend, LineElement, CategoryScale, LinearScale, PointElement)


export default {
  name: 'HelloWorld',
  components: {
    Line
  },

  props: ['handleLockFunction', 'handleLedFunction'],

  data: () => ({
    member: "Mother",
    isMember: false,
    enterHistory: null,
    temperature: 0,
    humidity: 0,
    lockState: false,
    ledState: false,
    doorState: false,

    timeLables: [],
    dataTemp: [],
    lablesHum: [],
    dataHum: [],
    chartOptions: {
        responsive: true
      },
    chartLabelTemp: "Temperature",
    chartItemBackgroundColorTemp: "#f87979",

    chartLabelHum: "Humidity",
    chartItemBackgroundColorHum: "#79c0f8"
  }),

  computed: {
    ChartDataTemp() {
        return {
          labels: this.timeLables.slice(),
          datasets: [ { data: this.dataTemp.slice(), label: this.chartLabelTemp, backgroundColor: this.chartItemBackgroundColorTemp }]
      };
    },

    ChartDataHum() {
      return {
          labels: this.timeLables.slice(),
          datasets: [ { data: this.dataHum.slice(), label: this.chartLabelHum, backgroundColor: this.chartItemBackgroundColorHum }]
      };
    },
    LastPersonEntered() {
      if (this.isMember) {
        var lastKey = Object.keys(this.enterHistory).pop();
        return this.enterHistory[lastKey].Member;
      } else {
        return 0;
      }
    },

    ArrayOfEnteredCards() {
      var result = [];
      // Make array out of object of enter history received from backend
      if (this.isMember) {
        for (var property in this.enterHistory) {
          if (Object.hasOwn(this.enterHistory, property)) {
            var object = this.enterHistory[property];
            var finalObject = {
              "Id": object.Id,
              "Member": object.Member,
              "Time": object.Time
            };
            result.push(finalObject);
          }
        }

        return result.reverse();
      } else {
        return 0;
      }
    },

    DoorSwitch() {
      return this.lockState ? "Lock is open" : "Lock is closed";
    },

    DoorState() {
      return this.doorState ? true : false;
    },

    LedState() {
      return this.ledState ? "Light is on" : "Light is off";
    }
  },

  async mounted() {
    await this.FetchAllEnterHistory();
  },

  watch: {
    ChartData() {
      this.chartUpdateFlag = false;
    },
    temperature(newTemperature) {
      if(newTemperature > 0) {
        if (this.dataTemp.length > 4) {
          this.dataTemp.shift();
          this.timeLables.shift();

          this.dataHum.shift();
          this.lablesHum.shift();
        }
        var date = new Date();
        var currentDate = date.toLocaleTimeString();
        this.dataTemp.push(this.temperature);
        this.dataHum.push(this.humidity);
        this.timeLables.push(currentDate);
      }
    }
  },

  methods: {
    async FetchAllEnterHistory() {
      var fetchUrl =  process.env.HistoryEnterURL || "http://localhost:3000/enterhistory";
      axios.get(fetchUrl).then(res => {
        this.enterHistory = res.data;
        this.isMember = true;
      })

      console.log("CALLED");
    },
    UpdateTempAndHumidity(message) {
      var messageValue = message.toString();
      const [temperature, humidity] = messageValue.split(",");

      this.temperature = temperature;
      this.humidity = humidity;

    },

    UpdateDoorState(message) {
      var messageValue = message.toString();
      console.log(messageValue);
      switch (messageValue) {
        case ("1"):
          this.doorState = true;
          break;
        case ("0"):
          this.doorState = false;
          break;
      }
    },

    HandleLock() {
      this.handleLockFunction(this.lockState);
    },

    HandleLed() {
      this.handleLedFunction(this.ledState);
    }
  }

}
</script>