<template>
  <div class="dp">
    <div class="dp-head">
      <button class="dp-nav" @click="prevMonth" aria-label="Previous month">‹</button>
      <div class="dp-title">{{ monthLabel }}</div>
      <button class="dp-nav" @click="nextMonth" aria-label="Next month">›</button>
    </div>

    <div class="dp-grid dp-dow">
      <div v-for="d in ['Mon','Tue','Wed','Thu','Fri','Sat','Sun']" :key="d" class="dp-dow-cell">{{ d }}</div>
    </div>

    <div class="dp-grid">
      <div v-for="(cell, i) in cells" :key="i"
           :class="['dp-cell', { 'is-out': !cell.inMonth, 'is-today': cell.isToday, 'is-selected': cell.isSelected }]"
           @click="cell.inMonth && select(cell.date)">
        <span v-if="cell.day" class="dp-num">{{ cell.day }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref, watch } from 'vue';

// v-model compatible
const props = defineProps({
  modelValue: { type: String, default: null } // expecting 'YYYY-MM-DD'
});
const emit = defineEmits(['update:modelValue']);

function toInputDate(d){ return d.toISOString().slice(0,10); }
function ymd(d){ return new Date(d.getFullYear(), d.getMonth(), d.getDate()); }

// internal cursor month (defaults to modelValue's month or today)
const today = new Date();
const initial = props.modelValue ? parseInputDate(props.modelValue) : today;
const year = ref(initial.getFullYear());
const month = ref(initial.getMonth()); // 0..11

// recompute when parent changes the value externally
watch(() => props.modelValue, (val) => {
  if (!val) return;
  const dt = parseInputDate(val)
  year.value = dt.getFullYear();
  month.value = dt.getMonth();
});

const monthLabel = computed(() => {
  const d = new Date(year.value, month.value, 1);
  return d.toLocaleDateString('en-GB', { month: 'long', year: 'numeric' });
});

const cells = computed(() => {
  const first = new Date(year.value, month.value, 1);
  const daysInMonth = new Date(year.value, month.value + 1, 0).getDate();
  // Monday-first lead
  let lead = first.getDay(); // 0=Sun..6=Sat
  lead = (lead === 0) ? 6 : (lead - 1);

  const total = lead + daysInMonth;
  const tail = (7 - (total % 7)) % 7;

  const sel = props.modelValue ? ymd(parseInputDate(props.modelValue)) : null;
  const cells = [];

  // leading blanks
  for (let i = 0; i < lead; i++) {
    cells.push({ inMonth:false });
  }
  // days
  for (let d = 1; d <= daysInMonth; d++) {
    const dt = new Date(year.value, month.value, d);
    const inMonth = true;
    const isToday = ymd(dt).getTime() === ymd(today).getTime();
    const isSelected = sel ? (ymd(dt).getTime() === sel.getTime()) : false;
    cells.push({ inMonth, day:d, date:dt, isToday, isSelected });
  }
  // trailing blanks
  for (let i = 0; i < tail; i++) {
    cells.push({ inMonth:false });
  }
  return cells;
});

function parseInputDate(str){
  const [y,m,d] = str.split('-').map(Number);
  return new Date(y, m-1, d); // local midnight
}
function formatInputDateLocal(d){
  const y = d.getFullYear();
  const m = String(d.getMonth()+1).padStart(2,'0');
  const day = String(d.getDate()).padStart(2,'0');
  return `${y}-${m}-${day}`;
}


function select(date){
  emit('update:modelValue', formatInputDateLocal(date));
}

function prevMonth(){
  if (month.value === 0){ month.value = 11; year.value -= 1; }
  else month.value -= 1;
}
function nextMonth(){
  if (month.value === 11){ month.value = 0; year.value += 1; }
  else month.value += 1;
}
</script>

<style scoped>
.dp{ background: var(--panel, #151923); border: 1px solid var(--border, #252c3b); border-radius: 12px; padding: 10px; }
.dp-head{ display:flex; align-items:center; justify-content:space-between; margin-bottom:6px; }
.dp-title{ font-weight:600 }
.dp-nav{ all:unset; cursor:pointer; padding:6px 10px; border-radius:8px; background: var(--chip, #222836); border:1px solid var(--border, #252c3b); }
.dp-grid{ display:grid; grid-template-columns: repeat(7, 1fr); gap: 6px; }
.dp-dow{ margin-bottom:6px; }
.dp-dow-cell{ text-align:center; font-size:12px; color: var(--muted,#8b93a7); padding:4px 0 }
.dp-cell{ height:38px; border:1px solid var(--border,#252c3b); border-radius:8px; display:flex; align-items:center; justify-content:center; background: var(--chip,#222836); cursor:pointer; }
.dp-cell.is-out{ visibility:hidden } /* keep grid shape */
.dp-cell.is-today{ box-shadow: 0 0 0 2px rgba(90,168,255,.35) inset; }
.dp-cell.is-selected{ background: var(--chip-active, #2b3345); border-color: var(--accent,#5aa8ff); }
.dp-num{ font-size:13px }
</style>
