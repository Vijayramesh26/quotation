<template>
  <v-container class="py-10 grey lighten-4" fluid>
    <v-row justify="center">
      <v-col cols="12" lg="10" xl="8">
        
        <div class="d-flex align-center mb-6">
          <v-btn icon @click="$emit('back')" class="mr-4">
            <v-icon>mdi-arrow-left</v-icon>
          </v-btn>
          <h1 class="text-h4 font-weight-black teal--text text--darken-3">Estimate Creator</h1>
          <v-spacer></v-spacer>
          <!-- <v-chip color="teal lighten-5" text-color="teal" label font-weight-bold>
            <v-icon left size="20">mdi-file-document-edit</v-icon> Draft Mode
          </v-chip> -->
        </div>

        <v-form ref="form" v-model="valid">
          
          <v-card class="mb-8 pa-8 rounded-xl elevation-2 border-top-teal">
            <div class="text-subtitle-1 font-weight-bold mb-6 grey--text text--darken-2">
              <v-icon left color="teal">mdi-information</v-icon> Project Information
            </div>
            <v-row>
              
              <v-col cols="12" md="4">
                <v-text-field v-model="estimate.date" label="Date" type="date" flat solo-inverted hide-details class="rounded-lg"></v-text-field>
              </v-col>
              <v-col cols="12" md="4">
                <v-text-field v-model="estimate.companyName" label="Company Name" placeholder="e.g. Acme Corp" flat solo-inverted hide-details class="rounded-lg"></v-text-field>
              </v-col>
              <v-col cols="12" md="4">
                <v-text-field v-model="estimate.purpose" label="Purpose" flat solo-inverted hide-details class="rounded-lg"></v-text-field>
              </v-col>
               <v-col cols="12" md="4">
                <v-text-field v-model="estimate.clientName" label="Client Name" flat solo-inverted hide-details class="rounded-lg"></v-text-field>
              </v-col>
              <v-col cols="12" md="4">
                <v-text-field v-model="estimate.clientNumber" label="Client Phone Number" type="tel" flat solo-inverted hide-details class="rounded-lg"></v-text-field>
              </v-col>
              <v-col cols="12" md="4">
                <v-autocomplete v-model="estimate.billType" :items="['ESTIMATE', 'BILL', 'TAX INVOICE']" label="Bill Type" flat solo-inverted hide-details class="rounded-lg"></v-autocomplete>
              </v-col>
              <v-col cols="12" md="4">
                <v-text-field v-model="estimate.siteName" label="Site Name" placeholder="e.g. Riverside Project" flat solo-inverted hide-details class="rounded-lg"></v-text-field>
              </v-col>
              <v-col cols="12" md="4">
                <v-text-field v-model="estimate.personName" label="Contact Person" flat solo-inverted hide-details class="rounded-lg"></v-text-field>
              </v-col>
              <v-col cols="12" md="4">
                <v-text-field v-model="estimate.phoneNumber1" label="Phone Number" type="tel" flat solo-inverted hide-details class="rounded-lg"></v-text-field>
              </v-col>
               <!-- <v-col cols="12" md="4">
                <v-text-field v-model="estimate.phoneNumber2" label="Phone Number" type="tel" flat solo-inverted hide-details class="rounded-lg"></v-text-field>
              </v-col> -->
            </v-row>
          </v-card>

          <div v-for="(category, categoryIndex) in estimate.categories" :key="categoryIndex" class="mb-10">
            <div class="d-flex align-center mb-4">
              <v-avatar color="teal" size="20" class="white--text font-weight-bold mr-3">{{ categoryIndex + 1 }}</v-avatar>
              <v-text-field
                v-model="category.name"
                placeholder="Category Name (e.g. Civil Works, Electrical)"
                hide-details
                full-width
                class=" font-weight-bold"
              ></v-text-field>
              <v-btn icon color="error" @click="removeCategory(categoryIndex)" v-if="estimate.categories.length > 1">
                <v-icon>mdi-delete-outline</v-icon>
              </v-btn>
            </div>

            <v-card class="rounded-xl overflow-hidden elevation-1 bg-white">
              <div v-for="(item, itemIndex) in category.items" :key="itemIndex" class="pa-6 position-relative border-bottom">
                
                <v-row align="center">
                  <v-col cols="12" md="4">
                    <v-text-field v-model="item.description" label="Item Description" dense outlined hide-details color="teal"></v-text-field>
                  </v-col>
                  <v-col cols="6" md="1">
                    <v-text-field v-model.number="item.length" label="L" dense outlined hide-details @input="calculateItemTotal(categoryIndex, itemIndex)"></v-text-field>
                  </v-col>
                  <v-col cols="6" md="1">
                    <v-text-field v-model.number="item.width" label="W" dense outlined hide-details @input="calculateItemTotal(categoryIndex, itemIndex)"></v-text-field>
                  </v-col>
                  <v-col cols="6" md="2">
                    <v-select v-model="item.unit" :items="unitOptions" label="Unit" dense outlined hide-details @input="calculateItemTotal(categoryIndex, itemIndex)"></v-select>
                  </v-col>
                  <v-col cols="6" md="1">
                    <v-text-field v-model.number="item.quantity" label="Qty" dense outlined hide-details @input="calculateItemTotal(categoryIndex, itemIndex)"></v-text-field>
                  </v-col>
                  <v-col cols="6" md="1">
                    <v-text-field v-model.number="item.rate" label="Rate" dense outlined hide-details @input="calculateItemTotal(categoryIndex, itemIndex)"></v-text-field>
                  </v-col>
                  <v-col cols="6" md="2">
                    <v-text-field :value="item.total" label="Total" dense filled readonly hide-details color="teal" class="font-weight-bold"></v-text-field>
                  </v-col>
                </v-row>

                <v-row class="mt-2" align="center">
                   <v-col cols="10">
                    <v-file-input
                      v-model="item.imageFile"
                      label="Attach Photo"
                      prepend-icon="mdi-camera"
                      dense
                      flat
                      hide-details
                      class="caption"
                    ></v-file-input>
                   </v-col>
                   <v-col cols="2" class="text-right">
                     <v-btn icon small color="grey" @click="removeItem(categoryIndex, itemIndex)" v-if="category.items.length > 1">
                        <v-icon>mdi-close-circle-outline</v-icon>
                      </v-btn>
                      <v-btn icon small color="teal" @click="addItem(categoryIndex)">
                        <v-icon>mdi-plus-circle</v-icon>
                      </v-btn>
                   </v-col>
                </v-row>
              </div>
            </v-card>
          </div>

          <v-btn block x-large dashed color="teal" class="rounded-xl border-dashed mb-10" @click="addCategory" outlined>
            <v-icon left>mdi-plus-box</v-icon> Add Another Category
          </v-btn>

        </v-form>
      </v-col>
    </v-row>

    <v-footer 
    app 
    inset 
    :height="$vuetify.breakpoint.xs ? 70 : 80" 
    class="elevation-10 bg-white border-top-teal px-0 rounded-xl mx-4 mb-4  px-4"
  >
    <v-container class="py-0 fill-height">
      <v-row align="center" no-gutters>
        <v-col cols="5" sm="6">
          <div class="d-flex flex-column justify-center">
            <span class="text-caption grey--text text-uppercase font-weight-black line-height-1 mb-1" style="font-size: 0.65rem !important;">
              Total <span class="hidden-xs-only">Estimate</span>
            </span>
            <div :class="$vuetify.breakpoint.xs ? 'text-subtitle-1' : 'text-h5'" class="font-weight-black teal--text line-height-1">
              ₹ {{ grandTotal.toLocaleString() }}
            </div>
          </div>
        </v-col>

        <v-col cols="7" sm="6" class="text-right">
          <v-btn
            color="teal"
            :large="!$vuetify.breakpoint.xs"
            
            depressed
            :block="$vuetify.breakpoint.xs"
            :class="$vuetify.breakpoint.xs ? 'px-2 rounded-lg' : 'px-10 rounded-lg elevation-4'"
            @click="previewEstimate"
            :disabled="isButtonDisabled"
          >
            <span :class="{'caption font-weight-bold': $vuetify.breakpoint.xs}">
              {{ $vuetify.breakpoint.xs ? 'Review' : 'Review & Export' }}
            </span>
            <v-icon :right="!$vuetify.breakpoint.xs" :small="$vuetify.breakpoint.xs">
              mdi-chevron-right
            </v-icon>
          </v-btn>
        </v-col>
      </v-row>
    </v-container>
  </v-footer>
  </v-container>
</template>

<style scoped>
.border-top-teal {
  border-top: 4px solid #009688 !important;
}
.border-bottom {
  border-bottom: 1px solid #eeeeee;
}
.border-bottom:last-child {
  border-bottom: none;
}
/* .category-input >>> input {
  font-size: 1.0rem;
} */
.border-dashed {
  border-style: dashed !important;
  border-width: 2px;
  text-transform: none;
  letter-spacing: 0;
}
.bg-white {
  background-color: white !important;
}
/* Ensure the footer doesn't feel too cramped on modern notched phones */
@media (max-width: 600px) {
  .v-footer {
    padding-bottom: env(safe-area-inset-bottom);
    height: auto !important;
    min-height: 70px;
  }
}
</style>

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
        "Qty.",
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
      } else if (item.unit === "Qty.") {
        item.type = "-";
        unitPrice = rate;
        totalValue = rate;
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
