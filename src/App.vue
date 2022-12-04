<template>
  <v-app>
    <v-main>
      <HelloWorld ref="mainPageComponent" />
    </v-main>
  </v-app>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import mqtt from "mqtt/dist/mqtt.min";

var topics = {
    RFID_Check: "RFID_Check",
    RFID_Check_Result: "RFID_Check_Result"
};


export default {
  name: 'App',

  components: {
    HelloWorld,
  },

  data: () => ({
    connection: {
        host: "j9f09aa1-internet-facing-ef1d133022690295.elb.us-east-1.amazonaws.com",
        protocol: "ws",
        clientId: "node-front-client",
        username: "ivanov",
        password: "ivanivanov",
        port: "8083",
        endpoint: "/mqtt",
        clean: true
    }
  }),
  methods: {
    createConnection() {
      try {
        this.connecting = true;
        const { protocol, host, port, endpoint, ...options } =
            this.connection;
        const connectUrl = `${protocol}://${host}:${port}${endpoint}`;
        this.client = mqtt.connect(connectUrl, options);
        if (this.client.on) {
          this.client.on("connect", () => {
            this.connecting = false;
            console.log("Connection succeeded!");
          });
          this.client.on("error", (error) => {
            console.log("Connection failed", error);
          });
          this.client.on("message", (topic, message) => {
            console.log(`Received message ${message} from topic ${topic}`);
            if (topic == topics.RFID_Check_Result) {
              this.$refs.mainPageComponent.FetchAllEnterHistory();
            }
          });

          this.client.subscribe(topics.RFID_Check_Result, () => {
            console.log("subscribed to", topics.RFID_Check);
          })
        }
      } catch (error) {
        this.connecting = false;
        console.log("mqtt.connect error", error);
      }
    }
  },
  
  mounted() {
    this.createConnection();
  }
}
</script>
