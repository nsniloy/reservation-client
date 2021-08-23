
<template>
  <section>
    <v-row   class="align-baseline">
      <v-col md="4" class="pa-5">
        <v-overflow-btn @change="getSlots"
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
        <v-btn @click="getSlots" color="secondary" dark large> search </v-btn>
      </v-col>
    </v-row>

    <v-data-table
      v-if="data.length"
      :headers="headers"
      :items="data"
      :search="search"
      class="elevation-1"
    >
      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title>Slot List</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
        </v-toolbar>
      </template>
      <template v-slot:[`item.date`]="{ item }">
        <span>{{ moment(item.date).tz('Asia/Singapore').format("DD MMM YYYY") }}</span>
      </template>
      <template v-slot:[`item.time`]="{ item }">
        <span>{{ moment(item.date).tz('Asia/Singapore').format("HH:mm A") }}</span>
      </template>
      <template v-slot:[`item.createdAt`]="{ item }">
        <span>{{ moment(item.createdAt).tz('Asia/Singapore').format("DD MMM YYYY HH:mm A") }}</span>
      </template>
      
      <template v-slot:no-data>
        <v-btn color="primary" @click="getSlots"> Reset </v-btn>
      </template>
    </v-data-table>
  </section>
</template>

<script>
import * as moment from "moment-timezone";
export default {
  data: () => ({
    dialog: false,
    moment: moment,
    dialogDelete: false,
    headers: [
      { text: "Slot Date", value: "date" },
      { text: "Slot Time", value: "time" },
      { text: "Remaining Quota", value: "quota_remaining" },
      { text: "Created time", value: "createdAt" },
    ],
    data: [],
    centre_data: [],
    menu: false,
    selectedCentre: "",
    search: "",
    editedIndex: -1,
    dates: [
      moment().format("YYYY-MM-DD"),
      moment().add(2, "weeks").format("YYYY-MM-DD"),
    ],
    datepicker: false
  }),

  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "New Item" : "Edit Item";
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
    this.getSlots();
  },

  methods: {
    async getSlots() {
      this.$router.push("/admin/slots?centre_id=" + this.selectedCentre);
      let url = `http://localhost:6001/slot?start_date=${this.dates[0]}&end_date=${this.dates[1]}&centre_id=${this.selectedCentre}`;
      try {
        let response = await this.$axios.$get(url);
        this.data = response.data;
      } catch (e) {
        console.log(e);
      }
    },

    async getCentres() {
      try {
        let response = await this.$axios.$get("http://localhost:6001/centre");
        this.centre_data = response.data;
        if (!this.selectedCentre && response.data.length) {
          this.selectedCentre = response.data[0]._id;
        }
      } catch (e) {
        console.log(e);
      }
    },
  },
};
</script>