<template>
  <v-container>
    <v-layout wrap>
      <!-- ADD TODO -->
      <v-flex xs12 sm12 md12>
        <div
          style="font-size: 50px; color: white; margin-top: 5%; margin-bottom: 3%; text-align: center"
        >ToDoList</div>
      </v-flex>
      <div class="add-item">
        <v-text-field
          label="What you want ToDo?"
          solo
          :items="toDos"
          v-model="newToDo"
          style="margin-right: 10px"
        ></v-text-field>
        <v-select
          solo
          :items="priorities"
          v-model="selectedPriority"
          item-value="selectedPriority"
          label="Select priority"
          class="select-priority"
        ></v-select>
        <v-menu
          ref="menu"
          v-model="menu"
          :close-on-content-click="false"
          :return-value.sync="selectedDate"
          transition="scale-transition"
          offset-y
        >
          <template v-slot:activator="{ on, attrs }">
            <v-text-field
              v-model="selectedDate"
              label="Select due date"
              readonly
              v-bind="attrs"
              class="date-picker-custom"
              v-on="on"
            ></v-text-field>
          </template>
          <v-date-picker v-model="selectedDate" no-title scrollable>
            <v-spacer></v-spacer>
            <v-btn text color="primary" @click="menu = false">Cancel</v-btn>
            <v-btn text color="primary" @click="$refs.menu.save(selectedDate)">OK</v-btn>
          </v-date-picker>
        </v-menu>
        <v-btn dark @click="AddToDo" class="custom-button">Add</v-btn>
      </div>
      <div class="tabs">
        <div
          class="tab-item"
          @click="tab = 'list'"
          :class="tab === 'list' ? 'underlined-text' : ''"
        >List</div>
        <div
          class="tab-item"
          @click="tab = 'calendar'"
          :class="tab === 'calendar' ? 'underlined-text' : ''"
        >Calendar</div>
      </div>
      <div class="tab-content" v-if="tab === 'list'">
        <!-- FILTERS -->
        <v-flex xs12 sm12 md12 row justify-center>
          <v-select
            solo
            :items="filterPriorities"
            v-model="filterPriority"
            item-value="filterPriority"
            label="Select priority"
            class="select-priority"
          ></v-select>
      <v-menu
        ref="menu3"
        v-model="menu3"
        :close-on-content-click="false"
        :return-value.sync="filterDate"
        transition="scale-transition"
        offset-y
        min-width="290px"
      >
        <template v-slot:activator="{ on, attrs }">
          <v-text-field
            v-model="filterDate"
            readonly
            v-bind="attrs"
            v-on="on"
          ></v-text-field>
        </template>
        <v-date-picker v-model="filterDate" range no-title scrollable>
          <v-spacer></v-spacer>
          <v-btn text color="primary" @click="menu3 = false">Cancel</v-btn>
          <v-btn text color="primary" @click="$refs.menu3.save(filterDate)">OK</v-btn>
        </v-date-picker>
      </v-menu>
      <v-btn dark class="custom-button" @click="reset">Reset</v-btn>
        </v-flex>
        <!-- LIST TODO -->
        <div class="todo-list">
          <div v-for="(item, index) in items" :key="index" style="margin-bottom: 20px;">
            <div class="todo-details">
              <v-icon
                class="todo-priority-icon"
                :color="getColorForStatus(item.Priority)"
              >mdi-checkbox-blank-circle</v-icon>

              <div>{{ item.Text }}</div>

              <div class="todo-due-date">- {{item.Date}}</div>

              <v-spacer></v-spacer>
              <div class="todo-actions">
                <v-icon
                  class="todo-status-icon"
                  @click="doneToDo(index)"
                  color="green"
                  v-if="item.Status !== 'Done'"
                >mdi-check</v-icon>
                <v-icon
                  class="todo-status-icon"
                  @click="editToDo(index)"
                  color="orange"
                  v-if="item.Status !== 'Done'"
                >mdi-pencil</v-icon>
                <v-icon class="todo-status-icon" @click="deleteToDo(index)" color="red">mdi-delete</v-icon>
              </div>
            </div>
            <v-divider></v-divider>
          </div>
        </div>
      </div>
      <div class="tab-content" v-if="tab === 'calendar'">
        <v-sheet tile height="54" color="grey lighten-3" class="d-flex">
          <v-btn icon class="ma-2" @click="$refs.calendar.prev()">
            <v-icon>mdi-chevron-left</v-icon>
          </v-btn>
          <v-spacer></v-spacer>
          <v-btn icon class="ma-2" @click="$refs.calendar.next()">
            <v-icon>mdi-chevron-right</v-icon>
          </v-btn>
        </v-sheet>
        <v-sheet height="600">
          <v-calendar
            ref="calendar"
            v-model="value"
            :weekdays="weekday"
            type="month"
            :events="events"
            event-overlap-mode="column"
            :event-overlap-threshold="30"
            :event-color="getEventColor"
            @change="getEvents"
          ></v-calendar>
        </v-sheet>
      </div>
    </v-layout>

    <!-- EDIT TODO DIALOG -->
    <v-dialog v-model="edit" width="500px">
      <v-card>
        <v-card-title>Edit ToDo</v-card-title>
        <v-card-text class="dialog-content">
          <v-text-field solo v-model="selectedItem.Text" class="full-width"></v-text-field>
          <v-select
            solo
            :items="priorities"
            v-model="selectedItem.Priority"
            item-value="selectedItem.Priority"
            label="Select priority"
            class="select-priority full-width"
          ></v-select>

          <v-menu
            ref="menu2"
            v-model="menu2"
            :close-on-content-click="false"
            :return-value.sync="selectedItem.Date"
            transition="scale-transition"
            offset-y
          >
            <template v-slot:activator="{ on, attrs }">
              <v-text-field
                v-model="selectedItem.Date"
                label="Select due date"
                readonly
                v-bind="attrs"
                class="date-picker-custom full-width"
                v-on="on"
              ></v-text-field>
            </template>
            <v-date-picker v-model="selectedItem.Date" no-title scrollable>
              <v-spacer></v-spacer>
              <v-btn text color="primary" @click="menu2 = false">Cancel</v-btn>
              <v-btn text color="primary" @click="$refs.menu2.save(selectedItem.Date)">OK</v-btn>
            </v-date-picker>
          </v-menu>
        </v-card-text>
        <v-card-actions>
          <v-btn class="custom-button" @click="edit = false">Cancel</v-btn>
          <v-spacer></v-spacer>
          <v-btn class="custom-button" @click="saveToDo">Save</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import * as moment from 'moment';
export default {
  name: "HelloWorld",
  data() {
    return {
      toDos: [],
      events: [],
      newToDo: "",
      weekday: [0, 1, 2, 3, 4, 5, 6],
      selectedDate: new Date().toISOString().substr(0, 10),
      tab: "list",
      value: "",
      selectedItem: {},
      menu: false,
      menu2: false,
      menu3: false,
      edit: false,
      toEditIndex: null,
      filterPriority: "All",
      filterPriorities: ["All", "High", "Medium", "Low"],
      priorities: ["High", "Medium", "Low"],
      selectedPriority: "Low",
      all: true,
      high: false,
      medium: false,
      low: false,
      filterDate: ['2000-01-01', '2050-01-01']
    };
  },
  computed: {
    items() {
      return this.toDos.filter(todo => {
        if (this.filterPriority.toLowerCase() === "all") {
          return true;
        } else {
          return (
            todo.Priority.toLowerCase() === this.filterPriority.toLowerCase()
          );
        }
      }).filter(todoItem => moment(todoItem.Date).isBetween(this.filterDate[0], this.filterDate[1]));
    }
  },
  methods: {
    reset() {
      this.filterDate = ['2000-01-01', '2050-01-01'];
      this.filterPriority = 'All';
    },
    getEventColor(event) {
      return event.color;
    },
    getEvents({ start, end }) {
      const events = [];
      console.log("test");

      const min = new Date(`${start.date}T00:00:00`);
      const max = new Date(`${end.date}T23:59:59`);

      for (let i = 0; i < this.toDos; i++) {
        const firstTimestamp = this.rnd(min.getTime(), max.getTime());
        const first = new Date(firstTimestamp - (firstTimestamp % 900000));
        console.log(first);
        events.push({
          name: this.toDos[i].Text,
          start: this.toDos[i].Date,
          color: this.getColorForStatus(this.toDos[i].Priority),
          timed: false
        });
      }
      this.events = events;
    },
    rnd(a, b) {
      return Math.floor((b - a + 1) * Math.random()) + a;
    },
    getColorForStatus(priority) {
      return priority === "High"
        ? "red"
        : priority === "Medium"
        ? "yellow"
        : "green";
    },
    AddToDo() {
      this.toDos.push({
        Text: this.newToDo,
        Status: "incomplete",
        Priority: this.selectedPriority,
        Date: this.selectedDate
      });
      this.newToDo = "";
    },
    doneToDo(value) {
      this.toDos[value].Status = "Done";
    },
    editToDo(value) {
      this.edit = true;
      this.toEditIndex = value;
      this.selectedItem = {
        Text: this.toDos[value].Text,
        Status: this.toDos[value].Status,
        Priority: this.toDos[value].Priority,
        Date: this.toDos[value].Date
      };
    },
    saveToDo() {
      this.toDos[this.toEditIndex] = this.selectedItem;
      localStorage.setItem("toDos", JSON.stringify(this.toDos));
      this.edit = false;
    },
    deleteToDo(value) {
      this.toDos.splice(value, 1);
      localStorage.setItem("toDos", JSON.stringify(this.toDos));
    },
    checkAll() {
      if (this.all === true) {
        this.high = false;
        this.medium = false;
        this.low = false;
      }
    },
    check() {
      if (this.high === true || this.medium === true || this.low === true) {
        this.all = false;
      }
    }
  },
  mounted() {
    if (localStorage.getItem("toDos"))
      this.toDos = JSON.parse(localStorage.getItem("toDos"));
  },
  watch: {
    toDos: {
      handler() {
        console.log("Todos changed!");
        localStorage.setItem("toDos", JSON.stringify(this.toDos));
      },
      deep: true
    }
  }
};
</script>

<style scoped>
.v-divider {
  border-top: 1px solid rgba(255, 255, 255, 0.3) !important;
}
.dialog-content {
  display: flex;
  width: 100%;
  flex-flow: column;
  align-items: center;
}
.dialog-content > .select-priority {
  max-width: unset;
}
.todo-actions {
  display: flex;
}
.tab-content {
  display: flex;
  flex-flow: column;
  width: 100%;
}
.tabs {
  display: flex;
  width: 100%;
  flex-flow: row;
  justify-content: center;
}
.underlined-text {
  text-decoration: underline;
}
.v-menu__content {
  min-width: auto !important;
}
.full-width {
  width: 100%;
}
.tab-item {
  margin: 0px 10px;
  color: white;
  font-weight: bold;
  font-size: x-large;
  text-transform: lowercase;
}
.todo-list {
  display: flex;
  flex-flow: column;
  width: 100%;
  color: white;
}
.todo-details {
  display: flex;
  align-items: center;
}
.todo-priority-icon {
  padding-right: 10px;
  font-size: smaller !important;
}
.todo-status-icon {
  padding-left: 10px;
}
.todo-due-date {
  font-size: smaller;
  padding-left: 5px;
  opacity: 0.6;
  display: flex;
  flex-flow: row;
  min-width: fit-content;
}
.todo-details {
  padding-bottom: 10px;
}
.v-input__slot {
  width: auto;
}
.layout {
  width: 100%;
}
.add-item {
  width: 100%;
  display: flex;
  margin: 0px 100px;
  flex-flow: row;
}
.select-priority {
  max-width: 15%;
}
.custom-button {
  float: right;
  height: 48px !important;
  width: 68px;
  font-size: larger;
  font-weight: bold;
  text-transform: capitalize;
  margin-left: 10px;
  color: white !important;
  background-color: rgb(195 120 4) !important;
}
</style>