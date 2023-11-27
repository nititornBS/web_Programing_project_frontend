<template>
  <q-page class="flex flex-col">
    <div class="flex justify-center items-center h-[20vh] w-full bg-gray-500">
      <div class="bg-white h-[80%] w-[95%] border rounded-md flex">
        <div class="w-full h-full flex">
          <div class="q-pa-md w-[40%] h-full">
            <q-input filled v-model="date" mask="date" :rules="['date']">
              <template v-slot:append>
                <q-icon name="event" class="cursor-pointer">
                  <q-popup-proxy
                    cover
                    transition-show="scale"
                    transition-hide="scale"
                  >
                    <q-date v-model="date" @input="onDateInput">
                      <div class="row items-center justify-end">
                        <q-btn
                          v-close-popup
                          label="Close"
                          color="primary"
                          flat
                        />
                      </div>
                    </q-date>
                  </q-popup-proxy>
                </q-icon>
              </template>
            </q-input>
            <div>{{ date }}</div>
          </div>
          <div class="my-[25px]">
            <q-btn push color="primary" label="Push" @click="getIncome()" />
          </div>
        </div>
        <!-- <q-datetime v-model="selectedDateTime" label="Select Date and Time" /> -->
      </div>
    </div>
    <div class="container mx-auto p-4">
      <div class="grid grid-cols-2 gap-4">
        <div class="bg-white p-4 rounded shadow">
          <h2 class="text-xl font-bold mb-4">Today's Overview</h2>
          <div class="flex justify-between mb-2">
            <div>Total Income Today: {{ totalIncomeToday }}</div>
          </div>
          <!-- Other sections for total users and total hours -->
        </div>

        <div class="bg-white p-4 rounded shadow">
          <h2 class="text-xl font-bold mb-4">Weekly Overview</h2>
          <canvas ref="chart"></canvas>
        </div>
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent } from "vue";
import { useLoginUserStore } from "../stores/loginUserStrore";
import { Notify } from "quasar";
import { ref } from "vue";

export default defineComponent({
  name: "DashBoard",
  setup() {
    const currentDate = new Date();
    const formattedDate = ref(currentDate.toISOString().split("T")[0]); // Extracts the date part
    console.log(formattedDate);
    const selectedDate = ref({
      day: currentDate.getDate(),
      month: currentDate.getMonth() + 1,
      year: currentDate.getFullYear(),
    });

    return {
      date: formattedDate,
      selectedDate,
    };
  },
  data() {
    return {
      totalIncomeToday: 0,
      storeLogUser: useLoginUserStore(),
      dialog: {
        icon: "",
        msg: "",
        btnType: "",
        iconColor: "primary",
      },
    };
  },
  methods: {
    onDateInput(selectedDate) {
      // This method will be called whenever the user selects a date
      console.log("Selected Date:", selectedDate);
      // Update any other logic based on the selected date if needed
      this.getIncome();
    },
    getIncome() {
      const data = {
        BookingDate: this.date,
      };
      const headers = {
        "x-access-token": this.storeLogUser.accessToken,
      };
      this.$api
        .put("booking/getTotalIncome", data, { headers })
        .then((res) => {
          if (res.status == 200) {
            Notify.create({
              type: "positive",
              message: "get data  ",
            });
            console.log(res.data)
            this.totalIncomeToday = res.data
          }
        })
        .catch((err) => {
          console.log(err);
          this.showErrDialog(err);
        });
    },
    showErrDialog(err) {
      this.showDialog = true;
      this.dialog.icon = "error";
      this.dialog.iconColor = "negative";
      this.dialog.msg = err;
      this.dialog.btnType = "Error";
    },
    fetchWeeklyOverview() {
      // Replace this with your actual API call to get weekly overview data
      // Example: axios.get('/api/weeklyOverview').then(response => { ... });

      // For this example, I'll create a dummy dataset
      const labels = ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"];
      const data = [1500, 2000, 3000, 2500, 1800, 4000, 6000];

      // Render the chart
      this.renderChart(labels, data);
    },
  },

  async mounted() {
    await this.getIncome();
  },
});
</script>
