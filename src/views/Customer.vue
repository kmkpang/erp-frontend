<template>
  <div class="main-page">
    <!-- call navigate tab -->
    <!-- <Navigate /> -->
    <div class="page-body">
      <div class="mb-4">
        <h2 v-if="this.selectedType === 'B'">{{ t("tapcustomerPart") }}</h2>
        <h2 v-if="this.selectedType === 'A'">{{ t("headerCustomer2") }}</h2>
      </div>
      <div class="row mb-3">
        <div class="col-4 col-sm-4 col-md-2 col-lg-2">
          <select class="form-control form-select size-font-md" v-model="dropDownStatus" aria-label="Status select">
            <option value="" selected hidden>{{ t("filter") }}</option>
            <option value="active">{{ t("statusActive") }}</option>
            <option value="not_active">{{ t("statusNotActive") }}</option>
          </select>
        </div>
      </div>
      <div class="row mb-3">
        <div class="col-6 col-sm-6 col-md-3 col-lg-3">
          <input v-model="searchQuery" type="text" class="form-control me-3 size-font-md" :placeholder="$t('Search')" />
        </div>
        <div class="col-6 col-sm-6 col-md-9 col-lg-9">
          <a class="btn btn-success float-right size-font-md" @click="openPopup">{{ t("addCustomer2") }}</a>
        </div>
      </div>

      <div class="card-view-customs">
        <!-- ปุ่ม Expand/Collapse All -->
        <div class="container">
          <div class="text-start">
            {{ allExpanded ? t("CollapseItemsAll") : t("expandedItemsAll") }}
            <span :class="allExpanded ? 'mdi mdi-chevron-up' : 'mdi mdi-chevron-down'
              " @click="toggleAll">
            </span>
          </div>
        </div>
        <div class="row">
          <div v-for="customer in filteredCustomer" :key="customer.ID" class="col-12 mb-3">
            <div class="card d-flex flex-column" style="font-size: 16px">
              <div class="card-header d-flex justify-content-between align-items-center"
                style="background-color: transparent; border-bottom: none;">
                <div class="fw-bold">{{ customer['Customer Name'] }}</div>
                <div class="d-flex gap-3">
                  <!-- <span
                    :class="{'text-success': customer.status === 'เปิดใช้งาน' || customer.status === 'Active', 'text-danger': customer.status !== 'เปิดใช้งาน' && customer.status !== 'Active'}">
                     {{ customer.status }}
                   </span> -->
                  <span class="mdi mdi-pencil-outline" @click="handleEdit(customer)"
                    style="cursor: pointer; font-size: 20px;"></span>
                  <span class="mdi mdi-trash-can-outline text-danger" @click="handleDelete(customer)"
                    style="cursor: pointer; font-size: 20px;"></span>
                </div>
              </div>

              <div class="card-body pt-0" style="line-height: 1.8;">
                <div class="d-flex justify-content-between">
                  <span>{{ t("cusNameHeaderTable") }}</span>
                  <span class="text-end text-break">{{ customer['Customer Name'] }}</span>
                </div>
                <div class="d-flex justify-content-between">
                  <span>{{ t("cusAddressHeaderTable") }}</span>
                  <div class="text-end text-break" style="max-width: 60%;">{{ customer['Customer Address'] }}</div>
                </div>
                <div class="d-flex justify-content-between">
                  <span>{{ t("cusTaxHeaderTable") }}</span>
                  <span class="text-end">{{ customer['Customer Tax'] }}</span>
                </div>
                <div class="d-flex justify-content-between">
                  <span>{{ t("cusTelHeaderTable") }}</span>
                  <span class="text-end">{{ customer['Customer Tel'] }}</span>
                </div>

                <div v-show="isExpanded(customer.ID)">
                  <div class="d-flex justify-content-between">
                    <span>{{ t("cusEmailHeaderTable") }}</span>
                    <span class="text-end text-break">{{ customer['Customer Email'] }}</span>
                  </div>
                  <div class="d-flex justify-content-between">
                    <span>{{ t("cusPurchaseHeaderTable") }}</span>
                    <span class="text-end">{{ customer['Purchase by'] }}</span>
                  </div>
                  <div class="d-flex justify-content-between">
                    <span>{{ t("cusPurchasePhone") }}</span>
                    <span class="text-end">{{ customer['Purchase Phone'] }}</span>
                  </div>
                  <div class="d-flex justify-content-between">
                    <span>{{ t("cusPurchaseEmail") }}</span>
                    <span class="text-end text-break">{{ customer['Purchase Email'] }}</span>
                  </div>
                  <div class="d-flex justify-content-between" v-if="customer['Purchase Remark']">
                    <span>{{ t("quotationRemark") }}</span>
                    <span class="text-end text-break">{{ customer['Purchase Remark'] }}</span>
                  </div>
                </div>
              </div>
              <div class="card-footer text-center bg-transparent border-0 pt-0" @click="toggleCollapse(customer.ID)">
                <span :class="isExpanded(customer.ID) ? 'mdi mdi-chevron-up' : 'mdi mdi-chevron-down'"
                  style="font-size: 24px; cursor: pointer;"></span>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="show-only-desktop sale_hide">
        <CategoryList :initialTableData="filteredCustomer" :tableHeaders="tableHeaders" :columnEditAndDelete="true"
          @handleEdit="handleEdit" @handleDelete="handleDelete" :isLoading="isLoading" :documentName="documentName"
          :showAllowButton="true" :hiddenColumns="['status']" />
      </div>
      <div v-if="isLoading" class="d-flex justify-content-center align-items-center">
        <div class="spinner-border text-primary" role="status">
          <span class="visually-hidden">Loading...</span>
        </div>
      </div>
    </div>
  </div>
  <div>
    <Popup :isOpen="isPopupOpen" :closePopup="closePopup">
      <h2 v-if="isAddingMode" style="padding:0px !important">
        {{ t("headerPopupAddCustomer") }}
      </h2>
      <h2 v-if="isEditMode">{{ t("headerPopupEditCustomer") }}</h2>
      <div class="mb-3">
        <span style="color: red">*</span><label class="col-sm-5 col-md-6">{{ t("customerName") }}</label>
        <input v-model="formData.cus_name" type="text" id="input-text" required
          :class="{ error: isEmpty.cus_name, 'form-control': true }" :placeholder="t('customerNamePlaceholder')"
          @click="resetError('cus_name')" @focus="closeErrorPopup" />
      </div>
      <div class="mb-3">
        <span style="color: red">*</span><label class="col-sm-5 col-md-6">{{ t("customerAddress") }}</label>
        <input :class="{ error: isEmpty.cus_address, 'form-control': true }" v-model="formData.cus_address" type="text"
          id="input-text" :placeholder="t('customerPurchasePlaceholderAddress')" @click="resetError('cus_address')"
          required @focus="closeErrorPopup" />
      </div>
      <div class="mb-3">
        <label class="col-sm-5 col-md-6">{{ t("phoneNum") }}</label>
        <input :class="{ error: isEmpty.cus_tel, 'form-control': true }" v-model="formData.cus_tel" type="text"
          id="input-text" @keypress="validateInput" :placeholder="t('customerPurchasePlaceholderPhoneNum')"
          @click="resetError('cus_tel')" maxlength="10" required @focus="closeErrorPopup" />
      </div>
      <div class="mb-3">
        <label class="col-sm-5 col-md-6">{{ t("email") }}</label>
        <input :class="{ error: isEmpty.cus_email, 'form-control': true }" v-model="formData.cus_email" type="email"
          id="input-text" :placeholder="t('enterEmail')" @click="resetError('cus_email')" @focus="closeErrorPopup" />
      </div>

      <div class="mb-3">
        <span style="color: red">*</span><label class="col-sm-5 col-md-6">{{ t("taxID") }}</label>
        <input :class="{ error: isEmpty.cus_tax, 'form-control': true }" v-model="formData.cus_tax" type="text"
          id="input-text" @keypress="validateInput" :placeholder="t('customerPurchasePlaceholderTaxID')"
          @click="resetError('cus_tax')" maxlength="13" required @focus="closeErrorPopup" />
      </div>
      <div class="mb-3">
        <label class="col-sm-5 col-md-6">{{ t("customerPurchaseBy") }}</label>
        <input :class="{ error: isEmpty.cus_purchase, 'form-control': true }" v-model="formData.cus_purchase"
          type="text" id="input-text" :placeholder="t('enterContactPerson')" @click="resetError('cus_purchase')"
          @focus="closeErrorPopup" />
      </div>

      <div class="mb-3">
        <label class="col-sm-5 col-md-6">{{ t("cusPurchasePhone") }}</label>
        <input class="form-control" v-model="formData.cus_purchase_phone" type="text" id="input-text"
          @keypress="validateInput" :placeholder="t('customerPurchasePlaceholderPhoneNum')"
          @click="resetError('cus_purchase_phone')" maxlength="10" @focus="closeErrorPopup" />
      </div>

      <div class="mb-3">
        <label class="col-sm-5 col-md-6">{{ t("cusPurchaseEmail") }}</label>
        <input class="form-control" v-model="formData.cus_purchase_email" type="email" id="input-text"
          :placeholder="t('enterEmail')" @click="resetError('cus_purchase_email')" @focus="closeErrorPopup" />
      </div>


      <div class="modal-footer">
        <button :disabled="isLoading" class="btn btn-primary me-3" v-if="isAddingMode" @click="addCustomer">
          <span v-if="isLoading" class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
          <span v-else>{{ t("buttonAdd") }}</span>
        </button>
        <button :disabled="isLoading" class="btn btn-primary me-3" v-if="isEditMode" @click="editCustomer">
          <span v-if="isLoading" class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
          <span v-else>{{ t("buttonSave") }}</span>
        </button>
        <button class="btn btn-secondary" @click="closePopup">
          {{ t("buttonCancel") }}
        </button>
      </div>
    </Popup>
    <Popup :isOpen="isPopupOpen2" :closePopup="closePopup2">
      <h2 style="padding: 0px" v-if="isAddingMode">
        {{ t("headerPopupEditCustomer3") }}
      </h2>
      <h2 style="padding: 0px" v-if="isEditMode">
        {{ t("headerPopupEditCustomer2") }}
      </h2>

      <div class="mb-3">
        <label class="col-sm-5 col-md-6"><span style="color: red">*</span>{{ t("cusNameHeaderTable2") }}</label>
        <input v-model="formDataCustomer.company_person_name" type="text" id="input-text" required
          :class="{ error: isEmpty2.company_person_name, 'form-control': true }" :placeholder="t('enterContactPerson')"
          @click="resetError('company_person_name')" @focus="closeErrorPopup" />
      </div>
      <div class="mb-3">
        <label class="col-sm-5 col-md-6" hidden>
          <span style="color: red">*</span>{{ t("cusAddressHeaderTable2") }}</label>
        <input :class="{
          error: isEmpty2.company_person_address,
          'form-control': true,
        }" v-model="formDataCustomer.company_person_address" type="text" id="input-text" required hidden
          @click="resetError('company_person_address')" @focus="closeErrorPopup" />
      </div>
      <div class="mb-3">
        <label class="col-sm-5 col-md-6"><span style="color: red">*</span>{{ t("cusTelHeaderTable2") }}</label>
        <input :class="{ error: isEmpty2.company_person_tel, 'form-control': true }"
          v-model="formDataCustomer.company_person_tel" type="text" id="input-text" maxlength="10"
          @keypress="validateInput" required :placeholder="t('customerPurchasePlaceholderPhoneNum')"
          @click="resetError('company_person_tel')" @focus="closeErrorPopup" />
      </div>
      <div class="mb-3">
        <label class="col-sm-5 col-md-6"><span style="color: red">*</span>{{ t("cusEmailHeaderTable2") }}</label>
        <input :class="{
          error: isEmpty2.company_person_email,
          'form-control': true,
        }" v-model="formDataCustomer.company_person_email" type="text" id="input-text" required
          :placeholder="t('enterEmail')" @click="resetError('company_person_email')" @focus="closeErrorPopup" />
      </div>
      <div class="mb-3">
        <label class="col-sm-5 col-md-6">{{ t("quotationRemark") }}</label>
        <textarea class="form-control" id="exampleFormControlTextarea1" rows="2" :placeholder="t('remarkPlaceholder')"
          @focus="closeErrorPopup" @click="closeErrorPopup"></textarea>
      </div>
      <div class="mb-3">
        <label class="col-sm-5 col-md-6"><span style="color: red">*</span>{{ t("cusCompany") }}</label>
        <select class="form-control col-sm-7 col-md-6 form-select" v-model="formDataCustomer.company_person_customer"
          :class="{ error: isEmpty2.company_person_customer }" id="cus_id" @focus="closeErrorPopup"
          @click="closeErrorPopup">
          <option v-for="CustomerDropown in CustomerDropown" :key="CustomerDropown.cus_id"
            :value="CustomerDropown.cus_id">
            {{ CustomerDropown.cus_name }}
          </option>
        </select>
      </div>
      <div class="modal-footer">
        <button v-if="this.isEditMode" class="btn btn-primary me-3" @click="EditCustomer2">
          Save
        </button>
        <button :disabled="isLoading" class="btn btn-primary me-3" v-if="isAddingMode" @click="addCustomer2">
          <span v-if="isLoading" class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
          <span v-else>{{ t("buttonAdd") }}</span>
        </button>
        <!-- <button
          :disabled="isLoading"
          class="btn btn-primary me-3"
          v-if="isEditMode"
          @click="editCustomer"
        >
          <span
            v-if="isLoading"
            class="spinner-border spinner-border-sm"
            role="status"
            aria-hidden="true"
          ></span>
          <span v-else>{{ t("buttonSave") }}</span>
        </button> -->
        <button class="btn btn-secondary" @click="closePopup2">
          {{ t("buttonCancel") }}
        </button>
      </div>
    </Popup>
    <div class="delete-popup">
      <Popup :isOpen="isDeleteConfirmPopupOpen" :closePopup="closeDeleteConfirmPopup">
        <div class="mb-5">
          <a>{{ t("deleteConfirmSentence") }}</a>
        </div>
        <div class="modal-footer">
          <button :disabled="isLoading" class="btn btn-danger me-3" @click="deleteCustomer">
            <span v-if="isLoading" class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
            <span v-else>{{ t("buttonDelete") }}</span>
          </button>
          <button class="btn btn-secondary" @click="closeDeleteConfirmPopup">
            {{ t("buttonCancel") }}
          </button>
        </div>
      </Popup>
    </div>
    <div v-if="isPopupVisible" class="popup-success">
      <div class="popup-content-success">
        <a>{{ popupMessage }}</a>
      </div>
    </div>
    <!-- <div v-if="isPopupVisible_error" class="popup-success">
      <div class="popup-content-error">
        <h3>{{ $t("validate_popupError") }}</h3>
        <ul>
          <li v-for="(msg, index) in errorMessages" :key="index">{{ msg }}</li>
        </ul>
      </div>
    </div> -->
    <div v-if="isPopupVisible_error" class="popup-error2">
      <div class="text-end">
        <button type="button" class="btn-close" aria-label="Close" @click="closeErrorPopup"
          style="color: #9f9999"></button>
      </div>
      <div class="popup-content-error2">
        <ul>
          <li v-for="(msg, index) in errorMessages" :key="index">
            {{ msg }}
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import Navigate from "../components/Navigation.vue";
import CategoryList from "../components/tableList.vue";
import Popup from "../components/popup.vue";
import { config } from "../../constant.js";
import { useI18n } from "vue-i18n";
import { computed, watch, ref } from "vue";

const API_CALL = config["url"];
const accessToken = localStorage.getItem("@accessToken");

export default {
  name: "Customer",
  components: {
    Navigate, // Navigation component for page navigation
    CategoryList, // Table list component to display customers
    Popup, // Popup component for adding/editing/deleting customers
  },
  setup() {
    const { t } = useI18n(); // Localization setup for translated texts

    const documentName2 = computed(() => t("dontHaveCompany"));
    const documentName = computed(() => t("dontHaveCustomerCompany"));
    return { t, documentName, documentName2 };
  },
  data() {
    return {
      dropDownStatus: "",
      CustomerDropown: [],
      CompanyPerson: [],
      selectedType: "",
      category_file: "",
      errorMessages: [],
      isPopupVisible_error: false, // Control for displaying error popup
      Customers: [], // List of customers fetched from the server
      isPopupOpen: false, // Control for add/edit popup visibility
      isPopupOpen2: false, // Control for add/edit popup visibility
      isDeleteConfirmPopupOpen: false, // Control for delete confirmation popup visibility
      isLoading: false, // Loading state for API requests
      isPopupVisible: false, // Control for success message popup
      inputValue: "",
      inputError: false, // Flag for form input validation
      formData: {
        // Data model for customer form
        cus_id: "",
        cus_name: "",
        cus_address: "",
        cus_tel: "",
        cus_email: "",
        cus_tax: "",
        cus_purchase: "",
        cus_purchase_phone: "",
        cus_purchase_email: "",
        cus_purchase_remark: "",
        status: "active",
      },
      formDataCustomer: {
        // Data model for customer form
        company_person_id: "",
        company_person_name: "",
        company_person_address: "",
        company_person_tel: "",
        company_person_email: "",
        company_person_customer: "",
      },
      isEmpty2: {
        // Data model for customer form
        company_person_id: "",
        company_person_name: "",
        company_person_address: "",
        company_person_tel: "",
        company_person_email: "",
        company_person_customer: "",
      },
      isEmpty: {
        // Data model for customer form
        cus_id: "",
        cus_name: "",
        cus_address: "",
        cus_tel: "",
        cus_email: "",
        cus_tax: "",
        cus_purchase: "",
        status: "active",
      },
      searchQuery: "",
      expandedItems: [],
      allExpanded: false,
    };
  },
  computed: {
    // Define table headers for displaying customer data
    tableHeaders() {
      return [
        { label: this.t("cusNameHeaderTable"), key: "Customer Name" },
        { label: this.t("cusAddressHeaderTable"), key: "Customer Address" },
        { label: this.t("cusTelHeaderTable"), key: "Customer Tel" },
        { label: this.t("cusEmailHeaderTable"), key: "Customer Email" },
        { label: this.t("cusTaxHeaderTable"), key: "Customer Tax" },
        { label: this.t("cusPurchaseHeaderTable"), key: "Purchase by" },
        { label: this.t("cusPurchasePhone"), key: "Purchase Phone" },
        { label: this.t("cusPurchaseEmail"), key: "Purchase Email" },
      ];
    },
    tableHeaders2() {
      return [
        { label: this.t("onlystatusHeaderTable"), key: "status" },
        { label: this.t("cusNameHeaderTable2"), key: "Customer Name" },
        // { label: this.t("cusAddressHeaderTable2"), key: "Customer Address" },
        { label: this.t("cusTelHeaderTable2"), key: "Customer Tel" },
        { label: this.t("cusEmailHeaderTable2"), key: "Customer Email" },
        { label: this.t("cusCompany"), key: "Customer Company" },
      ];
    },
    filteredCustomer() {
      let filteredCus = [...this.Customers];
      // กรองข้อมูล Categories โดยเปรียบเทียบกับ searchQuery
      // if (this.formData.status) {
      //   filteredCus = filteredCus.filter(
      //     (cus) =>
      //       cus.status.toLowerCase() === this.formData.status.toLowerCase()
      //   );
      // }

      if (this.searchQuery.trim()) {
        filteredCus = filteredCus.filter((cus) =>
          cus["Customer Name"]
            .toLowerCase()
            .includes(this.searchQuery.toLowerCase())
        );
      }

      filteredCus.forEach((cus) => {
        const lang = this.t("headerLang");

        if (lang === "TH") {
          if (cus.status === "Active" || cus.status === "active") {
            cus.status = "เปิดใช้งาน";
          } else if (
            cus.status === "Not Active" ||
            cus.status === "not active"
          ) {
            cus.status = "ปิดใช้งาน";
          }
        } else {
          // กรณีภาษาอังกฤษหรือภาษาอื่นๆ อยากแสดงกลับเป็น status เดิม (กรณีมีการแปลงไปแล้ว)
          if (cus.status === "เปิดใช้งาน" || cus.status === "active") {
            cus.status = "Active";
          } else if (
            cus.status === "ปิดใช้งาน" ||
            cus.status === "not active"
          ) {
            cus.status = "Not Active";
          }
        }
      });

      // กรองจาก dropDownStatus
      if (this.selectedType === "A") {
        if (this.dropDownStatus === "active") {
          filteredCus = filteredCus.filter(
            (cus) =>
              cus.status.toLowerCase() === "active" ||
              cus.status.toLowerCase() === "เปิดใช้งาน"
          );
        } else if (this.dropDownStatus === "not_active") {
          filteredCus = filteredCus.filter(
            (cus) =>
              cus.status.toLowerCase() === "not active" ||
              cus.status.toLowerCase() === "ปิดใช้งาน"
          );
        }
      }
      return filteredCus; // ถ้าไม่มีการค้นหาแสดงทั้งหมด
    },
    filteredCompany() {
      let filteredCus = [...this.CompanyPerson];
      // console.log("filteredCus__>", filteredCus);
      // กรองข้อมูล Categories โดยเปรียบเทียบกับ searchQuery

      filteredCus.forEach((cus) => {
        const lang = this.t("headerLang");

        if (lang === "TH") {
          if (cus.status === "Active" || cus.status === "active") {
            cus.status = "เปิดใช้งาน";
          } else if (
            cus.status === "Not Active" ||
            cus.status === "not active"
          ) {
            cus.status = "ปิดใช้งาน";
          }
        } else {
          // กรณีภาษาอังกฤษหรือภาษาอื่นๆ อยากแสดงกลับเป็น status เดิม (กรณีมีการแปลงไปแล้ว)
          if (cus.status === "เปิดใช้งาน" || cus.status === "active") {
            cus.status = "Active";
          } else if (
            cus.status === "ปิดใช้งาน" ||
            cus.status === "not active"
          ) {
            cus.status = "Not Active";
          }
        }
      });

      if (this.dropDownStatus === "active") {
        filteredCus = filteredCus.filter(
          (cus) =>
            cus.status.toLowerCase() === "active" ||
            cus.status.toLowerCase() === "เปิดใช้งาน"
        );
      } else if (this.dropDownStatus === "not_active") {
        filteredCus = filteredCus.filter(
          (cus) =>
            cus.status.toLowerCase() === "not active" ||
            cus.status.toLowerCase() === "ปิดใช้งาน"
        );
      }

      if (this.searchQuery.trim()) {
        filteredCus = filteredCus.filter((cus) =>
          cus["Customer Name"]
            .toLowerCase()
            .includes(this.searchQuery.toLowerCase())
        );
      }
      // console.log("filteredCus", filteredCus);
      // // กรองจาก dropDownStatus
      // if (this.dropDownStatus === "active") {
      //   filteredCus = filteredCus.filter(
      //     (cus) => cus.status.toLowerCase() === "active"
      //   );
      // } else if (this.dropDownStatus === "not_active") {
      //   filteredCus = filteredCus.filter(
      //     (cus) => cus.status.toLowerCase() === "not active"
      //   );
      // }
      // console.log("filteredCus", filteredCus);
      // console.log(filteredCus);
      return filteredCus; // ถ้าไม่มีการค้นหาแสดงทั้งหมด
    },
  },
  methods: {
    toggleCollapse(id) {
      if (this.expandedItems.includes(id)) {
        this.expandedItems = this.expandedItems.filter((itemId) => itemId !== id);
      } else {
        this.expandedItems.push(id);
      }
    },
    isExpanded(id) {
      if (this.allExpanded) {
        return true;
      }
      return this.expandedItems.includes(id);
    },
    toggleAll() {
      this.allExpanded = !this.allExpanded;
      if (this.allExpanded) {
        this.expandedItems = this.filteredCustomer.map((item) => item.ID);
      } else {
        this.expandedItems = [];
      }
    },
    validateInput(event) {
      const charCode = event.which ? event.which : event.keyCode;
      if (charCode < 48 || charCode > 57) {
        event.preventDefault();
      }
    },
    formatPhoneNumber(phone) {
      if (!phone || phone === '-') return "-";
      const cleaned = ('' + phone).replace(/\D/g, '');
      const match10 = cleaned.match(/^(\d{3})(\d{3})(\d{4})$/);
      if (match10) {
        return `${match10[1]}-${match10[2]}-${match10[3]}`;
      }
      const match9 = cleaned.match(/^(\d{2})(\d{3})(\d{4})$/);
      if (match9) {
        return `${match9[1]}-${match9[2]}-${match9[3]}`;
      }
      return phone;
    },
    closeErrorPopup() {
      this.isPopupVisible_error = false;
    },
    resetError(field) {
      if (field && this.isEmpty[field] !== undefined) {
        this.isEmpty[field] = false;
      }
      if (field && this.isEmpty2[field] !== undefined) {
        this.isEmpty2[field] = false;
      }
      this.isPopupVisible_error = false;
    },
    async validateFormData() {
      this.isEmpty.cus_name = false;
      this.isEmpty.cus_address = false;
      this.isEmpty.cus_tel = false;
      this.isEmpty.cus_email = false;
      this.isEmpty.cus_purchase = false;
      this.isEmpty.cus_tax = false;

      const errorMessages = [];

      if (this.formData.cus_name.trim() === "") {
        this.isEmpty.cus_name = true;
        errorMessages.push(this.$t("validation.cus_name"));
      }

      if (this.formData.cus_address.trim() === "") {
        this.isEmpty.cus_address = true;
        errorMessages.push(this.$t("validation.cus_address"));
      }

      if (this.formData.cus_tel.trim() !== "") {
        if (!/^\d+$/.test(this.formData.cus_tel)) {
          console.log("cus_tel", this.formData.cus_tel);
          this.isEmpty.cus_tel = true;
          errorMessages.push(this.$t("validation.phone_numeric"));
        } else if (
          this.formData.cus_tel.length < 9 ||
          this.formData.cus_tel.length > 10
        ) {
          this.isEmpty.cus_tel = true;
          errorMessages.push(this.$t("validation.cus_tel_length"));
        }
      }

      if (
        this.formData.cus_email.trim() !== "" &&
        !this.validateEmail(this.formData.cus_email)
      ) {
        this.isEmpty.cus_email = true;
        errorMessages.push(this.$t("validation.cus_email2"));
      }

      if (this.formData.cus_tax.trim() === "") {
        this.isEmpty.cus_tax = true;
        errorMessages.push(this.$t("validation.cus_tax"));
      } else if (this.formData.cus_tax.length !== 13) {
        this.isEmpty.cus_tax = true;
        errorMessages.push(this.$t("validation.cus_tax_length"));
      }



      if (this.formData.cus_purchase_phone && !/^\d+$/.test(this.formData.cus_purchase_phone)) {
        errorMessages.push(this.$t("validation.phone_numeric"));
      }

      if (errorMessages.length > 0) {
        this.showPopup_validate(errorMessages);
        return false;
      } else {
        return true;
      }
    },
    showPopup_validate(messages) {
      if (Array.isArray(messages)) {
        this.errorMessages = messages; // เก็บข้อความใน errorMessages

        // this.showErrorPopup = true; // แสดง Popup
        this.isPopupVisible_error = true;
        // setTimeout(() => {
        //   this.isPopupVisible_error = false; // ซ่อน Popup หลังจากหน่วงเวลา
        // }, 3000); // หน่วงเวลา 3 วินาที (3000 มิลลิวินาที)
      } else {
        this.showPopup_error(messages);
      }
    },
    // Open the popup for adding a new customer
    async openPopup() {
      this.isPopupOpen = true;
      this.isAddingMode = true; // Add mode
      this.isEditMode = false; // Disable edit mode
    },
    // Close the popup and reset form data
    closePopup() {
      this.isPopupOpen = false;
      this.isAddingMode = false;
      this.isEditMode = false;
      this.formData = {
        // Reset form data
        cus_id: "",
        cus_name: "",
        cus_address: "",
        cus_tel: "",
        cus_email: "",
        cus_tax: "",
        cus_purchase: "",
        cus_purchase_phone: "",
        cus_purchase_email: "",
        cus_purchase_remark: "",
        status: "",
      };
      this.isEmpty.cus_name = false;
      this.isEmpty.cus_address = false;
      this.isEmpty.cus_tel = false;
      this.isEmpty.cus_email = false;
      this.isEmpty.cus_purchase = false;
      this.isEmpty.cus_tax = false;

      this.category_file = ""; // Reset file input
      this.inputError = false; // Reset validation error state
      this.isPopupVisible_error = false;
    },
    // Close the delete confirmation popup
    closeDeleteConfirmPopup() {
      this.isDeleteConfirmPopupOpen = false;
    },
    // Open the popup in edit mode and pre-fill the form with selected customer data
    handleEdit(item) {
      console.log("Edit item:", item);
      this.isPopupOpen = true;
      this.isAddingMode = false; // Disable add mode
      this.isEditMode = true; // Enable edit mode
      this.formData = {
        // Populate form with selected customer data
        cus_id: item.ID,
        status: item.status,
        cus_name: item["Customer Name"],
        cus_address: item["Customer Address"],
        cus_tel: item["Customer Tel"] ? String(item["Customer Tel"]).replace(/\D/g, "") : "",
        cus_email: item["Customer Email"],
        cus_tax: item["Customer Tax"],
        cus_purchase: item["Purchase by"],
        cus_purchase_phone: item["Purchase Phone"] ? String(item["Purchase Phone"]).replace(/\D/g, "") : "",
        cus_purchase_email: item["Purchase Email"],
        cus_purchase_remark: item["Purchase Remark"],
      };
    },
    // Open the delete confirmation popup and store the customer ID to be deleted
    handleDelete(item) {
      console.log("Delete button clicked for item:", item);
      this.isDeleteConfirmPopupOpen = true;
      this.formData = {
        cus_id: item.ID,
      };
    },
    // Show success message popup
    showPopup(message) {
      this.popupMessage = message;
      this.isPopupVisible = true;
      setTimeout(() => {
        this.isPopupVisible = false; // Auto-hide after 2 seconds
      }, 2000);
    },
    // Show error message popup
    showPopup_error(message) {
      this.popupMessage_error = message;

      this.isPopupVisible_error = true;
      setTimeout(() => {
        this.isPopupVisible_error = false; // Auto-hide after 2 seconds
      }, 2000);
    },
    // Validate email format using regular expression
    validateEmail(email) {
      const emailRegex = /^[^\s@]+@[^\s@]+\.[a-zA-Z]{2,}(\.[a-zA-Z]{2,})?$/;
      return emailRegex.test(email);
    },
    // Import customer data from a CSV file
    async importCategory() {
      const accessToken = localStorage.getItem("@accessToken");
      this.isLoading = true; // Set loading state
      try {
        const formDataFile = new FormData();
        formDataFile.append("file", this.category_file);

        const response = await fetch(`${API_CALL}/migrate/importcsv`, {
          method: "POST",
          headers: {
            Authorization: `Bearer ${accessToken}`,
          },
          body: formDataFile,
        });
        const json = await response.json();

        if (json.statusCode === 200) {
          this.getCustomer(); // Refresh customer list
          console.log("Category imported successfully");
          this.category_file = ""; // Reset file input
        } else {
          console.log("Import category error:", json);
        }
      } catch (error) {
        console.error("Error fetching data:", error);
      } finally {
        this.isLoading = false; // Reset loading state
      }
    },
    // Fetch the list of customers from the API
    async getCustomer() {
      const accessToken = localStorage.getItem("@accessToken");
      this.isLoading = true; // Set loading state
      try {
        const response = await fetch(`${API_CALL}/Quotation/getCustomer`, {
          headers: {
            Authorization: `Bearer ${accessToken}`,
          },
        });
        const json = await response.json();
        console.log("json", json);

        if (json.statusCode === 200) {
          // Map API response data to the expected table structure
          this.Customers = json.data
            .filter((item) => item.Status !== "not active")
            .map((item) => {
              let initialTableData = {
                ID: item.cus_id,
                status: item.Status,
                "Customer Name": item.cus_name,
                "Customer Address": item.cus_address,
                "Customer Tel": this.formatPhoneNumber(item.cus_tel),
                "Customer Email": item.cus_email,
                "Customer Tax": item.cus_tax,
                "Purchase by": item.cus_purchase,
                "Purchase Phone": this.formatPhoneNumber(item.cusPurchasePhone),
                "Purchase Email": item.cusPurchaseEmail,
                "Purchase Remark": item.cusPurchaseRemark,
              };
              return initialTableData;
            });
        } else {
          this.showPopup_error(json.data); // Show error message
          console.log(json);
        }
      } catch (error) {
        console.error("Error fetching data:", error);
      } finally {
        this.isLoading = false; // Reset loading state
      }
    },
    async addCustomer() {
      const accessToken = localStorage.getItem("@accessToken");
      if (!(await this.validateFormData())) return;
      this.isLoading = true;
      try {
        const response = await fetch(`${API_CALL}/Quotation/addCustomer`, {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
            Authorization: `Bearer ${accessToken}`,
          },
          body: JSON.stringify(this.formData),
        });
        const json = await response.json();

        if (json.statusCode === 200) {
          this.isPopupVisible_error = false;
          this.getCustomer(); // Refresh customer list
          this.showPopup(this.$t("validation.AddSucc"));
          this.formData = {
            // Reset form data
            cus_id: "",
            cus_name: "",
            cus_address: "",
            cus_tel: "",
            cus_email: "",
            cus_tax: "",
            cus_purchase: "",
            cus_purchase_phone: "",
            cus_purchase_email: "",
            cus_purchase_remark: "",
            status: "",
          };
          this.isPopupOpen = false; // Close the popup
        } else {
          this.errorMessages = [];
          if (json.data.includes("tax")) {
            this.errorMessages.push(this.$t("validation.cus_tax_already"));
            this.isEmpty.cus_tax = true;
          } else if (json.data.includes("Contact")) {
            this.errorMessages.push(this.$t("validation.cus_name_Contact"));
            this.isEmpty.cus_tel = true;
          } else if (json.data.includes("Customer")) {
            this.errorMessages.push(this.$t("validation.cus_name_already"));
            this.isEmpty.cus_name = true;
          }
          this.showPopup_error(json.data);
          console.log("Add customer error:", json);
        }
      } catch (error) {
        console.error("Error adding customer:", error);
      } finally {
        this.isLoading = false; // Reset loading state
      }
    },
    // Edit an existing customer via an API call, with form validation
    async editCustomer() {
      const accessToken = localStorage.getItem("@accessToken");
      // Validate if required fields are filled
      if (!(await this.validateFormData())) return;
      this.errorMessages = [];

      this.isLoading = true; // Set loading state
      const cusID = this.formData.cus_id;
      try {
        const response = await fetch(
          `${API_CALL}/Quotation/editCustomer/${cusID}`,
          {
            method: "PUT",
            headers: {
              "Content-Type": "application/json",
              Authorization: `Bearer ${accessToken}`,
            },
            body: JSON.stringify(this.formData),
          }
        );
        const json = await response.json();

        if (json.statusCode === 200) {
          this.getCustomer(); // Refresh customer list
          this.showPopup(this.$t("validation.EditSucc"));
          this.closePopup(); // Close the popup
        } else {
          if (json.data.includes("tax")) {
            this.errorMessages.push(this.$t("validation.cus_tax_already"));
            this.isEmpty.cus_tax = true;
          } else if (json.data.includes("Contact")) {
            this.errorMessages.push(this.$t("validation.cus_name_Contact"));
            this.isEmpty.cus_tel = true;
          } else if (json.data.includes("Customer")) {
            this.errorMessages.push(this.$t("validation.cus_name_already"));
            this.isEmpty.cus_name = true;
          }
          this.showPopup_error(json.data);
          console.log("Edit customer error:", json);
        }
      } catch (error) {
        console.error("Error editing customer:", error);
      } finally {
        this.isLoading = false; // Reset loading state
      }
    },
    // Delete a customer via an API call
    async deleteCustomer() {
      const accessToken = localStorage.getItem("@accessToken");
      let cusID = this.formData.cus_id;
      this.isLoading = true;

      try {
        const endpoint = `${API_CALL}/Quotation/deleteCustomer/${cusID}`;

        // ส่งคำขอลบ
        const response = await fetch(endpoint, {
          method: "DELETE",
          headers: {
            "Content-Type": "application/json",
            Authorization: `Bearer ${accessToken}`,
          },
        });

        if (!response.ok) {
          throw new Error("Failed to delete customer.");
        }

        const json = await response.json();

        if (json.statusCode === 200) {
          this.Customers = this.Customers.filter(
            (item) => item.cus_id !== cusID
          );
          this.getCustomer();
          this.showPopup(this.$t("validation.DelateSucc"));
          this.closeDeleteConfirmPopup();
        } else {
          this.showPopup(this.$t("validation.DelateFail"));
        }
      } catch (error) {
        console.error("Error deleting customer:", error);
        this.showPopup(this.$t("validation.ErrorOccurred"));
      } finally {
        this.isLoading = false; // Reset loading state
      }
    },
  },
  // Fetch the customer list when the component is created
  created() {
    this.selectedType = "A";
    this.getCustomer();
  },
};
</script>
