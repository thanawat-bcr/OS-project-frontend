<template>
  <div>
    <div class="box" v-if="!editMode">
      <div class="temp">{{ temp }}°C</div>
      <div>
        <div class="date">{{ date }} {{ this.formattedMonth }} {{ year }}</div>
        <div class="time">{{ hour }}:{{ minute }}:{{ second }}</div>
      </div>
      <div class="icon-group">
        <div class="icon delete" @click="onDelete"></div>
        <div class="icon edit" @click="onUpdate"></div>
      </div>
    </div>
    <div class="box-input" v-else>
      <div><input class="temp" v-model="updated.temp" />°C</div>
      <div>
        <div class="input-date">
          <input type="text" v-model="updated.date" />
          <input type="text" v-model="updated.month" />
          <input type="text" class="input-year" v-model="updated.year" />
        </div>
        <div class="input-time">
          <input type="text" v-model="updated.hour" />
          <input type="text" v-model="updated.minute" />
          <input type="text" v-model="updated.second" />
        </div>
      </div>
      <div class="submit" @click="onSubmit">
        <div>OK</div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  props: ["temp", "date", "month", "year", "hour", "minute", "second", "id"],
  computed: {
    formattedMonth() {
      if (this.month == "01") return "Jan";
      if (this.month == "02") return "Feb";
      if (this.month == "03") return "Mar";
      if (this.month == "04") return "Apr";
      if (this.month == "05") return "May";
      if (this.month == "06") return "Jun";
      if (this.month == "07") return "July";
      if (this.month == "08") return "Aug";
      if (this.month == "09") return "Sep";
      if (this.month == "10") return "Oct";
      if (this.month == "11") return "Nov";
      if (this.month == "12") return "Dec";
      return "month";
    },
  },
  data() {
    return {
      editMode: false,
      updated: {
        temp: this.temp,
        date: this.date,
        month: this.month,
        year: this.year,
        hour: this.hour,
        minute: this.minute,
        second: this.second,
      },
    };
  },
  methods: {
    async onDelete() {
      await axios.get("http://localhost:3000/temp/remove", {
        params: { id: this.id },
      });
      location.reload();
    },
    onUpdate() {
      this.editMode = true;
    },
    async onSubmit() {
      await axios.get("http://localhost:3000/temp/update", {
        params: {
          id: this.id,
          temp: this.updated.temp,
          date: this.updated.date,
          month: this.updated.month,
          year: this.updated.year,
          hour: this.updated.hour,
          minute: this.updated.minute,
          second: this.updated.second,
        },
      });
      location.reload();
    },
  },
};
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
}
.box {
  background: white;
  width: 22rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.5rem 1rem;
  border-bottom-width: 1px;
  border-bottom-style: solid;
  border-bottom-color: black;
}
.box-input {
  background: white;
  width: 22rem;
  display: flex;
  align-items: center;
  padding: 0.5rem 1rem;
  border-bottom-width: 1px;
  border-bottom-style: solid;
  border-bottom-color: black;
}
.temp {
  font-weight: bold;
  font-size: 2.75rem;
}
.date {
  font-weight: 500;
  font-size: 1.35rem;
}
.time {
  font-weight: 300;
  font-size: 1rem;
}
.icon {
  padding: 0.5rem;
  cursor: pointer;
  width: 1.5rem;
  height: 1.5rem;
  background-repeat: no-repeat;
  background-size: 1.5rem;
  margin: 0 0.5rem;
}
.delete {
  background-image: url("../assets/icons/delete.svg");
}
.edit {
  background-image: url("../assets/icons/edit.svg");
}
.icon-group {
  display: flex;
}
.box-input input.temp {
  width: 4rem;
  margin-right: 0.5rem;
  text-align: center;
}
.input-date,
.input-time {
  display: flex;
}
.input-date input,
.input-time input {
  width: 3rem;
  margin: 0.5rem;
  padding: 0.5rem 0;
  text-align: center;
}
.input-date input.input-year {
  width: 4rem;
}
.input-date input {
  font-weight: 500;
  font-size: 1.35rem;
}
.input-time input {
  font-weight: 300;
  font-size: 1rem;
}
.submit {
  padding: 0.5rem 1rem;
  font-size: 1.5rem;
  cursor: pointer;
  border-radius: 0.5rem;
  background: lightcoral;
}
.submit:hover {
  background: lightsalmon;
}
</style>