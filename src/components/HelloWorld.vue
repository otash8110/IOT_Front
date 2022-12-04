<template>
  <v-container>
    <v-row justify="center" class="flex-column">
      <v-col>
        <v-card>
        <v-card-title v-if="enterHistory">
          Last entered: {{LastPersonEntered}}
        </v-card-title>
      </v-card>
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
import axios from 'axios'

export default {
  name: 'HelloWorld',

  data: () => ({
    member: "Mother",
    isMember: false,
    enterHistory: null
  }),

  computed: {
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
    }
  },

  async mounted() {
    await this.FetchAllEnterHistory();
  },

  methods: {
    async FetchAllEnterHistory() {
      var fetchUrl = "http://localhost:3000/enterhistory";
      axios.get(fetchUrl).then(res => {
        this.enterHistory = res.data;
        this.isMember = true;
      })

      console.log("CALLED");
    }
  }

}
</script>