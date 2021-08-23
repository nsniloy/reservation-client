
<template>
  <section>
    <v-row>
      <v-col md="4" class="pa-5">
        <v-overflow-btn
          @change="getReservations"
          :items="centre_data"
          label="Select Centre"
          no-data-text="No Centre Found"
          editable
          clearable
          autocomplete="off"
          dense
          v-model="selectedCentre"
          item-text="name"
          item-value="_id"
        ></v-overflow-btn>
      </v-col>
      <v-col md="4" class="pa-5">
        <v-menu
          v-model="datepicker"
          :close-on-content-click="false"
          transition="scale-transition"
          offset-y
          min-width="200px"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-text-field
              v-model="dateRangeText"
              range
              label="Select Date Range"
              append-icon="mdi-calendar"
              readonly
              dense
              outlined
              v-bind="attrs"
              v-on="on"
            ></v-text-field>
          </template>
          <v-date-picker
            v-model="dates"
            range
            @input="menu2 = false"
          ></v-date-picker>
        </v-menu>
      </v-col>
      <v-col md="4" class="pa-5">
        <v-btn @click="getReservations" color="secondary" dark large>
          search
        </v-btn>
      </v-col>
    </v-row>

    <v-data-table
      :headers="headers"
      :items="data"
      :search="search"
      class="elevation-1"
    >
      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title>Reservations</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" @click:outside="close" max-width="500px">
            <template v-slot:activator="{ on, attrs }">
              <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
                Create Reservation
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
                      <v-text-field v-if="editedIndex<0"
                        v-model="editedItem.full_name"
                        label="Full Name"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col cols="12" sm="6" md="12">
                      <v-text-field v-if="editedIndex<0"
                        v-model="editedItem.national_id"
                        label="National ID"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col cols="12" sm="6" md="12">
                      <v-text-field v-if="editedIndex<0"
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
                <v-btn color="blue darken-1" text @click="close">
                  Cancel
                </v-btn>
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
      <template v-slot:[`item.date`]="{ item }">
        <span>{{ moment(item.date).format("DD MMM YYYY") }}</span>
      </template>
      <template v-slot:[`item.time`]="{ item }">
        <span>{{ moment(item.date).format("HH:mm A") }}</span>
      </template>
      <template v-slot:[`item.actions`]="{ item }">
        <v-icon small class="mr-2" @click="editItem(item)"> mdi-pencil </v-icon>
        <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
      </template>
      <template v-slot:no-data>
        <v-btn color="primary" @click="getReservations"> Reset </v-btn>
      </template>
    </v-data-table>
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
  </section>
</template>

<script>
import * as moment from "moment";
export default {
  data: () => ({
    dialog: false,
    moment: moment,
    dialogDelete: false,
    headers: [
      { text: "Date", value: "date" },
      { text: "Time", value: "time" },
      { text: "Full Name", value: "full_name" },
      { text: "National ID", value: "national_id" },
      { text: "Actions", value: "actions" },
    ],
    data: [],
    centre_data: [],
    min_date: moment().add(1,'day').format(),
    max_date: moment().add(1,'month').format(),
    picker: new Date(Date.now() - new Date().getTimezoneOffset() * 60000)
      .toISOString()
      .substr(0, 10),
    menu: false,
    show: false,
    errorMessage: '',
    selectedCentre: "",
    selectedDate: moment().add(1, "day").format(),
    dateRange: {},
    search: "",
    editedIndex: -1,
    dialog: false,
    dates: [
      moment().format("YYYY-MM-DD"),
      moment().add(2, "weeks").format("YYYY-MM-DD"),
    ],
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
      name: "",
      address: "",
      vaccination_duration: "",
    },
  }),

  computed: {
    formTitle() {
      return "Reservation";
    },
    dateRangeText() {
      if (this.dates.length > 0) {
        return this.dates.join(" ~ ");
      } else {
        return "Select Date Range";
      }
    },
  },

  async created() {
    await this.getCentres();
    await this.getReservations();
  },

  methods: {
    async getReservations() {
      this.$router.push("/admin/reservations?centre_id=" + this.selectedCentre);
      let url = `http://localhost:6001/reservation?start_date=2020-02-02&end_date=2022-01-01&centre_id=${this.selectedCentre}`;
      try {
        let response = await this.$axios.$get(url);
        this.data = response.data;
      } catch (e) {
        console.log(e);
      }
    },

    editItem(item) {
      this.editedIndex = this.data.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },
    async getCentres() {
      try {
        let response = await this.$axios.$get("http://localhost:6001/centre");
        this.centre_data = response.data;
        if (!this.selectedCentre && response.data.length) {
          this.selectedCentre = response.data[0]._id;
        }
      } catch (e) {
        this.errorMessage = e.response.data.message
        this.show = true
        console.log(e);
      }
    },

    async deleteItemConfirm() {
      try {
        const data = await this.$axios.$delete(
          `http://localhost:6001/reservation/${this.editedItem._id}`
        );
        console.log(data);
        this.closeDelete();
        this.data.splice(this.editedIndex, 1);
      } catch (e) {
        console.log(e);
      }
    },

    deleteItem(item) {
      console.log(2222);
      this.editedIndex = this.data.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    close() {
      this.dialog = false;

      this.editedIndex = -1;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
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
      if (this.editedIndex > -1) {
        let url = `http://localhost:6001/reservation/${this.editedItem._id}`;
        //EDIT
        let payload = {
          date: this.picker,
          centre_id: this.editedItem.centre_id
        };
        try {
          const data = await this.$axios.$put(url, payload);
          Object.assign(this.desserts[this.editedIndex], this.editedItem);
        } catch (e) {
          this.errorMessage = e.response.data.message
          this.show = true
          console.log(e);
        }
      } else {
        let url = "http://localhost:6001/reservation";
        let payload = {
          date: this.picker,
          full_name: this.editedItem.full_name,
          email: this.editedItem.email,
          national_id: this.editedItem.national_id,
          centre_id: this.editedItem.centre_id,
        };
        console.log(payload);
        try {
          await this.$axios.$post(url, payload);
          this.getReservations();
          this.close();
        } catch (e) {
          this.errorMessage = e.response.data.message
          this.show = true
          console.log(e);
        }
      }
    },
  },
};
</script>