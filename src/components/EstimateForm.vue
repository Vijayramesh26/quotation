<template>
  <v-container>
    <v-card class="pa-6 elevation-0">
      <v-card-title
        class="text-body-1 font-weight-bold text-md-h5 text-center d-flex align-center justify-center"
      >
        HEADER INFO
      </v-card-title>
      <v-form ref="form" v-model="valid">
        <v-card class="pa-6 rounded-xl" outlined>
          <v-row>
            <v-col cols="12" md="6">
              <v-text-field
                v-model="estimate.date"
                label="Date"
                type="date"
                outlined
                required
              ></v-text-field>
            </v-col>
            <v-col cols="12" md="6">
              <v-text-field
                v-model="estimate.companyName"
                label="Company Name"
                outlined
                required
              ></v-text-field>
            </v-col>
          </v-row>

          <v-row>
            <v-col cols="12" md="6">
              <v-text-field
                v-model="estimate.siteName"
                label="Site Name"
                outlined
                required
              ></v-text-field>
            </v-col>
            <v-col cols="12" md="6">
              <v-text-field
                v-model="estimate.purpose"
                label="Purpose (Header)"
                outlined
                required
              ></v-text-field>
            </v-col>
          </v-row>

          <v-row>
            <v-col cols="12" md="4">
              <v-text-field
                v-model="estimate.personName"
                label="Person Name"
                outlined
                required
              ></v-text-field>
            </v-col>
            <v-col cols="12" md="4">
              <v-text-field
                v-model="estimate.phoneNumber1"
                label="Phone Number 1"
                outlined
                type="tel"
              ></v-text-field>
            </v-col>
            <v-col cols="12" md="4">
              <v-text-field
                v-model="estimate.phoneNumber2"
                label="Phone Number 2 (Optional)"
                outlined
                type="tel"
              ></v-text-field>
            </v-col>
          </v-row>
        </v-card>
        <v-divider class="my-6"></v-divider>
        <v-card-subtitle
          class="text-body-1 font-weight-bold text-md-h5 text-center d-flex align-center justify-center"
          >CATEGORIES</v-card-subtitle
        >

        <v-card
          v-for="(category, categoryIndex) in estimate.categories"
          :key="categoryIndex"
          class="mb-6 pa-6 rounded-xl"
          outlined
        >
          <v-row>
            <v-col cols="10" md="11">
              <v-text-field
                v-model="category.name"
                label="Description / Category"
                outlined
              ></v-text-field>
            </v-col>
            <v-col cols="2" md="1" class="d-flex align-start justify-end">
              <v-btn
                color="error"
                icon
                small
                @click="removeCategory(categoryIndex)"
                class="mt-2 ml-4 ml-md-2"
              >
                <v-icon>mdi-delete</v-icon>
              </v-btn>
            </v-col>
          </v-row>

          <div
            v-for="(item, itemIndex) in category.items"
            :key="itemIndex"
            class="item-row"
          >
            <v-row>
              <v-col cols="12">
                <v-text-field
                  v-model="item.description"
                  label="Item Description"
                  outlined
                ></v-text-field>
              </v-col>
            </v-row>
            <v-row>
              <v-col cols="6" md="2">
                <v-text-field
                  v-model.number="item.length"
                  label="Length"
                  type="number"
                  outlined
                  @input="calculateItemTotal(categoryIndex, itemIndex)"
                ></v-text-field>
              </v-col>
              <v-col cols="6" md="2">
                <v-text-field
                  v-model.number="item.width"
                  label="Width"
                  type="number"
                  outlined
                  @input="calculateItemTotal(categoryIndex, itemIndex)"
                ></v-text-field>
              </v-col>
              <v-col cols="6" md="2">
                <v-select
                  v-model="item.unit"
                  :items="unitOptions"
                  label="Unit"
                  outlined
                  @input="calculateItemTotal(categoryIndex, itemIndex)"
                ></v-select>
              </v-col>
              <v-col cols="6" md="2">
                <v-text-field
                  v-model.number="item.quantity"
                  label="Qty"
                  type="number"
                  outlined
                  @input="calculateItemTotal(categoryIndex, itemIndex)"
                ></v-text-field>
              </v-col>
              <v-col cols="12" md="2">
                <v-text-field
                  v-model.number="item.rate"
                  label="Rate"
                  type="number"
                  outlined
                  @input="calculateItemTotal(categoryIndex, itemIndex)"
                ></v-text-field>
              </v-col>
              <v-col cols="12" md="2">
                <v-text-field
                  :value="item.total"
                  label="Total"
                  outlined
                  readonly
                ></v-text-field>
              </v-col>
            </v-row>
            <v-row class="mt-0">
              <v-col cols="12" class="d-flex align-center">
                <v-file-input
                  v-model="item.imageFile"
                  label="Upload Image for this Item (Optional)"
                  accept="image/*"
                  prepend-inner-icon="mdi-camera"
                  prepend-icon=""
                  outlined
                ></v-file-input>
                <v-btn
                  color="secondary"
                  icon
                  small
                  @click="removeItem(categoryIndex, itemIndex)"
                  class="ml-2 mr-2 mb-6"
                >
                  <v-icon class="">mdi-minus</v-icon>
                </v-btn>
                <v-btn
                  color="info"
                  icon
                  small
                  @click="addItem(categoryIndex)"
                  class="mb-6 ml-2"
                >
                  <v-icon class="">mdi-plus</v-icon>
                </v-btn>
              </v-col>
            </v-row>
          </div>
        </v-card>

        <v-row class="mb-4">
          <v-col cols="12" class="text-end">
            <v-btn
              color="teal"
              outlined
              @click="addCategory"
              class="rounded-lg text-body-2 text-md-subtitle-1"
            >
              <v-icon left>mdi-plus</v-icon>
              Add New Category
            </v-btn>
          </v-col>
        </v-row>

        <v-card class="pa-6 rounded-xl" outlined>
          <v-row>
            <v-col cols="5">
              <div class="text-body-2 text-md-subtitle-1">Grand Total:</div>
            </v-col>
            <v-col cols="7" class="text-right">
              <div class="text-subtitle-2 text-md-h6 font-weight-bold">
                Rs. {{ grandTotal.toLocaleString() }}
              </div>
            </v-col>
          </v-row>
        </v-card>

        <v-row class="mt-6">
          <v-col cols="12" class="text-center">
            <v-btn
              color="teal"
              class="white--text rounded-lg text-body-2 text-md-subtitle-1 font-weight-bold"
              large
              @click="previewEstimate"
              :disabled="isButtonDisabled"
            >
              Preview Estimate
            </v-btn>
          </v-col>
        </v-row>
      </v-form>
    </v-card>
  </v-container>
</template>

<script>
export default {
  name: "EstimateForm",
  props: {
    value: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      valid: false,
      unitOptions: [
        "Nos",
        "Pcs",
        "Sq.ft",
        "Cu.ft",
        "Mtrs",
        "Lumpsum",
        "Run.ft",
      ],
    };
  },
  computed: {
    estimate: {
      get() {
        return this.value;
      },
      set(newValue) {
        this.$emit("input", newValue);
      },
    },
    grandTotal() {
      return this.estimate.categories.reduce((sum, category) => {
        const categoryTotal = category.items.reduce(
          (itemSum, item) => itemSum + (parseFloat(item.total) || 0),
          0
        );
        return sum + categoryTotal;
      }, 0);
    },
    isButtonDisabled() {
      // 1️⃣ Header validation
      if (
        !this.estimate.date ||
        // !this.estimate.companyName ||
        !this.estimate.siteName
        // || !this.estimate.purpose ||
        // !this.estimate.personName
      ) {
        return true;
      }

      // 2️⃣ Categories + Items validation
      return this.estimate.categories.some((category) => {
        // category name required
        if (!category.name) return true;

        // items validation
        return category.items.some((item) => {
          return (
            // !item.description ||
            // item.quantity <= 0 ||
            item.rate <= 0 || item.total <= 0
          );
        });
      });
    },
  },
  methods: {
    addCategory() {
      this.estimate.categories.push({
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
      });
    },
    removeCategory(categoryIndex) {
      if (this.estimate.categories.length > 1) {
        this.estimate.categories.splice(categoryIndex, 1);
      }
    },
    addItem(categoryIndex) {
      this.estimate.categories[categoryIndex].items.push({
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
      });
    },
    removeItem(categoryIndex, itemIndex) {
      const category = this.estimate.categories[categoryIndex];
      if (category.items.length > 1) {
        category.items.splice(itemIndex, 1);
      }
    },
    calculateItemTotal(categoryIndex, itemIndex) {
      const item = this.estimate.categories[categoryIndex].items[itemIndex];
      const rate = parseFloat(item.rate) || 0;
      const quantity = parseFloat(item.quantity) || 0;

      // Check if length and width are provided for area-based calculation
      const hasDimensions = item.length > 0 && item.width > 0;

      let unitPrice;
      let totalValue;

      if (hasDimensions && item.unit === "Nos") {
        const area = parseFloat(item.length) * parseFloat(item.width);
        item.type = area.toFixed(2);
        unitPrice = area * rate;
        totalValue = unitPrice * quantity;
      } else {
        item.type = "-";
        unitPrice = rate;
        totalValue = rate * quantity;
      }

      item.unitPrice = unitPrice.toFixed(2);
      item.total = totalValue.toFixed(2);
    },
    async previewEstimate() {
      for (const category of this.estimate.categories) {
        for (const item of category.items) {
          if (item.imageFile) {
            item.imagePreviewUrl = await this.fileToBase64(item.imageFile);
          } else {
            item.imagePreviewUrl = null;
          }
        }
      }
      this.$emit("preview");
    },
    fileToBase64(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = () => resolve(reader.result);
        reader.onerror = (error) => reject(error);
      });
    },
    saveToLocalStorage() {
      try {
        localStorage.setItem("estimate-data", JSON.stringify(this.estimate));
      } catch (e) {
        console.warn("LocalStorage save failed", e);
      }
    },

    loadFromLocalStorage() {
      const saved = localStorage.getItem("estimate-data");
      if (saved) {
        try {
          const parsed = JSON.parse(saved);
          this.$emit("input", parsed); // IMPORTANT: update parent model
          this.localEstimate = JSON.parse(JSON.stringify(parsed));
        } catch (e) {
          console.warn("LocalStorage load failed", e);
        }
      } else {
        // First load
        this.localEstimate = JSON.parse(JSON.stringify(this.estimate));
      }
    },
  },
  created() {
    this.loadFromLocalStorage();
  },
  watch: {
    estimate: {
      deep: true,
      handler() {
        this.saveToLocalStorage();
      },
    },
  },
};
</script>
