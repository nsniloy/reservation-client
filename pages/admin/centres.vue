<template>
  <v-data-table
    :headers="headers"
    :items="data"
    :search="search"
    class="elevation-1"
  >
    <template v-slot:top>
      <v-toolbar flat>
        <v-toolbar-title>Centre List</v-toolbar-title>
        <v-divider class="mx-4" inset vertical></v-divider>
        <v-spacer></v-spacer>
        <v-dialog v-model="dialog" max-width="500px">
          <template v-slot:activator="{ on, attrs }">
            <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
              Add new centre
            </v-btn>
          </template>
          <v-card>
            <v-card-title>
              <span class="text-h5">{{ formTitle }}</span>
            </v-card-title>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col cols="12" sm="6" md="12">
                    <v-text-field
                      v-model="editedItem.name"
                      label="Centre name"
                    ></v-text-field>
                  </v-col>
                </v-row>
                <v-row>
                  <v-col cols="12" sm="6" md="12">
                    <v-text-field
                      v-model="editedItem.address"
                      label="Centre address"
                    ></v-text-field>
                  </v-col>
                </v-row>
                <v-row>
                  <v-col cols="12" sm="6" md="12">
                    <v-text-field
                      v-model="editedItem.vaccination_duration"
                      label="Vaccination duration for each person"
                    ></v-text-field>
                  </v-col>
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="close"> Cancel </v-btn>
              <v-btn color="blue darken-1" text @click="save"> Save </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-title class="text-h5"
              >Are you sure you want to delete this item?</v-card-title
            >
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="closeDelete"
                >Cancel</v-btn
              >
              <v-btn color="blue darken-1" text @click="deleteItemConfirm"
                >OK</v-btn
              >
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:[`item.createdAt`]="{ item }">
        <span>{{ moment(item.createdAt).format("DD MMM YYYY HH:mm") }}</span>
      </template>
    <template v-slot:[`item.actions`]="{ item }">
      <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
    </template>
    <template v-slot:no-data>
      <v-btn color="primary" @click="getCentres"> Reset </v-btn>
    </template>
  </v-data-table>
</template>
<script>
import * as moment from "moment-timezone";
export default {
  data: () => ({
    dialog: false,
    moment: moment,
    dialogDelete: false,
    headers: [
      { text: "Centre name", value: "name" },
      { text: "Centre address", value: "address" },
      { text: "Duration for each vaccination", value: "vaccination_duration" },
      { text: "Created time", value: "createdAt" },
      { text: "Actions", value: "actions", sortable: false },
    ],
    data: [],
    search: "",
    editedIndex: -1,
    editedItem: {
      name: "",
      address: "",
      vaccination_duration: "",
    },
    defaultItem: {
      name: "",
      address: "",
      vaccination_duration: "",
    },
  }),

  computed: {
    formTitle() {
      return "New Centre" ;
    },
  },

  created() {
    this.getCentres();
  },

  methods: {
    async getCentres() {
      try {
        let response = await this.$axios.$get("http://localhost:6001/centre");
        this.data = response.data;
      } catch (e) {
        console.log(e);
      }
    },

    deleteItem(item) {
      this.editedIndex = this.data.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    async deleteItemConfirm() {
      try {
        const data = await this.$axios.$delete(`http://localhost:6001/centre/${this.editedItem._id}`);
        console.log(data);
        this.closeDelete();
        this.data.splice(this.editedIndex, 1);
      } catch (e) {
        console.log(e);
      }
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    closeDelete() {
      this.dialogDelete = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    async save() {
      let url = "http://localhost:6001/centre";
      let payload = {
        name: this.editedItem.name,
        address: this.editedItem.address,
        vaccination_duration: this.editedItem.vaccination_duration,
      };
      try {
        const response = await this.$axios.$post(url, payload);
        this.data.push(response.data);
        this.close();
      } catch (e) {
        console.log(e);
      }
    },
  },
};
</script>