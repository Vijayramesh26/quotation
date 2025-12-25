<template>
  <v-app>
    <v-app-bar app dark color="teal ">
      <v-toolbar-title class="text-body-1 font-weight-bold">
        ESTIMATION GENERATOR
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn v-if="currentView === 'preview'" text @click="goBackToForm">
        <v-icon left>mdi-arrow-left</v-icon>
        Back to Form
      </v-btn>
      <v-btn color="error" @click="clearStorage" small class="text-caption">
        <v-icon small>mdi-delete</v-icon> Clear
      </v-btn>
    </v-app-bar>

    <v-main>
      <EstimateForm
        v-if="currentView === 'form'"
        v-model="estimateData"
        @preview="showPreview"
      />
      <EstimatePreview
        v-if="currentView === 'preview'"
        :estimate="formattedEstimate"
      />
    </v-main>

    <v-snackbar v-model="snackbar.show" :color="snackbar.color" timeout="3000">
      {{ snackbar.message }}
      <template v-slot:action="{ attrs }">
        <v-btn text v-bind="attrs" @click="snackbar.show = false">
          Close
        </v-btn>
      </template>
    </v-snackbar>
  </v-app>
</template>

<script>
import EstimateForm from "../components/EstimateForm.vue";
import EstimatePreview from "../components/EstimatePreview.vue";

export default {
  name: "App",
  components: {
    EstimateForm,
    EstimatePreview,
  },
  data() {
    return {
      currentView: "form",
      // The single source of truth for all form data
      estimateData: {
        date: new Date().toISOString().substr(0, 10),
        companyName: "",
        siteName: "",
        purpose: "",
        personName: "",
        phoneNumber1: "",
        phoneNumber2: "",
        categories: [
          {
            name: "",
            items: [
              {
                description: "",
                length: "",
                width: "",
                unit: "Nos",
                quantity: 1,
                rate: 0,
                unitPrice: 0,
                total: 0,
                type: "",
                imageFile: null,
                imagePreviewUrl: null,
              },
            ],
          },
        ],
      },
      snackbar: {
        show: false,
        message: "",
        color: "success",
      },
    };
  },
  computed: {
    formattedEstimate() {
      // The data is already correctly formatted and stored
      return {
        ...this.estimateData,
        date: this.formatDate(this.estimateData.date),
        phoneNumbers: this.formatPhoneNumbers(this.estimateData),
        categories: this.estimateData.categories.map((category) => ({
          ...category,
          items: category.items.map((item) => ({
            ...item,
            // description: category.name || "Item Description", // Use category name as description
          })),
        })),
      };
    },
  },
  methods: {
    clearStorage() {
      localStorage.removeItem("estimate-data");
      location.reload();
    },

    showPreview() {
      this.currentView = "preview";
      this.showSnackbar("Estimate preview generated successfully!", "success");
    },
    goBackToForm() {
      this.currentView = "form";
    },
    formatDate(dateString) {
      if (!dateString) return new Date().toLocaleDateString("en-GB");
      const date = new Date(dateString);
      return date.toLocaleDateString("en-GB");
    },
    formatPhoneNumbers(estimateData) {
      const phones = [];
      if (estimateData.phoneNumber1) {
        phones.push({ label: "Mobile", number: estimateData.phoneNumber1 });
      }
      if (estimateData.phoneNumber2) {
        phones.push({ label: "Office", number: estimateData.phoneNumber2 });
      }
      return phones;
    },
    showSnackbar(message, color = "success") {
      this.snackbar.message = message;
      this.snackbar.color = color;
      this.snackbar.show = true;
    },
  },
};
</script>
