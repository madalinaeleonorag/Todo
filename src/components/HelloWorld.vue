<template>
  <v-container>
    <v-layout wrap>
      <!-- ADD TODO -->
      <v-flex xs12 sm12 md12>
        <div
          style="font-size: 50px; color: white; margin-top: 5%; margin-bottom: 3%; text-align: center"
        >
          ToDoList
        </div>
      </v-flex>
      <v-flex xs12 sm12 md12 row justify-center>
        <v-flex xs12 sm6 md4>
          <v-text-field
            label="What you want ToDo?"
            solo
            :items="toDos"
            v-model="newToDo"
            style="margin-right: 10px"
          ></v-text-field>
        </v-flex>
        <v-flex xs12 sm1 md>
          <v-select
            solo
            :items="priorities"
            v-model="selectedPriority"
            item-value="selectedPriority"
            label="Select priority"
          ></v-select>
        </v-flex>
        <v-flex xs12 sm1 md1>
          <v-btn
            class="mx-2"
            fab
            dark
            color="indigo"
            @click="AddToDo"
            small
            style="float: right"
          >
            <v-icon dark>mdi-plus</v-icon>
          </v-btn>
        </v-flex>
      </v-flex>
      <!-- SELECT PRIORITY -->
      <v-flex xs12 sm12 md12 row justify-center>
        <v-flex xs12 sm12 md6 row justify-center>
          <v-flex xs3 sm3 md3>
            <v-checkbox
              label="All"
              v-model="all"
              @change="checkAll"
            ></v-checkbox>
          </v-flex>
          <v-flex xs3 sm3 md3>
            <v-checkbox
              label="High"
              v-model="high"
              @change="check"
            ></v-checkbox>
          </v-flex>
          <v-flex xs3 sm3 md3>
            <v-checkbox
              label="Medium"
              v-model="medium"
              @change="check"
            ></v-checkbox>
          </v-flex>
          <v-flex xs3 sm3 md3>
            <v-checkbox label="Low" v-model="low" @change="check"></v-checkbox>
          </v-flex>
        </v-flex>
      </v-flex>
      <!-- LIST TODO -->
      <v-flex xs12 sm12 md12 row justify-center>
        <v-flex xs12 sm12 md6>
          <v-card
            v-for="(item, index) in items"
            :key="index"
            style="margin-bottom: 10px; opacity: 0.7"
          >
            <v-card-text>
              <v-flex row>
                <v-flex xs12 sm9 md9>
                  <div v-if="item.Status === 'Done'" class="green--text">
                    {{ item.Text }}
                  </div>
                  <div v-else>{{ item.Text }}</div>
                </v-flex>
                <v-flex xs12 sm3 md3>
                  <div
                    v-if="item.Priority === 'High'"
                    class="red--text"
                    style="float:right"
                  >
                    {{ item.Priority }}
                  </div>
                  <div
                    v-if="item.Priority === 'Medium'"
                    class="orange--text"
                    style="float:right"
                  >
                    {{ item.Priority }}
                  </div>
                  <div
                    v-if="item.Priority === 'Low'"
                    class="yellow--text"
                    style="float:right"
                  >
                    {{ item.Priority }}
                  </div>
                </v-flex>
              </v-flex>
            </v-card-text>
            <v-card-actions>
              <v-card-text> </v-card-text>
              <v-spacer></v-spacer>

              <v-btn :disabled="item.Status === 'Done'" fab small>
                <v-icon @click="doneToDo(index)" color="green"
                  >mdi-check</v-icon
                >
              </v-btn>
              <v-btn :disabled="item.Status === 'Done'" fab small>
                <v-icon @click="editToDo(index)" color="orange"
                  >mdi-pencil</v-icon
                >
              </v-btn>
              <v-btn fab small>
                <v-icon @click="deleteToDo(index)" color="red"
                  >mdi-delete</v-icon
                >
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-flex>
      </v-flex>
    </v-layout>

    <!-- EDIT TODO DIALOG -->
    <v-dialog v-model="edit" width="500px">
      <v-card>
        <v-card-title>Edit ToDo</v-card-title>
        <v-card-text>
          <v-text-field solo v-model="newEdit"></v-text-field>
        </v-card-text>
        <v-card-actions>
          <v-btn @click="edit = false">Cancel</v-btn>
          <v-spacer></v-spacer>
          <v-btn @click="saveToDo">Save</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
export default {
  name: "HelloWorld",
  data() {
    return {
      toDos: [],
      newToDo: "",
      newEdit: "",
      edit: false,
      toEditIndex: null,
      priorities: ["High", "Medium", "Low"],
      selectedPriority: "Low",
      all: true,
      high: false,
      medium: false,
      low: false,
    };
  },
  computed: {
    items() {
      var filteredItems = [];
      filteredItems = this.toDos.filter((todo) => {
        if (this.all === true) {
          return true;
        } else if (this.high === true && todo.Priority === "High") {
          return true;
        } else if (this.low === true && todo.Priority === "Low") {
          return true;
        }
        if (this.medium === true && todo.Priority === "Medium") {
          return true;
        }
      });
      return filteredItems;
    },
  },
  methods: {
    AddToDo() {
      this.toDos.push({
        Text: this.newToDo,
        Status: "incomplete",
        Priority: this.selectedPriority,
      });
      this.newToDo = "";
    },
    doneToDo(value) {
      this.toDos[value].Status = "Done";
    },
    editToDo(value) {
      this.edit = true;
      this.toEditIndex = value;
    },
    saveToDo() {
      this.toDos[this.toEditIndex].Text = this.newEdit;
      this.newEdit = "";
      this.edit = false;
    },
    deleteToDo(value) {
      this.toDos.splice(value, 1);
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
    },
  },
  mounted() {
    console.log("tododo");
    if (localStorage.getItem("toDos"))
      this.toDos = JSON.parse(localStorage.getItem("toDos"));
  },
  watch: {
    toDos: {
      handler() {
        console.log("Todos changed!");
        localStorage.setItem("toDos", JSON.stringify(this.toDos));
      },
      deep: true,
    },
  },
};
</script>
