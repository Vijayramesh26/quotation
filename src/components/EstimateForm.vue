<template>
  <v-container>
    <v-card class="pa-6">
      <v-card-title class="text-h4 text-center mb-6">
        Construction Estimate Form
      </v-card-title>

      <v-form ref="form" v-model="valid">
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

        <v-divider class="my-6"></v-divider>
        <v-card-subtitle class="text-h6 pa-0 mb-4"
          >Estimate Categories</v-card-subtitle
        >

        <v-card
          v-for="(category, categoryIndex) in estimate.categories"
          :key="categoryIndex"
          class="mb-6 pa-4"
          outlined
        >
          <v-row>
            <v-col cols="12" md="10">
              <v-text-field
                v-model="category.name"
                label="Description / Category"
                outlined
                dense
              ></v-text-field>
            </v-col>
            <v-col cols="12" md="2" class="d-flex align-center justify-end">
              <v-btn
                color="error"
                icon
                small
                @click="removeCategory(categoryIndex)"
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
                  dense
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
                  dense
                  @input="calculateItemTotal(categoryIndex, itemIndex)"
                ></v-text-field>
              </v-col>
              <v-col cols="6" md="2">
                <v-text-field
                  v-model.number="item.width"
                  label="Width"
                  type="number"
                  outlined
                  dense
                  @input="calculateItemTotal(categoryIndex, itemIndex)"
                ></v-text-field>
              </v-col>
              <v-col cols="6" md="1">
                <v-select
                  v-model="item.unit"
                  :items="unitOptions"
                  label="Unit"
                  outlined
                  dense
                ></v-select>
              </v-col>
              <v-col cols="6" md="2">
                <v-text-field
                  v-model.number="item.quantity"
                  label="Qty"
                  type="number"
                  outlined
                  dense
                  @input="calculateItemTotal(categoryIndex, itemIndex)"
                ></v-text-field>
              </v-col>
              <v-col cols="8" md="2">
                <v-text-field
                  v-model.number="item.rate"
                  label="Rate"
                  type="number"
                  outlined
                  dense
                  @input="calculateItemTotal(categoryIndex, itemIndex)"
                ></v-text-field>
              </v-col>
              <v-col cols="4" md="2">
                <v-text-field
                  :value="item.total"
                  label="Total"
                  outlined
                  dense
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
                  prepend-icon="mdi-camera"
                  outlined
                  dense
                ></v-file-input>
                <v-btn
                  color="secondary"
                  icon
                  small
                  @click="removeItem(categoryIndex, itemIndex)"
                  class="ml-2 mr-2"
                >
                  <v-icon>mdi-minus</v-icon>
                </v-btn>
                <v-btn color="info" icon small @click="addItem(categoryIndex)">
                  <v-icon>mdi-plus</v-icon>
                </v-btn>
              </v-col>
            </v-row>
          </div>
        </v-card>

        <v-row class="mb-4">
          <v-col cols="12">
            <v-btn color="primary" outlined @click="addCategory">
              <v-icon left>mdi-plus</v-icon>
              Add New Category
            </v-btn>
          </v-col>
        </v-row>

        <v-row>
          <v-col cols="12" class="text-right">
            <v-card class="pa-4" color="grey lighten-4">
              <div class="text-h5">
                <strong
                  >Grand Total: Rs. {{ grandTotal.toLocaleString() }}</strong
                >
              </div>
            </v-card>
          </v-col>
        </v-row>

        <v-row class="mt-6">
          <v-col cols="12" class="text-center">
            <v-btn
              color="primary"
              large
              @click="previewEstimate"
              :disabled="!valid"
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
      unitOptions: ["Nos", "Pcs", "Sq.ft", "Cu.ft", "Mtrs", "Lumpsum","Run.ft"],
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

      if (hasDimensions) {
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
  },
};
</script>
