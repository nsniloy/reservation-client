
<template>
  <section>
    <v-row>
      <v-col md="4" class="pa-5">
        <v-overflow-btn
          @change="getSlots"
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
      :headers="headers"
      :items="data"
      :search="search"
      class="elevation-1"
    >
      <template v-slot:top>
        <v-toolbar flat>
          <v-toolbar-title>Nurse Supply History</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" max-width="500px">
            <template v-slot:activator="{ on, attrs }">
              <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
                Assign Nurse
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
                      <v-text-field
                        type="number"
                        v-model="editedItem.number_of_nurses"
                        label="Number of nurses"
                      ></v-text-field>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col cols="12" sm="6" md="12">
                      <date-range-picker
                        ref="picker"
                        :opens="opens"
                        :minDate="minDate"
                        :maxDate="maxDate"
                        :singleDatePicker="singleDatePicker"
                        :timePicker="timePicker"
                        :showDropdowns="showDropdowns"
                        :autoApply="autoApply"
                        :select="dateRange"
                        :update="dateRange"
                        :locale-data="{
                          format: 'dd-mm-yyyy HH:mm:ss',
                        }"
                        v-model="dateRange"
                      >
                        <template
                          v-slot:input="dateRange"
                          style="min-width: 350px"
                        >
                          {{ dateRange.startDate}} -
                          {{ dateRange.endDate}}
                        </template>
                      </date-range-picker>
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
        </v-toolbar>
      </template>
      <template v-slot:[`item.date`]="{ item }">
        <span>{{ moment(item.start_time).format("DD MMM YYYY") }}</span>
      </template>
      <template v-slot:[`item.start_time`]="{ item }">
        <span>{{ moment(item.start_time).format("HH:mm A") }}</span>
      </template>
      <template v-slot:[`item.end_time`]="{ item }">
        <span>{{ moment(item.end_time).format("HH:mm A") }}</span>
      </template>
      <template v-slot:[`item.actions`]="{ item }">
        <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
      </template>
      <template v-slot:no-data>
        <v-btn color="primary" @click="getSlots"> Reset </v-btn>
      </template>
    </v-data-table>
  </section>
</template>

<script>
import * as moment from "moment";
import DateRangePicker from "vue2-daterange-picker";
import "vue2-daterange-picker/dist/vue2-daterange-picker.css";
export default {
  components: { DateRangePicker },
  data: () => ({
    dialog: false,
    moment: moment,
    dialogDelete: false,
    headers: [
      { text: "Date", value: "date" },
      { text: "Number of Nurses", value: "number_of_nurses" },
      { text: "Start Time", value: "start_time" },
      { text: "End Time", value: "end_time" },
    ],
    data: [],
    centre_data: [],
    ranges: true,
    center: "centre",
    opens: "centre",
    minDate: moment().format(),
    maxDate: moment().add(1, "month").format(),
    singleDatePicker: "range",
    timePicker: true,
    showDropdowns: true,
    autoApply: true,

    menu: false,
    selectedCentre: "",
    selectedCentreForNurse: "",
    dateRange: {},
    search: "",
    editedIndex: -1,
    dates: [
      moment().format("YYYY-MM-DD"),
      moment().add(1, "day").format("YYYY-MM-DD"),
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
      centre_id: "",
      number_of_nurses: 0,
      start_time: "",
      end_time: "",
    },
  }),

  computed: {
    formTitle() {
      return "Assign Nurses";
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
    await this.getSlots();
  },

  methods: {
    async getSlots() {
      this.$router.push("/admin/nurses?centre_id=" + this.selectedCentre);
      let url = `http://localhost:6001/history?start_date=2020-02-02&end_date=2022-01-01&centre_id=${this.selectedCentre}`;
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

    deleteItem(item) {
      this.editedIndex = this.data.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },

    close() {
      this.dialog = false;
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem);
        this.editedIndex = -1;
      });
    },

    async save() {
      let url = "http://localhost:6001/centre/assign-nurse";
      let payload = {
        centre_id: this.editedItem.centre_id,
        number_of_nurses: this.editedItem.number_of_nurses,
        start_time: this.dateRange.startDate,
        end_time: this.dateRange.endDate,
      };
      console.log(payload);
      try {
        await this.$axios.$post(url, payload);
        this.getSlots();
        this.close();
      } catch (e) {
        console.log(e);
      }
    },
  },
};
</script>