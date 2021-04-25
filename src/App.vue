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
      <div class="tempList" v-if="showList">
        <div class="calendar">
          <label for="date">Date: </label>
          <input
            type="date"
            name="date"
            id="date"
            v-model="selectedDate"
            @change="onSelectDate"
          />
        </div>
        <div class="close" @click="showList = false">x</div>
        <TempItem
          v-for="(temp, i) in temps"
          :key="i"
          :temp="temp.temp"
          :date="temp.date"
          :month="temp.month"
          :year="temp.year"
          :hour="temp.hour"
          :minute="temp.minute"
          :second="temp.second"
          :id="temp._id"
        />
      </div>
    </div>
    <div class="button" @click="onShowList">+</div>
  </div>
</template>

<script>
import mqtt from "mqtt";
import TempItem from "./components/TempItem";
import axios from "axios";

export default {
  data() {
    return {
      connection: {
        // host: "192.168.1.37",
        // host: "172.16.167.2",
        host: "127.0.0.1",
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
        temp: null,
        timestamp: null,
      },
      color: {
        red: 255,
        green: 200,
        blue: 200,
      },
      showList: false,
      nowDate: new Date(),
      selectedDate: null,
      temps: [],
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
    onShowList() {
      this.showList = true;
    },
    async onSelectDate() {
      console.log(this.selectedDate);
      const tmpDate = this.selectedDate.split("-");
      const temp = await axios.get("http://localhost:3000/temp/date", {
        params: { date: tmpDate[2], month: tmpDate[1], year: tmpDate[0] },
      });
      console.log(temp.data.log);
      this.temps = temp.data.log;
    },
  },
  async mounted() {
    this.createConnection();
    this.doSubscribe();
    const month = this.nowDate.getMonth() + 1;
    this.selectedDate =
      this.nowDate.getFullYear() +
      "-" +
      (month < 10 ? "0" + month : month) +
      "-" +
      this.nowDate.getDate();
    const tmpDate = this.selectedDate.split("-");
    const temp = await axios.get("http://192.168.1.35:3000/temp/date", {
      params: { date: tmpDate[2], month: tmpDate[1], year: tmpDate[0] },
    });
    console.log(temp.data.log);
    this.temps = temp.data.log;
  },

  components: {
    TempItem,
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
.button {
  position: fixed;
  right: 5rem;
  bottom: 5rem;
  background: teal;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 4rem;
  height: 4rem;
  border-radius: 50%;
  color: white;
  font-weight: 500;
  font-size: 2rem;
  transition: 200ms;
  cursor: pointer;
}
.button:hover {
  background: rgb(2, 184, 184);
}
.tempList {
  height: 80vh;
  width: 25rem;
  position: fixed;
  top: 50%;
  left: 2rem;
  transform: translateY(-50%);
  overflow-y: auto;
  background: white;
  animation: fade 200ms;
  box-shadow: 0 0 30px 10px rgba(255, 255, 255, 0.6);
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-top:2rem;
  /* justify-content: center; */
}
.close {
  text-align: right;
  font-size: 1.5rem;
  font-weight: thin;
  padding: 1rem;
  cursor: pointer;
  position: fixed;
  top: 0;
  right: 0;
}

.calendar {
  display: flex;
  align-items: center;
  margin-bottom: 1rem;
}
.calendar > label {
  font-size: 1.5rem;
  margin-right: 1rem;
}

@keyframes fade {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
</style>
