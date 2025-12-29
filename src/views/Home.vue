<template>
  <v-app class="grey lighten-5">
    <v-app-bar
      flat
      :class="
        $vuetify.breakpoint.smAndDown
          ? 'px-2'
          : 'mx-4 mt-4 rounded-xl px-4 elevation-2 border-teal'
      "
      :color="$vuetify.breakpoint.smAndDown ? 'white' : 'white'"
      :height="$vuetify.breakpoint.smAndDown ? 64 : 75"
      style="z-index: 100"
    >
      <div class="d-flex align-center">
        <v-avatar
          :color="
            $vuetify.breakpoint.smAndDown ? 'transparent' : 'teal lighten-5'
          "
          :size="$vuetify.breakpoint.smAndDown ? 32 : 40"
          :class="$vuetify.breakpoint.smAndDown ? 'mr-1' : 'mr-3'"
        >
          <v-icon color="teal">
            mdi-file-document-edit-outline
          </v-icon>
        </v-avatar>

        <div v-if="$vuetify.breakpoint.mdAndUp">
          <div
            class="text-overline teal--text font-weight-black line-height-1 hidden-xs-only"
            style="font-size: 0.65rem !important"
          >
            Precision
          </div>
          <div
            :class="
              $vuetify.breakpoint.smAndDown
                ? 'text-subtitle-2'
                : 'text-subtitle-1'
            "
            class="font-weight-black grey--text text--darken-4 mt-n1"
          >
            ESTIMATE<span class="hidden-xs-only"> GENETATOR</span>
          </div>
        </div>
      </div>

      <v-spacer></v-spacer>

      <div class="hidden-sm-and-down d-flex align-center mr-8">
        <v-chip
          :color="currentView === 'form' ? 'teal' : 'grey lighten-3'"
          :text-color="currentView === 'form' ? 'white' : 'grey'"
          label
          class="font-weight-bold mr-2 caption"
        >
          1. DRAFTING
        </v-chip>
        <v-icon small color="grey lighten-1">mdi-chevron-right</v-icon>
        <v-chip
          :color="currentView === 'preview' ? 'teal' : 'grey lighten-3'"
          :text-color="currentView === 'preview' ? 'white' : 'grey'"
          label
          class="font-weight-bold ml-2 caption"
        >
          2. REVIEW
        </v-chip>
      </div>

      <div class="d-flex align-center">
        <v-btn
          v-if="currentView === 'preview'"
          depressed
          color="teal lighten-5 teal--text"
          @click="goBackToForm"
          :class="
            $vuetify.breakpoint.xs ? 'mr-1 px-2' : 'rounded-lg mr-2 caption'
          "
          :icon="$vuetify.breakpoint.xs"
          :small="$vuetify.breakpoint.xs"
        >
          <v-icon :left="!$vuetify.breakpoint.xs">mdi-arrow-left</v-icon>
          <span class="hidden-xs-only">Edit Form</span>
        </v-btn>

        <v-btn
          outlined
          color="error"
          @click="clearStorage"
          :class="$vuetify.breakpoint.xs ? 'px-2' : 'rounded-lg caption'"
          :icon="$vuetify.breakpoint.xs"
          :small="$vuetify.breakpoint.xs"
        >
          <v-icon :left="!$vuetify.breakpoint.xs">mdi-refresh</v-icon>
          <span class="hidden-xs-only">Reset</span>
        </v-btn>
      </div>
    </v-app-bar>

    <v-main class="mt-6">
      <v-container>
        <v-fade-transition mode="out-in">
          <EstimateForm
            v-if="currentView === 'form'"
            v-model="estimateData"
            @preview="showPreview"
            key="form"
          />

          <EstimatePreview
            v-if="currentView === 'preview'"
            :estimate="formattedEstimate"
            key="preview"
          />
        </v-fade-transition>
      </v-container>
    </v-main>

    <v-snackbar
      v-model="snackbar.show"
      :color="snackbar.color"
      elevation="24"
      rounded="pill"
      class="mb-6"
    >
      <v-icon left color="white">
        {{ snackbar.color === "success" ? "mdi-check-circle" : "mdi-alert" }}
      </v-icon>
      <span class="font-weight-bold">{{ snackbar.message }}</span>
      <template v-slot:action="{ attrs }">
        <v-btn icon v-bind="attrs" @click="snackbar.show = false">
          <v-icon>mdi-close</v-icon>
        </v-btn>
      </template>
    </v-snackbar>
  </v-app>
</template>

<script>
// ... (Your script logic remains exactly as you provided)
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
      estimateData: {
        date: new Date().toISOString().substr(0, 10),
        companyName: "",
        billType: "",
        clientName: "",
        clientNumber: "",
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
      snackbar: { show: false, message: "", color: "success" },
    };
  },
  computed: {
    formattedEstimate() {
      return {
        ...this.estimateData,
        date: this.formatDate(this.estimateData.date),
        phoneNumbers: this.formatPhoneNumbers(this.estimateData),
        categories: this.estimateData.categories.map((category) => ({
          ...category,
          items: category.items.map((item) => ({ ...item })),
        })),
      };
    },
  },
  methods: {
    clearStorage() {
      if (confirm("Are you sure? This will wipe all current data.")) {
        localStorage.removeItem("estimate-data");
        location.reload();
      }
    },
    showPreview() {
      this.currentView = "preview";
      this.showSnackbar("Reviewing generated estimate...", "success");
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
      if (estimateData.phoneNumber1)
        phones.push({ label: "Mobile", number: estimateData.phoneNumber1 });
      if (estimateData.phoneNumber2)
        phones.push({ label: "Office", number: estimateData.phoneNumber2 });
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

<style scoped>
.v-main {
  background-color: #f5f7fa;
}
.border-teal {
  border: 1px solid rgba(0, 150, 136, 0.2) !important;
}
.line-height-1 {
  line-height: 1 !important;
}
/* Ensure the app looks centered and professional on very wide screens */
@media (min-width: 1904px) {
  .v-container {
    max-width: 1400px;
  }
}
</style>
