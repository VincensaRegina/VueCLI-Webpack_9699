<template>
  <v-main class="list">
    <h3 class="text-h3 font-weight-medium mb-5">To Do List</h3>
    <v-card>
      <v-card-title>
        <v-text-field
          v-model="search"
          append-icon="mdi-magnify"
          label="Search"
          hide-details
        ></v-text-field>

        <v-spacer></v-spacer>
        <v-select
          v-model="prioritySelect"
          :items="['Penting', 'Tidak penting']"
          label="Urutkan"
          required
          style="margin-right: 50px"
        ></v-select>

        <v-btn color="success" dark @click="dialog = true"> Tambah </v-btn>
      </v-card-title>
      <v-data-table
        :headers="headers"
        :items="filteredItems"
        :search="search"
        :expanded.sync="expanded"
        item-key="task"
        class="elevation-1"
        show-expand
      >
        <template v-slot:[`item.priority`]="{ item }">
          <v-chip :color="getColor(item.priority)" label outlined dark>
            {{ item.priority }}
          </v-chip>
        </template>

        <template v-slot:[`item.actions`]="{ item }">
          <v-btn
            small
            class="mr-2"
            @click="
              dialogEdit = true;
              itemContent = item;
              fillField();
            "
          >
            {{ index }}edit
          </v-btn>

          <v-btn
            small
            @click="
              dialogDelete = true;
              indexItem = todos.indexOf(item);
            "
          >
            delete
          </v-btn>
        </template>

        <template v-slot:expanded-item="{ headers, item }">
          <td :colspan="headers.length">
            <br /><b>NOTE:</b>
            <p>{{ item.note }}</p>
          </td>
        </template>

        <template v-slot:[`item.checkboxDelete`]="{ item }">
          <input
            type="checkbox"
            class="custom-control-input"
            :value="item.task"
            v-model="selectedItem"
          />
        </template>
      </v-data-table>
    </v-card>
    <v-dialog v-model="dialog" persistent max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">Form Todo</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-text-field
              v-model="formTodo.task"
              label="Task"
              required
            ></v-text-field>
            <v-select
              v-model="formTodo.priority"
              :items="['Penting', 'Biasa', 'Tidak penting']"
              label="Priority"
              required
            ></v-select>
            <v-textarea
              v-model="formTodo.note"
              label="Note"
              required
            ></v-textarea>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="cancel"> Cancel </v-btn>
          <v-btn color="blue darken-1" text @click="save"> Save </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="dialogEdit" persistent max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">Form Todo</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-text-field
              v-model="formTodoEdit.task"
              label="Task"
              required
              :value="itemContent.task"
            ></v-text-field>
            <v-select
              v-model="formTodoEdit.priority"
              :items="['Penting', 'Biasa', 'Tidak penting']"
              label="Priority"
              required
            ></v-select>
            <v-textarea v-model="formTodoEdit.note" label="Note" required>{{
              itemContent.note
            }}</v-textarea>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="cancelEdit"> Cancel </v-btn>
          <v-btn color="blue darken-1" text @click="saveEdit"> Save </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="dialogDelete" persistent max-width="400px">
      <v-card>
        <v-card-title>
          <span class="headline"><b>Yakin ingin menghapus?</b></span>
        </v-card-title>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="green darken-1" text @click="cancelDelete">
            TIDAK
          </v-btn>
          <v-btn color="red darken-1" text @click="deleteItem(indexItem)">
            YA
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <br />
    <v-card v-if="selectedItem.length != 0">
      <v-card-title>Delete Multiple</v-card-title>
      <v-card-text>
        <ul>
          <li v-for="selected in selectedItem" :key="selected">
            {{ selected }}
          </li>
        </ul>
        <br />
        <v-btn color="error" @click="deleteMultiple"> HAPUS SEMUA </v-btn>
      </v-card-text>
    </v-card>
  </v-main>
</template>
<script>
export default {
  name: "List",
  data() {
    return {
      search: null,
      dialog: false,
      dialogDelete: false,
      dialogEdit: false,
      itemContent: [],
      indexItem: null,
      prioritySelect: null,
      expanded: [],
      selectedItem: [],
      selectedDelete: [],
      headers: [
        { text: "", value: "data-table-expand" },
        {
          text: "Task",
          align: "start",
          sortable: true,
          value: "task",
        },
        { text: "Priority", value: "priority" },
        // { text: "Note", value: "note" },
        { text: "Actions", value: "actions" },
        { text: "", value: "checkboxDelete" },
      ],
      todos: [
        {
          task: "bernafas",
          priority: "Penting",
          note: "huffttt",
        },
        {
          task: "nongkrong",
          priority: "Tidak penting",
          note: "bersama tman2",
        },
        {
          task: "masak",
          priority: "Biasa",
          note: "masak air 500ml",
        },
      ],
      formTodo: {
        task: null,
        priority: null,
        note: null,
      },
      formTodoEdit: {
        task: null,
        priority: null,
        note: null,
      },
    };
  },
  methods: {
    fillField() {
      this.formTodoEdit.task = this.itemContent.task;
      this.formTodoEdit.priority = this.itemContent.priority;
      this.formTodoEdit.note = this.itemContent.note;
    },
    save() {
      this.todos.push(this.formTodo);
      this.resetForm();
      this.dialog = false;
    },
    saveEdit() {
      this.itemContent.task = this.formTodoEdit.task;
      this.itemContent.priority = this.formTodoEdit.priority;
      this.itemContent.note = this.formTodoEdit.note;
      this.resetForm();
      this.dialogEdit = false;
    },
    cancel() {
      this.resetForm();
      this.dialog = false;
    },
    cancelEdit() {
      this.resetForm();
      this.dialogEdit = false;
    },
    cancelDelete() {
      this.dialogDelete = false;
    },
    deleteItem(i) {
      this.todos.splice(i, 1);
      this.dialogDelete = false;
    },
    resetForm() {
      this.formTodo = {
        task: null,
        priority: null,
        note: null,
      };
      this.formTodoEdit = {
        task: null,
        priority: null,
        note: null,
      };
    },
    getColor(priority) {
      if (priority == "Penting") return "red";
      else if (priority == "Tidak penting") return "green";
      else if (priority == "Biasa") return "blue";
    },
    deleteMultiple() {
      var i, j;
      for (i in this.selectedItem) {
        for (j in this.todos) {
          if (this.selectedItem[i] == this.todos[j].task) {
            console.log("Index", i, j);
            this.todos.splice(j, 1);
          }
        }
      }
      this.selectedItem = [];
    },
  },

  computed: {
    filteredItems() {
      return this.todos.filter((i) => {
        return !this.prioritySelect || i.priority === this.prioritySelect;
      });
    },
  },
};
</script>