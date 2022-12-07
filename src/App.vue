<template>
  <v-app>
    <v-main>
      <HelloWorld ref="mainPageComponent"
      :handle-lock-function="LockChangeRequest"
      :handle-led-function="LedStatChangeRequest" />
    </v-main>
  </v-app>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import mqtt from "mqtt/dist/mqtt.min";

var topics = {
    RFID_Check: "RFID_Check",
    RFID_Check_Result: "RFID_Check_Result",
    DHT_Send: "DHT_Send",
    DoorStateTopic: "Door_State",
    LockStateTopic: "Lock_State",
    LedStateTopic: "LED_State"
};


export default {
  name: 'App',

  components: {
    HelloWorld,
  },

  data: () => ({
    connection: {
        host: "vfc64260-internet-facing-376ebf692d75cbe3.elb.us-east-1.amazonaws.com",
        protocol: "wss",
        clientId: "node-front-client",
        username: "ivanov",
        password: "ivanivanov",
        port: "8084",
        endpoint: "/mqtt",
        clean: true,
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
            switch (topic) {
              case (topics.RFID_Check_Result):
                this.$refs.mainPageComponent.FetchAllEnterHistory();
                break;
              case (topics.DHT_Send): 
                this.$refs.mainPageComponent.UpdateTempAndHumidity(message);
                break;
              case (topics.DoorStateTopic):
                this.$refs.mainPageComponent.UpdateDoorState(message);
                break;
              }
            }
          );

          this.client.subscribe(topics.RFID_Check_Result, () => {
            console.log("subscribed to", topics.RFID_Check);
          })

          this.client.subscribe(topics.DHT_Send, () => {
            console.log("subscribed to", topics.DHT_Send);
          })

          this.client.subscribe(topics.DoorStateTopic, () => {
            console.log("subscribed to", topics.DoorStateTopic);
          })
        }
      } catch (error) {
        this.connecting = false;
        console.log("mqtt.connect error", error);
      }
    },

    LockChangeRequest(lockState) {
      var value;
      if (lockState) {
        value = "0";
        this.client.publish(topics.LockStateTopic, value)
      } else {
        value = "2";
        this.client.publish(topics.LockStateTopic, value);
      }
    },

    LedStatChangeRequest(ledState) {
      var value;
      if (ledState) {
        value = "1";
        this.client.publish(topics.LedStateTopic, value)
      } else {
        value = "0";
        this.client.publish(topics.LedStateTopic, value);
      }
    }
  },
  
  mounted() {
    this.createConnection();
  }
}
</script>
