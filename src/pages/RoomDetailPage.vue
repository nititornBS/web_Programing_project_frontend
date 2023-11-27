<template>
  <q-page class="flex flex-col">
    <div
      class="h-[95vh] w-full flex gap-4 flex-col items-center justify-center bg-gray-400"
    >
      <div
        class="w-[80%] h-[90%] border rounded-lg bg-white drop-shadow-2xl border-gray-500"
      >
        <div class="flex flex-col h-full w-full">
          <div
            class="grid grid-cols-2 justify-center items-center p-5 h-[20%] w-full"
          >
            <div class="font-bold text-2xl h-full text-center">
              <div>{{ roomDetails.RoomNumber }}</div>
              <div>{{ date }}</div>
            </div>
            <div class="text-gray-500">Saturday, November 25, 2023</div>
          </div>
          <div class="flex flex-col justify-center h-[70%] border">
            <div class="border border-y-black">
              <div
                v-for="(val, index) in timeID.slice(0, -1)"
                :key="index"
                class="flex items-center justify-between p-2 cursor-pointer"
                :class="{ ' bg-gray-400': index % 2 !== 0 }"
              >
                <div class="grid grid-cols-2 w-full">
                  <div
                    v-if="index < timeID.length - 1"
                    class="text-xl flex justify-center"
                  >
                    {{ val.Time }}
                    -------------
                    {{ timeID[index + 1].Time }}
                  </div>
                  <div class="w-full">
                    <q-btn
                      v-if="selectedTime[index]"
                      class="flex justify-center items-center"
                      @click="removeTime(index)"
                      label="selected"
                    />
                    <q-btn
                      v-else
                      class="flex justify-center items-center text-red-500"
                      color="purple"
                      label="booking"
                      @click="selectTime(index)"
                    />
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="p-4 h-[10%] items-center grid-cols-3 grid w-full">
            <div class="col-span-2">
              ToTal {{ counthour() }} hour
              {{ counthour() * roomdata.CurrentPricePerHour }}
            </div>
         

            <div class="justify-end flex">
              <button
                class="bg-blue-500 text-white rounded-md px-4 py-2"
                @click="showResave()"
              >
                Book Now
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent, ref } from "vue";
import { useLoginUserStore } from "src/stores/loginUserStrore";
import { Notify } from "quasar";

export default defineComponent({
  name: "RoomDetailPage",
setup() {
  const currentDate = new Date();
  const formattedDate = ref(currentDate.toISOString().split('T')[0]); // Extracts the date part
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
      timeID: [],
      roomDetails: {},
      roomdata: {},
      TotalPrice: 0,
      selectedTime: [],
      starttime: 0,
      endtime: 0,

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
    showResave() {
      console.log(
        "start Time " + this.starttime + "/// end time " + this.endtime
      );
    },
    counthour() {
      var count = 0;
      this.selectedTime.map((times) => {
        if (times != null) {
          count++;
        }
      });
      return count;
    },

    getAlltime() {
      console.log("Get All Time ->> token:" + this.storeLogUser.accessToken);
      const headers = {
        "x-access-token": this.storeLogUser.accessToken,
      };
      console.log("headers:" + JSON.stringify(headers));
      this.$api
        .get("/booking/gettime", { headers })
        .then((res) => {
          if (res.status == 200) {
            res.data.map((timesObj) => {
              this.timeID.push({
                Id: timesObj.Id,
                Time: timesObj.Time,
              });
            });
            console.log("show time " + this.timeID);
          }
        })
        .catch((err) => {
          console.log(err);
          Notify.create({
            type: "negative",
            message: "Unauthorized",
          });
        });
    },
    getRomdetail() {
      console.log("Get room data->> token:" + this.storeLogUser.accessToken);
      const headers = {
        "x-access-token": this.storeLogUser.accessToken,
      };
      console.log("headers:" + JSON.stringify(headers));
      this.$api
        .get("/booking/showAllroomdetail", { headers })
        .then((res) => {
          if (res.status == 200) {
            res.data.map((timesObj) => {
              if (timesObj.SizeID == this.roomDetails.SizeID) {
                this.roomdata = timesObj;
              }
            });
          }
        })
        .catch((err) => {
          console.log(err);
          Notify.create({
            type: "negative",
            message: "Unauthorized",
          });
        });
    },
    selectTime(index) {
      const selectedId = this.timeID[index].Id;

      if (this.starttime === 0 && this.endtime === 0) {
        // If both starttime and endtime are null, set both to the selected time
        this.starttime = selectedId;
        this.endtime = selectedId;
      } else {
        // If either starttime or endtime is not null, adjust accordingly
        if (selectedId < this.starttime) {
          this.starttime = selectedId;
        } else if (selectedId > this.endtime) {
          this.endtime = selectedId;
        }
      }

      // Store the selected time in the array
      this.selectedTime[index] = selectedId;

      console.log(
        "start Time " + this.starttime + "/// end time " + this.endtime
      );
    },
    removeTime(index) {
      // Remove the selected time
      var temp = this.selectedTime[index];

      // Recalculate start and end times
      if (this.starttime === this.endtime) {
        this.starttime = 0;
        this.endtime = 0;
      } else {
        this.selectedTime.map((time) => {
          console.log(time + "time");
          if (time !== null) {
            if (temp === this.starttime) {
              this.starttime = temp + 1;
            }
            if (temp === this.endtime) {
              this.endtime = temp - 1;
            }
          }
        });
      }

      this.selectedTime[index] = null;
      console.log(
        "start Time " + this.starttime + "/// end time " + this.endtime
      );
    },
  },
  calculateTotalPrice() {
    this.TotalPrice =
      this.selectedTime.length * this.roomdata.CurrentPricePerHour;
  },

  async mounted() {
    await this.getAlltime();
    // Access the route params to get the object passed from the previous page
    this.roomDetails = JSON.parse(decodeURIComponent(this.$route.params.val));
    await this.getRomdetail();
    console.log(this.roomDetails);
    console.log("Received room details:", this.roomDetails);
  },
});
</script>
