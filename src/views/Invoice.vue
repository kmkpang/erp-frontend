<template>
  <div class="main-page">
    <!-- call navigate tab -->
    <!-- <Navigate /> -->
    <div class="page-body" :isLoading="isLoading">
      <div class="mb-3">
        <h2>{{ t("headerInvoice") }}</h2>
      </div>
      <!-- <div class="add-btn mb-3">
        <input
          v-model="searchQuery"
          type="text"
          class="form-control custome-search-css"
          style="width: 20%"
          :placeholder="$t('Search')"
        />
      </div> -->
      <div class="row mb-3">
        <div class="col-6 col-sm-6 col-md-6 col-lg-3">
          <input
            v-model="searchQuery"
            type="text"
            class="form-control me-3 size-font-md"
            :placeholder="$t('Search')"
          />
        </div>
        <div class="col-6 col-sm-6 col-md-6 col-lg-9"></div>
      </div>

      <div class="card-view-customs" style="display: none">
        <!-- ปุ่ม Expand/Collapse All -->
        <div class="container">
          <div class="text-start">
            {{ allExpanded ? t("CollapseItemsAll") : t("expandedItemsAll") }}
            <span
              :class="
                allExpanded ? 'mdi mdi-chevron-up' : 'mdi mdi-chevron-down'
              "
              @click="toggleAll"
            >
            </span>
          </div>
        </div>

        <div class="row">
          <div
            class="col-md-4 mt-3"
            v-for="quotation in filteredIn"
            :key="quotation.sale_id"
          >
            <div class="card d-flex flex-column" style="font-size: 16px">
              <div class="card-header d-flex justify-content-between align-items-center" style="background-color: transparent; border-bottom: none;">
                <div class="fw-bold">{{ quotation.invoice_number }}</div>
                <div class="d-flex gap-3">
                   <span
                    v-if="['ยังไม่มีใบเสร็จรับเงิน', 'Receipt not Issued'].includes(quotation.tax_invoice_status)"
                    class="mdi mdi-check-circle text-success"
                    @click="handleAllow(quotation)"
                    style="cursor: pointer; font-size: 20px;"
                  ></span>
                  <span class="mdi mdi-pencil-outline" @click="handleEdit(quotation)" style="cursor: pointer; font-size: 20px;"></span>
                   <span
                     v-if="['ยังไม่มีใบเสร็จรับเงิน', 'Receipt not Issued'].includes(quotation.tax_invoice_status)"
                    class="mdi mdi-trash-can-outline text-danger"
                    @click="handleDelete(quotation)"
                    style="cursor: pointer; font-size: 20px;"
                  ></span>
                </div>
              </div>

              <div class="card-body pt-0" style="line-height: 1.8;">
                <div class="d-flex justify-content-between">
                  <span>{{ t("cusNameHeaderTable") }}</span>
                  <span class="text-end text-break">{{ quotation.cus_name }}</span>
                </div>
                <div class="d-flex justify-content-between">
                  <span>{{ t("employeeNameHeaderTable") }}</span>
                  <span class="text-end">{{ quotation.employeeName }}</span>
                </div>
                <div class="d-flex justify-content-between">
                  <span>{{ t("saleTotalpriceHeaderTable") }}</span>
                  <span class="text-end">{{ quotation.sale_totalprice }}</span>
                </div>
                <div class="d-flex justify-content-between">
                  <span>{{ t("netpriceHeaderTable") }}</span>
                  <span class="text-end">{{ quotation.net_price }}</span>
                </div>

                <div v-show="isExpanded(quotation.sale_id)">
                  <div class="d-flex justify-content-between">
                    <span>{{ t("cusAddressHeaderTable") }}</span>
                    <div class="text-end text-break" style="max-width: 60%;">{{ quotation.cus_address }}</div>
                  </div>
                  <div class="d-flex justify-content-between">
                    <span>{{ t("cusTelHeaderTable") }}</span>
                    <span class="text-end">{{ quotation.cus_tel }}</span>
                  </div>
                  <div class="d-flex justify-content-between">
                    <span>{{ t("cusEmailHeaderTable") }}</span>
                    <span class="text-end text-break">{{ quotation.cus_email }}</span>
                  </div>
                  <div class="d-flex justify-content-between">
                    <span>{{ t("cusTaxHeaderTable") }}</span>
                    <span class="text-end">{{ quotation.cus_tax }}</span>
                  </div>
                  <div class="d-flex justify-content-between">
                    <span>{{ t("cusPurchaseHeaderTable") }}</span>
                    <span class="text-end">{{ quotation.cus_purchase }}</span>
                  </div>
                  <div class="d-flex justify-content-between">
                    <span>{{ t("saleDateHeaderTable") }}</span>
                    <span class="text-end">{{ quotation.invoice_date }}</span>
                  </div>
                  <div class="d-flex justify-content-between">
                    <span>{{ t("billingStatusHeaderTable") }}</span>
                    <span class="text-end">{{ quotation.billing }}</span>
                  </div>

                  <div class="d-flex justify-content-end gap-3 mt-2">
                    <span class="mdi mdi-eye-outline" @click="handlePreview(quotation)" style="cursor: pointer; font-size: 20px;"></span>
                    <span class="mdi mdi-tray-arrow-down" @click="handleDownload(quotation)" style="cursor: pointer; font-size: 20px;"></span>
                  </div>
                </div>
              </div>

               <div class="card-footer text-center bg-transparent border-0 pt-0" @click="toggleCollapse(quotation.sale_id)">
                 <span :class="isExpanded(quotation.sale_id) ? 'mdi mdi-chevron-up' : 'mdi mdi-chevron-down'" style="font-size: 24px; cursor: pointer;"></span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="show-only-desktop sale_hide">
        <InvoiceList
          :initialTableData="filteredIn"
          :tableHeaders="tableHeaders"
          :columnforExport="true"
          :columnEditAndDelete="true"
          @handleEdit="handleEdit"
          @handleDelete="handleDelete"
          @handleExport="handleDownload"
          @handlePreview="handlePreview"
          @handleAllow="handleAllow"
          :isLoading="isLoading"
          :documentName="documentName"
          :showAllowButton="false"
        />
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
  <popup :isOpen="isPopupOpen" :closePopup="closePopup">
    <h2>{{ t("headerInvoice") }}</h2>
    <div class="border p-4 mb-3">
      <!-- <h2>{{ t("headerInvoice") }}</h2> -->
      <div class="mb-3 div-for-formControl">
        <label class="col-sm-6 col-md-6">{{ t("numberInvoice") }}</label>
        <input
          class="form-control readonly"
          v-model="formData.invoice_number"
          readonly
          disabled
        />
      </div>
      <div class="mb-3 div-for-formControl">
        <label class="col-sm-6 col-md-6">{{ t("dateInvoice") }}</label>
        <div class="col-6 col-sm-6 col-md-6">
          <!-- <DatePicker
            v-model:value="formData.invoice_date"
            format="DD/MM/YYYY"
            value-type="date"
            placeholder="DD/MM/YYYY"
            class="form-control"
            :formatter="momentFormat"
            :lang="currentLocale"
          /> -->
          <v-date-picker
            v-model="formData.invoice_date"
            locale="th-TH"
            :format="formatDatePicker"
          >
            <template v-slot="{ inputEvents }">
              <input
                class="custom-input"
                :value="formatDatePicker(formData.invoice_date)"
                v-on="inputEvents"
                :placeholder="t('selectDate')"
                style="width: 100%"
                @click="resetError()"
              />
            </template>
          </v-date-picker>
        </div>
      </div>
      <div class="mb-3 div-for-formControl" v-if="isEditMode">
        <label class="col-sm-6 col-md-6">{{ t("statusInvoice") }}</label>
        <div class="col-6 col-sm-6 col-md-6">
          <select
            class="form-control dropdown_selector form-select"
            v-model="formData.invoice_status"
            disabled
            readonly
          >
            <option value="Issue a receipt">{{ t("issueReceipt") }}</option>
            <option style="display: none" value="Pending" disabled>
              {{ t("PendingLG_invoice") }}
            </option>
            <option style="display: none" value="expired" disabled>
              {{ t("expiredStatus") }}
            </option>
          </select>
        </div>
      </div>
      <div class="mb-3 div-for-formControl">
        <label class="col-sm-5 col-md-2">{{ t("employeeName") }}</label>
        <input
          class="form-control dropdown_selector readonly"
          v-model="formData.employeeName"
          readonly
          disabled
        />
      </div>
    </div>
    <div class="border p-4 mb-3">
      <div>
        <h5 style="text-decoration-line: underline">{{ t("customerPart") }}</h5>
      </div>
      <div class="mb-3 div-for-InputDropdown">
        <label class="col-6 col-sm-6 col-md-6">{{ t("customerName") }}</label>
        <!-- <div class="relative-wrapper"> -->
        <div class="col-6 col-sm-6 col-md-6">
          <input
            class="form-control readonly"
            v-model="formData.cus_name"
            @input="filterItems"
            :class="{ error: inputError }"
            readonly
            disabled
            @click="resetError()"
          />
        </div>
        <!-- </div> -->
      </div>
      <div class="mb-3 div-for-formControl">
        <label class="col-sm-5 col-md-6">{{ t("customerAddress") }}</label>
        <input
          class="form-control readonly"
          v-model="formData.cus_address"
          readonly
          disabled
          @click="resetError()"
        />
      </div>
      <div class="mb-3 div-for-formControl">
        <label class="col-sm-5 col-md-6">{{ t("phoneNum") }}</label>
        <input
          class="form-control readonly"
          v-model="formData.cus_tel"
          readonly
          disabled
          @click="resetError()"
        />
      </div>
      <div class="mb-3 div-for-formControl">
        <label class="col-sm-5 col-md-6">{{ t("email") }}</label>
        <input
          class="form-control readonly"
          v-model="formData.cus_email"
          readonly
          disabled
          @click="resetError()"
        />
      </div>
      <div class="mb-3 div-for-formControl">
        <label class="col-sm-5 col-md-6">{{ t("taxID") }}</label>
        <input
          class="form-control readonly"
          v-model="formData.cus_tax"
          readonly
          disabled
          @click="resetError()"
        />
      </div>
      <div class="mb-3 div-for-formControl">
        <label class="col-sm-5 col-md-6">{{ t("customerPurchaseBy") }}</label>
        <input
          class="form-control readonly"
          v-model="formData.cus_purchase"
          readonly
          disabled
          @click="resetError()"
        />
      </div>
    </div>
    <div class="mb-3">
      <div class="Register-contain" style="padding: 20px; width: unset">
        <div>
          <h5 style="text-decoration-line: underline">{{ t("product") }}</h5>
          <table class="table">
            <thead>
              <tr>
                <th class="product-name-column">{{ t("productName") }}</th>
                <th class="price-column">{{ t("price") }}</th>
                <th class="quantity-column">{{ t("quantity") }}</th>
                <th class="unit-column">{{ t("pro_unit") }}</th>
                <th class="discount-column">{{ t("discount") }}</th>
                <th class="total-price-column">{{ t("totalPrice") }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(form, index) in productForms" :key="index">
                <td>
                  <input
                    class="form-control readonly"
                    v-model="form.productname"
                    readonly
                    disabled
                  />
                  <!-- <a
                    class="text-muted ng-star-inserted text-start"
                    href="javascript:void(0)"
                    @click="toggleProductDetail(form, index)"
                    ><div _ngcontent-vfm-c67="" class="description-row">
                      เพิ่มรายละเอียดสินค้า
                    </div></a
                  > -->
                  <textarea
                    v-if="form.showDetails || form.product_detail !== ''"
                    class="form-control"
                    v-model="form.product_detail"
                    rows="3"
                    readonly
                    disabled
                  ></textarea>
                </td>
                <td>
                  <input
                    class="form-control readonly"
                    v-model="form.price"
                    readonly
                    disabled
                  />
                </td>
                <td>
                  <input
                    class="form-control readonly"
                    v-model="form.sale_qty"
                    @input="updatePrice(form)"
                    readonly
                    disabled
                  />
                </td>
                <td>
                  <input
                    class="form-control"
                    v-model="form.pro_unti"
                    readonly
                    disabled
                  />
                </td>

                <td>
                  <div class="discount-type">
                    <select
                      class="form-control form-select"
                      v-model="form.discountType"
                      @change="updatePrice(form)"
                      :disabled="true"
                      style="
                        border-top-right-radius: 0px;
                        border-bottom-right-radius: 0px;
                      "
                    >
                      <option value="amount">
                        {{ t("productDiscountTypeAmount") }}
                      </option>
                      <option value="percent">
                        {{ t("productDiscountTypePercent") }}
                      </option>
                    </select>
                    <input
                      style="
                        min-width: 100px;
                        border-top-left-radius: 0px;
                        border-bottom-left-radius: 0px;
                        border-left: 1px solid rgba(0, 0, 0, 0) !important;
                      "
                      class="form-control"
                      v-model="form.sale_discount"
                      readonly
                      disabled
                      min="0"
                      @input="limitDiscount(form)"
                      @change="updatePrice(form)"
                    />
                  </div>
                  <!-- <div class="discount-type">
                    <input class="form-control" />
                    <input
                      style="min-width: 100px"
                      class="form-control readonly"
                      v-model="form.sale_discount"
                      @input="limitDiscount(form)"
                      @change="updatePrice(form)"
                      readonly
                    />
                  </div> -->
                </td>
                <td>
                  <input
                    class="form-control readonly"
                    v-model="form.sale_price"
                    readonly
                    disabled
                  />
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
    <div class="border p-4 mb-3">
      <div class="mb-3 div-for-formControl">
        <label class="col-sm-5 col-md-6">{{ t("totalDiscount") }}</label>
        <input
          class="form-control readonly"
          v-model="formData.discount_quotation"
          readonly
          disabled
          :class="{ error: inputError }"
          @click="resetError()"
        />
      </div>
      <!-- <div class="mb-3 div-for-formControl">
      <label class="col-sm-5 col-md-6">{{ t("consluPrice") }}</label>
      <input
        class="form-control readonly"
        v-model="formData.sale_totalprice"
        readonly
        :class="{ error: inputError }"
        disabled
      /> -->
      <!-- <label
        v-if="this.formData.vatType === 'VATincluding'"
        class="col-sm-5 col-md-6"
        >{{ t("consluPrice") }}</label
      >
      <label v-else class="col-sm-5 col-md-6">{{ t("consluPrice") }}</label>
      <input
        v-if="this.formData.vatType === 'VATincluding'"
        class="form-control readonly"
        v-model="formData.sale_totalprice"
        readonly
        :class="{ error: inputError }"
        disabled
      />
      <input
        v-else
        class="form-control readonly"
        v-model="formData.Net_price"
        readonly
        :class="{ error: inputError }"
        disabled
      /> -->
      <!-- </div> -->
      <div class="mb-3 div-for-formControl">
        <!-- <label class="col-sm-5 col-md-6">{{ t("consluPrice") }} </label> -->
        <label
          v-if="this.formData.vatType === 'VATincluding'"
          class="col-sm-5 col-md-6"
          >{{ t("consluPrice") }}</label
        >
        <label v-else class="col-sm-5 col-md-6">{{ t("consluPrice") }}</label>
        <input
          v-if="this.formData.vatType === 'VATincluding'"
          class="form-control readonly"
          v-model="formData.sale_totalprice"
          readonly
          :class="{ error: inputError }"
          disabled
          @click="resetError()"
        />
        <input
          v-else
          class="form-control readonly"
          v-model="formData.Net_price"
          readonly
          :class="{ error: inputError }"
          disabled
          @click="resetError()"
        />
      </div>
      <div class="row mb-3">
        <label class="col-sm-5 col-md-6">{{ t("typeVat") }}</label>
        <div class="col-md-6">
          <div class="form-check form-check-inline">
            <input
              class="form-check-input"
              type="radio"
              value="VATexcluding"
              v-model="formData.vatType"
              @change="vatTypeChange()"
              disabled
            />
            <label class="form-check-label" for="inlineCheckbox1">{{
              t("vatType1")
            }}</label>
          </div>
          <div class="form-check form-check-inline">
            <input
              class="form-check-input"
              type="radio"
              value="VATincluding"
              v-model="formData.vatType"
              @change="vatTypeChange()"
              disabled
            />
            <label class="form-check-label" for="inlineCheckbox2">{{
              t("vatType2")
            }}</label>
          </div>
        </div>
      </div>
      <div class="mb-3 div-for-formControl">
        <label class="col-sm-5 col-md-6">{{ t("vatPrice") }}</label>
        <input
          :placeholder="t('vatPricePlaceholder')"
          v-model="formData.vat"
          class="form-control readonly"
          readonly
          disabled
          :class="{ error: inputError }"
          @click="resetError()"
        />
      </div>
      <!-- <div class="mb-3 div-for-formControl">
      <label class="col-sm-5 col-md-6">{{ t("netPrice") }}</label>
      <input
        class="form-control readonly"
        v-model="formData.Net_price"
        readonly
        :class="{ error: inputError }"
      />
    </div> -->
      <div class="mb-3 div-for-formControl">
        <label
          v-if="this.formData.vatType === 'VATincluding'"
          class="col-sm-5 col-md-6"
          >{{ t("netPrice") }}</label
        >
        <label v-else class="col-sm-5 col-md-6">{{ t("netPrice") }}</label>
        <input
          v-if="this.formData.vatType === 'VATincluding'"
          class="form-control readonly"
          v-model="formData.Net_price"
          readonly
          :class="{ error: inputError }"
          disabled
          @click="resetError()"
        />
        <input
          v-else
          class="form-control readonly"
          v-model="formData.sale_totalprice"
          readonly
          :class="{ error: inputError }"
          disabled
          @click="resetError()"
        />
      </div>

      <div class="mb-5 div-for-formControl">
        <label class="col-sm-6 col-md-6 label-textarea">{{
          t("quotationRemark")
        }}</label>
        <div class="col-6 col-sm-6 col-md-6">
          <!-- <div class="text-editor col-6 col-sm-6 col-md-6"> -->
          <textarea
            v-model="formData.remark"
            class="form-control"
            maxlength="105"
            rows="3"
            @input="onInput"
          ></textarea>
          <p>
            {{ 105 - (formData.remark ? formData.remark.length : 0) }}
            {{ t("characters") }}
          </p>
          <!-- </div> -->
        </div>
      </div>

      <!-- <div class="mb-5 div-for-formControl-textarea">
        <label class="col-sm-5 col-md-6 label-textarea">ไฟล์แนบ</label>
        <div class="text-editor">
          <img
            :src="imageSrc"
            alt="Preview"
            style="max-width: 200px; max-height: 200px"
          /> -->
      <!-- </div> -->
      <!-- <input class="form-control" v-model="formData.remark"> -->
      <!-- </div> -->
    </div>
    <div class="modal-footer">
      <button
        :disabled="isLoading"
        class="btn btn-primary me-3"
        v-if="isEditMode"
        @click="editInvoice"
      >
        <span
          v-if="isLoading"
          class="spinner-border spinner-border-sm"
          role="status"
          aria-hidden="true"
        ></span>
        <span v-else>{{ t("buttonSave") }}</span>
      </button>
      <button class="btn btn-secondary" @click="closePopup">
        {{ t("buttonCancel") }}
      </button>
    </div>
  </popup>
  <popup :isOpen="isPopupPDFOpen" :closePopup="ClosePDFview">
    <embed :src="pdfUrl" type="application/pdf" width="100%" height="600px" />
    <div class="modal-footer">
      <button class="btn btn-secondary" @click="ClosePDFview">
        {{ t("buttonClose") }}
      </button>
    </div>
  </popup>
  <div class="delete-popup">
    <Popup
      :isOpen="isDeleteConfirmPopupOpen"
      :closePopup="closeDeleteConfirmPopup"
    >
      <div class="mb-5">
        <a>{{ t("deleteConfirmSentence") }}</a>
      </div>
      <div class="modal-footer mb-5">
        <button class="btn btn-danger me-2" @click="deleteInvoice">
          {{ t("buttonDelete") }}
        </button>
        <button class="btn btn-secondary" @click="closeDeleteConfirmPopup">
          {{ t("buttonCancel") }}
        </button>
      </div>
    </Popup>
  </div>
  <div class="delete-popup">
    <Popup
      :isOpen="isAllowConfirmPopupOpen"
      :closePopup="closeAllowConfirmPopup"
    >
      <div class="mb-5">
        <a>{{ t("AllowConfirmSentence") }}</a>
      </div>
      <div class="modal-footer mb-5">
        <button class="btn btn-success me-2" @click="editInvoice2">
          {{ t("issueReceipt") }}
        </button>
        <button class="btn btn-secondary" @click="closeAllowConfirmPopup">
          {{ t("buttonCancel") }}
        </button>
      </div>
    </Popup>
  </div>
  <transition name="fade">
    <div v-if="openPopupAllow" class="popup-overlay">
      <div class="popup-content-custome alert alert-light" role="alert">
        <span v-if="formData.invoice_status === 'Issue a receipt'">
          <i class="fa-solid fa-ban"></i> {{ t("popupCalcelTaxInvoic") }}
        </span>
        <span v-else-if="formData.invoice_status === 'Pending'">
          <i class="fa-solid fa-check"></i> {{ t("popupCreatedTaxInvoic") }}
        </span>
      </div>
    </div>
  </transition>
  <div>
    <div v-if="isPopupVisible" class="popup-success">
      <div class="popup-content-success">
        <a>{{ popupMessage }}</a>
      </div>
    </div>
    <!-- <div v-if="isPopupVisible_error" class="popup-success">
      <div class="popup-content-error">
        <a>{{ popupMessage_error }}</a>
      </div>
    </div> -->
    <div v-if="isPopupVisible_error" class="popup-error2">
      <div class="text-end">
        <button
          type="button"
          class="btn-close"
          aria-label="Close"
          @click="closeErrorPopup"
          style="color: #9f9999"
        ></button>
      </div>
      <div class="popup-content-error2">
        <a>{{ popupMessage_error }}</a>
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
</template>

<script>
import Navigate from "../components/Navigation.vue";
import InvoiceList from "../components/tableList.vue";
import { useI18n } from "vue-i18n";
import Popup from "../components/popup.vue";
import { config } from "../../constant.js";
import DatePicker from "vue-datepicker-next";
import "vue-datepicker-next/index.css";
import jsPDF from "jspdf";
import thSarabunFont from "../font/THSarabunNew-normal.js"; // ฟอนต์ที่แปลงแล้ว
import PromptBold from "../font/Prompt-Bold-bold.js"; // ฟอนต์ที่แปลงแล้ว
import PromptRegular from "../font/Prompt-Regular-normal.js"; // ฟอนต์ที่แปลงแล้ว
import PromptRegularLight from "../font/Prompt-Light-normal.js"; // ฟอนต์ที่แปลงแล้ว
import PromptThinnormal from "../font/Prompt-Thin-normal.js"; // ฟอนต์ที่แปลงแล้ว
import "jspdf-autotable";
import { computed, watch, ref } from "vue";
import moment from "moment";

// ✅ นำเข้า locale ภาษาไทยและอังกฤษ
import th from "vue-datepicker-next/locale/th.es";
import en from "vue-datepicker-next/locale/en.es";

const API_CALL = config["url"];
const accessToken = localStorage.getItem("@accessToken");

export default {
  name: "Invoice",
  components: {
    Navigate,
    InvoiceList,
    Popup,
    DatePicker,
  },
  setup() {
    const { t } = useI18n();
    const { locale } = useI18n();
    // ✅ สร้าง computed property ให้ใช้งาน t("lang") ได้ตลอด
    const lang = computed(() => t("lang"));
    // const currentLocale = computed(() => (locale.value === "th" ? th : en));
    const currentLocale = computed(() => {
      return {
        ...(locale.value === "th" ? th : en), // ✅ ใช้ค่าจาก locale ปัจจุบัน
        yearFormat:
          locale.value === "en" ? moment().year() : moment().year() + 543, // ✅ แปลงปี พ.ศ. หรือ ค.ศ.
      };
    });

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

    const documentName = computed(() => t("dontHaveInvoice"));

    return {
      momentFormat,
      currentLocale,
      locale,
      t,
      documentName,
    };
  },
  data() {
    return {
      openPopupAllow: false,
      shortcutAllow: false,
      isAllowConfirmPopupOpen: false,
      expandedItems: new Set(), // ใช้ Set() แทน isExpanded,
      langString: "th",
      imageSrc: null, // สำหรับเก็บ URL ของรูปภาพที่แสดง
      pdfUrl: null,
      AllBanks: [],
      isDeleteConfirmPopupOpen: false,
      Invoices: [],
      isLoading: false,
      isPopupVisible_error: false,
      inputError: false,
      isPopupVisible: false,
      isPopupOpen: false,
      isPopupPDFOpen: false,
      Business: [],
      Products: [],
      Employees: [],
      productForms: [],
      formData: {
        bus_id: "",
        cus_id: "",
        productID: "",
        employeeID: "",
        employeeName: "",
        position: "",
        employeeEmail: "",
        employeeTel: "",
        cus_name: "",
        cus_address: "",
        cus_tel: "",
        cus_email: "",
        cus_tax: "",
        cus_purchase: "",
        sale_qty: "",
        sale_date: "",
        credit_date_number: "",
        credit_expired_date: "",
        sale_discount: 0,
        sale_totalprice: 0, // total price cal discount and vat
        sale_price: 0, //total price before cal discount but *qty
        total_price: 0, ////////////cal all product
        total_discount: 0, ///////////////
        Net_price: 0, //cal after cal discount
        vat: 0, //price not percentage
        price: "", //price per product
        remark: "",
        invoice_id: "",
        invoice_number: "",
        invoice_status: "",
        invoice_date: "",
        pdfname: "",
        file: "",
      },
      searchQuery: "",
    };
  },
  computed: {
    allExpanded() {
      return this.expandedItems.size === this.Invoices.length; // ถ้าทั้งหมดขยาย ให้ return true
    },
    sale_totalprice() {
      return this.Net_price + this.vat;
    },
    sale_price() {
      return this, price * sale_qty;
    },
    total_price() {
      return this.productForms.reduce((total, form) => {
        return total + form.sale_qty * form.sale_price;
      }, 0);
    },
    total_discount() {
      return this.productForms.reduce((total, form) => {
        return total + parseFloat(form.sale_discount || 0);
      }, 0);
    },
    Net_price() {
      return this.total_price - this.total_discount;
    },
    vat() {
      return (7 / 100) * this.Net_price;
    },
    lines() {
      return this.formData.remark.split("\n");
    },
    tableHeaders() {
      return [
        { label: this.t("invoiceStatusHeaderTable"), key: "invoice_status" },
        { label: this.t("invoiceHeaderTable"), key: "invoice_number" },
        { label: this.t("salenumberHeaderTable"), key: "sale_number" },
        { label: this.t("employeeNameHeaderTable"), key: "employeeName" },
        { label: this.t("cusNameHeaderTable"), key: "cus_name" },
        { label: this.t("cusAddressHeaderTable"), key: "cus_address" },
        { label: this.t("cusTelHeaderTable"), key: "cus_tel" },
        { label: this.t("cusEmailHeaderTable"), key: "cus_email" },
        { label: this.t("cusTaxHeaderTable"), key: "cus_tax" },
        { label: this.t("cusPurchaseHeaderTable"), key: "cus_purchase" },
        { label: this.t("saleTotalpriceHeaderTable"), key: "sale_totalprice" },
        { label: this.t("netpriceHeaderTable"), key: "net_price" },
        { label: this.t("invoiceDateHeaderTable"), key: "invoice_date" },
        { label: this.t("remarkHeaderTable"), key: "remark" },
        { label: this.t("statusHide"), key: "statusHide" },
        // { label: this.t("billingStatusHeaderTable"), key: "billing" },
      ];
    },
    filteredIn() {
      // กรองข้อมูล Categories โดยเปรียบเทียบกับ searchQuery
      let filteredInvoices = [...this.Invoices];
      console.log("filteredInvoices", filteredInvoices);
      if (this.searchQuery.trim()) {
        filteredInvoices = filteredInvoices.filter((inv) =>
          inv["invoice_number"]
            .toLowerCase()
            .includes(this.searchQuery.toLowerCase())
        );
      }

      const monthMapping = {
        Jan: "ม.ค.",
        Feb: "ก.พ.",
        Mar: "มี.ค.",
        Apr: "เม.ย.",
        May: "พ.ค.",
        Jun: "มิ.ย.",
        Jul: "ก.ค.",
        Aug: "ส.ค.",
        Sep: "ก.ย.",
        Oct: "ต.ค.",
        Nov: "พ.ย.",
        Dec: "ธ.ค.",
      };

      const monthMappingTHtoEN = {
        "ม.ค.": "Jan",
        "ก.พ.": "Feb",
        "มี.ค.": "Mar",
        "เม.ย.": "Apr",
        "พ.ค.": "May",
        "มิ.ย.": "Jun",
        "ก.ค.": "Jul",
        "ส.ค.": "Aug",
        "ก.ย.": "Sep",
        "ต.ค.": "Oct",
        "พ.ย.": "Nov",
        "ธ.ค.": "Dec",
      };
      console.log("filteredInvoices ----", filteredInvoices);

      // แปลงค่าของ billing ถ้าภาษาเป็น TH
      if (this.t("headerLang") === "TH") {
        filteredInvoices = filteredInvoices.map((inv) => ({
          ...inv,
          showAllowButton: inv.deleted_at === "" || inv.deleted_at === null, // true ถ้า Pending, false ถ้าไม่ใช่
          invoice_status:
            inv.invoice_status === "Pending"
              ? this.t("PendingLG_invoice")
              : inv.invoice_status === "Issue a receipt"
              ? this.t("issueReceipt_invoice")
              : inv.invoice_status, // ถ้าไม่ตรงเงื่อนไขใด ๆ ใช้ค่าเดิม

          // billing:
          //   inv.billing === "Pending" ? this.t("PendingLG") : inv.billing, // แปลงค่า billing ด้วย
        }));

        // แปลง invoice_date ให้เป็นชื่อเดือนภาษาไทย
        filteredInvoices = filteredInvoices.map((sale) => ({
          ...sale,
          invoice_date: String(sale.invoice_date)
            .replace(
              /\b(Jan|Feb|Mar|Apr|May|Jun|Jul|Aug|Sep|Oct|Nov|Dec)\b/g,
              (match) => monthMapping[match] // ✅ แปลงชื่อเดือนเป็นไทย
            )
            .replace(/(\d{4})/, (match) => (parseInt(match) + 543).toString()), // ✅ เพิ่มปี ค.ศ. + 543 (เป็น พ.ศ.)
        }));
      } else {
        filteredInvoices = filteredInvoices.map((inv) => ({
          ...inv,
          showAllowButton: inv.deleted_at === "" || inv.deleted_at === null, // true ถ้า Pending, false ถ้าไม่ใช่
          invoice_status:
            inv.invoice_status === "Pending"
              ? this.t("PendingLG_invoice")
              : inv.invoice_status === "Issue a receipt"
              ? this.t("issueReceipt_invoice")
              : inv.invoice_status, // ถ้าไม่ตรงเงื่อนไขใด ๆ ใช้ค่าเดิม

          // billing: inv.billing === "Pending" ? "PendingLG" : inv.billing,
        }));
      }
      return filteredInvoices; // ถ้าไม่มีการค้นหาแสดงทั้งหมด
    },
  },
  watch: {
    "formData.sale_qty": function (newQty, oldQty) {
      this.formData.sale_price = newQty * this.formData.price;
    },
    "formData.price": function (newPrice, oldPrice) {
      this.formData.sale_price = this.formData.sale_qty * newPrice;
    },
    formData: {
      deep: true,
      handler(newValue, oldValue) {
        console.log("formData has been updated:", newValue);
      },
    },
  },
  methods: {
    formatDatePicker(date) {
      if (!date) return "";
      const d = new Date(date);
      const day = d.getDate().toString().padStart(2, "0");
      const month = (d.getMonth() + 1).toString().padStart(2, "0");
      const buddhistYear = d.getFullYear() + 543;

      return `${day}/${month}/${buddhistYear}`; // 🔸 แสดงเป็น พ.ศ.
    },
    closeErrorPopup() {
      this.isPopupVisible_error = false;
    },

    resetError(field) {
      this.inputError = false;
      this.isPopupVisible_error = false;
    },
    async handleAllow(row) {
      this.shortcutAllow = true;
      await this.handleEdit(row);
      console.log(this.formData.invoice_status);
      if (
        this.formData.invoice_status === "Pending" ||
        this.formData.invoice_status === "ยังไม่มีใบกํากับภาษี" ||
        this.formData.invoice_status === "Tax Invoice not Issued"
      ) {
        await this.editInvoice2();
        this.openPopupAllow = true;
        setTimeout(() => {
          this.openPopupAllow = false;
        }, 3000);
      } else {
        const saleNumber = this.formData.invoice_id; // "QT-2505080001"
        // await this.deleteTaxInvoice(saleNumber);
        await this.editInvoice_Custome();
        this.openPopupAllow = true;
        setTimeout(() => {
          this.openPopupAllow = false;
        }, 3000);
      }
      this.shortcutAllow = false;
      return false;
      // this.isAllowConfirmPopupOpen = true;
      // this.handleEdit(row);
      // this.isPopupOpen = false;
    },
    closeAllowConfirmPopup() {
      this.isAllowConfirmPopupOpen = false;
    },
    toggleAll() {
      if (this.allExpanded) {
        this.expandedItems.clear(); // ถ้าทุกอันถูกขยาย → ย่อทั้งหมด
      } else {
        this.expandedItems = new Set(this.Invoices.map((q) => q.sale_id)); // ถ้ายังขยายไม่หมด → ขยายทั้งหมด
      }
    },
    isExpanded(id) {
      return this.expandedItems.has(id);
    },
    toggleCollapse(id) {
      if (this.expandedItems.has(id)) {
        this.expandedItems.delete(id); // ถ้ากดซ้ำให้ซ่อน
      } else {
        this.expandedItems.add(id); // ถ้ากดใหม่ให้แสดง
      }
      this.expandedItems = new Set(this.expandedItems); // อัปเดต reactivity
    },
    getProductName(productID) {
      const product = this.Products.find((p) => p.productID === productID);
      return product ? product.productname : "Unknown";
    },
    showPopup_error(message) {
      this.popupMessage_error = message;
      this.isPopupVisible = false;
      this.isPopupVisible_error = true;
      // setTimeout(() => {
      //   this.isPopupVisible = false;
      //   this.isPopupVisible_error = false;
      // }, 2000); // 2 seconds
    },
    showPopup(message) {
      this.popupMessage = message;
      this.isPopupVisible = true;
      setTimeout(() => {
        this.isPopupVisible = false;
      }, 2000); // 2 seconds
    },
    async getQuotationImg(id) {
      const accessToken = localStorage.getItem("@accessToken");
      try {
        const response = await fetch(`${API_CALL}/Quotation/getQuotation_img`, {
          headers: {
            Authorization: `Bearer ${accessToken}`,
          },
        });
        const json = await response.json();

        if (json.statusCode === 200) {
          const quotations = json.data;

          // ใช้ filter เพื่อหาข้อมูลที่ตรงกับ id
          const filteredData = quotations.filter(
            (quotation) => quotation.quotation_id === id
          );

          return filteredData[0].quotation_img; // ส่งคืนรายการแรกหรือ null ถ้าไม่มี
        }
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    },
    calculateNat(discount) {
      const totalNet = this.productForms.reduce((total, form) => {
        const salePrice = parseFloat(form.sale_price.replace(/,/g, "")); // แปลง formatDecimal เป็นตัวเลข
        return total + salePrice;
      }, 0);

      if (discount > totalNet) {
        this.formData.discount_quotation = totalNet;
        this.formData.Net_price = 0;
      } else {
        this.formData.Net_price = totalNet - discount;
      }

      this.updateTotalDiscount();
      this.totalNetPrice();
      this.vat_price();
      this.total_pricesale();
      this.total_priceBeforeDiscount();
    },
    async handleEdit(row) {
      this.getInvoice();
      console.log("Edit item:", row);
      if (this.shortcutAllow === false) {
        this.isPopupOpen = true;
      }
      this.isEditMode = true;
      const formatDateForPicker = (date) => {
        if (!date) return null;
        const d = new Date(date);
        if (isNaN(d.getTime())) return null; // ตรวจสอบว่าเป็นวันที่ถูกต้อง
        return d;
      };

      let formattedInvoice; // ประกาศตัวแปรก่อน

      if (this.t("headerLang") === "TH") {
        const monthMapping = {
          "ม.ค.": 0,
          "ก.พ.": 1,
          "มี.ค.": 2,
          "เม.ย.": 3,
          "พ.ค.": 4,
          "มิ.ย.": 5,
          "ก.ค.": 6,
          "ส.ค.": 7,
          "ก.ย.": 8,
          "ต.ค.": 9,
          "พ.ย.": 10,
          "ธ.ค.": 11,
        };

        // ตรวจสอบว่า row.invoice_date มีค่า และเป็น string หรือไม่
        if (row.invoice_date && typeof row.invoice_date === "string") {
          const match = row.invoice_date.match(
            /(\d{1,2})\s(ม.ค.|ก.พ.|มี.ค.|เม.ย.|พ.ค.|มิ.ย.|ก.ค.|ส.ค.|ก.ย.|ต.ค.|พ.ย.|ธ.ค.)\s(\d{4})/
          );

          if (match) {
            let [_, day, thaiMonth, year] = match;
            let month = monthMapping[thaiMonth];
            formattedInvoice = new Date(year, month, day);
          } else {
            formattedInvoice = null; // ถ้าตรงกับรูปแบบไม่ได้ ให้เป็น null
          }
        } else {
          formattedInvoice = null; // ถ้า invoice_date ไม่มีค่าหรือไม่ใช่ string ให้เป็น null
        }
      } else {
        formattedInvoice = formatDateForPicker(row.invoice_date); // ใช้ฟังก์ชันที่กำหนดเอง
      }

      const quotationData = await this.getQuotationByID(row.sale_id);
      // const quotation_img = await this.getQuotationImg(row.sale_number);

      // if (quotation_img !== "") {
      //   this.imageSrc = quotation_img;
      // }

      // const formattedStart = formatDateForPicker(
      //   quotationData.quotation_start_date
      // );
      // const formattedExpired = formatDateForPicker(
      //   quotationData.quotation_expired_date
      // );

      const filteredInvoice = this.Invoices.filter(
        (inv) => inv.invoice_number === row.invoice_number
      );

      console.log("this.Invoices status:", filteredInvoice[0]?.invoice_status);
      
      // หา ID ให้เจอไม่ว่าจะจาก row หรือ filteredInvoice
      const invoiceIdVal = row.ID || (filteredInvoice[0] ? filteredInvoice[0].ID : null);
      console.log("Invoice ID resolved:", invoiceIdVal, "From row:", row.ID);

      this.formData = {
        employeeID: row.employeeID,
        employeeName: row.employeeName,
        cus_id: row.cus_id,
        cus_name: row.cus_name,
        cus_address: row.cus_address,
        cus_tel: row.cus_tel,
        cus_email: row.cus_email,
        cus_tax: row.cus_tax,
        cus_purchase: row.cus_purchase,
        sale_totalprice: row.sale_totalprice,
        remark: row.remark || "", 
        invoice_id: invoiceIdVal,
        invoice_number: row.invoice_number,
        invoice_status: filteredInvoice[0] ? filteredInvoice[0].invoice_status : row.invoice_status,
        // invoice_date: formattedInvoice,
        invoice_date: filteredInvoice[0] ? formatDateForPicker(filteredInvoice[0].invoice_date) : formatDateForPicker(row.invoice_date),
        discount_quotation: quotationData ? quotationData.discount_quotation : 0,
        vatType: quotationData ? quotationData.vatType : "VATexcluding",
      };

      this.productForms = (row.productForms || []).map((detail) => {
        const selectedProduct = this.Products.find(
          (product) => product.productID === detail.productID
        );
        let price = 0;
        let productname = "";
        console.log("selectedProduct", detail);
        if (selectedProduct) {
          price = this.formatDecimal(
            parseFloat(selectedProduct.price.toFixed(2))
          );
          productname = selectedProduct.productname;
        }

        const salePrice = detail.sale_qty * parseFloat(price.replace(/,/g, ""));
        let saleDiscount = detail.sale_discount;
        if (detail.discounttype === "percent") {
          saleDiscount = (detail.sale_discount / 100) * salePrice;
        }
        return {
          productID: detail.productID,
          price: price,
          productname: productname,
          sale_qty: detail.sale_qty,
          sale_price: this.formatDecimal(salePrice - saleDiscount),
          sale_discount: detail.sale_discount,
          discountType: detail.discounttype,
          product_detail: detail.product_detail,
          pro_unti: detail.pro_unti,
        };
      });
      this.updateTotalDiscount();
      this.totalNetPrice();
      // this.vat_price();
      this.total_pricesale();
      this.total_priceBeforeDiscount();

      // if (this.formData.vatType === "VATincluding") {
      //   this.formData.sale_totalprice = this.formatDecimal(
      //     parseFloat(this.formData.Net_price.replace(/,/g, "")) / 1.07
      //   );
      //   this.formData.vat = this.formatDecimal(
      //     parseFloat(this.formData.Net_price.replace(/,/g, "")) -
      //       parseFloat(this.formData.sale_totalprice.replace(/,/g, ""))
      //   );
      // } else {
      //   this.formData.vat = this.formatDecimal(
      //     parseFloat(this.formData.Net_price.replace(/,/g, "")) * 0.07
      //   );
      //   this.formData.sale_totalprice = this.formatDecimal(
      //     parseFloat(this.formData.Net_price.replace(/,/g, "")) +
      //       parseFloat(this.formData.vat)
      //   );
      // }
      // this.calculateNat(quotationData.discount_quotation);

      if (this.formData.vatType === "VATincluding") {
        this.formData.sale_totalprice = this.formatDecimal(
          parseFloat(this.formData.Net_price.replace(/,/g, "")) / 1.07
        );
        this.formData.vat = this.formatDecimal(
          parseFloat(this.formData.Net_price.replace(/,/g, "")) -
            parseFloat(this.formData.sale_totalprice.replace(/,/g, ""))
        );
      } else {
        this.formData.vat = this.formatDecimal(
          parseFloat(this.formData.Net_price.replace(/,/g, "")) * 0.07
        );
        this.formData.sale_totalprice = this.formatDecimal(
          parseFloat(this.formData.Net_price.replace(/,/g, "")) +
            parseFloat(this.formData.vat.replace(/,/g, ""))
        );
      }
    },
    handleDelete(item) {
      // console.log("-->", item);
      this.isDeleteConfirmPopupOpen = true;
      this.formData = {
        test: 0,
        invoice_id: item.ID,
        sale_id: item.sale_id,
      };
    },
    closeDeleteConfirmPopup() {
      this.isDeleteConfirmPopupOpen = false;
    },
    formatDate(date) {
      const d = new Date(date);
      const day = String(d.getDate()).padStart(2, "0");
      const month = String(d.getMonth() + 1).padStart(2, "0");
      const year = d.getFullYear();
      return `${day}/${month}/${year}`;
    },
    formatDecimal(value) {
      const floatValue = parseFloat(value);
      if (Math.round(floatValue) === floatValue) {
        return floatValue.toFixed(2).replace(/\d(?=(\d{3})+\.)/g, "$&,");
      }
      return floatValue
        .toFixed(2)
        .toString()
        .replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    },
    updateTotalDiscount() {
      const totalDiscount = this.productForms.reduce((total, form) => {
        if (form.sale_discount === "") {
          return 0;
        }
        const saleDiscount = parseFloat(form.sale_discount);
        return total + saleDiscount;
      }, 0);
      this.formData.total_discount = this.formatDecimal(totalDiscount);
    },
    totalNetPrice() {
      const totalNet = this.productForms.reduce((total, form) => {
        const salePrice = parseFloat(form.sale_price.replace(/,/g, "")); // แปลง formatDecimal เป็นตัวเลข
        return total + salePrice;
      }, 0);
      const saleDiscount = this.formData.total_discount.replace(/,/g, "");
      // this.formData.Net_price = this.formatDecimal(totalNet - saleDiscount);
      this.formData.Net_price = this.formatDecimal(
        parseFloat(totalNet) - parseFloat(this.formData.discount_quotation)
      );
    },
    vat_price() {
      // this.formData.vat = this.formatDecimal(
      //   (7 / 100) * parseFloat(this.formData.Net_price.replace(/,/g, ""))
      // );
    },
    total_priceBeforeDiscount() {
      const totalNet = this.productForms.reduce((total, form) => {
        const salePrice = parseFloat(form.sale_price.replace(/,/g, "")); // แปลง formatDecimal เป็นตัวเลข
        return total + salePrice;
      }, 0);
      this.formData.total_price = this.formatDecimal(totalNet);
    },
    total_pricesale() {
      // const vat = parseFloat(this.formData.vat.replace(/,/g, ""));
      // const net = parseFloat(this.formData.Net_price.replace(/,/g, ""));
      // this.formData.sale_totalprice = this.formatDecimal(net + vat);
    },
    handlePreview(row) {
      console.log("Preview item:", row);
      this.viewformpdf("view", row);
      // this.isPopupPDFOpen = true;
      this.isAddingMode = false;
      this.isEditMode = true;

      const formatDateForPicker = (date) => {
        if (!date) return null;
        const d = new Date(date);
        if (isNaN(d.getTime())) return null; // ตรวจสอบว่าเป็นวันที่ถูกต้อง
        return d;
      };
      const formattedInvoice = formatDateForPicker(row.invoice_date);

      this.formData = {
        employeeID: row.employeeID,
        employeeName: row.employeeName,
        cus_id: row.cus_id,
        cus_name: row.cus_name,
        cus_address: row.cus_address,
        cus_tel: row.cus_tel,
        cus_email: row.cus_email,
        cus_tax: row.cus_tax,
        cus_purchase: row.cus_purchase,
        sale_totalprice: row.sale_totalprice,
        remark: row.remark || "", // จัดการค่า remark ให้เป็น string ว่างถ้าเป็น null

        invoice_id: row.ID,
        invoice_number: row.invoice_number,
        invoice_status: row.invoice_status,
        invoice_date: formattedInvoice,
      };

      this.productForms = (row.details || []).map((detail) => {
        const selectedProduct = this.Products.find(
          (product) => product.productID === detail.productID
        );
        let price = 0;
        if (selectedProduct) {
          price = this.formatDecimal(
            parseFloat(selectedProduct.price.toFixed(2))
          );
        }

        const salePrice = detail.sale_qty * parseFloat(price.replace(/,/g, ""));
        let saleDiscount = detail.sale_discount;
        if (detail.discountType === "percent") {
          saleDiscount = (detail.sale_discount / 100) * salePrice;
        }

        return {
          productID: detail.productID,
          price: price,
          sale_qty: detail.sale_qty,
          sale_price: this.formatDecimal(salePrice - saleDiscount),
          sale_discount: saleDiscount,
          discountType: detail.discountType || "amount",
        };
      });
      this.updateTotalDiscount();
      this.totalNetPrice();
      this.vat_price();
      this.total_pricesale();
      this.total_priceBeforeDiscount();
    },
    handleDownload(row) {
      console.log("Edit item:", row);
      this.viewformpdf("download", row);
      const formatDateForPicker = (date) => {
        if (!date) return null;
        const d = new Date(date);
        if (isNaN(d.getTime())) return null; // ตรวจสอบว่าเป็นวันที่ถูกต้อง
        return d;
      };
      const formattedInvoice = formatDateForPicker(row.invoice_date);

      this.formData = {
        employeeID: row.employeeID,
        employeeName: row.employeeName,
        cus_id: row.cus_id,
        cus_name: row.cus_name,
        cus_address: row.cus_address,
        cus_tel: row.cus_tel,
        cus_email: row.cus_email,
        cus_tax: row.cus_tax,
        cus_purchase: row.cus_purchase,
        sale_totalprice: row.sale_totalprice,
        remark: row.remark || "", // จัดการค่า remark ให้เป็น string ว่างถ้าเป็น null

        invoice_id: row.ID,
        invoice_number: row.invoice_number,
        invoice_status: row.invoice_status,
        invoice_date: formattedInvoice,
      };
      this.productForms = (row.details || []).map((detail) => {
        const selectedProduct = this.Products.find(
          (product) => product.productID === detail.productID
        );
        let price = 0;
        if (selectedProduct) {
          price = this.formatDecimal(
            parseFloat(selectedProduct.price.toFixed(2))
          );
        }
        const salePrice = detail.sale_qty * parseFloat(price.replace(/,/g, ""));
        let saleDiscount = detail.sale_discount;
        if (detail.discountType === "percent") {
          saleDiscount = (detail.sale_discount / 100) * salePrice;
        }

        return {
          productID: detail.productID,
          price: price,
          sale_qty: detail.sale_qty,
          sale_price: this.formatDecimal(salePrice - saleDiscount),
          sale_discount: saleDiscount,
          discountType: detail.discountType || "amount",
        };
      });

      this.updateTotalDiscount();
      this.totalNetPrice();
      this.vat_price();
      this.total_pricesale();
      this.total_priceBeforeDiscount();
    },
    async getBanks() {
      const accessToken = localStorage.getItem("@accessToken");
      this.isLoading = true;
      try {
        const response = await fetch(`${API_CALL}/Quotation/getBank`, {
          method: "GET", // เปลี่ยน method เป็น 'POST'
          headers: {
            Authorization: `Bearer ${accessToken}`,
            "Content-Type": "application/json", // เพิ่ม header Content-Type
          },
        });
        const json = await response.json();

        if (json.statusCode === 200) {
          this.AllBanks = json.data;
        } else {
          this.showPopup_error(json.data);
          console.log("Employee ", json);
        }
      } catch (error) {
        console.error("Error fetching data:", error);
      } finally {
        this.isLoading = false;
      }
    },
    async viewformpdf(action, row) {
      // console.log("Preview Item: ", row)
      const productForms = row.productForms || [];
      const doc = new jsPDF();

      const quotationData = await this.getQuotationByID(row.sale_id);
      console.log(">>>>>>>>>>>>>>>>>", quotationData);

      const filteredInvoicesPDF = this.Invoices.filter(
        (invoice) => invoice.sale_id === row.sale_id
      );
      // console.log("filteredInvoicesPDF", filteredInvoicesPDF[0].invoice_date);

      const formatDate = { day: "2-digit", month: "short", year: "numeric" };
      // const Qdate = new Date(row.sale_date);
      // const Quo_date = Qdate.toLocaleDateString("en-GB", formatDate);
      const Qdate = new Date(filteredInvoicesPDF[0].invoice_date);
      const Quo_date = Qdate.toLocaleDateString("en-GB", formatDate);

      const Expdate = new Date(row.credit_expired_date);
      const ExpiredDate = Expdate.toLocaleDateString("en-GB", formatDate);
      // const Qdate = new Date(row.invoice_date);

      const IN_date = Quo_date;
      //header of prodyct table
      const headerText = [
        "No.",
        "Image",
        "Product Name",
        "Qty",
        "Unit price",
        "Discount",
        "Total Price",
      ];

      const tableData = productForms.map((form, index) => {
        // const product = this.Products.find(
        //   (p) => p.productID === form.productID.toString()
        // );
        const product = this.Products.find(
          (product) => product.productID === form.productID
        );
        // console.log(product);
        return [
          index + 1,
          product && product.productImg ? product.productImg : "---", // ดึงรูปภาพสินค้าถ้ามี
          product.productname +
            (form.product_detail ? "\n" + form.product_detail : ""),
          form.sale_qty,
          this.formatDecimal(product ? product.price : ""),
          this.formatDecimal(form.sale_discount),
          this.formatDecimal(form.sale_price),
        ];
      });

      console.log("Table:", tableData);

      const startY = 85;
      const margin = 10;
      const lineHeight = 15;
      const pageSize = 6;
      const pageCount = Math.ceil(tableData.length / pageSize);

      for (let i = 0; i < pageCount; i++) {
        const currentPageData = tableData.slice(
          i * pageSize,
          (i + 1) * pageSize
        );

        // doc.addPage();
        if (i > 0) {
          // ตรวจสอบหน้า PDF ที่ไม่ใช่หน้าแรกหรือ action เป็น 'download'
          doc.addPage();
        }

        doc.setTextColor(0, 0, 0);

        doc.setFontSize(16);

        doc.addFileToVFS("Prompt-Bold.ttf", PromptBold);
        doc.addFont("Prompt-Bold.ttf", "PromptBold", "bold");
        // ตั้งค่าฟอนต์ที่ต้องการ
        doc.setTextColor(0, 0, 0); // ตั้งสีดำสนิท (RGB 0,0,0)
        doc.setFont("PromptBold", "bold");
        doc.setFontSize(20);
        doc.text("Invoice", 175, 40);

        doc.addFileToVFS("Prompt-Regular.ttf", PromptRegular);
        doc.addFont("Prompt-Regular.ttf", "PromptRegular", "normal");
        doc.setFontSize(16);

        // ตั้งค่าฟอนต์ที่ต้องการ
        doc.setFont("PromptRegular", "normal");

        //{text, x,y}
        doc.text(`${this.Business.bus_name}`, 10, 12);
        // console.log("this.Business", this.Business.bank_id);

        const bank_detail = this.AllBanks.filter(
          (bank) => bank.bank_id === this.Business.bank_id
        );
        // console.log("bank_detail", bank_detail);

        const FormBank = [
          `${bank_detail[0].bank_name}`,
          `${bank_detail[0].bank_account}`,
          `${bank_detail[0].bank_number}`,
        ];
        doc.setFontSize(14);
        doc.text(FormBank, 10, 275, {
          align: "left",
          valign: "left",
          lineGap: 8,
        });

        doc.setFont("helvetica", "normal");
        doc.setFontSize(12);
        doc.setTextColor(0, 0, 0);
        // เพิ่มฟอนต์
        doc.addFileToVFS("Prompt-RegularLight.ttf", PromptRegularLight);
        doc.addFont("Prompt-RegularLight.ttf", "PromptRegularLight", "normal");
        doc.setFontSize(10);

        // ตั้งค่าฟอนต์ที่ต้องการ
        doc.setFont("PromptRegularLight", "normal");
        doc.text(
          `${this.Business.bus_name} ${this.Business.bus_website}`,
          10,
          19
        );
        doc.text(`${this.Business.bus_address}`, 10, 24);
        doc.text(`Tax ID.${this.Business.bus_tax}`, 10, 29);
        doc.text(`Tel.${this.Business.bus_tel}`, 10, 34);
        doc.text("Signature", 110, 265);
        doc.text("Name", 110, 275);
        doc.text("Position", 110, 285);

        // //line width
        // doc.setLineWidth(0.2);
        // //[start(x,y), end(x,y)]
        // doc.line(130, 265, 200, 265);
        // doc.line(130, 275, 200, 275);
        // doc.line(130, 285, 200, 285);

        // doc.setLineWidth(0.5);
        // doc.line(10, 35, 120, 35);
        // doc.line(10, 72, 200, 72);
        // doc.line(10, 210, 200, 210);
        // doc.line(10, 250, 200, 250);

        doc.setFontSize(10);
        doc.setTextColor(0, 0, 0);

        doc.text(`Customer Name: ` + row.cus_name, 10, 50);
        doc.text(`Address: ` + row.cus_address, 10, 55);
        doc.text(`Contact: ` + row.cus_tel, 10, 60);
        doc.text(`E-mail: ` + row.cus_email, 10, 65);
        doc.text(`Contact No: ` + row.cus_tel, 10, 70);
        doc.text(`Tax ID: ` + row.cus_tax, 10, 75);

        // const startX_header = 130; // ตำแหน่งเริ่มต้นของข้อความ
        // const valueX_header = 175; // ตำแหน่งค่าของตัวแปร (จัดให้อยู่แนวเดียวกัน)
        // const startY_header = 50; // จุดเริ่มต้นแนวตั้ง
        // const lineHeight_header = 5; // ระยะห่างระหว่างบรรทัด
        // const lineHeight_header2 = 10; // ระยะห่างระหว่างบรรทัด
        // // doc.text(`Date:     ${IN_date}`, 150, 55);
        // // doc.text(`Number: ${row.invoice_number}`, 150, 50);

        // doc.text(`Date:`, startX_header, startY_header);
        // doc.text(`${IN_date}`, valueX_header + 5.5, startY_header);

        // doc.text(
        //   `Invoice Number:`,
        //   startX_header,
        //   startY_header + lineHeight_header
        // );
        // doc.text(
        //   `${row.invoice_number}`,
        //   valueX_header + 2,
        //   startY_header + lineHeight_header
        // );

        // doc.text(
        //   `Quotation​ Number​:`,
        //   startX_header,
        //   startY_header + lineHeight_header2
        // );
        // doc.text(
        //   `${row.quotation_num}`,
        //   valueX_header,
        //   startY_header + lineHeight_header2
        // );
        // ===== Header Section (จัดระนาบ) =====
        const startX_header_custom = 130;
        const valueRightAlignX_custom = 200;
        const startY_header_custom = 50;
        const lineHeight_custom = 5;

        let currentY = startY_header_custom;

        function drawAlignedRow(label, value) {
          doc.text(label, startX_header_custom, currentY);

          const textWidth = doc.getTextWidth(value);
          doc.text(value, valueRightAlignX_custom - textWidth, currentY);

          currentY += lineHeight_custom;
        }

        // แสดงเฉพาะถึงบรรทัดนี้เท่านั้น
        drawAlignedRow("Date:", IN_date);
        drawAlignedRow("Invoice Number:", row.invoice_number);
        drawAlignedRow("Quotation Number:", row.quotation_num);

        // const FormCustomer = [
        // `${row.cus_name}`,
        // `${row.cus_address}`,
        // `${row.cus_purchase}`,
        // `${row.cus_email}`,
        // `${row.cus_tel}`,
        // `${row.cus_tax}`,
        // ];
        // doc.text(FormCustomer, 50, 55, {
        //   align: "left",
        //   valign: "middle",
        //   lineGap: 5,
        // });

        // doc.text(
        //   `Payment: ${row.credit_date_number} days     Valid until: ${ExpiredDate}`,
        //   200,
        //   70,
        //   { align: "right", valign: "middle" }
        // );
        // doc.text(`Payment:          ${row.credit_date_number} Day`, 150, 70);
        // doc.text(`Valid until:  ${ExpiredDate}`, 150, 75);

        doc.addFileToVFS("Prompt-RegularLight.ttf", PromptRegularLight);
        doc.addFont("Prompt-RegularLight.ttf", "PromptRegularLight", "normal");
        doc.setFontSize(10);

        // ตั้งค่าฟอนต์ที่ต้องการ
        doc.setFont("PromptRegularLight", "normal");
        const employ = this.Employees.find(
          (p) => p.employeeID === row.employeeID
        );

        // doc.text(`${employ.position}`, 10, 255);
        doc.text(`Name: `, 10, 255);
        doc.text(row.employeeName, 40, 255);
        doc.text(`Email: `, 10, 260);
        doc.text(employ.Email, 40, 260);
        doc.text(`Contact No.: `, 10, 265);
        doc.text(employ.Phone_num, 40, 265);
        doc.text(`Remark: `, 10, 215);

        // const FormEmployee_sale = [
        //   `${row.employeeName}`,
        //   // `${employ.Email}`,
        //   // `${employ.Phone_num}`,
        // ];
        // doc.text(FormEmployee_sale, 40, 255, {
        //   align: "left",
        //   valign: "middle",
        //   lineGap: 5,
        // });

        // doc.text(`Total Before Discount: `, 130, 215);
        // doc.text(`Total Before Vat: `, 130, 219);
        // doc.text(`Discount: `, 130, 223);
        // doc.text(`Vat 7%: `, 130, 227);
        // doc.text(`Net Price:  `, 130, 231);
        doc.text(`Total Before Discount: `, 130, 215);
        doc.text(`Total Before VAT: `, 130, 220);
        doc.text(`Discount: `, 130, 225);
        doc.text(`VAT 7%: `, 130, 230);
        doc.text(`Net Price:  `, 130, 235);

        const total_price = productForms.reduce((total, form) => {
          return total + parseFloat(form.sale_price);
        }, 0);

        const total_discount = productForms.reduce((total, form) => {
          return total + parseFloat(form.sale_discount || 0);
        }, 0);

        const net_price = total_price - row.discount_quotation;

        const vat = (7 / 100) * net_price;

        // const discount_pdf =
        //   parseFloat(net_price) - parseFloat(row.discount_quotation);
        // alert(discount_pdf);
        let netCal = this.formatDecimal(total_price * 0.07);
        let sale_data = this.formatDecimal(total_price + netCal);
        // alert(netCal);
        // const FormTotalprice = [];

        if (quotationData.vatType === "VATincluding") {
          //           this.formData.sale_totalprice = this.formatDecimal(
          //   parseFloat(this.formData.Net_price.replace(/,/g, "")) / 1.07
          // );
          // this.formData.vat = this.formatDecimal(
          //   parseFloat(this.formData.total_price.replace(/,/g, "")) -
          //     parseFloat(this.formData.sale_totalprice.replace(/,/g, ""))
          // );
          let FormTotalprice = [
            `${this.formatDecimal(total_price)}`,
            `${(
              this.formatDecimal(
                total_price - quotationData.discount_quotation
              ) / 1.07
            ).toFixed(2)}`,
            `${this.formatDecimal(quotationData.discount_quotation)}`,
            `${this.formatDecimal(
              total_price -
                quotationData.discount_quotation -
                (total_price - quotationData.discount_quotation) / 1.07
            )}`,
            `${this.formatDecimal(
              total_price - quotationData.discount_quotation
            )}`,
          ];
          // doc.text(FormTotalprice, 200, 215, {
          //   align: "right",
          //   valign: "middle",
          //   lineGap: 5,
          // });
          let startY = 215;
          const lineSpacing = 5; // ระยะห่างระหว่างบรรทัด

          FormTotalprice.forEach((line) => {
            doc.text(line, 200, startY, {
              align: "right",
              valign: "middle",
            });
            startY += lineSpacing; // เพิ่มพิกัด Y สำหรับแต่ละบรรทัด
          });
        } else {
          let FormTotalprice = [
            `${this.formatDecimal(total_price)}`,
            `${this.formatDecimal(
              total_price - quotationData.discount_quotation
            )}`,
            `${this.formatDecimal(quotationData.discount_quotation)}`,
            `${this.formatDecimal(
              (total_price - quotationData.discount_quotation) * 0.07
            )}`,
            `${this.formatDecimal(
              total_price -
                quotationData.discount_quotation +
                (total_price - quotationData.discount_quotation) * 0.07
            )}`,
          ];
          // doc.text(FormTotalprice, 200, 215, {
          //   align: "right",
          //   valign: "middle",
          //   lineGap: 5,
          // });
          let startY = 215;
          const lineSpacing = 5; // ระยะห่างระหว่างบรรทัด

          FormTotalprice.forEach((line) => {
            doc.text(line, 200, startY, {
              align: "right",
              valign: "middle",
            });
            startY += lineSpacing; // เพิ่มพิกัด Y สำหรับแต่ละบรรทัด
          });
        }

        doc.addFileToVFS("Prompt-Regular.ttf", PromptRegular);
        doc.addFont("Prompt-Regular.ttf", "PromptRegular", "normal");
        doc.setFontSize(16);

        doc.setLineWidth(0.5);
        doc.line(10, 210, 200, 210);
        doc.line(10, 238, 200, 238);
        doc.line(130, 265, 200, 265);
        doc.line(130, 275, 200, 275);
        doc.line(130, 285, 200, 285);

        // ตั้งค่าฟอนต์ที่ต้องการ
        doc.setFont("PromptRegular", "normal");
        doc.text("Receiver", 110, 250);
        doc.text("Buyer", 10, 42);
        doc.setFontSize(14);
        doc.text(`Sales Person`, 10, 248);
        // เพิ่มฟอนต์
        doc.addFileToVFS("Prompt-RegularLight.ttf", PromptRegularLight);
        doc.addFont("Prompt-RegularLight.ttf", "PromptRegularLight", "normal");
        doc.setFontSize(10);

        const maxWidth = 90;
        const maxWidth2 = 20;
        const maxHeight = 20;

        // ตั้งค่าฟอนต์ที่ต้องการ
        doc.setFont("PromptRegularLight", "normal");
        //set image (image , x,y,w,h)

        if (this.Business.bus_logo != null) {
          const img = new Image();
          img.src = `${this.Business.bus_logo}`;

          await new Promise((resolve) => {
            img.onload = resolve;
          });

          const maxBoxWidth = 35;
          const maxBoxHeight = 20;

          let imgWidth = img.width;
          let imgHeight = img.height;

          // คำนวณสัดส่วนให้ fit ภายในกล่อง 20x20 โดยไม่เสียสัดส่วน
          const widthRatio = maxBoxWidth / imgWidth;
          const heightRatio = maxBoxHeight / imgHeight;
          const scaleRatio = Math.min(widthRatio, heightRatio);

          // คำนวณขนาดใหม่ที่ fit
          const scaledWidth = imgWidth * scaleRatio;
          const scaledHeight = imgHeight * scaleRatio;

          // พิกัดกรอบบนขวา
          const boxX = doc.internal.pageSize.width - 10 - maxBoxWidth; // ขอบขวา - margin - boxWidth
          const boxY = 12; // ขอบบน

          // วางรูปในกรอบ โดยชิดขวา-บนของกล่อง (แต่ต้องเว้นให้ไม่เกินขอบ)
          const imgX = boxX + (maxBoxWidth - scaledWidth); // ชิดขวาภายในกรอบ
          const imgY = boxY; // ชิดบน

          // วาดภาพ
          doc.addImage(img, "JPEG", imgX, imgY, scaledWidth, scaledHeight);
        }

        // doc.addImage(`${this.Business.bus_logo}`, "JPEG", 165, 12, 20, 20);

        doc.setFont("helvetica", "normal");
        // เพิ่มฟอนต์
        // doc.addFileToVFS("THSarabunNew-normal.ttf", thSarabunFont);
        // doc.addFont("THSarabunNew-normal.ttf", "THSarabunNew", "normal");

        // // ตั้งค่าฟอนต์ที่ต้องการ
        // doc.setFont("THSarabunNew", "normal");
        doc.addFileToVFS("Prompt-RegularLight.ttf", PromptRegularLight);
        doc.addFont("Prompt-RegularLight.ttf", "PromptRegularLight", "normal");
        doc.setFontSize(10);

        // ตั้งค่าฟอนต์ที่ต้องการ
        doc.setFont("PromptRegularLight", "normal");

        this.shortcutAllow = true;
        await this.handleEdit(row);
        // doc.text(`${row.remark}`, 40, 235);
        doc.text(`${row.remark}`, 40, 215, { maxWidth });
        this.drawHeader(doc, headerText, startY, margin);
        this.drawTable(doc, currentPageData, startY, margin, lineHeight);
      }

      if (action === "view") {
        const pdfBlob = doc.output("blob");
        const pdfUrl = URL.createObjectURL(pdfBlob);
        this.pdfUrl = pdfUrl;
        // this.isPopupPDFOpen = true;
        // เปิดในแท็บใหม่
        window.open(pdfUrl, "_blank");
      } else if (action === "download") {
        doc.save(`quotation-${row.cus_name}-${row.sale_number}.pdf`);
      }
      this.shortcutAllow = false;
    },
    // async viewformpdf(action, row) {
    //   console.log("Preview Item: ", row);
    //   const productForms = row.productForms || [];
    //   const doc = new jsPDF();
    //   const formatDate = { day: "2-digit", month: "short", year: "numeric" };
    //   const Qdate = new Date(row.invoice_date);
    //   const IN_date = Qdate.toLocaleDateString("en-GB", formatDate);

    //   const Expdate = new Date(row.credit_expired_date);
    //   const ExpiredDate = Expdate.toLocaleDateString("en-GB", formatDate);

    //   const headerText = [
    //     "No.",
    //     "Image",
    //     "Description",
    //     "Qty",
    //     "Unit price",
    //     "Discount",
    //     "Amount",
    //   ];

    //   const tableData = productForms.map((form, index) => {
    //     // const product = this.Products.find(
    //     //   (p) => p.productID === form.productID
    //     // );
    //     const product = this.Products.find(
    //       (product) => product.productname === form.productID
    //     );
    //     return [
    //       index + 1,
    //       product ? product.productImg : "", // ดึงรูปภาพสินค้าถ้ามี
    //       product ? product.productname : "", // ดึงชื่อสินค้าถ้ามี
    //       form.sale_qty,
    //       this.formatDecimal(product ? product.price : ""),
    //       this.formatDecimal(form.sale_discount),
    //       this.formatDecimal(form.sale_price),
    //     ];
    //   });

    //   console.log("Table:", tableData);

    //   const startY = 77;
    //   const margin = 10;
    //   const lineHeight = 15;
    //   const pageSize = 6;
    //   const pageCount = Math.ceil(tableData.length / pageSize);

    //   for (let i = 0; i < pageCount; i++) {
    //     const currentPageData = tableData.slice(
    //       i * pageSize,
    //       (i + 1) * pageSize
    //     );

    //     if (i > 0) {
    //       // ตรวจสอบหน้า PDF ที่ไม่ใช่หน้าแรกหรือ action เป็น 'download'
    //       doc.addPage();
    //     }
    //     //

    //     doc.setFont("helvetica", "bold");
    //     doc.setFontSize(18);
    //     doc.setTextColor(0, 0, 0);

    //     // เพิ่มฟอนต์
    //     doc.addFileToVFS("THSarabunNew-normal.ttf", thSarabunFont);
    //     doc.addFont("THSarabunNew-normal.ttf", "THSarabunNew", "normal");

    //     // ตั้งค่าฟอนต์ที่ต้องการ
    //     doc.setFont("THSarabunNew", "normal");
    //     doc.text(`${this.Business.bus_name}`, 10, 15);
    //     doc.text("Invoice", 160, 40);

    //     const bank_detail = this.AllBanks.filter(
    //       (bank) => bank.bank_id === this.Business.bank_id
    //     );

    //     const FormBank = [
    //       `${bank_detail[0].bank_name}`,
    //       `${bank_detail[0].bank_account}`,
    //       `${bank_detail[0].bank_number}`,
    //     ];

    //     // const FormBank = [
    //     //   `${this.Business.banks[0].bank_name}`,
    //     //   `${this.Business.banks[0].bank_account}`,
    //     //   `${this.Business.banks[0].bank_number}`,
    //     // ];
    //     doc.text(FormBank, 30, 263, {
    //       align: "center",
    //       valign: "middle",
    //       lineGap: 8,
    //     });

    //     doc.setFont("helvetica", "normal");
    //     doc.setFontSize(12);
    //     doc.setTextColor(0, 0, 0);
    //     // เพิ่มฟอนต์
    //     doc.addFileToVFS("THSarabunNew-normal.ttf", thSarabunFont);
    //     doc.addFont("THSarabunNew-normal.ttf", "THSarabunNew", "normal");

    //     // ตั้งค่าฟอนต์ที่ต้องการ
    //     doc.setFont("THSarabunNew", "normal");
    //     doc.text(
    //       `${this.Business.bus_name} ${this.Business.bus_website}`,
    //       10,
    //       23
    //     );
    //     doc.text(`${this.Business.bus_address}`, 10, 28);
    //     doc.text(
    //       `Tax ID.${this.Business.bus_tax}  Tel.${this.Business.bus_tel}`,
    //       10,
    //       33
    //     );
    //     doc.text("Signature", 100, 265);
    //     doc.text("Name", 100, 275);
    //     doc.text("Position", 100, 285);

    //     doc.setLineWidth(0.2);
    //     doc.line(130, 265, 200, 265);
    //     doc.line(130, 275, 200, 275);
    //     doc.line(130, 285, 200, 285);

    //     doc.setLineWidth(0.5);
    //     doc.line(10, 35, 120, 35);
    //     doc.line(10, 72, 200, 72);
    //     doc.line(10, 210, 200, 210);
    //     doc.line(10, 250, 200, 250);

    //     doc.setFontSize(12);
    //     doc.setTextColor(0, 0, 0);

    //     doc.text(`Customer Name: `, 10, 45);
    //     doc.text(`Number: ${row.invoice_number}`, 150, 45);
    //     doc.text(`Address: `, 10, 50);
    //     doc.text(`Date: ${IN_date}`, 150, 50);
    //     doc.text(`Contact: `, 10, 55);
    //     doc.text(`E-mail: `, 10, 60);
    //     doc.text(`Contact No: `, 10, 65);
    //     doc.text(`Tax ID: `, 10, 70);

    //     const FormCustomer = [
    //       `${row.cus_name}`,
    //       `${row.cus_address}`,
    //       `${row.cus_purchase}`,
    //       `${row.cus_email}`,
    //       `${row.cus_tel}`,
    //       `${row.cus_tax}`,
    //     ];
    //     doc.text(FormCustomer, 50, 45, {
    //       align: "left",
    //       valign: "middle",
    //       lineGap: 5,
    //     });

    //     const employ = this.Employees.find(
    //       (p) => p.employeeID === row.employeeID
    //     );

    //     doc.text(`${employ.position}`, 10, 220);
    //     doc.text(`Email: `, 10, 225);
    //     doc.text(`Contact No.: `, 10, 230);
    //     doc.text(`Remark: `, 10, 235);

    //     const FormEmployee_sale = [
    //       `${row.employeeName}`,
    //       `${employ.Email}`,
    //       `${employ.Phone_num}`,
    //     ];
    //     doc.text(FormEmployee_sale, 40, 220, {
    //       align: "left",
    //       valign: "middle",
    //       lineGap: 5,
    //     });

    //     doc.text(`Total Before Vat: `, 130, 215);
    //     doc.text(`Discount: `, 130, 220);
    //     doc.text(`Net Price: `, 130, 225);
    //     doc.text(`Vat 7%: `, 130, 230);
    //     doc.text(`Total Amount: `, 130, 235);

    //     const total_price = productForms.reduce((total, form) => {
    //       return total + parseFloat(form.sale_price);
    //     }, 0);

    //     const total_discount = productForms.reduce((total, form) => {
    //       return total + parseFloat(form.sale_discount || 0);
    //     }, 0);

    //     const net_price = total_price - total_discount;

    //     const vat = (7 / 100) * net_price;

    //     const sale_totalprice = net_price + vat;
    //     const quotationData = await this.getQuotationByID(row.sale_id);

    //     // const discount_pdf =
    //     //   parseFloat(net_price) - parseFloat(row.discount_quotation);
    //     // alert(discount_pdf);
    //     let netCal = this.formatDecimal(total_price * 0.07);
    //     let sale_data = this.formatDecimal(total_price + netCal);
    //     // alert(netCal);
    //     // const FormTotalprice = [];

    //     if (quotationData.vatType === "VATincluding") {
    //       //           this.formData.sale_totalprice = this.formatDecimal(
    //       //   parseFloat(this.formData.Net_price.replace(/,/g, "")) / 1.07
    //       // );
    //       // this.formData.vat = this.formatDecimal(
    //       //   parseFloat(this.formData.total_price.replace(/,/g, "")) -
    //       //     parseFloat(this.formData.sale_totalprice.replace(/,/g, ""))
    //       // );
    //       let FormTotalprice = [
    //         `${this.formatDecimal(total_price)}`,
    //         `${(
    //           this.formatDecimal(
    //             total_price - quotationData.discount_quotation
    //           ) / 1.07
    //         ).toFixed(2)}`,
    //         `${this.formatDecimal(quotationData.discount_quotation)}`,
    //         `${this.formatDecimal(
    //           total_price -
    //             quotationData.discount_quotation -
    //             (total_price - quotationData.discount_quotation) / 1.07
    //         )}`,
    //         `${this.formatDecimal(
    //           total_price - quotationData.discount_quotation
    //         )}`,
    //       ];
    //       doc.text(FormTotalprice, 200, 215, {
    //         align: "right",
    //         valign: "middle",
    //         lineGap: 5,
    //       });
    //     } else {
    //       let FormTotalprice = [
    //         `${this.formatDecimal(total_price)}`,
    //         `${this.formatDecimal(
    //           total_price - quotationData.discount_quotation
    //         )}`,
    //         `${this.formatDecimal(quotationData.discount_quotation)}`,
    //         `${this.formatDecimal(
    //           (total_price - quotationData.discount_quotation) * 0.07
    //         )}`,
    //         `${this.formatDecimal(
    //           total_price -
    //             quotationData.discount_quotation +
    //             (total_price - quotationData.discount_quotation) * 0.07
    //         )}`,
    //       ];
    //       doc.text(FormTotalprice, 200, 215, {
    //         align: "right",
    //         valign: "middle",
    //         lineGap: 5,
    //       });
    //     }

    //     // const FormTotalprice = [
    //     //   `${this.formatDecimal(total_price)}`,
    //     //   `${this.formatDecimal(total_discount)}`,
    //     //   `${this.formatDecimal(net_price)}`,
    //     //   `${this.formatDecimal(vat)}`,
    //     //   `${this.formatDecimal(row.sale_totalprice)}`,
    //     // ];
    //     // doc.text(FormTotalprice, 200, 215, {
    //     //   align: "right",
    //     //   valign: "middle",
    //     //   lineGap: 5,
    //     // });

    //     doc.setFont("helvetica", "bold");
    //     doc.text("Receiver", 100, 255);
    //     doc.text("Buyer", 10, 40);
    //     doc.text(`Salesperson`, 10, 215);
    //     // เพิ่มฟอนต์
    //     doc.addFileToVFS("THSarabunNew-normal.ttf", thSarabunFont);
    //     doc.addFont("THSarabunNew-normal.ttf", "THSarabunNew", "normal");

    //     // ตั้งค่าฟอนต์ที่ต้องการ
    //     doc.setFont("THSarabunNew", "normal");

    //     const img = new Image();
    //     img.src = `${this.Business.bus_logo}`;

    //     await new Promise((resolve) => {
    //       img.onload = resolve;
    //     });

    //     const maxWidth = 20;
    //     const maxHeight = 20;
    //     let imgWidth = img.width;
    //     let imgHeight = img.height;

    //     // คำนวณอัตราส่วนภาพ
    //     let widthRatio = maxWidth / imgWidth;
    //     let heightRatio = maxHeight / imgHeight;
    //     let scaleRatio = Math.min(widthRatio, heightRatio); // เลือกอัตราส่วนที่เล็กกว่าเพื่อไม่ให้เกินขนาดที่กำหนด

    //     // ปรับขนาดภาพ
    //     imgWidth = imgWidth * scaleRatio;
    //     imgHeight = imgHeight * scaleRatio;

    //     // ใส่ภาพใน PDF ด้วยขนาดที่ปรับแล้ว
    //     doc.addImage(img, "JPEG", 165, 12, imgWidth, imgHeight);

    //     doc.setFont("helvetica", "normal");

    //     // เพิ่มฟอนต์
    //     doc.addFileToVFS("THSarabunNew-normal.ttf", thSarabunFont);
    //     doc.addFont("THSarabunNew-normal.ttf", "THSarabunNew", "normal");

    //     // ตั้งค่าฟอนต์ที่ต้องการ
    //     doc.setFont("THSarabunNew", "normal");
    //     doc.text(`${row.remark}`, 40, 235);
    //     this.drawHeader(doc, headerText, startY, margin);
    //     this.drawTable(doc, currentPageData, startY, margin, lineHeight);
    //   }

    //   if (action === "view") {
    //     const pdfBlob = doc.output("blob");
    //     const pdfUrl = URL.createObjectURL(pdfBlob);
    //     this.pdfUrl = pdfUrl;
    //     // this.isPopupPDFOpen = true;
    //     // เปิดในแท็บใหม่
    //     window.open(pdfUrl, "_blank");
    //   } else if (action === "download") {
    //     doc.save(`invoice-${row.cus_name}-${row.invoice_number}.pdf`);
    //   }
    // },
    drawHeader(doc, headerText, startY, margin) {
      doc.setFontSize(10);
      doc.setTextColor(0, 0, 0); // สีข้อความ (สีดำ)

      const cellWidth =
        (doc.internal.pageSize.width - margin * 2) / headerText.length;
      const startX = margin;
      let x = margin;
      let y = startY;
      const lineHeight = 7;

      headerText.forEach((header) => {
        let cellWidth = 0;
        if (header === "No.") {
          cellWidth = 10;
        } else if (header === "Image") {
          cellWidth = 30;
        } else if (header === "Product Name") {
          cellWidth = (doc.internal.pageSize.width - margin * 2) * 0.3;
        } else {
          cellWidth =
            (doc.internal.pageSize.width -
              margin * 2 -
              (doc.internal.pageSize.width - margin * 2) * 0.4 -
              20) /
            (headerText.length - 3);
        }

        doc.text(header, x + cellWidth / 2, y + 3, {
          align: "center",
          valign: "middle",
        });
        x += cellWidth;
      });

      doc.setLineWidth(0.1);
      doc.rect(
        margin,
        startY - 2,
        doc.internal.pageSize.width - 2 * margin + 1,
        lineHeight
      );
    },
    // drawTable(doc, data, startY, margin, lineHeight) {
    //   const startX = margin;

    //   let y = startY + 5;
    //   doc.setFontSize(10);

    //   const cellWidths = data[0].map((_, index) => {
    //     if (index === 0) {
    //       return 10; // กำหนดความยาวเซลล์สำหรับ column แรกเป็น 10
    //     } else if (index === 1) {
    //       return 30;
    //     } else if (index === 2) {
    //       return (doc.internal.pageSize.width - margin * 2) * 0.3;
    //     } else {
    //       return (
    //         (doc.internal.pageSize.width -
    //           margin * 2 -
    //           (doc.internal.pageSize.width - margin * 2) * 0.4 -
    //           20) /
    //         (data[0].length - 3)
    //       );
    //     }
    //   });
    //   data.forEach((row) => {
    //     let x = startX;
    //     row.forEach((cell, index) => {
    //       const currentCellWidth = cellWidths[index];
    //       doc.rect(x, y, currentCellWidth, lineHeight, "S");

    //       if (index === 1 && cell) {
    //         // ถ้าเป็น cell ที่ index เท่ากับ 1 (คอลัมน์ที่เป็น productImg) และมีรูปภาพใน cell
    //         const imgHeight = lineHeight * 0.5; // ความสูงของรูปภาพจะเท่ากับความสูงของบรรทัด
    //         const imgWidth = imgHeight * 2; // กำหนดความกว้างของรูปภาพให้เล็กกว่า cellWidth เพื่อไม่ให้รูปภาพเกินขอบเขตของเซลล์
    //         doc.addImage(
    //           cell,
    //           x + (currentCellWidth - imgWidth) / 2,
    //           y + (lineHeight - imgHeight) / 2 + 2,
    //           imgWidth,
    //           imgHeight
    //         );
    //       } else {
    //         doc.text(
    //           String(cell),
    //           x + currentCellWidth / 2,
    //           y + lineHeight / 2 + 2,
    //           { align: "center", valign: "middle" }
    //         );
    //       }
    //       x += currentCellWidth;
    //     });
    //     y += lineHeight;
    //   });
    // },
    drawTable(doc, data, startY, margin, lineHeight) {
      const startX = margin;
      let y = startY + 5;
      doc.setFontSize(10);

      const cellWidths = data[0].map((_, index) => {
        if (index === 0) return 10;
        else if (index === 1) return 30;
        else if (index === 2)
          return (doc.internal.pageSize.width - margin * 2) * 0.3;
        else {
          return (
            (doc.internal.pageSize.width -
              margin * 2 -
              (doc.internal.pageSize.width - margin * 2) * 0.4 -
              20) /
            (data[0].length - 3)
          );
        }
      });

      // วนลูปแต่ละแถวของข้อมูล
      data.forEach((row) => {
        // คำนวณหาความสูงที่มากที่สุดของแถวปัจจุบัน
        let maxRowHeight = lineHeight;
        const productDetailCell = row[2] || "";
        if (
          typeof productDetailCell === "string" &&
          productDetailCell.includes("\n")
        ) {
          const lines = productDetailCell.split("\n");
          const requiredHeight = lines.length * 5;
          if (requiredHeight > maxRowHeight) {
            maxRowHeight = requiredHeight;
          }
        }

        let x = startX;

        // วนลูปแต่ละเซลล์เพื่อวาด
        row.forEach((cell, index) => {
          const currentCellWidth = cellWidths[index];
          // วาดกรอบของเซลล์โดยใช้ความสูงที่คำนวณได้
          doc.rect(x, y, currentCellWidth, maxRowHeight, "S");

          if (index === 1 && cell && cell !== "---") {
            // คอลัมน์รูปภาพ
            try {
              const padding = 1;
              const imgWidth = currentCellWidth - padding * 2;
              const imgHeight = maxRowHeight - padding * 2;
              doc.addImage(
                cell,
                x + padding,
                y + padding,
                imgWidth,
                imgHeight,
                undefined,
                "FAST"
              );
            } catch (e) {
              console.warn("Could not add image:", e);
            }
          } else if (index === 2 && typeof cell === "string") {
            // คอลัมน์ชื่อสินค้า + รายละเอียด
            const lines = cell.split("\n");

            // --- ส่วนที่แก้ไขตามความต้องการล่าสุด ---
            if (lines.length > 1) {
              // ถ้ามีหลายบรรทัด (มี detail) ให้จัดชิดซ้าย
              doc.text(lines, x + 2, y + 4, { align: "left" });
            } else {
              // ถ้ามีบรรทัดเดียว (ไม่มี detail) ให้จัดกึ่งกลาง
              const textY = y + maxRowHeight / 2;
              doc.text(lines[0], x + currentCellWidth / 2, textY, {
                align: "center",
                valign: "middle",
              });
            }
            // --- จบส่วนที่แก้ไข ---
          } else {
            // เซลล์อื่นๆ
            // วาดข้อความให้อยู่กึ่งกลางแนวตั้งของเซลล์
            const textY = y + maxRowHeight / 2;
            doc.text(String(cell), x + currentCellWidth / 2, textY, {
              align: "center",
              valign: "middle",
            });
          }
          x += currentCellWidth;
        });

        // เพิ่มค่า y ตามความสูงจริงของแถวที่เพิ่งวาดไป
        y += maxRowHeight;
      });
    },
    ClosePDFview() {
      this.isPopupPDFOpen = false;
    },
    closePopup() {
      this.isPopupOpen = false;
      this.isEditMode = false;
      this.formData = {
        bus_id: "",
        cus_id: "",
        productID: "",
        employeeID: "",
        employeeName: "",
        position: "",
        employeeEmail: "",
        employeeTel: "",
        cus_name: "",
        cus_address: "",
        cus_tel: "",
        cus_email: "",
        cus_tax: "",
        cus_purchase: "",
        sale_qty: "",
        sale_date: "",
        credit_date_number: "",
        credit_expired_date: "",
        sale_discount: 0,
        sale_totalprice: 0, // total price cal discount and vat
        sale_price: 0, //total price before cal discount but *qty
        total_price: 0, ////////////cal all product
        total_discount: 0, ///////////////
        Net_price: 0, //cal after cal discount
        vat: 0, //price not percentage
        price: "", //price per product
        remark: "",
        invoice_id: "",
        invoice_number: "",
        invoice_status: "",
        invoice_date: "",
        pdfname: "",
        file: "",
      };
      this.productForms = [];
      this.getInvoice();
      this.isPopupVisible_error = false;
    },
    async getEmployee() {
      const accessToken = localStorage.getItem("@accessToken");
      try {
        const response = await fetch(
          `${API_CALL}/employee/getEmployeeQuotation`,
          {
            headers: {
              "Content-Type": "application/json",
              Authorization: `Bearer ${accessToken}`,
            },
          }
        );
        const json = await response.json();

        if (json.statusCode === 200) {
          this.Employees = json.data.map((item) => {
            let initialTableData = {
              employeeID: item.employeeID,
              Name: item.name,
              Phone_num: item.phone,
              Email: item.email,
              Department: item.department,
              position: item.position,
            };
            return initialTableData;
          });
        } else {
          this.showPopup_error(json.data);
          console.log("Employee ", json);
        }
      } catch (error) {
        console.error("Error fetching data:", error);
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
          this.Products = json.data.map((item) => {
            return {
              productID: item.productID,
              Category: item.product_category.categoryName,
              productname: item.productname,
              Detail: item.productdetail,
              price: item.price,
              Cost: item.productcost,
              Amount: item.amount,
              productImg: item.productImg,
              productTypeID: item.productTypeID,
              categoryID: item.categoryID,
            };
          });
        } else {
          this.showPopup_error(json.data);
          console.log("Product: ", json);
        }
      } catch (error) {
        console.error("Error fetching data:", error);
      } finally {
        this.isLoading = false;
      }
    },
    async getQuotationByID(id) {
      const accessToken = localStorage.getItem("@accessToken");
      try {
        const response = await fetch(`${API_CALL}/quotation/getQuotation`, {
          headers: {
            Authorization: `Bearer ${accessToken}`,
          },
        });
        const json = await response.json();

        if (json.statusCode === 200) {
          const quotations = json.data;

          // ใช้ filter เพื่อหาข้อมูลที่ตรงกับ id
          const filteredData = quotations.filter(
            (quotation) => quotation.sale_id === id
          );

          return filteredData.length > 0 ? filteredData[0] : null; // ส่งคืนรายการแรกหรือ null ถ้าไม่มี
        }
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    },
    async getBusiness() {
      const accessToken = localStorage.getItem("@accessToken");
      this.isLoading = true;
      try {
        const response = await fetch(`${API_CALL}/quotation/getBusinessByID`, {
          method: "GET", // เปลี่ยน method เป็น 'POST'
          headers: {
            Authorization: `Bearer ${accessToken}`,
            "Content-Type": "application/json", // เพิ่ม header Content-Type
          },
        });
        const json = await response.json();

        if (json.statusCode === 200) {
          this.Business = json.data.business;
        } else {
          this.showPopup_error(json.data);
          console.log("Employee ", json);
        }
      } catch (error) {
        console.error("Error fetching data:", error);
      } finally {
        this.isLoading = false;
      }
    },
    async getInvoice() {
      const accessToken = localStorage.getItem("@accessToken");
      try {
        const response = await fetch(`${API_CALL}/quotation/getInvoice`, {
          headers: {
            Authorization: `Bearer ${accessToken}`,
          },
        });
        const json = await response.json();
        console.log("json", json);

        if (json.statusCode === 200) {
          console.log("getInvoice", json.data);
          this.Invoices = json.data.map((item) => {
            const QTDate = new Date(item.quotation_start_date);
            const EXPD = new Date(item.quotation_expired_date);
            const IND = new Date(item.invoice_date);
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

            const formatNumber = (number) => {
              return new Intl.NumberFormat("en-US", {
                minimumFractionDigits: 2,
                maximumFractionDigits: 2,
              }).format(number);
            };
            let total_before_vat;
            let vat_in;
            // alert(item.vatType);
            if (item.vatType === "VATexcluding") {
              total_before_vat = (item.sale_totalprice * 100) / 107;
              vat_in = total_before_vat * 1.07;
            } else {
              total_before_vat = item.sale_totalprice;
              vat_in = (
                item.sale_totalprice +
                (item.sale_totalprice * 7) / 100
              ).toFixed(2);
            }

            // จัดรูปแบบตัวเลขให้อยู่ในรูปแบบลูกน้ำ
            total_before_vat = formatNumber(total_before_vat);
            vat_in = formatNumber(vat_in);

            let initialTableData = {
              sale_id: item.sale_id,
              invoice_status: item.invoice_status,
              invoice_number: item.invoice_number,
              quotation_num: item.quotation_num,
              employeeID: item.employeeID,
              employeeName: item.employee_name,
              cus_id: item.cus_id,
              cus_name: item.cus_name,
              cus_address: item.cus_address,
              cus_tel: item.cus_tel,
              cus_email: item.cus_email,
              cus_tax: item.cus_tax,
              cus_purchase: item.cus_purchase,
              sale_totalprice: total_before_vat,
              net_price: vat_in,

              ID: item.invoice_id,

              invoice_date: InvoiceDate,
              remark: item.invoice_remark || "",
              deleted_at: item.deleted_at,
              // billing: item.billing,
              productForms: item.details.map((detail) => ({
                productID: detail.productID,
                sale_price: detail.sale_price,
                discounttype: detail.discounttype,
                sale_discount: detail.sale_discount,
                sale_qty: detail.sale_qty,
                product_detail: detail.product_detail,
                pro_unti: detail.pro_unti,
              })),
            };
            return initialTableData;
          });
        } else {
          this.showPopup_error(json.data);
          console.log("Employee ", json);
        }
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    },
    async editInvoice() {
      // alert(this.formData.remark);
      const accessToken = localStorage.getItem("@accessToken");
      console.log("Testtttttttt: ", this.formData.invoice_status);

      // const date_invoice_date = new Date(this.formData.invoice_date); // ค่าที่ได้
      // date_invoice_date.setFullYear(date_invoice_date.getFullYear() - 543); // ลบ 543 ปี
      if (
        this.formData.invoice_number === "" ||
        this.formData.invoice_date === "" ||
        this.formData.invoice_status === ""
      ) {
        this.inputError = true;
        this.showPopup_error("Please fill all data");
        this.result = false;
        console.log("Space");
      } else {
        console.log("Sale number: ", this.formData.invoice_date);
        try {
          console.log("Not Space");
          const in_id = this.formData.invoice_id;
          const response = await fetch(
            `${API_CALL}/quotation/editInvoice/${in_id}`,
            {
              method: "PUT",
              headers: {
                "Content-Type": "application/json",
                Authorization: `Bearer ${accessToken}`,
              },
              body: JSON.stringify({
                invoice_number: this.formData.invoice_number,
                invoice_date: this.formData.invoice_date,
                // invoice_date:
                //   this.t("lang") === "en"
                //     ? date_invoice_date
                //     : this.formData.invoice_date,
                invoice_status: this.formData.invoice_status,
                remark: this.formData.remark,
              }),
            }
          );
          const json = await response.json();
          console.log("=================================>>", json);

          if (json.statusCode === 200) {
            this.getInvoice();
            if (this.shortcutAllow === false) {
              this.showPopup(this.$t("validation.EditSucc"));
              this.closePopup();
            }
          } else {
            this.showPopup_error(json.data);
            console.log("Employee ", json);
          }
        } catch (error) {
          console.error("Error fetching data:", error);
        }
      }
    },
    async editInvoice2() {
      // alert(this.formData.remark);
      const accessToken = localStorage.getItem("@accessToken");
      console.log("Testtttttttt: ", this.formData.invoice_status);

      // const date_invoice_date = new Date(this.formData.invoice_date); // ค่าที่ได้
      // date_invoice_date.setFullYear(date_invoice_date.getFullYear() - 543); // ลบ 543 ปี
      if (
        this.formData.invoice_number === "" ||
        this.formData.invoice_date === "" ||
        this.formData.invoice_status === ""
      ) {
        this.inputError = true;
        this.showPopup_error("Please fill all data");
        this.result = false;
        console.log("Space");
      } else {
        console.log("Sale number: ", this.formData.invoice_date);
        try {
          console.log("Not Space");
          const in_id = this.formData.invoice_id;
          if (!in_id || in_id === "null") {
            console.error("Invalid Invoice ID:", in_id);
            this.showPopup_error("Error: Invalid Invoice ID");
            return;
          }
          const response = await fetch(
            `${API_CALL}/quotation/editInvoice/${in_id}`,
            {
              method: "PUT",
              headers: {
                "Content-Type": "application/json",
                Authorization: `Bearer ${accessToken}`,
              },
              body: JSON.stringify({
                invoice_number: this.formData.invoice_number,
                invoice_date: this.formData.invoice_date,
                // invoice_date:
                //   this.t("lang") === "en"
                //     ? date_invoice_date
                //     : this.formData.invoice_date,
                invoice_status: "Issue a receipt",
                remark: this.formData.remark,
              }),
            }
          );
          const json = await response.json();

          if (json.statusCode === 200) {
            this.getInvoice();
            // this.showPopup(this.$t("validation.EditSucc"));
            // this.closePopup();
          } else {
            this.showPopup_error(json.data);
            console.log("Employee ", json);
          }
        } catch (error) {
          console.error("Error fetching data:", error);
        }
      }
    },
    async editInvoice_Custome() {
      // alert(this.formData.remark);
      const accessToken = localStorage.getItem("@accessToken");
      console.log("Testtttttttt: ", this.formData.invoice_status);

      // const date_invoice_date = new Date(this.formData.invoice_date); // ค่าที่ได้
      // date_invoice_date.setFullYear(date_invoice_date.getFullYear() - 543); // ลบ 543 ปี
      if (
        this.formData.invoice_number === "" ||
        this.formData.invoice_date === "" ||
        this.formData.invoice_status === ""
      ) {
        // this.inputError = true;
        // // this.showPopup_error("Please fill all data");
        // this.result = false;
        // console.log("Space");
      } else {
        // console.log("Sale number: ", this.formData.invoice_date);
        try {
          console.log("Not Space");
          const in_id = this.formData.invoice_id;
          const response = await fetch(
            `${API_CALL}/quotation/editInvoice/${in_id}`,
            {
              method: "PUT",
              headers: {
                "Content-Type": "application/json",
                Authorization: `Bearer ${accessToken}`,
              },
              body: JSON.stringify({
                invoice_number: this.formData.invoice_number,
                invoice_date: this.formData.invoice_date,
                // invoice_date:
                //   this.t("lang") === "en"
                //     ? date_invoice_date
                //     : this.formData.invoice_date,
                invoice_status: "Pending",
                remark: this.formData.remark,
              }),
            }
          );
          const json = await response.json();

          if (json.statusCode === 200) {
            this.getInvoice();
          } else {
            // this.showPopup_error(json.data);
            // console.log("Employee ", json);
          }
        } catch (error) {
          console.error("Error fetching data:", error);
        }
      }
    },
    async deleteInvoice() {
      const accessToken = localStorage.getItem("@accessToken");
      this.isLoading = true;
      const qty_id = this.formData.sale_id;

      // alert(qty_id);
      try {
        const response = await fetch(
          `${API_CALL}/quotation/deleteInvoice/${qty_id}`,
          {
            method: "DELETE",
            headers: {
              "Content-Type": "application/json",
              Authorization: `Bearer ${accessToken}`,
            },
            body: JSON.stringify({
              invoice_id: qty_id,
            }),
          }
        );

        const json = await response.json();

        const saleNumber = this.formData.invoice_id; // "QT-2505080001"
        // alert(saleNumber);

        // const response_delte = await this.deleteTaxInvoice(saleNumber);
        // console.log("response_delte", response_delte);

        if (json.statusCode === 200) {
          this.getInvoice();

          // this.showPopup(this.$t("validation.DelateSucc"));
          console.log("delete employee success");
          this.closeDeleteConfirmPopup();
        } else {
          console.log("delete employee error", json);
          this.showPopup_error(json.data);
        }
      } catch (error) {
        console.error("Error fetching data:", error);
      } finally {
        this.isLoading = false;
      }
    },
    async deleteTaxInvoice(qty_id) {
      const accessToken = localStorage.getItem("@accessToken");
      this.isLoading = true;
      // const qty_id = this.formData.sale_id;
      // alert(qty_id);
      try {
        const response = await fetch(
          `${API_CALL}/quotation/deleteTaxInvoice/${qty_id}`,
          {
            method: "DELETE",
            headers: {
              "Content-Type": "application/json",
              Authorization: `Bearer ${accessToken}`,
            },
            body: JSON.stringify({
              invoice_id: qty_id,
            }),
          }
        );
        const json = await response.json();

        if (json.statusCode === 200) {
          this.getInvoice();
          // this.showPopup(this.$t("validation.DelateSucc"));
          console.log("delete employee success");
          // this.closeDeleteConfirmPopup();
        } else {
          console.log("delete employee error", json);
          this.showPopup_error(json.data);
        }
      } catch (error) {
        console.error("Error fetching data:", error);
      } finally {
        this.isLoading = false;
      }
    },
  },
  created() {
    this.getInvoice();
    this.getBusiness();
    this.getProduct();
    this.getEmployee();
    this.getBanks();
  },
};
</script>

<style>
/* ปรับขนาดของแต่ละคอลัมน์ */
.product-name-column {
  width: 30%; /* กำหนดพื้นที่มากที่สุดสำหรับชื่อสินค้า */
  min-width: 150px !important;
}
.price-column {
  width: 15%; /* รองรับข้อมูลยาวสำหรับ Price */
  min-width: 130px !important;
}
.quantity-column {
  width: 5%;
  min-width: 100px !important;
}
.unit-column {
  width: 5%;
  min-width: 120px !important;
}
.discount-column {
  width: auto; /* ให้ขนาดปรับตามเนื้อหา */
  white-space: nowrap; /* บังคับให้ข้อความอยู่บรรทัดเดียว */
  text-align: center;
}
.total-price-column {
  width: 15%; /* รองรับข้อมูลยาวสำหรับ Price */
  min-width: 130px !important;
}
.action-column {
  width: 10%; /* คอลัมน์ปุ่มลบ */
  min-width: 10px !important;
}

.show-only-desktop table thead tr th:nth-child(18),
.show-only-desktop table tbody tr td:nth-child(18) {
  display: none;
}
</style>
