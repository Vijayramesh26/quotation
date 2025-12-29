<template>
  <v-container  class="pa-0 grey lighten-4"  >
    <div class="a4-container" >
      <div class="responsive-wrapper">
        <div class="a4-sheet shadow-lg" ref="estimateSheet">
          <div class="header-section" > 
            <v-row no-gutters align="center">
              <v-col cols="8">
                <div class="company-brand">{{ localEstimate.companyName }}</div>
                <div class="site-location mt-1">
                  <v-icon small color="teal">mdi-map-marker</v-icon>
                  {{ localEstimate.siteName }}
                </div>
              </v-col>
              <v-col cols="4" class="text-right">
                <div class="doc-type">{{ localEstimate.billType }}</div>
                <div class="date-label">DATE: {{ localEstimate.date }}</div>
              </v-col>
            </v-row>
          </div>

          <div class="info-grid-bar my-4">
            <v-row no-gutters>
              <v-col cols="6" class="pa-4 border-right">
                <div class="info-group">
                  <span class="info-label">CLIENT / வாடிக்கையாளர்</span>
                  <div class="info-value text-uppercase">
                    {{ localEstimate.clientName || "Valued Client" }}
                    <div
                    class="info-sub-value"
                    v-if="
                      localEstimate.clientNumber 
                    "
                  >
                    <v-icon x-small color="teal">mdi-phone</v-icon>
                    {{ localEstimate.clientNumber }}
                  </div>
                  </div>
                </div>
              </v-col>

              <v-col cols="6" class="pa-4 text-right">
                <div class="info-group">
                  <span class="info-label">OWNER / உரிமையாளர்</span>
                  <div class="info-value text-uppercase">
                    {{ localEstimate.personName }}
                  </div>
                  <div
                    class="info-sub-value"
                    v-if="
                      localEstimate.phoneNumbers &&
                      localEstimate.phoneNumbers.length
                    "
                  >
                    <v-icon x-small color="teal">mdi-phone</v-icon>
                    {{ localEstimate.phoneNumbers[0].number }}
                  </div>
                </div>
              </v-col>
            </v-row>
          </div>

          <div
            class="purpose-banner text-center my-4"
            v-if="localEstimate.purpose"
          >
            {{ localEstimate.purpose }}
          </div>

          <div class="estimate-table">
            <table class="professional-table">
              <thead>
                <tr>
                  <th class="text-left">Description / விவரம்</th>
                  <th class="text-center">Dimensions</th>
                  <th class="text-center">Sq.ft</th>
                  <th class="text-center">Unit</th>
                  <th class="text-center">Qty</th>
                  <th class="text-right">Rate</th>
                  <th class="text-right">Total (Rs.)</th>
                </tr>
              </thead>
              <tbody>
                <template
                  v-for="(
                    category, categoryIndex
                  ) in localEstimate.categories || []"
                >
                  <tr
                    v-if="category.name"
                    class="category-divider-row"
                    :key="`cat-${categoryIndex}`"
                  >
                    <td colspan="7" class="category-name-cell">
                      {{ category.name }}
                    </td>
                  </tr>
                  <tr
                    v-for="(item, itemIndex) in category.items || []"
                    :key="`item-${categoryIndex}-${itemIndex}`"
                    class="data-row"
                  >
                    <td class="desc-cell">{{ item.description || "—" }}</td>
                    <td class="text-center dim-cell">
                      {{
                        item.length && item.width
                          ? `${item.length} × ${item.width}`
                          : "-"
                      }}
                    </td>
                    <td class="text-center">{{ item.type || "-" }}</td>
                    <td class="text-center">{{ item.unit || "Nos" }}</td>
                    <td class="text-center font-weight-bold">
                      {{ item.quantity }}
                    </td>
                    <td class="text-right">₹{{ item.rate }}</td>
                    <td class="text-right font-weight-bold">
                      ₹{{ item.total }}
                    </td>
                  </tr>
                </template>
              </tbody>
              <tfoot>
                <tr class="grand-total-row">
                  <td colspan="6" class="text-right total-label">
                    GRAND TOTAL (மொத்தம்)
                  </td>
                  <td class="text-right total-amount">
                    ₹ {{ calculateTotal() }}
                  </td>
                </tr>
              </tfoot>
            </table>
          </div>

          <div v-if="hasImages" class="image-gallery-section mt-10">
            <div class="gallery-title d-flex align-center">
              <v-divider class="mr-4"></v-divider>
              <span>IMAGE GALLERY / பட தொகுப்பு</span>
              <v-divider class="ml-4"></v-divider>
            </div>
            <v-row class="mt-4">
              <v-col
                v-for="(image, index) in allImages"
                :key="index"
                cols="6"
                class="pa-2"
              >
                <div class="gallery-card">
                  <img :src="image.url" class="gallery-img" />
                  <div class="gallery-caption">{{ image.description }}</div>
                </div>
              </v-col>
            </v-row>
          </div>
        </div>
      </div>
    </div>

    <v-card class="action-bar-fixed px-6 py-3 rounded-pill elevation-10">
      <v-btn color="teal" dark @click="exportToPDF" class="mx-2 rounded-lg" :small="$vuetify.breakpoint.xsOnly">
        <v-icon left>mdi-file-pdf-box</v-icon> PDF
      </v-btn>
      <v-btn
        color="blue-grey darken-3"
        dark
        @click="exportToWord"
        class="mx-2 rounded-lg"
        :small="$vuetify.breakpoint.xsOnly"
      >
        <v-icon left>mdi-file-word-box</v-icon> Word
      </v-btn>
      <v-btn
        color="deep-purple"
        @click="translateUserContent"
        class="mx-2 white--text rounded-lg"
        :loading="loading"
        :small="$vuetify.breakpoint.xsOnly"
      >
        <span class="mr-2 font-weight-black">அ</span> PDF (தமிழ்)
      </v-btn>
    </v-card>
  </v-container>
</template>

<script>
// ... (All your imports remain identical)
import jsPDF from "jspdf";
import html2canvas from "html2canvas";
import {
  Document,
  Packer,
  Paragraph,
  Table,
  TableRow,
  TableCell,
  TextRun,
  AlignmentType,
  ImageRun,
} from "docx";
import { saveAs } from "file-saver";
import axios from "axios";

export default {
  name: "EstimatePreview",
  data() {
    return {
      localEstimate: null,
      isTamil: false,
      loading: false,
    };
  },
  created() {
    this.initializeData();
  },
  props: {
    estimate: {
      type: Object,
      default: () => ({
        companyName: "ABC CONSTRUCTION",
        date: "2025-08-21",
        personName: "P.Ramesh",
        phoneNumbers: [{ label: "Mobile", number: "9876543210" }],
        siteName: "RESIDENTIAL PROJECT",
        purpose: "Proposed Construction Estimate",
        categories: [],
      }),
    },
  },
  computed: {
    allImages() {
      const images = [];
      if (this.estimate && this.estimate.categories) {
        this.estimate.categories.forEach((cat) => {
          cat.items.forEach((item) => {
            if (item.imagePreviewUrl)
              images.push({
                url: item.imagePreviewUrl,
                description: item.description || "",
              });
          });
        });
      }
      return images;
    },
    hasImages() {
      return this.allImages.length > 0;
    },
  },
  methods: {
    initializeData() {
      this.localEstimate = JSON.parse(JSON.stringify(this.estimate));
    },
    calculateTotal() {
      let total = 0;
      this.estimate.categories.forEach((cat) => {
        cat.items.forEach((item) => {
          total += parseFloat(item.total) || 0;
        });
      });
      return total.toLocaleString(undefined, { minimumFractionDigits: 2 });
    },
    async exportToPDF() {
      this.loading = true;
      const element = this.$refs.estimateSheet;

      try {
        const canvas = await html2canvas(element, {
          scale: 3, // High resolution
          useCORS: true,
          backgroundColor: "#ffffff",
          logging: false,
        });

        const imgData = canvas.toDataURL("image/png", 1.0);
        const pdf = new jsPDF("p", "mm", "a4");
        const imgWidth = 210;
        const imgHeight = (canvas.height * imgWidth) / canvas.width;

        pdf.addImage(
          imgData,
          "PNG",
          0,
          0,
          imgWidth,
          imgHeight,
          undefined,
          "FAST"
        );
        pdf.save(`${this.localEstimate.siteName || "Estimate"}.pdf`);
      } catch (e) {
        console.error("PDF Error", e);
      } finally {
        this.loading = false;
      }
    },
    async getTamil(text) {
      if (!text || !isNaN(text) || text.length < 2) return text;
      try {
        const res = await axios.get("https://api.mymemory.translated.net/get", {
          params: { q: text, langpair: "en|ta" },
        });
        return res.data.responseData.translatedText;
      } catch (e) {
        return text;
      }
    },
    async translateUserContent() {
      this.loading = true;
      this.initializeData();
      try {
        this.localEstimate.companyName = await this.getTamil(
          this.localEstimate.companyName
        );
        this.localEstimate.siteName = await this.getTamil(
          this.localEstimate.siteName
        );
        this.localEstimate.personName = await this.getTamil(
          this.localEstimate.personName
        );

        for (let i = 0; i < this.localEstimate.categories.length; i++) {
          let category = this.localEstimate.categories[i];
          if (category.name) category.name = await this.getTamil(category.name);
          for (let j = 0; j < category.items.length; j++) {
            category.items[j].description = await this.getTamil(
              category.items[j].description
            );
          }
        }
        await this.$nextTick();
        await new Promise((r) => setTimeout(r, 1000));
        await this.exportToPDF();
        this.initializeData(); // Reset to English
      } catch (error) {
        console.error(error);
      } finally {
        this.loading = false;
      }
    },

    async waitForImages(container) {
      const images = container.querySelectorAll("img");
      const promises = [];

      images.forEach((img) => {
        if (img.complete && img.naturalHeight !== 0) return;

        promises.push(
          new Promise((resolve) => {
            img.onload = resolve;
            img.onerror = resolve; // don't block PDF for bad images
          })
        );
      });

      await Promise.all(promises);
    },

    async exportToWord() {
      const doc = new Document({
        sections: [
          {
            properties: {},
            children: [
              // Company Name
              new Paragraph({
                children: [
                  new TextRun({
                    text: this.estimate.companyName,
                    bold: true,
                    size: 32,
                  }),
                ],
                alignment: AlignmentType.CENTER,
              }),

              // Date and Person Info
              new Paragraph({
                children: [
                  new TextRun({
                    text: `Date: ${this.estimate.date}`,
                    size: 24,
                  }),
                  new TextRun({
                    text: `\t\t\tPerson: ${this.estimate.personName}`,
                    size: 24,
                  }),
                ],
              }),

              // Site Name
              new Paragraph({
                children: [
                  new TextRun({
                    text: this.estimate.siteName || "SITE NAME",
                    bold: true,
                    size: 28,
                  }),
                ],
                alignment: AlignmentType.CENTER,
              }),

              // Purpose
              new Paragraph({
                children: [
                  new TextRun({
                    text: this.estimate.purpose || "PURPOSE (HEADER)",
                    bold: true,
                    size: 24,
                  }),
                ],
              }),

              // Description
              new Paragraph({
                children: [
                  new TextRun({
                    text: this.estimate.description || "Project Description",
                  }),
                ],
                alignment: AlignmentType.CENTER,
              }),
              new Table({
                rows: this.createTableRows(),
              }),
              ...this.createImageParagraphs(),
            ],
          },
        ],
      });
      const blob = await Packer.toBlob(doc);
      saveAs(blob, `estimate-${this.estimate.date || "document"}.docx`);
    },
    createTableRows() {
      const rows = [];
      rows.push(
        new TableRow({
          children: [
            new TableCell({
              children: [
                new Paragraph({
                  children: [new TextRun({ text: "Description", bold: true })],
                }),
              ],
            }),
            new TableCell({
              children: [
                new Paragraph({
                  children: [new TextRun({ text: "Dimensions", bold: true })],
                }),
              ],
            }),
            new TableCell({
              children: [
                new Paragraph({
                  children: [new TextRun({ text: "Unit", bold: true })],
                }),
              ],
            }),
            new TableCell({
              children: [
                new Paragraph({
                  children: [new TextRun({ text: "Sq.ft", bold: true })],
                }),
              ],
            }),
            new TableCell({
              children: [
                new Paragraph({
                  children: [new TextRun({ text: "Qty", bold: true })],
                }),
              ],
            }),
            new TableCell({
              children: [
                new Paragraph({
                  children: [new TextRun({ text: "Rate", bold: true })],
                }),
              ],
            }),
            new TableCell({
              children: [
                new Paragraph({
                  children: [new TextRun({ text: "Price", bold: true })],
                }),
              ],
            }),
            new TableCell({
              children: [
                new Paragraph({
                  children: [new TextRun({ text: "Total (Rs.)", bold: true })],
                }),
              ],
            }),
          ],
        })
      );
      this.estimate.categories.forEach((category) => {
        if (category.name) {
          rows.push(
            new TableRow({
              children: [
                new TableCell({
                  children: [
                    new Paragraph({
                      children: [
                        new TextRun({ text: category.name, bold: true }),
                      ],
                    }),
                  ],
                }),
                ...Array(7)
                  .fill()
                  .map(
                    () =>
                      new TableCell({
                        children: [
                          new Paragraph({
                            children: [new TextRun({ text: "" })],
                          }),
                        ],
                      })
                  ),
              ],
            })
          );
        }
        category.items.forEach((item) => {
          const hasDimensions = item.length > 0 && item.width > 0;
          rows.push(
            new TableRow({
              children: [
                new TableCell({
                  children: [
                    new Paragraph({
                      children: [new TextRun({ text: item.description || "" })],
                    }),
                  ],
                }),
                new TableCell({
                  children: [
                    new Paragraph({
                      children: [
                        new TextRun({
                          text: hasDimensions
                            ? `${item.length} x ${item.width}`
                            : "-",
                        }),
                      ],
                    }),
                  ],
                }),
                new TableCell({
                  children: [
                    new Paragraph({
                      children: [new TextRun({ text: item.unit || "No" })],
                    }),
                  ],
                }),
                new TableCell({
                  children: [
                    new Paragraph({
                      children: [new TextRun({ text: item.type || "-" })],
                    }),
                  ],
                }),
                new TableCell({
                  children: [
                    new Paragraph({
                      children: [new TextRun({ text: item.quantity || "1" })],
                    }),
                  ],
                }),
                new TableCell({
                  children: [
                    new Paragraph({
                      children: [new TextRun({ text: item.rate || "" })],
                    }),
                  ],
                }),
                new TableCell({
                  children: [
                    new Paragraph({
                      children: [new TextRun({ text: item.unitPrice || "" })],
                    }),
                  ],
                }),
                new TableCell({
                  children: [
                    new Paragraph({
                      children: [new TextRun({ text: `${item.total}` })],
                    }),
                  ],
                }),
              ],
            })
          );
        });
      });
      rows.push(
        new TableRow({
          children: [
            ...Array(6)
              .fill()
              .map(
                () =>
                  new TableCell({
                    children: [
                      new Paragraph({ children: [new TextRun({ text: "" })] }),
                    ],
                  })
              ),
            new TableCell({
              children: [
                new Paragraph({
                  children: [new TextRun({ text: "Total", bold: true })],
                }),
              ],
            }),
            new TableCell({
              children: [
                new Paragraph({
                  children: [
                    new TextRun({
                      text: `${this.calculateTotal()}`,
                      bold: true,
                    }),
                  ],
                }),
              ],
            }),
          ],
        })
      );
      return rows;
    },
    createImageParagraphs() {
      const paragraphs = [];
      paragraphs.push(
        new Paragraph({
          children: [
            new TextRun({ text: "Image Gallery", bold: true, size: 28 }),
          ],
        })
      );
      this.allImages.forEach((image) => {
        paragraphs.push(
          new Paragraph({
            children: [
              new ImageRun({
                data: image.url,
                transformation: {
                  width: 500,
                  height: 300,
                },
              }),
            ],
          })
        );
        paragraphs.push(
          new Paragraph({
            children: [new TextRun({ text: image.description, italics: true })],
            alignment: AlignmentType.CENTER,
          })
        );
      });
      return paragraphs;
    },
  },
};
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&family=Noto+Sans+Tamil:wght@400;700&display=swap");

.a4-container {
  background-color: #f0f2f5;
  padding: 40px 0;
  min-height: 100vh; 
}
.a4-sheet {
  width: 210mm;
  min-height: 297mm;
  background: white;
  padding: 15mm 12mm;
  margin: 0 auto;
  font-family: "Inter", "Noto Sans Tamil", sans-serif;
  color: #1a1c23;
  position: relative;
  overflow-y: auto;
  scroll-behavior: smooth;
}

/* Header */
.header-section {
  border-bottom: 3px solid #009688;
  padding-bottom: 15px;
  margin-bottom: 10px;
}
.company-brand {
  font-size: 28px;
  font-weight: 900;
  color: #00796b;
  text-transform: uppercase;
  letter-spacing: -1px;
}
.site-location {
  font-size: 13px;
  font-weight: 600;
  color: #666;
}
.doc-type {
  font-size: 22px;
  font-weight: 900;
  color: #e0e0e0;
  letter-spacing: 2px;
}
.date-label {
  font-size: 12px;
  font-weight: 700;
  color: #333;
}

/* Client Bar */
.client-info-bar {
  background: #f8f9fa;
  padding: 10px 15px;
  border-radius: 8px;
  margin-bottom: 20px;
}
.client-detail .label {
  font-size: 10px;
  font-weight: 800;
  color: #999;
  display: block;
}
.client-detail .value {
  font-size: 14px;
  font-weight: 700;
  color: #333;
}

/* Table */
.professional-table {
  width: 100%;
  border-collapse: collapse;
}
/* Table Header: Light & Sharp */
.professional-table thead th {
  background: #ffffff;             /* Pure white background */
  color: rgb(13, 83, 83);          /* Dark teal only for text */
  padding: 12px 8px;
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 1.5px;
  border-top: 2px solid rgb(13, 83, 83);    /* Solid line on top */
  border-bottom: 1px solid rgb(13, 83, 83); /* Thin line on bottom */
}

/* Category Row: Subtle Wash */
.category-name-cell {
  background: #f4f8f8;             /* Very faint teal wash */
  color: rgb(13, 83, 83);
  font-weight: 800;
  padding: 10px;
  font-size: 13px;
  border-bottom: 1px solid #e0eaea;
}
.data-row td {
  padding: 12px 8px;
  border-bottom: 1px solid #edf2f7;
  font-size: 13px;
}
.desc-cell {
  font-weight: 500;
  width: 30%;
}
.grand-total-row td {
  padding: 20px 10px;
}
.total-label {
  font-size: 14px;
  font-weight: 800;
  color: #666;
}
/* Grand Total: Clear Emphasis */
.total-amount {
  font-size: 20px;
  font-weight: 900;
  color: rgb(13, 83, 83);
  border-top: 1px solid rgb(13, 83, 83); /* Line above the total */
}

/* Gallery */
.gallery-title {
  color: #999;
  font-size: 12px;
  font-weight: 800;
  letter-spacing: 2px;
}
.gallery-card {
  border: 1px solid #eee;
  border-radius: 8px;
  overflow: hidden;
}
.gallery-img {
  width: 100%;
  height: 180px;
  object-fit: cover;
}
.gallery-caption {
  padding: 8px;
  font-size: 11px;
  text-align: center;
  background: #f8f9fa;
  font-style: italic;
}

/* Fixed Actions */
.action-bar-fixed {
  position: fixed;
  bottom: 30px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 100;
  display: flex;
  align-items: center;
}
.sig-line {
  border-top: 1px solid #333;
  width: 150px;
  margin-left: auto;
}

.info-grid-bar {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  background-color: #fafafa;
  overflow: hidden;
}
.border-right {
  border-right: 1px solid #e0e0e0;
}
.info-label {
  font-size: 10px;
  font-weight: 800;
  color: #888;
  letter-spacing: 0.5px;
  text-transform: uppercase;
}
.info-value {
  font-size: 15px;
  font-weight: 700;
  color: #222;
}
.info-sub-value {
  font-size: 13px;
  font-weight: 600;
  color: #00796b;
}
/* ===== Mobile Scroll Without Breaking PDF ===== */
@media (max-width: 768px) {
  .a4-container {
    height: 100vh;
    padding: 0;
  }

  .responsive-wrapper {
    height: 100vh;
    overflow-y: auto;
    -webkit-overflow-scrolling: touch;
  }

  /* A4 stays full size for PDF */
  .a4-sheet {
    min-height: 297mm;
    overflow: visible; /* VERY IMPORTANT */
  }
}


</style>
