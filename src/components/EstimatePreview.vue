<template>
  <v-container fluid class="pa-0">
    <div class="a4-container">
      <div class="a4-sheet" ref="estimateSheet">
        <div class="header-section">
          <div class="company-name">
            {{ estimate.companyName }}
          </div>

          <div class="info-row">
            <div class="date-section">
              {{ estimate.date }}
            </div>
            <div class="person-section">
              <div class="person-name">{{ estimate.personName }}</div>
              <div
                class="phone-numbers"
                v-if="estimate.phoneNumbers && estimate.phoneNumbers.length"
              >
                <div
                  v-for="(phone, index) in estimate.phoneNumbers"
                  :key="index"
                  class="phone-bar"
                >
                  <span class="phone-label">{{ phone.label }}:</span>
                  <span class="phone-number">{{ phone.number }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="site-name">
          {{ estimate.siteName }}
        </div>

        <div class="purpose-header text-center">
          {{ estimate.purpose || "CONSTRUCTION ESTIMATE" }}
        </div>

        <div class="calculation-info">
          *Calculations: Total = (Sq.ft x Qty x Rate) or (Qty x Rate) for Sq.ft
          Item*
        </div>

        <div class="estimate-table">
          <table>
            <thead>
              <tr>
                <th class="description-header">Description</th>
                <th class="dimensions-header">Dimensions</th>
                <th class="unit-header">Unit</th>
                <th class="type-header">Sq.ft</th>
                <th class="quantity-header">Qty</th>
                <th class="rate-header">
                  Rate
                  <br />
                  <span class="calculation-guide"> (Per Unit) </span>
                </th>
                <th class="price-header">
                  Price
                  <br />
                  <span class="calculation-guide"> (Sq.ft x Rate) </span>
                </th>
                <th class="total-header">
                  Total (Rs.)
                  <br />
                  <span class="calculation-guide"> (Price x Qty) </span>
                </th>
              </tr>
            </thead>
            <tbody>
              <template
                v-for="(category, categoryIndex) in estimate.categories || []"
              >
                <tr
                  v-if="category.name"
                  class="category-row"
                  :key="`category-${categoryIndex}`"
                >
                  <td colspan="8" class="category-cell">{{ category.name }}</td>
                </tr>
                <tr
                  v-for="(item, itemIndex) in category.items || []"
                  :key="`item-${categoryIndex}-${itemIndex}`"
                  class="item-row-table"
                >
                  <td class="description-cell">{{ item.description || "" }}</td>
                  <td class="dimensions-cell">
                    <span v-if="item.length && item.width"
                      >{{ item.length }} x {{ item.width }}</span
                    >
                    <span v-else>-</span>
                  </td>
                  <td class="unit-cell">{{ item.unit || "Nos" }}</td>
                  <td class="type-cell">{{ item.type || "-" }}</td>
                  <td class="quantity-cell">{{ item.quantity }}</td>
                  <td class="rate-cell">{{ item.rate }}</td>
                  <td class="price-cell">{{ item.unitPrice }}</td>
                  <td class="total-cell">{{ item.total }}</td>
                </tr>
              </template>
              <tr class="total-row-table">
                <td colspan="7" class="total-label-cell">Total</td>
                <td class="total-amount-cell">{{ calculateTotal() }}</td>
              </tr>
            </tbody>
          </table>
        </div>

        <div v-if="hasImages" class="image-gallery-section">
          <div class="image-gallery-header">Image Gallery</div>
          <div
            v-for="(image, index) in allImages"
            :key="`img-${index}`"
            class="image-figure"
          >
            <img :src="image.url" alt="Item Image" class="item-image" />
            <figcaption>{{ image.description }}</figcaption>
          </div>
        </div>
      </div>
    </div>

    <div class="text-center mt-2">
      <v-btn color="primary" @click="exportToPDF" class="mr-4">
        <v-icon left>mdi-file-pdf-box</v-icon>
        Download PDF
      </v-btn>
      <v-btn color="success" @click="exportToWord">
        <v-icon left>mdi-file-word-box</v-icon>
        Download Word
      </v-btn>
    </div>
  </v-container>
</template>

<script>
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

export default {
  name: "EstimatePreview",
  props: {
    estimate: {
      type: Object,
      default: () => ({
        companyName: "ABC CONSTRUCTION COMPANY",
        date: "21-08-2025",
        personName: "P.Ramesh",
        phoneNumbers: [
          { label: "Mobile", number: "9876543210" },
          { label: "Office", number: "0431-2345678" },
        ],
        siteName: "RESIDENTIAL BUILDING PROJECT",
        purpose: "",
        description:
          "This is a detailed description of the construction estimate.",
        categories: [
          {
            name: "",
            items: [
              {
                description: "",
                length: "6.5",
                width: "5",
                unit: "Nos",
                quantity: "1",
                rate: "32.5",
                unitPrice: "32.5",
                total: "11050",
                type: "32.50",
                imagePreviewUrl: null,
              },
              {
                description: "",
                length: "",
                width: "",
                unit: "Nos",
                quantity: "1",
                rate: "27.5",
                unitPrice: "27.5",
                total: "9350",
                type: "-",
                imagePreviewUrl: null,
              },
            ],
          },
        ],
      }),
    },
  },
  computed: {
    allImages() {
      const images = [];
      if (this.estimate && this.estimate.categories) {
        this.estimate.categories.forEach((category) => {
          category.items.forEach((item) => {
            if (item.imagePreviewUrl) {
              images.push({
                url: item.imagePreviewUrl,
                description: item.description || "",
              });
            }
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
    calculateTotal() {
      let total = 0;
      if (this.estimate && this.estimate.categories) {
        this.estimate.categories.forEach((category) => {
          if (category.items) {
            category.items.forEach((item) => {
              total += parseFloat(item.total) || 0;
            });
          }
        });
      }
      return total.toFixed(2);
    },
    async exportToPDF() {
      const element = this.$refs.estimateSheet;

      // ✅ Wait for Vue DOM update
      await this.$nextTick();

      // ✅ Wait for all images inside estimate sheet
      await this.waitForImages(element);

      const pdf = new jsPDF("p", "mm", "a4");
      const pageHeight = pdf.internal.pageSize.height;
      let y = 10;

      const mainContentSelectors = [
        ".header-section",
        ".site-name",
        ".purpose-header",
        ".calculation-info",
        ".estimate-table",
      ];

      for (const selector of mainContentSelectors) {
        const contentElement = element.querySelector(selector);
        if (!contentElement) continue;

        const canvas = await html2canvas(contentElement, {
          scale: 2,
          useCORS: true,
          allowTaint: false,
          backgroundColor: "#ffffff",
        });

        if (!canvas.width || !canvas.height) continue;

        const imgData = canvas.toDataURL("image/png");
        const imgWidth = pdf.internal.pageSize.width - 20;
        const imgHeight = (canvas.height * imgWidth) / canvas.width;

        if (y + imgHeight > pageHeight - 10) {
          pdf.addPage();
          y = 10;
        }

        pdf.addImage(imgData, "PNG", 10, y, imgWidth, imgHeight);
        y += imgHeight + 5;
      }

      // ===== IMAGE GALLERY =====
      const imageGalleryElement = element.querySelector(
        ".image-gallery-section"
      );

      if (imageGalleryElement) {
        await this.waitForImages(imageGalleryElement);

        const canvas = await html2canvas(imageGalleryElement, {
          scale: 2,
          useCORS: true,
          allowTaint: false,
          backgroundColor: "#ffffff",
        });

        if (canvas.width && canvas.height) {
          const imgData = canvas.toDataURL("image/png");
          const imgWidth = pdf.internal.pageSize.width - 20;
          const imgHeight = (canvas.height * imgWidth) / canvas.width;

          if (y + imgHeight > pageHeight - 10) {
            pdf.addPage();
            y = 10;
          }

          pdf.addImage(imgData, "PNG", 10, y, imgWidth, imgHeight);
        }
      }

      pdf.save(
        `${this.estimate.siteName}-${this.estimate.date || "document"}.pdf`
      );
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
.a4-container {
  display: flex;
  justify-content: center;
  padding: 20px;
  background-color: #f5f5f5;
  min-height: 100vh;
}
.a4-sheet {
  width: 210mm;
  min-height: 297mm;
  max-width: 210mm;
  background: white;
  padding: 20mm 15mm;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
  font-family: "Times New Roman", serif;
  font-size: 14px;
  line-height: 1.4;
  color: #000;
  position: relative;
}
.header-section {
  margin-bottom: 8px;
  border-bottom: 1px solid #000;
  padding-bottom: 5px;
}
.company-name {
  text-align: center;
  font-size: 18px;
  font-weight: bold;
  margin-bottom: 8px;
  text-transform: uppercase;
  letter-spacing: 1px;
  color: #000;
}
.info-row {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 5px;
}
.date-section {
  font-size: 14px;
  font-weight: bold;
  color: #000;
}
.person-section {
  text-align: right;
}
.person-name {
  font-size: 14px;
  font-weight: bold;
  margin-bottom: 3px;
  color: #000;
}
.phone-numbers {
  display: flex;
  flex-direction: column;
  gap: 2px;
}
.phone-bar {
  display: flex;
  align-items: center;
  gap: 5px;
  font-size: 10px;
}
.phone-label {
  min-width: 35px;
  font-weight: 500;
}
.phone-number {
  min-width: 80px;
  font-size: 10px;
  font-weight: 500;
  color: #000;
}
.site-name {
  text-decoration: underline;
  text-decoration-thickness: 2px;
  text-align: center;
  font-size: 16px;
  font-weight: bold;
  margin: 8px 0 5px 0;
  text-transform: uppercase;
  letter-spacing: 1px;
  color: #000;
}
.purpose-header {
  font-size: 14px;
  font-weight: bold;
  margin-bottom: 8px;
  text-transform: uppercase;
  color: #000;
}
.calculation-info {
  font-style: italic;
  font-size: 12px;
  margin-bottom: 10px;
  text-align: left;
}
.estimate-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 15px;
}
table {
  width: 100%;
  border-collapse: collapse;
  font-family: "Times New Roman", serif;
}
thead th {
  background-color: #f2f2f2;
  border-bottom: 2px solid #000;
  padding: 8px 10px;
  text-align: left;
  font-weight: bold;
}
th.description-header {
  width: 20%;
}
th.dimensions-header {
  width: 10%;
  text-align: center;
}
th.unit-header {
  width: 8%;
  text-align: center;
}
th.type-header {
  width: 8%;
  text-align: center;
}
th.quantity-header {
  width: 8%;
  text-align: center;
}
th.rate-header {
  width: 10%;
  text-align: right;
}
th.price-header {
  width: 15%;
  text-align: right;
}
th.total-header {
  width: 15%;
  text-align: right;
  line-height: 1.2;
}
.category-row .category-cell {
  font-weight: bold;
  padding: 10px 0 5px 0;
  border-bottom: 1px solid #ccc;
}
.item-row-table td {
  padding: 5px 10px;
  vertical-align: top;
  border-bottom: 1px dashed #eee;
}
.description-cell {
  font-style: italic;
  font-size: 13px;
}
.dimensions-cell,
.unit-cell,
.type-cell,
.quantity-cell {
  text-align: center;
}
.rate-cell,
.price-cell,
.total-cell {
  text-align: right;
}
.total-row-table {
  font-weight: bold;
  border-top: 2px solid #000;
}
.total-label-cell {
  text-align: right;
  padding: 10px 10px 10px 0;
}
.total-amount-cell {
  padding: 10px 10px 10px 0;
  text-align: right;
  font-size: 16px;
  color: #000;
}

.image-gallery-section {
  margin-top: 20px;
  border-top: 1px solid #000;
  padding-top: 10px;
}
.image-gallery-header {
  font-size: 16px;
  font-weight: bold;
  margin-bottom: 10px;
  text-align: center;
}
.image-figure {
  text-align: center;
  margin-bottom: 20px;
  page-break-inside: avoid;
}
.item-image {
  max-width: 100%;
  height: auto;
  display: block;
  margin: 0 auto;
}
figcaption {
  font-style: italic;
  margin-top: 5px;
}
.calculation-guide {
  font-size: 10px;
  font-weight: normal;
  display: block;
}
</style>
