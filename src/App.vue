<script setup>
import moment from "moment";
import { reactive, ref } from "vue";
import { VueDatePicker } from "@vuepic/vue-datepicker";
import "@vuepic/vue-datepicker/dist/main.css";

const rooms = reactive([
  {
    name: "A",
    books: [
      { start: "2025-12-21 09:00", end: "2025-12-21 10:00" },
      { start: "2025-12-21 14:00", end: "2025-12-21 15:00" },
    ],
  },
  {
    name: "B",
    books: [
      { start: "2025-12-21 10:00", end: "2025-12-21 12:00" },
      { start: "2025-12-21 16:00", end: "2025-12-21 18:00" },
    ],
  },
]);

const date = ref(moment().format('YYYY-MM-DD'));
const timeStart = ref(moment().format('HH:mm'));
const timeEnd = ref(moment().format('HH:mm'));
const result = ref(null);
const error = ref(null);

const toMinutes = (m) => m.hours() * 60 + m.minutes()

// function
function checkBooked(d, ts, te) {
  error.value = null;
  result.value = null;

  // input validate
  if (!d || !ts || !te) {
    error.value = "กรุณาเลือกวันที่ เวลาเริ่ม และเวลาสิ้นสุด";
    return;
  }

  // datetime
  const start = moment(`${d} ${ts}`, "YYYY-MM-DD HH:mm");
  const end = moment(`${d} ${te}`, "YYYY-MM-DD HH:mm");
  const workStart = moment(`${d} 9:00`, "YYYY-MM-DD HH:mm");
  const workEnd = moment(`${d} 18:00`, "YYYY-MM-DD HH:mm");
  const breakStart = moment(`${d} 12:00`, "YYYY-MM-DD HH:mm");
  const breakEnd = moment(`${d} 13:00`, "YYYY-MM-DD HH:mm");
  const startMn = toMinutes(start)
  const endMn = toMinutes(end)
  const workStartMn = toMinutes(workStart)
  const workEndMn = toMinutes(workEnd)
  const breakStartMn = toMinutes(breakStart)
  const breakEndMn = toMinutes(breakEnd)

  // validate
  if (startMn >= endMn) {
    error.value = "เวลาเริ่มต้องน้อยกว่าเวลาสิ้นสุด";
    return;
  }

  if (startMn < workStartMn || endMn > workEndMn) {
    error.value = "อยู่นอกเวลาทำงาน (09:00 - 18:00)";
    return;
  }

  if (startMn <= breakEndMn && endMn >= breakStartMn) {
    error.value = "ตรงกับเวลาพัก (12:00 - 13:00)";
    return;
  }

  // check booking
  const roomResult = []
  for (const room of rooms) {
    const isConflict = room.books.some((b) => {
      const bookedStart = moment(b.start, "YYYY-MM-DD HH:mm");
      const bookedEnd = moment(b.end, "YYYY-MM-DD HH:mm");
      const bookedStartMn = toMinutes(bookedStart);
      const bookedEndMn = toMinutes(bookedEnd);
      return startMn < bookedEndMn && endMn > bookedStartMn;
    });
    roomResult.push({ room: room.name, isEmpty: !isConflict });
  }
  result.value = roomResult;
}

function onBookRoom(room) {
  const index = rooms.findIndex((r) => r.name === room);
  if (index !== -1) {
    rooms[index].books.push({ start: date.value + " " + timeStart.value, end: date.value + " " + timeEnd.value });
  }
}

</script>

<template>
  <div style="max-width: 400px">
    <label for="date">วันที่จอง</label><br>
    <input type="date" id="date" name="date" v-model="date" required />

    <label for="time-start" style="margin-top: 12px; display: block">เวลาที่จอง</label>
    <input id="time-start" type="time" name="time-start" min="9:00" max="18:00" v-model="timeStart" required /> -
    <input id="time-end" type="time" name="time-end" min="9:00" max="18:00" v-model="timeEnd" required />
    <br>
    <button style="margin-top: 16px" @click="checkBooked(date, timeStart, timeEnd)">
      เช็คห้องประชุม
    </button>

    <div v-if="result">
      <p v-for="r in result" :key="r.room" :style="{ color: r.isEmpty ? 'green' : 'red' }">
        ห้อง {{ r.room }} :
        {{ r.isEmpty ? 'ว่าง' : 'ไม่ว่าง' }} <button :disabled="!r.isEmpty" @click="onBookRoom(r.room)">จอง</button>
      </p>
    </div>
    <p v-if="error" style="color: red">ข้อผิดพลาด: {{ error }}</p>
    <div>---------------------------------------------------</div>
    <table>
      <thead>
        <th>ห้อง</th>
        <th>วันที่เวลา</th>
      </thead>
      <tbody>
        <template v-for="(room, index) in rooms" :key="index">
          <tr v-for="(book, bIdx) in room.books" :key="bIdx">
            <td>{{ room.name }}</td>
            <td>
              {{ moment(book.start).format("DD/MM/YYYY (HH:mm") }} - {{ moment(book.end).format("HH:mm)") }}
            </td>
          </tr>
        </template>
      </tbody>
    </table>
  </div>
</template>
