<script setup>
import moment from "moment";
import { reactive, ref } from "vue";
import { VueDatePicker } from "@vuepic/vue-datepicker";
import "@vuepic/vue-datepicker/dist/main.css";

/* ---------------- data ---------------- */
const rooms = reactive([
  {
    name: "A",
    books: [
      { start: "19/12/1999 09:00", end: "19/12/1999 10:00" },
      { start: "19/12/1999 14:00", end: "19/12/1999 15:00" },
    ],
  },
  {
    name: "B",
    books: [
      { start: "19/12/1999 10:00", end: "19/12/1999 12:00" },
      { start: "19/12/1999 16:00", end: "19/12/1999 18:00" },
    ],
  },
]);

const dateSel = ref(null);
const timeSel = ref(null);
const result = ref(null);
const error = ref(null);

const toMinutes= (date) => date.getHours() * 60 + date.getMinutes();

// function
function autoBookMeetingRoom() {
  error.value = null;
  result.value = null;

  // input validate
  if (!dateSel.value || !timeSel.value) {
    error.value = "กรุณาเลือกวันที่และเวลา";
    return;
  }

  const [startTime, endTime] = timeSel.value;

  // datetime
  const start = toMinutes(moment(dateSel.value)
    .hour(moment(startTime).hour())
    .minute(moment(startTime).minute()))

  const end = toMinutes(moment(dateSel.value)
    .hour(moment(endTime).hour())
    .minute(moment(endTime).minute()));

  const workStart = moment(dateSel.value).hour(9).minute(0);
  const workEnd = moment(dateSel.value).hour(18).minute(0);

  const breakStart = moment(dateSel.value).hour(12).minute(0);
  const breakEnd = moment(dateSel.value).hour(13).minute(0);

  // validate
  if (!start.isBefore(end)) {
    error.value = "เวลาเริ่มต้องน้อยกว่าเวลาสิ้นสุด";
    return;
  }

  if (start.isBefore(workStart) || end.isAfter(workEnd)) {
    error.value = "อยู่นอกเวลาทำงาน (09:00 - 18:00)";
    return;
  }

  if (start.isBefore(breakEnd) && end.isAfter(breakStart)) {
    error.value = "ตรงกับเวลาพัก (12:00 - 13:00)";
    return;
  }

  // check booking
  // const checkBook = []
  // for (const room of rooms) {
  //   const hasConflict = room.books.some((b) => {
  //     const bookedStart = moment(b.start, "DD/MM/YYYY HH:mm");
  //     const bookedEnd = moment(b.end, "DD/MM/YYYY HH:mm");

  //     return moment(start).isBetween(bookedStart,bookedEnd) && moment(end).isBetween(bookedStart, bookedEnd);
  //   });
  //   checkBook.push({name: room.name, isUsing: hasConflict, msg: hasConflict ? 'ห้องไม่ว่าง': 'ห้องว่าง'})
  // }

  // error.value = "ไม่มีห้องว่างในช่วงเวลานี้";
}
</script>

<template>
  <div style="max-width: 400px">
    <label>วันที่จอง</label>
    <VueDatePicker
      v-model="dateSel"
      :time-config="{ enableTimePicker: false }"
      :formats="{ preview: 'dd/MM/yyyy' }"
    />

    <label style="margin-top: 12px; display: block">เวลาที่จอง</label>
    <VueDatePicker v-model="timeSel" time-picker range />

    <button style="margin-top: 16px" @click="autoBookMeetingRoom">
      เช็คห้องประชุม
    </button>

    <p v-if="result" style="color: green">{{ result }}</p>
    <p v-if="error" style="color: red">ข้อผิดพลาด: {{ error }}</p>
    <div>---------------------------------------------------</div>
    <table>
      <thead>
        <th>ห้อง</th>
        <th>วันที่เวลา</th>
      </thead>
      <tbody>
        <template v-for="(room, index) in rooms" :key="index">
          <tr v-for="(book,bIdx) in room.books" :key="bIdx">
            <td>{{ room.name}}</td>
            <td>
              {{ book.start }} - {{ book.end }}
            </td>
          </tr>
        </template>
      </tbody>
    </table>
  </div>
</template>
