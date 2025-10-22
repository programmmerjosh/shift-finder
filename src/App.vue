<template>
  <div class="container">
    <h1>Production Shift Finder</h1>

    <!-- Controls -->
    <div class="card" style="margin-bottom: 14px">
      <div class="row" style="justify-content: space-between">
        <div class="tabs">
          <div
            v-for="s in shifts"
            :key="s"
            class="tab"
            :class="{ active: shift === s }"
            @click="shift = s"
          >
            {{ s }}-Shift
          </div>
        </div>
        <div class="toggle" role="tablist" aria-label="Search mode">
          <button :class="{ active: mode === 'month' }" @click="mode = 'month'">
            Search By Month
          </button>
          <button :class="{ active: mode === 'date' }" @click="mode = 'date'">
            Search By Date
          </button>
        </div>
      </div>

      <!-- Inputs -->
      <div class="row" style="margin-top: 12px; gap: 14px">
        <template v-if="mode === 'date'">
          <div style="width: 300px; max-width: 100%">
            <label class="muted" style="display: block; margin-bottom: 6px"
              >Select a date</label
            >
            <DatePicker v-model="dateInput" />
          </div>
        </template>

        <template v-else>
          <div>
            <label for="inputMonth">Select month</label><br />
            <div class="month-input-row">
              <input
                id="inputMonth"
                type="month"
                v-model="monthInput"
                :title="
                  isDesktop
                    ? 'Tip: Click the field, then type the first letter of the month (e.g., “J” for January/June), or use ↑/↓ to move through months. You can use ↑/↓ to move to a different year OR key in the exact year you want.'
                    : ''
                "
              />
              <!-- Help button: desktop only -->
              <button
                v-if="isDesktop"
                class="help-btn"
                type="button"
                aria-label="How to use the month picker"
                @mouseenter="showMonthHelp = true"
                @mouseleave="showMonthHelp = false"
                @focus="showMonthHelp = true"
                @blur="showMonthHelp = false"
              >
                ?
              </button>

              <!-- Tooltip -->
              <div
                v-if="isDesktop && showMonthHelp"
                class="help-tip"
                role="tooltip"
              >
                <strong>Month picker tips</strong><br />
                • Click the field, then type the first letter of a month (e.g.,
                “J”).<br />
                • Use ↑ / ↓ to move through months.<br />
                • Use ↑/↓ to move to a different year OR key in the exact year<br />
              </div>
            </div>
          </div>
        </template>
      </div>

      <!-- <div class="tip">
        Base cycle start (editable in code): <strong>{{ baseHuman }}</strong
        >. Pattern: A (day) & B (night) for 4 days, then C (day) & D (night) for
        4 days, repeat.
      </div> -->
    </div>

    <!-- Date search result -->
    <div class="card" v-if="mode === 'date' && resultDate">
      <div
        class="row"
        style="justify-content: space-between; align-items: center"
      >
        <div>
          <div style="font-weight: 600">
            Result for {{ shift }} on {{ fmtYMD(resultDate.queryDate) }}
          </div>
          <div class="muted">
            {{
              !resultDate.working
                ? "OFF"
                : resultDate.shiftType === "day"
                ? "Day shift (07:00–19:00)"
                : "Night shift (19:00–07:00)"
            }}
          </div>
        </div>
        <span class="pill" :class="resultDate.working ? 'yes' : 'no'">
          {{ resultDate.working ? "WORKING" : "NOT working" }}
        </span>
      </div>

      <div class="kv">
        <div class="muted">{{ resultDate.blockLabel }}</div>
        <div>
          <span
            class="pill"
            :class="
              resultDate.blockPhase === 'day'
                ? 'day'
                : resultDate.blockPhase === 'night'
                ? 'night'
                : 'off'
            "
            style="margin-right: 8px"
          >
            <template v-if="resultDate.blockPhase === 'day'"
              >☀️ DAY WORK</template
            >
            <template v-else-if="resultDate.blockPhase === 'night'"
              >🌙 NIGHT WORK</template
            >
            <template v-else>🛌 OFF</template>
          </span>
          {{ fmtFull(resultDate.blockStart) }} →
          {{ fmtFull(resultDate.blockEnd) }}
        </div>

        <div class="muted">You’re scheduled</div>
        <div>
          <template v-if="resultDate.working">
            {{
              resultDate.shiftType === "day"
                ? "DAY (07:00–19:00)"
                : "NIGHT (19:00–07:00)"
            }}
          </template>
          <template v-else> Off on the selected date </template>
        </div>

        <div class="muted">Previous off</div>
        <div>
          {{ fmtDate(resultDate.prevOffStart) }} →
          {{ fmtDate(resultDate.prevOffEnd) }}
        </div>

        <div class="muted">Next off</div>
        <div>
          {{ fmtDate(resultDate.nextOffStart) }} →
          {{ fmtDate(resultDate.nextOffEnd) }}
        </div>

        <div class="muted">Nearest work blocks</div>
        <div>
          <div>
            Previous: {{ fmtFull(resultDate.prevWorkStart) }} →
            {{ fmtFull(resultDate.prevWorkEnd) }}
          </div>
          <div>
            Next: {{ fmtFull(resultDate.nextWorkStart) }} →
            {{ fmtFull(resultDate.nextWorkEnd) }}
          </div>
        </div>
      </div>
    </div>

    <!-- Month search result -->
    <div class="card" v-if="mode === 'month' && resultMonth">
      <div
        class="row"
        style="justify-content: space-between; align-items: center"
      >
        <div>
          <div style="font-weight: 600">
            Month view for {{ shift }} — {{ monthLabel }}
          </div>
          <div class="legend">
            <span class="dot day"></span> Day ON
            <span class="dot night"></span> Night ON
          </div>
        </div>
      </div>

      <!-- Calendar -->
      <div class="calendar">
        <div class="cal-head">
          <div
            v-for="d in ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']"
            :key="d"
          >
            {{ d }}
          </div>
        </div>
        <div class="cal-grid">
          <div v-for="c in calendarCells" :key="c.key" class="cell">
            <div class="date-num" v-if="c.inMonth">{{ c.day }}</div>
            <div class="badge" v-if="c.inMonth && c.badge" :class="c.badge">
              {{ c.badge === "day" ? "DAY" : "NIGHT" }}
            </div>
          </div>
        </div>
      </div>

      <div class="list">
        <div
          v-for="(b, i) in resultMonth.blocks"
          :key="i"
          style="margin-bottom: 6px"
        >
          <span
            class="pill"
            :class="b.phase === 'day' ? 'day' : 'night'"
            style="margin-right: 8px"
          >
            <template v-if="b.phase === 'day'">☀️ DAY</template>
            <template v-else>🌙 NIGHT</template>
          </span>
          {{ fmtFull(b.start) }} → {{ fmtFull(b.end) }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import DatePicker from "./components/DatePicker.vue";
import { ref, computed, watch, onMounted, onUnmounted } from "vue";

/* ========= Configuration ========= */
// Start of each shift's 16-day cycle (index 0 = Day ON start for that shift)
const BASE_STARTS = {
  A: new Date("2025-07-03T00:00:00"), // Day ON: 4d → OFF 4d → Night ON 4d → OFF 4d
  B: new Date("2025-07-11T00:00:00"), // offset +4d from A
  C: new Date("2025-07-15T00:00:00"), // offset +8d
  D: new Date("2025-07-07T00:00:00"), // offset +12d
};

/* ========= State ========= */
const shifts = ["A", "B", "C", "D"];
const shift = ref("A");
const mode = ref("month");

const today = new Date();
const dateInput = ref(toInputDate(today));
const monthInput = ref(toInputMonth(today));

const resultDate = ref(null);
const resultMonth = ref(null);

const isDesktop = ref(false);
let mq, mqHandler;
const showMonthHelp = ref(false);

/* ========= Helpers ========= */
function toInputDate(d) {
  const y = d.getFullYear();
  const m = String(d.getMonth() + 1).padStart(2, "0");
  const day = String(d.getDate()).padStart(2, "0");
  return `${y}-${m}-${day}`; // local YYYY-MM-DD
}
function toInputMonth(d) {
  const y = d.getFullYear();
  const m = String(d.getMonth() + 1).padStart(2, "0");
  return `${y}-${m}`; // local YYYY-MM
}

function ymd(d) {
  return new Date(d.getFullYear(), d.getMonth(), d.getDate());
}
function addDays(d, n) {
  const x = new Date(d);
  x.setDate(x.getDate() + n);
  return x;
}
function fmtDate(d) {
  return d.toLocaleDateString("en-GB", {
    weekday: "short",
    day: "2-digit",
    month: "short",
    year: "numeric",
  });
}
function fmtFull(d) {
  return d.toLocaleString("en-GB", {
    weekday: "short",
    day: "2-digit",
    month: "short",
    year: "numeric",
    hour: "2-digit",
    minute: "2-digit",
  });
}
function fmtYMD(d) {
  return d.toLocaleDateString("en-GB", {
    weekday: "long",
    day: "2-digit",
    month: "long",
    year: "numeric",
  });
}

/* ========= Derived display of current shift's base (REPLACE THIS) ========= */
const baseHuman = computed(() => fmtDate(BASE_STARTS[shift.value]));

/* ========= 16-day cycle helpers (REPLACE OLD HELPERS WITH ALL BELOW) ========= */

// 16-day cycle layout (per shift):
// idx 0..3   : DAY ON (07:00–19:00)
// idx 4..7   : OFF
// idx 8..11  : NIGHT ON (19:00–07:00 next day)
// idx 12..15 : OFF
function cycleIndex(date, shiftLetter) {
  const base = ymd(BASE_STARTS[shiftLetter]);
  const days = Math.floor((ymd(date) - base) / (1000 * 60 * 60 * 24));
  return ((days % 16) + 16) % 16; // 0..15
}

function phaseFromIndex(i) {
  if (i < 4) return "day"; // working (day)
  if (i < 8) return "off"; // off
  if (i < 12) return "night"; // working (night)
  return "off"; // off
}

// Helper to convert a start date + phase to start/end timestamps
function workBounds(startDate, phase) {
  if (phase === "day") {
    const s = new Date(startDate);
    s.setHours(7, 0, 0, 0);
    const e = new Date(addDays(startDate, 3));
    e.setHours(19, 0, 0, 0);
    return { start: s, end: e };
  } else {
    // night
    const s = new Date(startDate);
    s.setHours(19, 0, 0, 0);
    const e = new Date(addDays(startDate, 4));
    e.setHours(7, 0, 0, 0);
    return { start: s, end: e };
  }
}

// Returns { working:Boolean, type:'day'|'night'|null, phase:'day'|'night'|'off' }
function workingState(shiftLetter, date) {
  const idx = cycleIndex(date, shiftLetter);
  const phase = phaseFromIndex(idx);
  const working = phase === "day" || phase === "night";
  return { working, type: working ? phase : null, phase, idx };
}

// Start date (00:00) for the 4-day phase window that contains `date`
function phaseStartForDate(shiftLetter, date) {
  const { idx } = workingState(shiftLetter, date);
  const startIdx = idx - (idx % 4); // align to 0,4,8,12 within cycle
  // Move from current date back to the aligned start of its 4-day window
  return addDays(ymd(date), -(idx % 4));
}

// Returns timestamps describing the selected shift's *current* phase window.
// If phase is 'day' or 'night', includes exact start/end times for the whole 4-day work block.
function currentPhaseWindow(shiftLetter, date) {
  const { phase } = workingState(shiftLetter, date);
  const start = phaseStartForDate(shiftLetter, date); // 00:00 at first calendar day
  const endExclusive = addDays(start, 4); // 00:00 the day after the 4th day

  // For display purposes (human-friendly time bounds)
  const dayStart = new Date(start);
  dayStart.setHours(7, 0, 0, 0);
  const dayEnd = new Date(addDays(start, 3));
  dayEnd.setHours(19, 0, 0, 0);

  const nightStart = new Date(start);
  nightStart.setHours(19, 0, 0, 0);
  const nightEnd = new Date(addDays(start, 4));
  nightEnd.setHours(7, 0, 0, 0);

  return {
    phase, // 'day' | 'night' | 'off'
    start, // 4-day window calendar start (00:00)
    endExclusive, // window end (exclusive)
    dayStart,
    dayEnd, // full 4-day DAY ON block timestamps
    nightStart,
    nightEnd, // full 4-day NIGHT ON block timestamps
  };
}

// Find previous/next WORK windows (nearest) for this shift around `date`.
// Always returns the DAY and NIGHT work windows adjacent to the *current* phase window,
// skipping OFF blocks automatically.
function neighborWorkWindows(shiftLetter, date) {
  // use the window aligned to our current phase (on or off)
  const currStart = phaseStartForDate(shiftLetter, date);
  const currIdx = cycleIndex(currStart, shiftLetter);

  // On-phase starts are indices 0 (day) and 8 (night) (mod 16).
  // We step in 4-day increments until we hit a working phase.
  function findPrevOn(startIdx) {
    let i = startIdx;
    do {
      i -= 4;
    } while (phaseFromIndex(((i % 16) + 16) % 16) === "off");
    return i;
  }
  function findNextOn(startIdx) {
    let i = startIdx;
    do {
      i += 4;
    } while (phaseFromIndex(i % 16) === "off");
    return i;
  }

  const prevOnIdx = findPrevOn(currIdx);
  const nextOnIdx = findNextOn(currIdx);

  const prevStart = addDays(currStart, prevOnIdx - currIdx);
  const nextStart = addDays(currStart, nextOnIdx - currIdx);

  const prevPhase = phaseFromIndex(((prevOnIdx % 16) + 16) % 16); // 'day' or 'night'
  const nextPhase = phaseFromIndex(nextOnIdx % 16); // 'day' or 'night'

  const prev = workBounds(prevStart, prevPhase);
  const next = workBounds(nextStart, nextPhase);

  return {
    prevPhase,
    prevStart: prev.start,
    prevEnd: prev.end,
    nextPhase,
    nextStart: next.start,
    nextEnd: next.end,
  };
}

// For convenience in outputs: OFF windows adjacent to a WORK block
function offWindowsAroundWork(workStartDate) {
  // Off before = the 4 days immediately before the workStart
  const prevOffStart = addDays(workStartDate, -4);
  const prevOffEnd = workStartDate;
  // Off after = the 4 days immediately after the 4-day work block
  const nextOffStart = addDays(workStartDate, 4);
  const nextOffEnd = addDays(workStartDate, 8);
  return { prevOffStart, prevOffEnd, nextOffStart, nextOffEnd };
}

/* ========= Actions ========= */
function submit() {
  if (mode.value === "date") {
    // Parse the selected date at local midnight
    const q = new Date(dateInput.value + "T00:00:00");

    const state = workingState(shift.value, q); // { working, type, phase, idx }
    const win = currentPhaseWindow(shift.value, q); // current 4-day phase window (on/off)
    const neighbors = neighborWorkWindows(shift.value, q); // nearest prev/next work windows

    // --- Current work/off block with accurate timestamps ---
    // Working → use workBounds(win.start, 'day'|'night')
    // Off     → OFF block spans from prev work END to next work START
    let blockStartTs, blockEndTs, blockPhase, blockLabel;
    blockLabel = "Current work/off block";

    if (state.working) {
      blockPhase = state.type; // 'day' | 'night'
      ({ start: blockStartTs, end: blockEndTs } = workBounds(
        win.start,
        state.type
      ));
    } else {
      blockPhase = "off";
      blockStartTs = neighbors.prevEnd; // when last work ended (07:00 or 19:00)
      blockEndTs = neighbors.nextStart; // when next work begins (07:00 or 19:00)
    }

    // Off windows (keep your existing behavior):
    // If working → around the current work block; if off → around the next work block
    const offRefStart = state.working
      ? ymd(win.start)
      : ymd(neighbors.nextStart);
    const around = offWindowsAroundWork(offRefStart);

    resultDate.value = {
      working: state.working,
      shiftType: state.type ?? "off",
      blockPhase, // 'day' | 'night' | 'off' (useful for badges)
      blockLabel, // "Current work/off block"
      queryDate: q,
      blockStart: blockStartTs,
      blockEnd: blockEndTs,
      prevOffStart: around.prevOffStart,
      prevOffEnd: around.prevOffEnd,
      nextOffStart: around.nextOffStart,
      nextOffEnd: around.nextOffEnd,
      prevWorkStart: neighbors.prevStart,
      prevWorkEnd: neighbors.prevEnd,
      nextWorkStart: neighbors.nextStart,
      nextWorkEnd: neighbors.nextEnd,
    };
  } else {
    // Month mode → list 4-day work blocks (DAY/NIGHT) that intersect the month
    const [y, m] = monthInput.value.split("-").map(Number);
    const blocks = listWorkBlocksForMonth(shift.value, y, m);
    resultMonth.value = { blocks, year: y, month: m };
  }
}

/* ========= Calendar ========= */
const monthLabel = computed(() => {
  if (!resultMonth.value) return "";
  const d = new Date(resultMonth.value.year, resultMonth.value.month - 1, 1);
  return d.toLocaleDateString("en-GB", { month: "long", year: "numeric" });
});
const calendarCells = computed(() => {
  if (!resultMonth.value) {
    const tmp = new Date();
    return buildCalendar(tmp.getFullYear(), tmp.getMonth() + 1, shift.value);
  }
  return buildCalendar(
    resultMonth.value.year,
    resultMonth.value.month,
    shift.value
  );
});

function buildCalendar(year, month, letter) {
  const first = new Date(year, month - 1, 1);
  const daysInMonth = new Date(year, month, 0).getDate();
  let lead = first.getDay();
  lead = lead === 0 ? 6 : lead - 1; // Mon-first

  const total = lead + daysInMonth;
  const tail = (7 - (total % 7)) % 7;
  const cells = [];

  for (let i = 0; i < lead; i++) cells.push({ key: "L" + i, inMonth: false });

  for (let d = 1; d <= daysInMonth; d++) {
    const dt = new Date(year, month - 1, d);
    const st = workingState(letter, dt);
    let badge = null;
    if (st.working) badge = st.type === "day" ? "day" : "night";
    cells.push({ key: "D" + d, inMonth: true, day: d, badge });
  }

  for (let i = 0; i < tail; i++) cells.push({ key: "T" + i, inMonth: false });
  return cells;
}

// Return all 4-day WORK blocks (DAY or NIGHT) for a given month, for a shift.
// Each item: { phase: "day"|"night", start: Date, end: Date }
function listWorkBlocksForMonth(shiftLetter, year, month) {
  const monthStart = new Date(year, month - 1, 1);
  const monthEndExclusive = new Date(year, month, 1); // first day of next month

  // Start scanning a little before the month to catch a block that begins in the previous month
  const scanStart = addDays(phaseStartForDate(shiftLetter, monthStart), -12); // 3 windows earlier
  let cursor = new Date(scanStart);

  const blocks = [];
  while (cursor < addDays(monthEndExclusive, 12)) {
    // scan slightly past the month
    const idx = cycleIndex(cursor, shiftLetter);
    const phase = phaseFromIndex(idx); // 'day' | 'night' | 'off'
    const windowStart = new Date(cursor); // aligned 4-day window (00:00)

    if (phase === "day" || phase === "night") {
      const { start, end } = workBounds(windowStart, phase);
      // Include any work block that overlaps the month in time
      if (end > monthStart && start < monthEndExclusive) {
        blocks.push({ phase, start, end });
      }
    }

    // Step to next 4-day window in the 16-day cycle
    cursor = addDays(cursor, 4);
  }
  // Sort by start (just in case)
  blocks.sort((a, b) => a.start - b.start);
  return blocks;
}

// run once on load
// Recalculate results automatically on relevant UI changes
// Auto-update whenever the user changes shift/mode/date/month
watch([shift, mode, dateInput, monthInput], () => {
  submit();
}, { immediate: true });

watch(mode, (m) => {
  if (m === "month") {
    if (!monthInput.value || !/^\d{4}-\d{2}$/.test(monthInput.value)) {
      monthInput.value = toInputMonth(new Date()); // e.g. 2025-10
    }
    submit(); // ensure month view re-renders now
  }
});

// Run once on initial load so results show immediately
onMounted(() => {
  // true on desktops / trackpads (hover + fine pointer)
  mq = window.matchMedia("(hover: hover) and (pointer: fine)");
  mqHandler = (e) => { isDesktop.value = e.matches; };
  isDesktop.value = mq.matches;

  if (mq.addEventListener) mq.addEventListener("change", mqHandler);
  else if (mq.addListener) mq.addListener(mqHandler); // Safari fallback
});
onUnmounted(() => {
  if (!mq) return;
  if (mq.removeEventListener) mq.removeEventListener("change", mqHandler);
  else if (mq.removeListener) mq.removeListener(mqHandler);
});
</script>

<style>
:root {
  --bg: #0f1115;
  --panel: #151923;
  --muted: #8b93a7;
  --text: #e8ebf2;
  --accent: #5aa8ff;
  --green: #21c16b;
  --red: #ff5c5c;
  --night: #e0b3ff;
  --day: #7fe3a2;
  --chip: #222836;
  --chip-active: #7c5775;
  --grid: #1c2331;
  --border: #252c3b;
}
* {
  box-sizing: border-box;
}
body {
  margin: 0;
  background: var(--bg);
  color: var(--text);
  font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Arial;
}
.container {
  max-width: 980px;
  margin: 32px auto;
  padding: 0 16px;
}
h1 {
  font-size: 20px;
  font-weight: 600;
  margin: 0 0 16px;
}
.card {
  background: var(--panel);
  border: 1px solid var(--border);
  border-radius: 14px;
  padding: 16px 16px;
}
.row {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
  align-items: center;
}
.tabs {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}
.tab {
  padding: 8px 14px;
  border: 1px solid var(--border);
  background: var(--chip);
  border-radius: 10px;
  cursor: pointer;
}
.tab.active {
  background: var(--chip-active);
  border-color: var(--accent);
  color: #fff;
}
.toggle {
  display: flex;
  border: 1px solid var(--border);
  background: var(--chip);
  border-radius: 12px;
  overflow: hidden;
}
.toggle button {
  all: unset;
  padding: 8px 14px;
  cursor: pointer;
}
.toggle button.active {
  background: var(--chip-active);
  color: #fff;
  border-left: 1px solid var(--border);
  border-right: 1px solid var(--border);
}
label {
  font-size: 13px;
  color: var(--muted);
}
input,
select {
  background: #0f1420;
  border: 1px solid var(--border);
  color: var(--text);
  padding: 10px 12px;
  border-radius: 10px;
  outline: none;
}
input[type="month"] {
  padding: 8px 10px;
}
.pill {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 6px 10px;
  border-radius: 999px;
  font-size: 12px;
}
.pill.yes {
  background: rgba(33, 193, 107, 0.18);
  color: var(--green);
  border: 1px solid rgba(33, 193, 107, 0.35);
}
.pill.no {
  background: rgba(255, 92, 92, 0.18);
  color: var(--red);
  border: 1px solid rgba(255, 92, 92, 0.35);
}
.pill.day {
  background: var(--day-bg);
  color: var(--day-fg);
  border-color: var(--day-bd);
}
.pill.night {
  background: var(--night-bg);
  color: var(--night-fg);
  border-color: var(--night-bd);
}
.pill.off {
  background: var(--off-bg);
  color: var(--off-fg);
  border-color: var(--off-bd);
}
.kv {
  display: grid;
  grid-template-columns: 160px 1fr;
  gap: 6px 12px;
  margin-top: 10px;
}
.kv div {
  padding: 6px 0;
  border-bottom: 1px dashed var(--grid);
}
.calendar {
  margin-top: 14px;
  border: 1px solid var(--border);
  border-radius: 12px;
  overflow: hidden;
}
.cal-head {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  background: var(--chip);
  border-bottom: 1px solid var(--border);
}
.cal-head div {
  padding: 10px;
  text-align: center;
  font-size: 12px;
  color: var(--muted);
}
.cal-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.02), transparent);
}
.cell {
  min-height: 82px;
  border-right: 1px solid var(--border);
  border-bottom: 1px solid var(--border);
  padding: 8px;
  position: relative;
}
.cell:nth-child(7n) {
  border-right: none;
}
.date-num {
  font-size: 12px;
  color: var(--muted);
}
.badge {
  position: absolute;
  right: 6px;
  bottom: 6px;
  font-size: 10px;
  padding: 4px 6px;
  border-radius: 8px;
  border: 1px solid var(--border);
}
.badge.day {
  background: rgba(127, 227, 162, 0.15);
  color: var(--day);
  border-color: rgba(127, 227, 162, 0.35);
}
.badge.night {
  background: rgba(224, 179, 255, 0.12);
  color: var(--night);
  border-color: rgba(224, 179, 255, 0.35);
}
.legend {
  display: flex;
  gap: 10px;
  align-items: center;
  margin-top: 8px;
  color: var(--muted);
  font-size: 12px;
}
.legend .dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
}
.dot.day {
  background: var(--day);
}
.dot.night {
  background: var(--night);
}
.list {
  margin-top: 10px;
  display: flex;
  gap: 18px;
  flex-wrap: wrap;
}
.list .block {
  flex: 1 1 280px;
  border: 1px solid var(--border);
  border-radius: 10px;
  padding: 10px;
  background: var(--chip);
}
.muted {
  color: var(--muted);
}
.tip {
  font-size: 12px;
  color: var(--muted);
  margin-top: 10px;
}
@media (max-width: 560px) {
  .kv {
    grid-template-columns: 1fr;
  }
  .kv div {
    border-bottom: none;
  }
}

/* styles for search-by-month tooltip ~ desktop only */
.month-input-row{
  display:inline-flex;
  align-items:center;
  gap:8px;
  position:relative;
}

.help-btn{
  all:unset;
  width:26px;height:26px; line-height:26px;
  text-align:center; font-weight:700;
  border-radius:999px; cursor:pointer;
  background:var(--chip,#222836);
  border:1px solid var(--border,#252c3b);
  color:var(--muted,#8b93a7);
  transition:box-shadow .12s ease, transform .06s ease, background-color .12s ease;
}
.help-btn:hover{ transform: translateY(-1px); }
.help-btn:focus-visible{ box-shadow:0 0 0 3px var(--ring, rgba(90,168,255,.5)); }

.help-tip{
  position:absolute;
  top:100%; left:0;
  margin-top:8px; z-index:5;
  max-width:360px;
  background:var(--panel,#151923);
  color:var(--text,#e8ebf2);
  border:1px solid var(--border,#252c3b);
  border-radius:8px; padding:10px 12px;
  box-shadow:0 8px 28px rgba(0,0,0,.35);
  font-size:12px; line-height:1.35;
}

/* only show help UI on devices with hover + fine pointer (desktop) */
@media (hover: none), (pointer: coarse){
  .help-btn, .help-tip { display:none !important; }
}
</style>
