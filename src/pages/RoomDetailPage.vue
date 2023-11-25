<template>
  <q-page class="flex flex-col">
    <div
      class="h-[80vh] w-full bg-stone-600 flex gap-4 p-4 flex-col items-center"
    >
      <div class="bg-white w-full h-[30%]">
      <div style="width: 50%">Token: {{ this.storeLogUser.accessToken }}</div>

      </div>
      <div
        class="w-[80%] h-[50%] border rounded-lg bg-white flex flex-col"
      ></div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent } from "vue";
import { useLoginUserStore } from "../stores/loginUserStrore";
export default defineComponent({
  name: "RoomDetailPage",
  data() {
    return {
      storeLogUser: useLoginUserStore()

    }
      
    
  },
});
</script>
