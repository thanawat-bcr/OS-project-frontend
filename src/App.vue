<template>
  <div id="app">
    <div
      class="container"
      :style="{
        backgroundColor: `rgb(${color.red},${color.green},${
          color.blue - msg.temp
        })`,
      }"
    >
      <div>{{ msg.temp }} °C</div>
      <div>{{ msg.timestamp }}</div>
    </div>
  </div>
</template>

<script>
import mqtt from "mqtt";

export default {
  data() {
    return {
      connection: {
        host: "172.16.167.2",
        // host: "127.0.0.1",
        port: 9001,
        endpoint: "",
        clean: true, // Reserved session
        // connectTimeout: 4000, // Time out
        // reconnectPeriod: 4000, // Reconnection interval
        // Certification Information
        clientId: "vuejs0001",
        username: "tutorism",
        password: "1234",
      },
      subscription: {
        topic: "test",
        qos: 0,
      },
      publish: {
        topic: "test",
        qos: 0,
        payload: '{ "msg": "Hello, I am browser." }',
      },
      receiveNews: "",
      qosList: [
        { label: 0, value: 0 },
        { label: 1, value: 1 },
        { label: 2, value: 2 },
      ],
      client: {
        connected: false,
      },
      subscribeSuccess: false,
      msg: {
        temp: Math.round(Math.random() * 100),
        timestamp: new Date(Date.now()).toString().split("G")[0],
      },
      color: {
        red: 255,
        green: 200,
        blue: 200,
      },
    };
  },

  methods: {
    createConnection() {
      const { host, port, endpoint, ...options } = this.connection;
      const connectUrl = `mqtt://${host}:${port}${endpoint}`;
      try {
        this.client = mqtt.connect(connectUrl, options);
      } catch (error) {
        console.log("mqtt.connect error", error);
      }
      this.client.on("connect", () => {
        console.log("Connection succeeded!");
      });
      this.client.on("error", (error) => {
        console.log("Connection failed", error);
      });
      this.client.on("message", (topic, message) => {
        this.receiveNews = this.receiveNews.concat(message);
        console.log(`Received message ${message} from topic ${topic}`);
        if (topic == "temp") this.msg.temp = message;
        if (topic == "timestamp") this.msg.timestamp = message;
      });
    },
    doPublish() {
      // test publishing
      this.client.publish("test", "Hello mqtt");
    },
    doSubscribe() {
      this.client.subscribe("temp");
      this.client.subscribe("timestamp");
    },
  },
  mounted() {
    this.createConnection();
    this.doSubscribe();
    // Test color running
    // setInterval(() => {
    //   this.msg.temp = Math.round(Math.random() * 100);
    //   this.msg.timestamp = new Date(Date.now()).toString().split("G")[0];
    // }, 1000);
  },
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  min-height: 100vh;
}
.container {
  min-height: 100vh;
  width: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  transition: 1s;
  font-size: 2rem;
}
</style>
