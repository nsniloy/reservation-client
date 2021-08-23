<template>
  <section>
    <v-card max-width="700px">
      <v-card-title>
        <span class="text-h5">{{ formTitle }}</span>
      </v-card-title>

      <v-card-text>
        <v-container>
          <v-row>
            <v-col cols="12" sm="6" md="12">
              <v-overflow-btn
                :items="centre_data"
                label="Select Centre"
                no-data-text="No Centre Found"
                editable
                clearable
                autocomplete="off"
                dense
                v-model="editedItem.centre_id"
                item-text="name"
                item-value="_id"
              ></v-overflow-btn>
            </v-col>
          </v-row>
          <v-row>
            <v-col cols="12" sm="6" md="12">
              <v-text-field
                v-if="editedIndex < 0"
                v-model="editedItem.full_name"
                label="Full Name"
              ></v-text-field>
            </v-col>
          </v-row>
          <v-row>
            <v-col cols="12" sm="6" md="12">
              <v-text-field
                v-if="editedIndex < 0"
                v-model="editedItem.national_id"
                label="National ID"
              ></v-text-field>
            </v-col>
          </v-row>
          <v-row>
            <v-col cols="12" sm="6" md="12">
              <v-text-field
                v-if="editedIndex < 0"
                v-model="editedItem.email"
                type="email"
                label="Email"
              ></v-text-field>
            </v-col>
          </v-row>
          <v-row>
            <v-col cols="12" sm="6" md="12">
              <v-date-picker v-model="picker"></v-date-picker>
            </v-col>
          </v-row>
        </v-container>
      </v-card-text>

      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn depressed color="success" text @click="save"> Submit </v-btn>
      </v-card-actions>
    </v-card>
    <div name="snackbars">
      <v-snackbar v-model="show" :color="'error'" :timeout="2000" :top="false">
        {{ errorMessage }}

        <template v-slot:action="{ attrs }">
          <v-btn dark text v-bind="attrs" @click="snackbar = false">
            Close
          </v-btn>
        </template>
      </v-snackbar>
    </div>
    <div name="snackbars">
      <v-snackbar
        v-model="show_success"
        :color="'success'"
        :timeout="4000"
        :top="false"
      >
        {{ successMessage }}

        <template v-slot:action="{ attrs }">
          <v-btn dark text v-bind="attrs" @click="snackbar = false">
            Close
          </v-btn>
        </template>
      </v-snackbar>
    </div>
  </section>
</template>

<script>
import * as moment from "moment-timezone";
export default {
  layout: "resident",
  data: () => ({
    moment: moment,
    dialogDelete: false,
    headers: [
      { text: "Date", value: "date" },
      { text: "Time", value: "time" },
      { text: "Full Name", value: "full_name" },
      { text: "National ID", value: "national_id" },
      { text: "Actions", value: "actions" },
    ],
    centre_data: [],
    picker: new Date(Date.now() - new Date().getTimezoneOffset() * 60000)
      .toISOString()
      .substr(0, 10),
    menu: false,
    show: false,
    show_success: false,
    errorMessage: "",
    successMessage: "",
    selectedCentre: "",
    selectedDate: moment().add(1, "day").format(),
    dateRange: {},
    search: "",
    editedIndex: -1,
    dialog: false,
    datepicker: false,
    picker: new Date(Date.now() - new Date().getTimezoneOffset() * 60000)
      .toISOString()
      .substr(0, 10),
    editedItem: {
      centre_id: "",
      number_of_nurses: 0,
      start_time: "",
      end_time: "",
    },
    defaultItem: {
      centre_id: "",
      number_of_nurses: 0,
      start_time: "",
      end_time: "",
    },
  }),

  computed: {
    formTitle() {
      return "Reservation";
    },
  },

  async created() {
    await this.getCentres();
  },

  methods: {
    async getCentres() {
      try {
        let response = await this.$axios.$get("http://localhost:6001/centre");
        this.centre_data = response.data;
        if (!this.selectedCentre && response.data.length) {
          this.selectedCentre = response.data[0]._id;
        }
      } catch (e) {
        this.errorMessage = e.response.data.message;
        this.show = true;
        console.log(e);
      }
    },
    async save() {
      let url = "http://localhost:6001/reservation";
      let payload = {
        date: this.picker,
        full_name: this.editedItem.full_name,
        email: this.editedItem.email,
        national_id: this.editedItem.national_id,
        centre_id: this.editedItem.centre_id,
      };
      try {
        let res = await this.$axios.$post(url, payload);
        this.editedItem = this.defaultItem;
        this.show_success = true;
        this.successMessage = res.message;
      } catch (e) {
        this.errorMessage = e.response.data.message;
        this.show = true;
        console.log(e);
      }
    },
  },
};
</script>