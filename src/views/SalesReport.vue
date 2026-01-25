<template>
  <div class="main-page">
    <!-- call navigate tab -->
    <!-- <navigate /> -->
    <div class="page-body">
      <div class="mb-3">
        <h2>{{ t("salesreport") }}</h2>
      </div>
      <div class="row text-center">
        <div class="col-md-12">
          <div class="card bg-light">
            <div class="card-body">
              <div class="card-body-p header-dash">
                <p class="date-period-dash">
                  {{ t("sentenceSummaryYear") }} {{ startDateFilter }} -
                  {{ endDateFilter }}
                </p>
                <div
                  class="div-for-formControl custome-filter-date pr-3"
                  v-if="toggleCustomDate"
                >
                  <div class="mb-3 div-for-formControl form-filter-home">
                    <!-- <DatePicker
                      v-model:value="custom_filter_start_date"
                      format="DD/MM/YYYY"
                      value-type="date"
                      placeholder="DD/MM/YYYY"
                      class="form-control"
                      :formatter="momentFormat"
                      :lang="currentLocale"
                      @change="filterDataByOption"
                    /> -->
                    <v-date-picker
                      v-model="custom_filter_start_date"
                      locale="th-TH"
                      :format="formatDatePicker"
                    >
                      <template v-slot="{ inputEvents }">
                        <input
                          class="custom-input"
                          :value="formatDatePicker(custom_filter_start_date)"
                          v-on="inputEvents"
                          placeholder="เลือกวันที่"
                          style="width: 100%"
                        />
                      </template>
                    </v-date-picker>
                  </div>
                  <span class="mdi mdi-minus" style="padding: 10px"></span>
                  <div class="mb-3 div-for-formControl form-filter-home">
                    <!-- <DatePicker
                      v-model:value="custom_filter_end_date"
                      format="DD/MM/YYYY"
                      value-type="date"
                      placeholder="DD/MM/YYYY"
                      class="form-control"
                      :formatter="momentFormat"
                      :lang="currentLocale"
                      @change="filterDataByOption"
                    /> -->
                    <v-date-picker
                      v-model="custom_filter_end_date"
                      locale="th-TH"
                      :format="formatDatePicker"
                    >
                      <template v-slot="{ inputEvents }">
                        <input
                          class="custom-input"
                          :value="formatDatePicker(custom_filter_end_date)"
                          v-on="inputEvents"
                          placeholder="เลือกวันที่"
                          style="width: 100%"
                        />
                      </template>
                    </v-date-picker>
                  </div>
                </div>
                <div class="select-date-period-dash">
                  <select
                    class="form-control col-sm-9 col-md-6 form-select"
                    @change="filterDataByOption"
                    v-model="selectedOption"
                  >
                    <option value="all">{{ $t("filterDate.all") }}</option>
                    <option value="last7Days">
                      {{ t("filterDate.last7Days") }}
                    </option>
                    <option value="last14Days">
                      {{ t("filterDate.last14Days") }}
                    </option>
                    <option value="last30Days">
                      {{ t("filterDate.last30Days") }}
                    </option>
                    <option value="thisMonth">
                      {{ t("filterDate.thisMonth") }}
                    </option>
                    <option value="lastMonth">
                      {{ t("filterDate.lastMonth") }}
                    </option>
                    <option value="thisQuarter">
                      {{ t("filterDate.thisQuarter") }}
                    </option>
                    <option value="lastQuarter">
                      {{ t("filterDate.lastQuarter") }}
                    </option>
                    <option value="thisYear">
                      {{ t("filterDate.thisYear") }}
                    </option>
                    <option value="lastYear">
                      {{ t("filterDate.lastYear") }}
                    </option>
                    <option value="customDate">
                      {{ t("filterDate.customDate") }}
                    </option>
                  </select>
                  <!-- <select
                    class="form-control form-select"
                    v-model="selectedOption"
                    @change="filterDataByOption"
                  >
                    <option value="all">{{ t("all") }}</option>
                    <option value="thisYear">{{ t("thisyear") }}</option>
                    <option value="thisQuarter">{{ t("thisquarter") }}</option>
                    <option value="thisMonth">{{ t("thismonth") }}</option>
                    <option value="today">{{ t("today") }}</option>
                    <option value="thisWeek">{{ t("thisweek") }}</option>
                    <option value="lastYear">{{ t("lastyear") }}</option>
                  </select> -->
                </div>
                <!-- <div class="header-dash" style="margin-left: 10px; margin-right: 10px;">
                                    <p class="select-date-period-dash-from">{{ t('from') }}</p>
                                    <datepicker v-model:value="startDate" format="DD/MM/YYYY" value-type="date"
                                        placeholder="DD/MM/YYYY" class="form-control datepicker-dash"
                                        @change="filterDataByDate" />
                                </div>
                                <div class="header-dash">
                                    <p class="select-date-period-dash-to">{{ t('to') }}</p>
                                    <datepicker v-model:value="endDate" format="DD/MM/YYYY" value-type="date"
                                        placeholder="DD/MM/YYYY" class="form-control datepicker-dash"
                                        @change="filterDataByDate" />
                                </div> -->
              </div>
              <div class="card-body-detail">
                <div class="card-body-left">
                  <div class="card-body-left-sale">
                    <div class="sale-card">
                      <p>{{ t("totalsaleDash") }}</p>
                    </div>
                    <p>
                      {{
                        totalSalescon.toLocaleString(undefined, {
                          minimumFractionDigits: 2,
                          maximumFractionDigits: 2,
                        })
                      }}
                    </p>
                  </div>
                  <div class="card-body-left-cost">
                    <div class="cost-card">
                      <p>{{ t("costDash") }}</p>
                    </div>
                    <p>{{ ConvertTo2DigitCost }}</p>
                  </div>
                </div>
                <div class="card-body-right">
                  <div class="card-body-right-profit">
                    <div class="profit-card">
                      <p>{{ t("profitDAsh") }}</p>
                    </div>
                    <p>{{ ConvertTo2DigitProfit }}</p>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="row text-center">
        <div class="col-md-6 body-dashboard">
          <div class="card bg-light" style="height: 100%">
            <div class="card-body">
              <p class="card-body-p">{{ t("valueofsales") }}</p>
              <div class="card-body-detail">
                <div ref="donutChart" style="width: 500px; height: 400px"></div>
              </div>
            </div>
          </div>
        </div>
        <div class="col-md-6 body-dashboard">
          <div class="card bg-light">
            <div class="card-body">
              <p class="card-body-p">{{ t("summarySaleMonth") }}</p>
              <div class="card-body-barchart-detail">
                <div class="menu-barchart">
                  <!-- <div class="">
                                        <p>{{ t('totalbillDash') }}</p>
                                    </div> -->
                  <!-- <p>{{ salesData }}</p> -->
                </div>
              </div>
              <div class="card-body-detail">
                <div ref="barChart" style="width: 100%; height: 400px"></div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="row text-center">
        <div class="col-md-6 body-dashboard">
          <div class="card bg-light" style="height: 100%">
            <div class="card-body">
              <p class="card-body-p">{{ t("currentstock") }}</p>
              <div class="card-body-detail">
                <div
                  ref="donutChartAmountProduct"
                  style="width: 500px; height: 400px"
                ></div>
              </div>
            </div>
          </div>
        </div>
        <div class="col-md-6 body-dashboard">
          <div class="card bg-light">
            <div class="card-body">
              <p class="card-body-p">{{ t("servicevalue") }}</p>
              <div class="card-body-detail">
                <div
                  ref="barChartService"
                  style="width: 100%; height: 400px"
                ></div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div
      v-if="isLoading"
      class="d-flex justify-content-center align-items-center"
    >
      <div class="spinner-border text-primary" role="status">
        <span class="visually-hidden">Loading...</span>
      </div>
    </div>
  </div>
</template>

<script>
import navigate from "../components/Navigation.vue";
import * as echarts from "echarts";
import { config } from "../../constant.js";
import { useI18n } from "vue-i18n";
import DatePicker from "vue-datepicker-next";
import "vue-datepicker-next/index.css";
import { computed, watch, ref } from "vue";
import moment from "moment";
moment.locale("th");
const API_CALL = config["url"];
const accessToken = localStorage.getItem("@accessToken");

// ✅ นำเข้า locale ภาษาไทยและอังกฤษ
import th from "vue-datepicker-next/locale/th.es";
import en from "vue-datepicker-next/locale/en.es";

export default {
  name: "Sale Report",
  components: {
    navigate,
    DatePicker,
  },
  setup() {
    const { t } = useI18n();
    const { locale } = useI18n();
    const lang = computed(() => t("lang"));

    const currentLocale = computed(() => {
      return {
        ...(locale.value === "th" ? th : en), // ✅ ใช้ค่าจาก locale ปัจจุบัน
        yearFormat:
          locale.value === "en" ? moment().year() : moment().year() + 543, // ✅ แปลงปี พ.ศ. หรือ ค.ศ.
      };
    });

    const date = ref(null); // ✅ ค่าที่เก็บเป็น YYYY-MM-DD (ค.ศ.)

    const momentFormat = computed(() => ({
      // 📌 Date → String (แสดงผลเป็น พ.ศ. ถ้าภาษาเป็นไทย)
      stringify: (date) => {
        if (!date) return "";
        const yearOffset = lang.value === "en" ? 543 : 0; // ✅ ตรวจสอบภาษาผ่าน computed
        return moment(date).add(yearOffset, "years").format("DD/MM/YYYY");
      },
      // 📌 String → Date (แปลงกลับเป็น ค.ศ. ถ้าภาษาเป็นไทย)
      parse: (value) => {
        if (!value) return null;
        const yearOffset = lang.value === "en" ? 543 : 0;
        return moment(value, "DD/MM/YYYY")
          .subtract(yearOffset, "years")
          .toDate();
      },
      // 📌 ใช้ moment คำนวณเลขสัปดาห์ (ถ้าจำเป็น)
      getWeek: (date) => {
        return moment(date).week();
      },
    }));

    return {
      date,
      momentFormat,
      currentLocale,
      t,
      lang, // ✅ นำ lang ไปใช้ใน template ได้
    };
  },
  data() {
    return {
      Categories: [],
      startDateFilter: [],
      endDateFilter: [],
      startDateFilterChart: [],
      endDateFilterChart: [],
      Expenses: [],
      BusinessJson: [],
      custom_filter_start_date: new Date(),
      custom_filter_end_date: new Date(),
      toggleCustomDate: false,
      toggleCustomDate2: false,
      filteredData: [],
      filteredProductData: [],
      totalSalescon: "", // Stores total sales
      totalProductCost: "", // Stores total product cost
      productTypeA: [], // Stores product type A data
      productTypeB: [], // Stores product type B data
      Billings: [], // Stores billing data
      ProductFilter: [], // Stores filtered product data
      originalBillings: [], // Stores original (unfiltered) billing data
      originalProductFilter: [], // Stores original (unfiltered) product data
      startDate: "", // Start date for filtering data
      endDate: "", // End date for filtering data
      isLoading: false, // Loading state for API requests
      selectedOption: "all", // Selected filter option (All, Today, This Month, This Year)
      allTypeofproduct: [], // Stores all product types
    };
  },
  methods: {
    formatDate2(date) {
      const monthIndex = date.getMonth(); // ดึงเดือนเป็นตัวเลข
      const year = date.getFullYear(); // ดึงปี ค.ศ.

      // เรียกชื่อเดือนตามภาษาที่เลือก
      const monthName = this.t(`months.${this.getMonthKey(monthIndex)}`);

      // แปลงปี ค.ศ. เป็น พ.ศ. ถ้าภาษาไทยถูกเลือก
      const formattedYear = this.$i18n.locale === "th" ? year + 543 : year;

      // แสดงวันที่แบบ "เดือน ปี" พร้อมกับการแปล
      return `${date.getDate()} ${monthName} ${formattedYear}`;
    },
    formatDatePicker(date) {
      if (!date) return "";
      const d = new Date(date);
      const day = d.getDate().toString().padStart(2, "0");
      const month = (d.getMonth() + 1).toString().padStart(2, "0");
      const buddhistYear = d.getFullYear() + 543;

      return `${day}/${month}/${buddhistYear}`; // 🔸 แสดงเป็น พ.ศ.
    },
    formatDateForPicker(date) {
      if (!date) return null;
      const d = new Date(date);
      if (isNaN(d.getTime())) return null; // ตรวจสอบว่าเป็นวันที่ถูกต้อง
      return d;
    },
    async getBilling() {
      const accessToken = localStorage.getItem("@accessToken");
      try {
        const response = await fetch(`${API_CALL}/quotation/getBilling`, {
          headers: {
            Authorization: `Bearer ${accessToken}`,
          },
        });
        const json = await response.json();

        if (json.statusCode === 200) {
          // console.log("raw::", json.data);

          // ใช้ map เพื่อสร้างข้อมูล Billings และเก็บข้อมูลทั้งหมดใน this.Billings
          this.Billings = json.data.map((item) => {
            const QTDate = new Date(item.quotation_start_date);
            const EXPD = new Date(item.quotation_expired_date);
            const IND = new Date(item.invoice_date);
            const BILL = new Date(item.billing_date);
            const formatDate = {
              day: "2-digit",
              month: "short",
              year: "numeric",
            };
            const Quotationdate = QTDate.toLocaleDateString(
              "en-GB",
              formatDate
            );
            const ExpiredDate = EXPD.toLocaleDateString("en-GB", formatDate);
            const InvoiceDate = IND.toLocaleDateString("en-GB", formatDate);
            const BillingDate = BILL.toLocaleDateString("en-GB", formatDate);

            // สร้างข้อมูล formData สำหรับแต่ละรายการ และ return เพื่อเก็บใน Billings
            return {
              billing_number: item.billing_number,
              employeeID: item.employeeID,
              employeeName: item.employee_name,
              cus_id: item.cus_id,
              cus_name: item.cus_name,
              cus_address: item.cus_address,
              cus_tel: item.cus_tel,
              cus_email: item.cus_email,
              cus_tax: item.cus_tax,
              cus_purchase: item.cus_purchase,
              sale_totalprice: item.sale_totalprice, // ตรวจสอบค่าที่ใช้คำนวณ
              invoice_id: item.invoice_id,
              remark: item.remark || "",
              billing_id: item.billing_id,
              billing_date: this.formatDateForPicker(BillingDate),
              billing_status: item.billing_status,
              payments: item.payments,
              productForms: item.details.map((detail) => ({
                ID: detail.productID,
                sale_price: detail.sale_price,
                sale_discount: detail.sale_discount,
                sale_qty: detail.sale_qty,
              })),
            };
          });
          this.filteredData = json.data.map((item) => {
            const QTDate = new Date(item.quotation_start_date);
            const EXPD = new Date(item.quotation_expired_date);
            const IND = new Date(item.invoice_date);
            const BILL = new Date(item.billing_date);
            const formatDate = {
              day: "2-digit",
              month: "short",
              year: "numeric",
            };
            const Quotationdate = QTDate.toLocaleDateString(
              "en-GB",
              formatDate
            );
            const ExpiredDate = EXPD.toLocaleDateString("en-GB", formatDate);
            const InvoiceDate = IND.toLocaleDateString("en-GB", formatDate);
            const BillingDate = BILL.toLocaleDateString("en-GB", formatDate);

            // สร้างข้อมูล formData สำหรับแต่ละรายการ และ return เพื่อเก็บใน Billings
            return {
              billing_number: item.billing_number,
              employeeID: item.employeeID,
              employeeName: item.employee_name,
              cus_id: item.cus_id,
              cus_name: item.cus_name,
              cus_address: item.cus_address,
              cus_tel: item.cus_tel,
              cus_email: item.cus_email,
              cus_tax: item.cus_tax,
              cus_purchase: item.cus_purchase,
              sale_totalprice: item.sale_totalprice, // ตรวจสอบค่าที่ใช้คำนวณ
              invoice_id: item.invoice_id,
              remark: item.remark || "",
              billing_id: item.billing_id,
              billing_date: this.formatDateForPicker(BillingDate),
              billing_status: item.billing_status,
              payments: item.payments,
              productForms: item.details.map((detail) => ({
                ID: detail.ID,
                sale_price: detail.sale_price,
                sale_discount: detail.sale_discount,
                sale_qty: detail.sale_qty,
              })),
            };
          });
          this.filteredDataEachYear = json.data.map((item) => {
            const QTDate = new Date(item.quotation_start_date);
            const EXPD = new Date(item.quotation_expired_date);
            const IND = new Date(item.invoice_date);
            const BILL = new Date(item.billing_date);
            const formatDate = {
              day: "2-digit",
              month: "short",
              year: "numeric",
            };
            const Quotationdate = QTDate.toLocaleDateString(
              "en-GB",
              formatDate
            );
            const ExpiredDate = EXPD.toLocaleDateString("en-GB", formatDate);
            const InvoiceDate = IND.toLocaleDateString("en-GB", formatDate);
            const BillingDate = BILL.toLocaleDateString("en-GB", formatDate);

            // สร้างข้อมูล formData สำหรับแต่ละรายการ และ return เพื่อเก็บใน Billings
            return {
              billing_number: item.billing_number,
              employeeID: item.employeeID,
              employeeName: item.employee_name,
              cus_id: item.cus_id,
              cus_name: item.cus_name,
              cus_address: item.cus_address,
              cus_tel: item.cus_tel,
              cus_email: item.cus_email,
              cus_tax: item.cus_tax,
              cus_purchase: item.cus_purchase,
              sale_totalprice: item.sale_totalprice, // ตรวจสอบค่าที่ใช้คำนวณ
              invoice_id: item.invoice_id,
              remark: item.remark || "",
              billing_id: item.billing_id,
              billing_date: this.formatDateForPicker(BillingDate),
              billing_status: item.billing_status,
              payments: item.payments,
              productForms: item.details.map((detail) => ({
                ID: detail.ID,
                sale_price: detail.sale_price,
                sale_discount: detail.sale_discount,
                sale_qty: detail.sale_qty,
              })),
            };
          });

          this.filterDataByOption();
        } else {
          //this.showPopup_error(json.data);
          console.log("Error fetching billings:", json);
        }
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    },
    async getTypeOfProduct() {
      const accessToken = localStorage.getItem("@accessToken");
      this.isLoading = true;
      try {
        const response = await fetch(`${API_CALL}/product/getProductType`, {
          headers: {
            Authorization: `Bearer ${accessToken}`,
          },
        });
        const json = await response.json();

        if (json.statusCode === 200) {
          this.allTypeofproduct = json.data;
        } else {
          // console.log("ProductType", json);
        }
      } catch (err) {
        console.log("error fetching: ", err);
      } finally {
        this.isLoading = false;
      }
    },
    async getProduct() {
      const accessToken = localStorage.getItem("@accessToken");
      this.isLoading = true;
      try {
        const response = await fetch(`${API_CALL}/product/getProduct`, {
          headers: {
            Authorization: `Bearer ${accessToken}`,
          },
        });
        const json = await response.json();

        if (json.statusCode === 200) {
          // กรองข้อมูลตามประเภทสินค้าที่เลือก
          this.ProductFilter = json.data.map((item) => {
            const PD = new Date(item.product_date);
            const formatDate = {
              day: "2-digit",
              month: "short",
              year: "numeric",
            };
            const Productdate = PD.toLocaleDateString("en-GB", formatDate);

            let product = {
              ID: item.ID,
              Category: item.product_category.categoryName,
              "Product Name": item.productname,
              Detail: item.productdetail,
              Price: item.price,
              Cost: item.productcost,
              "Product Image": item.productImg,
              productTypeID: item.productTypeID,
              categoryID: item.categoryID,
              product_date: this.formatDateForPicker(Productdate),
              productTypeName: item.product_type.productTypeName,
              Amount: item.amount,
            };
            console.log(item.product_type.productTypeName);
            // if (this.selectedType === 'A') {
            //     product.Amount = item.amount;
            // }
            return product;
          });
          this.filteredProductData = json.data.map((item) => {
            const PD = new Date(item.product_date);
            const formatDate = {
              day: "2-digit",
              month: "short",
              year: "numeric",
            };
            const Productdate = PD.toLocaleDateString("en-GB", formatDate);

            let product = {
              ID: item.ID,
              Category: item.product_category.categoryName,
              "Product Name": item.productname,
              Detail: item.productdetail,
              Price: item.price,
              Cost: item.productcost,
              "Product Image": item.productImg,
              productTypeID: item.productTypeID,
              categoryID: item.categoryID,
              product_date: this.formatDateForPicker(Productdate),
              productTypeName: item.product_type.productTypeName,
              Amount: item.amount,
            };
            console.log(item.product_type.productTypeName);
            // if (this.selectedType === 'A') {
            //     product.Amount = item.amount;
            // }
            return product;
          });

          // console.log(this.ProductFilter)

          // this.originalProductFilter = [...this.ProductFilter];
          this.filterDataByOption();
        } else {
          //this.showPopup_error(json.data);
          // console.log("ProductByType", json);
        }
      } catch (error) {
        console.error("Error fetching data:", error);
      } finally {
        this.isLoading = false;
      }
    },
    convertThaiDateToISO(thaiDate) {
      // แยกส่วนวันที่
      const [day, monthThai, yearThai] = thaiDate.split(" ");

      // แผนที่เดือนภาษาไทยไปยังลำดับเดือน
      const months = {
        มกราคม: "01",
        กุมภาพันธ์: "02",
        มีนาคม: "03",
        เมษายน: "04",
        พฤษภาคม: "05",
        มิถุนายน: "06",
        กรกฎาคม: "07",
        สิงหาคม: "08",
        กันยายน: "09",
        ตุลาคม: "10",
        พฤศจิกายน: "11",
        ธันวาคม: "12",
      };

      // แปลงปี พ.ศ. เป็น ค.ศ.
      const yearCE = parseInt(yearThai, 10) - 543;

      // แปลงวันที่เป็นรูปแบบ ISO
      return `${yearCE}-${months[monthThai]}-${day.padStart(2, "0")}`;
    },
    async renderChart() {
      // ตรวจสอบว่ามีการ mount ของ donutChart แล้วหรือไม่
      const chartDom = this.$refs.donutChart;
      if (!chartDom) return;

      // ตรวจสอบและทำลาย instance เก่าของกราฟ
      let chart = echarts.getInstanceByDom(chartDom);
      if (chart) {
        chart.dispose();
      }
      if (this.startDateFilter !== "") {
        this.startDateFilterChart = this.convertThaiDateToISO(
          this.startDateFilter
        ); // 26 พฤษภาคม 2568 convert to 2025-05-28
        this.endDateFilterChart = this.convertThaiDateToISO(this.endDateFilter);
      }
      // สร้าง instance ใหม่
      chart = echarts.init(chartDom);

      try {
        const accessToken = localStorage.getItem("@accessToken");
        // เรียก API เพื่อดึงข้อมูลยอดขายแยกตามประเภทสินค้า
        const response = await fetch(
          `${API_CALL}/quotation/get_sale_report_product_type`,
          {
            method: "POST",
            headers: {
              Authorization: `Bearer ${accessToken}`,
              "Content-Type": "application/json",
            },
            body: JSON.stringify({
              startDate: this.startDateFilterChart,
              endDate: this.endDateFilterChart,
            }),
          }
        );
        console.log("---->", response);
        // const response = await fetch(
        //   `${API_CALL}/Quotation/get_sale_report_product_type`,
        //   {
        //     method: "POST",
        //     headers: {
        //       "Content-Type": "application/json",
        //       Authorization: `Bearer ${accessToken}`,
        //     },
        //     body: JSON.stringify({
        //       startDate: this.startDate,
        //       endDate: this.endDate,
        //     }),
        //   }
        // );

        if (!response.ok) {
          throw new Error("Failed to fetch data from the API");
        }

        const data = await response.json();

        // เตรียมข้อมูลสำหรับกราฟ
        const chartData = data.data.map((item) => ({
          value: item.total_sale_price,
          name: item.product_type,
        }));

        // ตั้งค่ากราฟ
        const option = {
          tooltip: {
            trigger: "item",
            formatter: "{a} <br/>{b} : {c} ({d}%)",
          },
          legend: {
            orient: "vertical",
            left: "left",
          },
          series: [
            {
              name: this.t("inventoryVolume"),
              type: "pie",
              radius: ["50%", "70%"],
              avoidLabelOverlap: false,
              label: {
                show: false,
                position: "center",
              },
              emphasis: {
                label: {
                  show: true,
                  fontSize: "30",
                  fontWeight: "bold",
                },
              },
              labelLine: {
                show: false,
              },
              data: chartData, // ข้อมูลที่ได้รับจาก API
            },
          ],
        };

        // ตั้งค่ากราฟลงใน chart instance
        chart.setOption(option);
      } catch (error) {
        console.error("Error rendering chart:", error.message);
      }
    },

    // renderChart() {
    //   // ตรวจสอบว่ามีการ mount ของ donutChart แล้วหรือไม่
    //   const chartDom = this.$refs.donutChart;
    //   if (!chartDom) return;

    //   let chart = echarts.getInstanceByDom(chartDom);
    //   if (chart) {
    //     chart.dispose(); // ทำลาย instance เก่า
    //   }

    //   chart = echarts.init(chartDom);

    //   // กรองข้อมูล Billings ตามช่วงวันที่ startDate และ endDate
    //   const start = new Date(this.startDate);
    //   const end = new Date(this.endDate);

    //   // const filteredBillings = this.Billings.filter(item => {
    //   //     const billingDate = new Date(item.billing_date);
    //   //     return billingDate >= start && billingDate <= end;
    //   // });

    //   // ดึงข้อมูลจาก Billings ที่ถูกกรองแล้ว เพื่อรวมยอดขายแยกตามประเภทสินค้า
    //   const productTypeSales = {};

    //   // console.log("---------------->>", this.filteredData);

    //   this.filteredData.forEach((billing) => {
    //     billing.productForms.forEach((product) => {
    //       const productInFilter = this.ProductFilter.find(
    //         (p) => p["Product Name"] === product.ID || 0
    //       );

    //       // if (!productInFilter) {
    //       //     console.warn(`Product with ID ${product.ID} not found in ProductFilter`);
    //       // }

    //       const productTypeName = productInFilter
    //         ? productInFilter.productTypeName
    //         : "Unknown"; // ดึงชื่อ productType จาก ProductFilter

    //       const totalPrice = product.sale_price - product.sale_discount;

    //       if (!productTypeSales[productTypeName]) {
    //         productTypeSales[productTypeName] = 0;
    //       }

    //       productTypeSales[productTypeName] += totalPrice;
    //       // console.log("productTypeName", productTypeName);
    //     });
    //   });

    //   const chartData = Object.keys(productTypeSales).map((productTypeName) => {
    //     return {
    //       value: productTypeSales[productTypeName],
    //       name: productTypeName,
    //     };
    //   });

    //   // console.log("chartData", chartData);

    //   // this.Billings.productForms;

    //   // const data = [
    //   //   {
    //   //     product_type: "สินค้า",
    //   //     total_price: 50000,
    //   //   },
    //   //   {
    //   //     product_type: "บริการ",
    //   //     total_price: 20000,
    //   //   },
    //   // ];

    //   // const chartData = data.map((item) => ({
    //   //   value: item.total_price,
    //   //   name: item.product_type,
    //   // }));

    //   const option = {
    //     // title: {
    //     //   text: this.t("inventoryVolume"),
    //     //   left: "center",
    //     // },
    //     tooltip: {
    //       trigger: "item",
    //       formatter: "{a} <br/>{b} : {c} ({d}%)",
    //     },
    //     legend: {
    //       orient: "vertical",
    //       left: "left",
    //     },
    //     series: [
    //       {
    //         name: this.t("inventoryVolume"),
    //         type: "pie",
    //         radius: ["50%", "70%"],
    //         avoidLabelOverlap: false,
    //         label: {
    //           show: false,
    //           position: "center",
    //         },
    //         emphasis: {
    //           label: {
    //             show: true,
    //             fontSize: "30",
    //             fontWeight: "bold",
    //           },
    //         },
    //         labelLine: {
    //           show: false,
    //         },
    //         data: chartData, // ข้อมูลที่เตรียมไว้
    //       },
    //     ],
    //   };

    //   chart.setOption(option);
    // },
    async renderAmountProductChart() {
      try {
        const chartDom = this.$refs.donutChartAmountProduct; // ตรวจสอบว่า element ถูก mount แล้วหรือไม่
        if (!chartDom) return;

        let chart = echarts.getInstanceByDom(chartDom);
        if (chart) {
          chart.dispose(); // ทำลาย instance เดิมก่อนสร้างใหม่
        }

        chart = echarts.init(chartDom);

        if (this.startDateFilter !== "") {
          this.startDateFilterChart = this.convertThaiDateToISO(
            this.startDateFilter
          ); // 26 พฤษภาคม 2568 convert to 2025-05-28
          this.endDateFilterChart = this.convertThaiDateToISO(
            this.endDateFilter
          );
        }

        // ดึงข้อมูลจาก API
        const response = await fetch(
          `${API_CALL}/quotation/get_sale_report_product_rank`,
          {
            method: "POST",
            headers: {
              Authorization: `Bearer ${accessToken}`,
              "Content-Type": "application/json",
            },
            body: JSON.stringify({
              startDate: this.startDateFilterChart,
              endDate: this.endDateFilterChart,
            }),
          }
        );

        if (!response.ok) {
          throw new Error("Failed to fetch product rank data");
        }

        const result = await response.json();
        console.log(result);

        // แปลงข้อมูลให้เหมาะสมกับกราฟ
        let sortedData = result.data
          .map((item) => ({
            name: item.product,
            value: item.total_sale_price,
          }))
          .sort((a, b) => b.value - a.value); // เรียงจากมากไปน้อย

        // แยก top 7 และรวมอื่น ๆ
        const top7 = sortedData.slice(0, 7);
        const others = sortedData.slice(7);

        const otherTotal = others.reduce((sum, item) => sum + item.value, 0);

        if (otherTotal > 0) {
          top7.push({
            name: this.t("others"), // หรือใช้ชื่อว่า 'อื่นๆ'
            value: otherTotal,
          });
        }

        const option = {
          tooltip: {
            trigger: "item",
            formatter: "{a} <br/>{b} : {c} ({d}%)",
          },
          legend: {
            orient: "vertical",
            left: "left",
          },
          series: [
            {
              name: this.t("stockP"),
              type: "pie",
              radius: ["50%", "70%"],
              center: ["65%", "50%"], // ขยับกราฟไปทางขวา
              avoidLabelOverlap: false,
              label: {
                show: false,
                position: "center",
              },
              emphasis: {
                label: {
                  show: true,
                  fontSize: "30",
                  fontWeight: "bold",
                },
              },
              labelLine: {
                show: false,
              },
              data: top7,
            },
          ],
        };

        chart.setOption(option); // สร้างกราฟด้วย option ที่ตั้งไว้
      } catch (error) {
        console.error("Error rendering donut chart:", error);
      }
    },

    // renderAmountProductChart() {
    //   const chartDom = this.$refs.donutChartAmountProduct;
    //   if (!chartDom) return;

    //   let chart = echarts.getInstanceByDom(chartDom);
    //   if (chart) {
    //     chart.dispose();
    //   }

    //   chart = echarts.init(chartDom);

    //   const productData = {};

    //   this.ProductFilter.forEach((product) => {
    //     const productType = product.productTypeName;
    //     if (productType === "product") {
    //       const productName = product["Product Name"];
    //       const amount = product.Amount || 0;

    //       if (!productData[productName]) {
    //         productData[productName] = 0;
    //       }

    //       productData[productName] += amount;
    //     }
    //   });

    //   // แปลง object เป็น array เพื่อจัดอันดับ
    //   let sortedData = Object.keys(productData)
    //     .map((name) => ({
    //       name,
    //       value: productData[name],
    //     }))
    //     .sort((a, b) => b.value - a.value); // เรียงจากมากไปน้อย

    //   // แยก top 7 และอื่น ๆ
    //   const top7 = sortedData.slice(0, 7);
    //   const others = sortedData.slice(7);

    //   const otherTotal = others.reduce((sum, item) => sum + item.value, 0);

    //   if (otherTotal > 0) {
    //     top7.push({
    //       name: this.t("others"), // หรือใช้ชื่อว่า 'อื่นๆ'
    //       value: otherTotal,
    //     });
    //   }

    //   const option = {
    //     tooltip: {
    //       trigger: "item",
    //       formatter: "{a} <br/>{b} : {c} ({d}%)",
    //     },
    //     legend: {
    //       orient: "vertical",
    //       left: "left",
    //     },
    //     series: [
    //       {
    //         name: this.t("stockP"),
    //         type: "pie",
    //         radius: ["50%", "70%"],
    //         center: ["65%", "50%"], // ขยับกราฟไปทางขวา
    //         avoidLabelOverlap: false,
    //         label: {
    //           show: false,
    //           position: "center",
    //         },
    //         emphasis: {
    //           label: {
    //             show: true,
    //             fontSize: "30",
    //             fontWeight: "bold",
    //           },
    //         },
    //         labelLine: {
    //           show: false,
    //         },
    //         data: top7,
    //       },
    //     ],
    //   };

    //   chart.setOption(option);
    // },

    async renderServiceBarChart() {
      try {
        const chartDom = this.$refs.barChartService; // ตรวจสอบว่า DOM มีอยู่หรือไม่
        if (!chartDom) return;

        let chart = echarts.getInstanceByDom(chartDom);
        if (chart) {
          chart.dispose(); // ทำลาย instance เก่าหากมี
        }

        chart = echarts.init(chartDom); // สร้าง instance ใหม่

        // ดึงข้อมูลจาก API
        const response = await fetch(
          `${API_CALL}/quotation/get_sale_report_service`,
          {
            method: "POST",
            headers: {
              Authorization: `Bearer ${accessToken}`,
              "Content-Type": "application/json",
            },
            body: JSON.stringify({
              startDate: this.startDateFilterChart, // วันที่เริ่มต้น
              endDate: this.endDateFilterChart, // วันที่สิ้นสุด
            }),
          }
        );

        if (!response.ok) {
          throw new Error("Failed to fetch service data");
        }

        const result = await response.json();
        console.log(result);

        // แปลงข้อมูลจาก API ให้เหมาะสมกับกราฟ
        const serviceNames = result.data.map((item) => item.product_name); // ชื่อ service
        const serviceData = result.data.map((item) => item.total_sale_price); // ราคารวม

        // ตั้งค่ากราฟ
        const option = {
          tooltip: {
            trigger: "axis",
            axisPointer: { type: "shadow" }, // ใช้เส้น shadow เมื่อ hover
          },
          xAxis: {
            type: "value", // แกน x แสดงข้อมูลเชิงตัวเลข
            boundaryGap: [0, 0.01], // กำหนดขอบให้แถบไม่ชนขอบ
          },
          yAxis: {
            type: "category", // แกน y แสดงชื่อของ service
            data: serviceNames, // ใช้ชื่อ service เป็นแกน y
          },
          series: [
            {
              name: "Value", // ชื่อซีรีส์
              type: "bar", // ประเภทกราฟเป็น bar
              data: serviceData, // ข้อมูลราคาของ service
              label: {
                show: true,
                position: "right", // แสดง label ที่ปลายกราฟแท่ง
              },
            },
          ],
        };

        chart.setOption(option); // ตั้งค่า option ให้กับกราฟ
      } catch (error) {
        console.error("Error rendering service bar chart:", error);
      }
    },

    // renderServiceBarChart() {
    //   const chartDom = this.$refs.barChartService;
    //   if (!chartDom) return;

    //   let chart = echarts.getInstanceByDom(chartDom);
    //   if (chart) {
    //     chart.dispose(); // ทำลาย instance เก่า
    //   }

    //   chart = echarts.init(chartDom);

    //   const serviceData = [];
    //   const serviceNames = [];

    //   this.ProductFilter.forEach((product) => {
    //     // console.log("In loop Service Chart4:");
    //     const productType = product.productTypeName; // ชื่อประเภทสินค้า
    //     if (productType === "service") {
    //       // เงื่อนไข: เอาเฉพาะประเภท 'service'
    //       const serviceName = product["Product Name"]; // ชื่อของ service
    //       const price = product.Price || 0; // ราคาของ service

    //       serviceNames.push(serviceName); // เพิ่มชื่อ service ใน array
    //       serviceData.push(price); // เพิ่มราคาใน array
    //       // console.log("In loop Service Chart4:",serviceNames)
    //     }
    //   });
    //   // console.log("Service Chart4:", serviceNames);
    //   const option = {
    //     // title: {
    //     //   text: this.t("serviceprice"), // หัวข้อของกราฟ
    //     //   left: "center",
    //     // },
    //     tooltip: {
    //       trigger: "axis",
    //       axisPointer: { type: "shadow" }, // ใช้เส้น shadow เมื่อ hover
    //     },
    //     xAxis: {
    //       type: "value", // แกน x แสดงข้อมูลเชิงตัวเลข (ราคาของ service)
    //       boundaryGap: [0, 0.01], // กำหนดขอบให้แถบไม่ชนขอบ
    //     },
    //     yAxis: {
    //       type: "category", // แกน y แสดงชื่อของ service
    //       data: serviceNames, // แสดงรายชื่อ service เป็นแกน y
    //     },
    //     series: [
    //       {
    //         name: "Value", // ชื่อซีรีส์
    //         type: "bar", // ประเภทกราฟเป็น bar
    //         data: serviceData, // ข้อมูลราคาของ service
    //         label: {
    //           show: true,
    //           position: "right", // แสดง label ที่ปลายกราฟแท่ง
    //         },
    //       },
    //     ],
    //   };

    //   chart.setOption(option);
    // },

    async renderBarChart() {
      try {
        const chartDom = this.$refs.barChart; // ตรวจสอบว่า barChart มีอยู่หรือไม่
        if (!chartDom) return;

        // ทำลาย instance ก่อนหน้า หากมี
        let chart = echarts.getInstanceByDom(chartDom);
        if (chart) {
          chart.dispose();
        }

        chart = echarts.init(chartDom); // สร้าง instance ใหม่

        if (this.startDateFilter !== "") {
          this.startDateFilterChart = this.convertThaiDateToISO(
            this.startDateFilter
          ); // แปลงวันที่เริ่มต้น
          this.endDateFilterChart = this.convertThaiDateToISO(
            this.endDateFilter
          ); // แปลงวันที่สิ้นสุด
        }

        // ดึงข้อมูลจาก API
        const response = await fetch(
          `${API_CALL}/quotation/get_sale_report_category`,
          {
            method: "POST",
            headers: {
              Authorization: `Bearer ${accessToken}`,
              "Content-Type": "application/json",
            },
            body: JSON.stringify({
              startDate: this.startDateFilterChart, // ใช้วันที่เริ่มต้น
              endDate: this.endDateFilterChart, // ใช้วันที่สิ้นสุด
            }),
          }
        );

        if (!response.ok) {
          throw new Error("Failed to fetch category data");
        }

        const result = await response.json();

        // แปลงข้อมูลให้เหมาะสมกับการแสดงผล
        const Sale_date = result.data.map((item) => item.categoryName); // ดึงชื่อหมวดหมู่
        const SaleData = result.data.map((item) => item.total_sale_price); // ดึงยอดขายรวม

        // ตัวเลือกของกราฟ
        const option = {
          tooltip: {
            trigger: "axis",
            axisPointer: { type: "shadow" },
          },
          xAxis: {
            type: "category",
            data: Sale_date,
          },
          yAxis: {
            type: "value",
            boundaryGap: [0, 0.01],
          },
          series: [
            {
              name: "Value",
              type: "bar",
              data: SaleData,
              label: {
                show: true,
                position: "top",
              },
            },
          ],
        };

        chart.setOption(option); // ตั้งค่า option ให้กับกราฟ
      } catch (error) {
        console.error("Error rendering bar chart:", error);
      }
    },

    destroyCharts() {
      if (this.$refs.donutChart) {
        const donutChart = echarts.getInstanceByDom(this.$refs.donutChart);
        if (donutChart) donutChart.dispose(); // ทำลาย instance เดิม
      }

      if (this.$refs.barChart) {
        const barChart = echarts.getInstanceByDom(this.$refs.barChart);
        if (barChart) barChart.dispose(); // ทำลาย instance เดิม
      }

      if (this.$refs.barChartService) {
        const barChart = echarts.getInstanceByDom(this.$refs.barChartService);
        if (barChart) barChart.dispose(); // ทำลาย instance เดิม
      }

      if (this.$refs.donutChartAmountProduct) {
        const donutChart = echarts.getInstanceByDom(
          this.$refs.donutChartAmountProduct
        );
        if (donutChart) donutChart.dispose(); // ทำลาย instance เดิม
      }
    },

    filterDataByOption() {
      const now = new Date();
      const currentYear = now.getFullYear();
      const currentMonth = now.getMonth();
      const currentQuarter = Math.floor((currentMonth + 3) / 3);
      const lastQuarter = currentQuarter === 1 ? 4 : currentQuarter - 1;
      const lastQuarterYear =
        currentQuarter === 1 ? currentYear - 1 : currentYear;

      let startDate, endDate;

      switch (this.selectedOption) {
        case "last7Days":
          startDate = new Date();
          startDate.setDate(now.getDate() - 6);
          endDate = now;
          this.toggleCustomDate = false;
          break;
        case "last14Days":
          startDate = new Date();
          startDate.setDate(now.getDate() - 13);
          endDate = now;
          this.toggleCustomDate = false;
          break;
        case "last30Days":
          startDate = new Date();
          startDate.setDate(now.getDate() - 29);
          endDate = now;
          this.toggleCustomDate = false;
          break;
        case "thisMonth":
          startDate = new Date(currentYear, currentMonth, 1);
          endDate = new Date(currentYear, currentMonth + 1, 0);
          this.toggleCustomDate = false;
          break;
        case "lastMonth":
          startDate = new Date(currentYear, currentMonth - 1, 1);
          endDate = new Date(currentYear, currentMonth, 0);
          this.toggleCustomDate = false;
          break;
        case "thisQuarter":
          startDate = new Date(currentYear, (currentQuarter - 1) * 3, 1);
          endDate = new Date(currentYear, currentQuarter * 3, 0);
          this.toggleCustomDate = false;
          break;
        case "lastQuarter":
          startDate = new Date(lastQuarterYear, (lastQuarter - 1) * 3, 1);
          endDate = new Date(lastQuarterYear, lastQuarter * 3, 0);
          this.toggleCustomDate = false;
          break;
        case "thisYear":
          startDate = new Date(currentYear, 0, 1);
          endDate = new Date(currentYear, 11, 31);
          this.toggleCustomDate = false;
          break;
        case "lastYear":
          startDate = new Date(currentYear - 1, 0, 1);
          endDate = new Date(currentYear - 1, 11, 31);
          this.toggleCustomDate = false;
          break;
        case "customDate":
          startDate = this.custom_filter_start_date;
          endDate = this.custom_filter_end_date;
          this.toggleCustomDate = true;
          break;
        default:
          // startDate = new Date(
          //   this.parseThaiDateTime(this.BusinessJson.TokenCreate)
          // );
          startDate = new Date(
            this.parseThaiDateTime(localStorage.TokenCreate)
          );
          // startDate = new Date(currentYear, 0, 1);
          endDate = now;
          this.toggleCustomDate = false;
      }

      const isInRange = (date) => {
        const d = new Date(date);
        return d >= startDate && d <= endDate;
      };

      // ใช้ช่วงวันที่กับข้อมูลที่มีอยู่
      this.filteredData = this.Billings.filter((item) =>
        isInRange(item.billing_date)
      );
      this.filteredProductData = this.ProductFilter.filter((item) =>
        isInRange(item.product_date)
      );

      // ต่อจากนี้คือส่วนคำนวณยอดรวมและ renderChart ต่าง ๆ ที่คุณมีอยู่แล้ว
      this.totalSalescon = this.filteredData.reduce(
        (sum, item) => sum + (item.sale_totalprice || 0),
        0
      );

      // this.totalProductCost = this.filteredProductData.reduce(
      //   (sum, product) => {
      //     if (product.productTypeID === 1) {
      //       return sum + product.Cost * (product.Amount || 1);
      //     } else {
      //       return sum + (product.Cost || 0);
      //     }
      //   },
      //   0
      // );

      this.startDateFilter = this.formatToThaiDate(startDate);
      this.endDateFilter = this.formatToThaiDate(endDate);

      this.renderChart();
      this.renderAmountProductChart();
      this.renderServiceBarChart();
      this.renderBarChart();
    },
    formatToThaiDate(date) {
      if (!date) return "";

      const d = new Date(date);
      const day = d.getDate();
      const month = d.toLocaleString("th-TH", { month: "long" }); // "พฤษภาคม"
      const year = d.getFullYear() + 543; // Convert to พ.ศ.

      return `${day} ${month} ${year}`;
    },
    parseThaiDateTime(thaiDateTimeStr) {
      // แยกวันที่ เวลา
      const [datePart, timePart] = thaiDateTimeStr.split(" "); // "7/4/2568", "15:46:12"
      const [day, month, yearBE] = datePart.split("/"); // "7", "4", "2568"

      const year = parseInt(yearBE) - 543; // แปลง พ.ศ. -> ค.ศ.

      // สร้าง string ให้ JS เข้าใจ: "2025-04-07T15:46:12"
      const isoDateStr = `${year}-${month.padStart(2, "0")}-${day.padStart(
        2,
        "0"
      )}T${timePart}`;

      return new Date(isoDateStr);
    },

    // filterDataByOption() {
    //   const currentYear = new Date().getFullYear();
    //   const currentMonth = new Date().getMonth();
    //   const currentDay = new Date().getDate();
    //   const lastYear = currentYear - 1;
    //   const getQuarter = (month) => {
    //     if (month >= 0 && month <= 2) return 1; // Q1: ม.ค.-มี.ค.
    //     if (month >= 3 && month <= 5) return 2; // Q2: เม.ย.-มิ.ย.
    //     if (month >= 6 && month <= 8) return 3; // Q3: ก.ค.-ก.ย.
    //     return 4; // Q4: ต.ค.-ธ.ค.
    //   };
    //   const currentQuarter = getQuarter(currentMonth); // ไตรมาสปัจจุบัน
    //   const getMonday = () => {
    //     const today = new Date();
    //     const day = today.getDay();
    //     const diff = today.getDate() - day + (day === 0 ? -6 : 1); // Adjust when day is Sunday
    //     return new Date(today.setDate(diff));
    //   };
    //   const getSunday = () => {
    //     const monday = getMonday();
    //     return new Date(monday.setDate(monday.getDate() + 6));
    //   };
    //   const monday = getMonday(); // วันจันทร์แรกของสัปดาห์
    //   const sunday = getSunday(); // วันอาทิตย์สุดท้ายของสัปดาห์

    //   if (this.selectedOption === "thisYear") {
    //     this.filteredData = this.Billings.filter((item) => {
    //       const itemYear = new Date(item.billing_date).getFullYear();
    //       return itemYear === currentYear;
    //     });

    //     this.filteredProductData = this.ProductFilter.filter((item) => {
    //       const itemYear = new Date(item.product_date).getFullYear();
    //       return itemYear === currentYear;
    //     });

    //     //this.filteredDataEachYear = this.filterBillingsByYear(currentYear);
    //     // this.showSalesDataForYear(currentYear);
    //   } else if (this.selectedOption === "lastYear") {
    //     this.filteredData = this.Billings.filter((item) => {
    //       const itemYear = new Date(item.billing_date).getFullYear();
    //       return itemYear === lastYear;
    //     });

    //     this.filteredProductData = this.ProductFilter.filter((item) => {
    //       const itemYear = new Date(item.product_date).getFullYear();
    //       return itemYear === lastYear;
    //     });

    //     //this.filteredDataEachYear = this.filterBillingsByYear(lastYear);
    //     //this.showSalesDataForYear(lastYear);
    //   } else if (this.selectedOption === "thisMonth") {
    //     this.filteredData = this.Billings.filter((item) => {
    //       const itemDate = new Date(item.billing_date);
    //       return (
    //         itemDate.getFullYear() === currentYear &&
    //         itemDate.getMonth() === currentMonth
    //       );
    //     });

    //     this.filteredProductData = this.ProductFilter.filter((item) => {
    //       const itemDate = new Date(item.product_date);
    //       return (
    //         itemDate.getFullYear() === currentYear &&
    //         itemDate.getMonth() === currentMonth
    //       );
    //     });
    //   } else if (this.selectedOption === "today") {
    //     this.filteredData = this.Billings.filter((item) => {
    //       const itemDate = new Date(item.billing_date);
    //       return (
    //         itemDate.getFullYear() === currentYear &&
    //         itemDate.getMonth() === currentMonth &&
    //         itemDate.getDate() === currentDay
    //       );
    //     });

    //     this.filteredProductData = this.ProductFilter.filter((item) => {
    //       const itemDate = new Date(item.product_date);
    //       return (
    //         itemDate.getFullYear() === currentYear &&
    //         itemDate.getMonth() === currentMonth &&
    //         itemDate.getDate() === currentDay
    //       );
    //     });
    //   } else if (this.selectedOption === "thisQuarter") {
    //     let startMonth, endMonth;
    //     if (currentQuarter === 1) {
    //       startMonth = 0; // มกราคม
    //       endMonth = 2; // มีนาคม
    //     } else if (currentQuarter === 2) {
    //       startMonth = 3; // เมษายน
    //       endMonth = 5; // มิถุนายน
    //     } else if (currentQuarter === 3) {
    //       startMonth = 6; // กรกฎาคม
    //       endMonth = 8; // กันยายน
    //     } else {
    //       startMonth = 9; // ตุลาคม
    //       endMonth = 11; // ธันวาคม
    //     }

    //     this.filteredData = this.Billings.filter((item) => {
    //       const itemDate = new Date(item.billing_date);
    //       return (
    //         itemDate.getFullYear() === currentYear &&
    //         itemDate.getMonth() >= startMonth &&
    //         itemDate.getMonth() <= endMonth
    //       );
    //     });

    //     this.filteredProductData = this.ProductFilter.filter((item) => {
    //       const itemDate = new Date(item.product_date);
    //       return (
    //         itemDate.getFullYear() === currentYear &&
    //         itemDate.getMonth() >= startMonth &&
    //         itemDate.getMonth() <= endMonth
    //       );
    //     });
    //   } else if (this.selectedOption === "thisWeek") {
    //     this.filteredData = this.Billings.filter((item) => {
    //       const itemDate = new Date(item.billing_date);
    //       return itemDate >= monday && itemDate <= sunday; // ฟิลเตอร์ตามวันที่อยู่ระหว่างวันจันทร์ถึงวันอาทิตย์
    //     });

    //     this.filteredProductData = this.ProductFilter.filter((item) => {
    //       const itemDate = new Date(item.product_date);
    //       return itemDate >= monday && itemDate <= sunday;
    //     });
    //   } else {
    //     this.filteredData = this.Billings; // แสดงข้อมูลทั้งหมดถ้าเลือก "All"
    //     this.filteredProductData = this.ProductFilter;
    //   }

    //   this.totalSalescon = this.filteredData.reduce(
    //     (sum, item) => sum + (item.sale_totalprice || 0),
    //     0
    //   );
    //   this.totalProductCost = this.filteredProductData.reduce(
    //     (sum, product) => {
    //       if (product.productTypeID === 1) {
    //         // ประเภทที่ 1: คูณ Cost กับ Amount
    //         return sum + product.Cost * (product.Amount || 1); // ถ้าไม่มี Amount จะใช้ค่า 1
    //       } else {
    //         // ประเภทที่ 2: รวมแค่ Cost
    //         return sum + (product.Cost || 0);
    //       }
    //     },
    //     0
    //   );

    //   this.renderChart();
    //   this.renderAmountProductChart();
    //   this.renderServiceBarChart();
    //   this.renderBarChart();
    // },
    async getBusiness() {
      const accessToken = localStorage.getItem("@accessToken");
      this.isLoading = true;
      try {
        const response = await fetch(`${API_CALL}/Quotation/getBusinessByID`, {
          method: "GET",
          headers: {
            Authorization: `Bearer ${accessToken}`,
          },
        });
        const json = await response.json();
        this.BusinessJson = json.data;
      } catch (error) {
        console.error("Error fetching data:", error);
        this.showPopup_error("Error fetching data");
      } finally {
        this.isLoading = false;
      }
    },
    async getExpense() {
      const accessToken = localStorage.getItem("@accessToken");
      this.errorMessage = [];
      this.isLoading = true;
      try {
        const response = await fetch(`${API_CALL}/product/getExpenses`, {
          method: "GET",
          headers: {
            "Content-Type": "application/json",
            Authorization: `Bearer ${accessToken}`,
          },
        });
        const json = await response.json();
        // this.Expenses = json.data;
        this.Expenses = json.data.map((item) => ({
          expense_id: item.expense_id,
          expense_date: item.expense_date,
          expense_category: item.expense_category,
          expense_amount: item.expense_amount,
          quantity_remark: item.quantity_remark,
        }));
        console.log(json.data);
        if (json.statusCode === 200) {
        } else {
          this.showPopup_error(json.data);
        }
      } catch (error) {
        console.log(error);
      } finally {
        this.isLoading = false;
      }
    },
    async getProductCategory() {
      const accessToken = localStorage.getItem("@accessToken");
      this.isLoading = true;
      try {
        const response = await fetch(`${API_CALL}/product/getCategory`, {
          headers: {
            Authorization: `Bearer ${accessToken}`,
          },
        });
        const json = await response.json();

        if (json.statusCode === 200) {
          this.Categories = json.data.map((item) => ({
            ID: item.categoryID,
            "Category Name": item.categoryName,
          }));
          console.log(json);
        } else {
          this.showPopup_error(json.data);
          console.log(this.Categories);
        }
      } catch (error) {
        console.error("Error fetching categories:", error);
      } finally {
        this.isLoading = false;
      }
    },
  },
  watch: {
    custom_filter_start_date() {
      this.filterDataByOption();
    },
    custom_filter_end_date() {
      this.filterDataByOption();
    },
    locale() {
      this.destroyCharts();
      this.$nextTick(() => {
        this.renderChart();
        //this.prepareBarChartData();
        this.renderAmountProductChart();
        this.renderServiceBarChart();
      });
    },
  },
  computed: {
    ConvertTo2DigitCost() {
      if (!Array.isArray(this.Expenses)) {
        return "0.00";
      }

      const totalCost = this.Expenses.reduce((sum, item) => {
        const amount = parseFloat(item.expense_amount);
        return sum + (isNaN(amount) ? 0 : amount);
      }, 0);

      return totalCost.toLocaleString(undefined, {
        minimumFractionDigits: 2,
        maximumFractionDigits: 2,
      });
    },

    ConvertTo2DigitProfit() {
      const totalSales = this.totalSalescon || 0;

      // รวมค่าทั้งหมดใน this.Expenses
      const totalExpense = Array.isArray(this.Expenses)
        ? this.Expenses.reduce((sum, item) => {
            const amount = parseFloat(item.expense_amount);
            return sum + (isNaN(amount) ? 0 : amount);
          }, 0)
        : 0;

      const profit = totalSales - totalExpense;

      if (isNaN(profit)) {
        return "0.00";
      }

      return profit.toLocaleString(undefined, {
        minimumFractionDigits: 2,
        maximumFractionDigits: 2,
      });
    },
  },
  created() {
    this.getProductCategory();
    this.getBilling();
    this.getProduct();
    this.getTypeOfProduct();
    this.getExpense();
    this.$nextTick(() => {
      // this.destroyCharts();
      this.renderChart();
      //this.prepareBarChartData();
      this.renderAmountProductChart();
      this.renderServiceBarChart();
    });
  },
};
</script>
