<template>
  <div class="main-page">
    <!-- call navigate tab -->
    <!-- <Navigate /> -->
    <div class="page-body" :isLoading="isLoading">
      <div class="mb-3">
        <h2>{{ t("headerBilling") }}</h2>
      </div>
      <!-- ✅ Add New Billing Button -->
      <div class="add-btn mb-3">
        <button class="btn btn-primary" @click="openPopup">
          <i class="mdi mdi-plus"></i> {{ t("addBilling") }}
        </button>
      </div>
      <div class="row mb-3">
        <div class="col-6 col-sm-6 col-md-6 col-lg-3">
          <input v-model="searchQuery" type="text" class="form-control me-3 size-font-md" :placeholder="$t('Search')" />
        </div>
        <div class="col-6 col-sm-6 col-md-6 col-lg-9"></div>
      </div>
      <div class="card-view-customs" style="display: none">
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
          <div v-for="quotation in filteredBill" :key="quotation.sale_id">
            <div class="card d-flex flex-column" style="font-size: 16px">
              <div class="card-header d-flex justify-content-between align-items-center"
                style="background-color: transparent; border-bottom: none;">
                <div class="fw-bold">{{ quotation.billing_number }}</div>
                <div class="d-flex gap-3">
                  <span class="mdi mdi-pencil-outline" @click="handleEdit(quotation)" style="cursor: pointer;"></span>
                  <span class="mdi mdi-trash-can-outline text-danger" @click="handleDelete(quotation)"
                    style="cursor: pointer;"></span>
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
                    <span>{{ t("billingDateHeaderTable") }}</span>
                    <span class="text-end">{{ quotation.billing_date }}</span>
                  </div>
                  <div class="d-flex justify-content-between">
                    <span>{{ t("paymentsHeaderTable") }}</span>
                    <span class="text-end">{{ quotation.payments }}</span>
                  </div>

                  <div class="d-flex justify-content-end gap-3 mt-2">
                    <span class="mdi mdi-eye-outline" @click="handlePreview(quotation)"
                      style="cursor: pointer; font-size: 20px;"></span>
                    <span class="mdi mdi-tray-arrow-down" @click="handleDownload(quotation)"
                      style="cursor: pointer; font-size: 20px;"></span>
                  </div>
                </div>
              </div>

              <div class="card-footer text-center bg-transparent border-0 pt-0"
                @click="toggleCollapse(quotation.sale_id)">
                <span :class="isExpanded(quotation.sale_id) ? 'mdi mdi-chevron-up' : 'mdi mdi-chevron-down'"
                  style="font-size: 24px; cursor: pointer;"></span>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="show-only-desktop sale_hide">
        <!-- table billing -->
        <BillingList :initialTableData="filteredBill" :tableHeaders="tableHeaders" :columnforExport="true"
          :columnEditAndDelete="true" @handleEdit="handleEdit" @handleDelete="handleDelete"
          @handleExport="handleDownload" @handlePreview="handlePreview" @handleAllow="handleAllow"
          :isLoading="isLoading" :documentName="documentName" :showAllowButton="false" />
      </div>
    </div>
    <div v-if="isLoading" class="d-flex justify-content-center align-items-center">
      <div class="spinner-border text-primary" role="status">
        <span class="visually-hidden">Loading...</span>
      </div>
    </div>
  </div>
  <!-- popup for edit or add data -->
  <popup :isOpen="isPopupOpen" :closePopup="closePopup">
    <h2>{{ t("headerBilling") }}</h2>
    <div class="border p-4 mb-3">
      <div class="mb-3 div-for-formControl">
        <label class="col-sm-5 col-md-6">{{ t("numberBilling") }}</label>
        <input class="form-control readonly" v-model="formData.billing_number" readonly disabled />
      </div>
      <div class="mb-3 div-for-formControl">
        <label class="col-sm-6 col-md-6">{{ t("dateBilling") }}</label>
        <!-- <DatePicker
          v-model:value="formData.billing_date"
          format="DD/MM/YYYY"
          value-type="date"
          placeholder="DD/MM/YYYY"
          class="form-control"
          :formatter="momentFormat"
          :lang="currentLocale"
        /> -->
        <v-date-picker v-model="formData.billing_date" locale="th-TH" :format="formatDatePicker">
          <template v-slot="{ togglePopover }">
            <input class="custom-input" :value="formatDatePicker(formData.billing_date)" @focus="togglePopover" readonly
              :placeholder="t('selectDate')" style="cursor: pointer" />
          </template>
        </v-date-picker>
      </div>
    </div>
    <div class="border p-4 mb-3">
      <div>
        <h5 style="text-decoration-line: underline">{{ t("customerPart") }}</h5>
      </div>
      <div class="mb-3 div-for-InputDropdown">
        <label class="col-sm-6 col-md-6">{{ t("customerName") }}</label>
        <div class="relative-wrapper">
          <!-- ✅ Add mode: Customer selection dropdown -->
          <div v-if="!isEditMode" class="relative-wrapper">
            <input class="form-control" v-model="selectedCusName" @input="onCustomerInput"
              @focus="showCustomerDropdown = true" @blur="closeCustomerList" :class="{ error: isEmpty.cus_name }"
              autoComplete="off" style="width: 100%; padding-right: 30px; font-size: 14px"
              :placeholder="t('customerNamePlaceholder')" @click="resetError('cus_name')" />
            <span style="
                position: absolute;
                right: 16px;
                top: 50%;
                transform: translateY(-50%) scaleX(1.5);
                font-size: 8px;
                color: #888;
                pointer-events: none;
              ">
              ▼
            </span>

            <div v-if="showCustomerDropdown && filteredCustomers.length > 0" class="custom-datalist-options show">
              <div v-for="item in filteredCustomers" :key="item.cus_id" class="custom-datalist-option"
                @mousedown.prevent="selectCustomer(item)">
                {{ item.cus_name }}
              </div>
            </div>
          </div>
          <!-- Edit mode: Readonly -->
          <input v-else class="form-control readonly" v-model="formData.cus_name" :class="{ error: inputError }"
            readonly disabled />
        </div>
      </div>
      <div class="mb-3 div-for-formControl">
        <label class="col-sm-5 col-md-6">{{ t("customerAddress") }}</label>
        <input class="form-control" v-model="formData.cus_address" :class="{ error: isEmpty.cus_address }"
          :placeholder="t('customerPurchasePlaceholderAddress')" @click="resetError('cus_address')" />
      </div>
      <div class="mb-3 div-for-formControl">
        <label class="col-sm-5 col-md-6">{{ t("phoneNum") }}</label>
        <input class="form-control" v-model="formData.cus_tel" :class="{ error: isEmpty.cus_tel }"
          :placeholder="t('customerPurchasePlaceholderPhoneNum')" @click="resetError('cus_tel')" />
      </div>

      <div class="mb-3 div-for-formControl">
        <label class="col-sm-5 col-md-6">{{ t("taxID") }}</label>
        <input class="form-control" v-model="formData.cus_tax" :class="{ error: isEmpty.cus_tax }"
          :placeholder="t('customerPurchasePlaceholderTaxID')" @click="resetError('cus_tax')" />
      </div>

    </div>
    <div class="mb-3">
      <div class="Register-contain" :class="{ error: isEmpty.productForms }" style="padding: 20px; width: unset">
        <div>
          <h5 style="text-decoration-line: underline">{{ t("product") }}</h5>
          <!-- ✅ Add Product Button (only in Add mode) -->
          <div v-if="!isEditMode" class="mb-3">
            <button @click="showingAddProduct" class="add-product-btn">
              {{ t("addProduct") }}
            </button>
          </div>
          <table class="table">
            <thead>
              <tr>
                <th class="product-name-column">{{ t("productName") }}</th>
                <th class="price-column">{{ t("price") }}</th>
                <th class="quantity-column">{{ t("quantity") }}</th>
                <th class="unit-column">{{ t("pro_unit") }}</th>
                <th class="discount-column">{{ t("discount") }}</th>
                <th class="total-price-column">{{ t("totalPrice") }}</th>
                <th v-if="!isEditMode" class="action-column"></th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(form, index) in productForms" :key="index">
                <td class="product-name-column">
                  <!-- ✅ Add mode: Product selection -->
                  <div v-if="!isEditMode" class="relative-wrapper">
                    <div style="position: relative">
                      <input class="form-control" v-model="form.productName" @input="onProductInput(form, index)"
                        @focus="form.showProductDropdown = true" @blur="closeProductList(form)"
                        :class="{ error: inputError }" autoComplete="off"
                        style="width: 100%; padding-right: 30px; font-size: 14px" :placeholder="t('selectProduct')" />
                      <span style="
                          position: absolute;
                          right: 16px;
                          top: 50%;
                          transform: translateY(-50%) scaleX(1.5);
                          font-size: 8px;
                          color: #888;
                          pointer-events: none;
                        ">
                        ▼
                      </span>

                      <div v-if="form.showProductDropdown && getFilteredProducts(form.productName).length > 0"
                        class="custom-datalist-options show">
                        <div v-for="item in getFilteredProducts(form.productName)" :key="item.productID"
                          class="custom-datalist-option" @mousedown.prevent="selectProduct(form, item, index)">
                          {{ item.productname }}
                        </div>
                      </div>
                    </div>

                    <a class="text-muted ng-star-inserted text-start" href="javascript:void(0)"
                      @click="toggleProductDetail(form)">
                      <div class="description-row">
                        {{ t('addProductDetail') }}
                      </div>
                    </a>
                    <textarea v-if="form.showDetails || form.product_detail !== ''" class="form-control"
                      v-model="form.product_detail" rows="3"></textarea>
                  </div>
                  <!-- Edit mode: Readonly -->
                  <div v-else>
                    <input class="form-control readonly" v-model="form.productname" readonly disabled />
                    <textarea v-if="form.showDetails || form.product_detail !== ''" class="form-control"
                      v-model="form.product_detail" rows="3" readonly disabled></textarea>
                  </div>
                </td>
                <td class="price-column">
                  <input class="form-control readonly" v-model="form.price" type="number" min="0"
                    @input="updatePrice2(form, index)" :readonly="isEditMode || form.isReadonly2"
                    :disabled="isEditMode || form.isDisabled2" />
                </td>
                <td class="quantity-column">
                  <input class="form-control" v-model="form.sale_qty" type="number" min="1"
                    @input="updatePrice2(form, index)" :readonly="isEditMode" :disabled="isEditMode" />
                </td>
                <td class="unit-column">
                  <input class="form-control" v-model="form.pro_unti" :readonly="isEditMode" :disabled="isEditMode" />
                </td>

                <td class="discount-column">
                  <div class="discount-type">
                    <select class="form-control form-select" v-model="form.discounttype"
                      @change="updatePrice2(form, index)" :disabled="isEditMode" style="
                        border-top-right-radius: 0px;
                        border-bottom-right-radius: 0px;
                      ">
                      <option value="amount">
                        {{ t("productDiscountTypeAmount") }}
                      </option>
                      <option value="percent">
                        {{ t("productDiscountTypePercent") }}
                      </option>
                    </select>
                    <input style="
                        min-width: 100px;
                        border-top-left-radius: 0px;
                        border-bottom-left-radius: 0px;
                        border-left: 1px solid rgba(0, 0, 0, 0) !important;
                      " class="form-control" v-model="form.sale_discount" type="number" min="0"
                      @input="limitDiscount(form)" @change="updatePrice2(form, index)" :readonly="isEditMode"
                      :disabled="isEditMode" />
                  </div>
                </td>
                <td class="total-price-column">
                  <input class="form-control readonly" v-model="form.sale_price" readonly disabled />
                </td>
                <!-- ✅ Delete button (only in Add mode) -->
                <td v-if="!isEditMode" class="action-column">
                  <button class="btn btn-danger btn-sm" @click="closingProduct(index)">
                    <i class="mdi mdi-delete"></i>
                  </button>
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
        <input class="form-control readonly" :value="calculatedTotalDiscount.toFixed(2)" readonly disabled
          :class="{ error: inputError }" />
      </div>
      <div class="mb-3 div-for-formControl">
        <!-- ราคารวม (ก่อนหัก VAT หรือหลังหัก VAT ขึ้นอยู่กับประเภท) -->
        <label v-if="this.formData.vatType === 'VATincluding'" class="col-sm-5 col-md-6">{{ t("consluPrice") }}</label>
        <label v-else class="col-sm-5 col-md-6">{{ t("consluPrice") }}</label>
        <input v-if="this.formData.vatType === 'VATincluding'" class="form-control readonly"
          :value="calculatedFinalPrice.toFixed(2)" readonly :class="{ error: inputError }" disabled />
        <input v-else class="form-control readonly" :value="calculatedNetPrice.toFixed(2)" readonly
          :class="{ error: inputError }" disabled />
      </div>
      <div class="row mb-3">
        <label class="col-sm-5 col-md-6">{{ t("typeVat") }}</label>
        <div class="col-md-6">
          <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" value="VATexcluding" v-model="formData.vatType"
              @change="vatTypeChange()" :disabled="isEditMode" />
            <label class="form-check-label" for="inlineCheckbox1">{{
              t("vatType1")
            }}</label>
          </div>
          <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" value="VATincluding" v-model="formData.vatType"
              @change="vatTypeChange()" :disabled="isEditMode" />
            <label class="form-check-label" for="inlineCheckbox2">{{
              t("vatType2")
            }}</label>
          </div>
        </div>
      </div>
      <div class="mb-3 div-for-formControl">
        <label class="col-sm-5 col-md-6">{{ t("vatPrice") }}</label>
        <input :placeholder="t('vatPricePlaceholder')" :value="calculatedVat.toFixed(2)" class="form-control readonly"
          readonly disabled :class="{ error: inputError }" />
      </div>
      <div class="mb-3 div-for-formControl">
        <label v-if="this.formData.vatType === 'VATincluding'" class="col-sm-5 col-md-6">{{ t("netPrice") }}</label>
        <label v-else class="col-sm-5 col-md-6">{{ t("netPrice") }}</label>
        <input v-if="this.formData.vatType === 'VATincluding'" class="form-control readonly"
          :value="calculatedNetPrice.toFixed(2)" readonly :class="{ error: inputError }" disabled />
        <input v-else class="form-control readonly" :value="calculatedFinalPrice.toFixed(2)" readonly
          :class="{ error: inputError }" disabled />
      </div>
      <!-- <div class="mb-3 div-for-formControl">
      <label class="col-sm-5 col-md-6">{{ t("consluPrice") }}</label>
      <input
        class="form-control readonly"
        v-model="formData.sale_totalprice"
        readonly
        :class="{ error: inputError }"
      />
    </div> -->
      <div class="mb-3 div-for-formControl">
        <label class="col-sm-5 col-md-6">{{ t("payments") }}</label>
        <select class="form-control form-select" v-model="formData.payments" :class="{ error: inputError }">
          <option value="Cash">{{ t("cash") }}</option>
          <option value="MobileBank">{{ t("mobileBanking") }}</option>
          <option value="Cheque">{{ t("cheque") }}</option>
        </select>
      </div>

      <!-- ✅ New: Payment details for Bank/Cheque -->
      <div v-if="formData.payments === 'MobileBank' || formData.payments === 'Cheque'"
        class="border p-3 mb-3 bg-light rounded">
        <div class="mb-2 div-for-formControl">
          <label class="col-sm-5 col-md-6">{{ t("pay_bank") }}</label>
          <select class="form-select" v-model="formData.pay_bank" :class="{ error: inputError }"
            @click="resetError('pay_bank')">
            <option value="" disabled selected>{{ t("select_bank") }}</option>
            <option value="ธนาคารกรุงเทพ">ธนาคารกรุงเทพ (BBL)</option>
            <option value="ธนาคารกสิกรไทย">ธนาคารกสิกรไทย (KBANK)</option>
            <option value="ธนาคารกรุงไทย">ธนาคารกรุงไทย (KTB)</option>
            <option value="ธนาคารไทยพาณิชย์">ธนาคารไทยพาณิชย์ (SCB)</option>
            <option value="ธนาคารกรุงศรีอยุธยา">ธนาคารกรุงศรีอยุธยา (BAY)</option>
            <option value="ธนาคารทหารไทย">ธนาคารทหารไทย (TMB)</option>
            <option value="ธนาคารธนชาต">ธนาคารธนชาต (TBANK)</option>
            <option value="ธนาคารเกียรตินาคิน">ธนาคารเกียรตินาคิน (KK)</option>
            <option value="ธนาคารทิสโก้">ธนาคารทิสโก้ (TISCO)</option>
            <option value="ธนาคารซีไอเอ็มบีไทย">ธนาคารซีไอเอ็มบีไทย (CIMBT)</option>
            <option value="ธนาคารแลนด์แอนด์เฮ้าส">ธนาคารแลนด์แอนด์เฮ้าส (LH)</option>
            <option value="ธนาคารยูโอบี">ธนาคารยูโอบี (UOB)</option>
            <option value="ธนาคารเพื่อการเกษตรและสหกรณ์การเกษตร">ธนาคารเพื่อการเกษตรและสหกรณ์การเกษตร (BACC)</option>
            <option value="ธนาคารไอซีบซี">ธนาคารไอซีบซี (ICBC)</option>
            <option value="ธนาคารไอซีบีซี (ไทย)">ธนาคารไอซีบีซี (ไทย)</option>
            <option value="ธนาคารออมสิน">ธนาคารออมสิน (GSB)</option>
          </select>
        </div>
        <div class="mb-2 div-for-formControl">
          <label class="col-sm-5 col-md-6">{{ t("pay_number") }}</label>
          <input class="form-control" v-model="formData.pay_number"
            :placeholder="formData.payments === 'Cheque' ? t('checkNumber') : t('transactionNumber')" />
        </div>
        <div class="mb-2 div-for-formControl">
          <label class="col-sm-5 col-md-6">{{ t("pay_branch") }}</label>
          <input class="form-control" v-model="formData.pay_branch" :placeholder="t('bankBranchPlaceholder')" />
        </div>
        <div class="mb-2 div-for-formControl">
          <label class="col-sm-5 col-md-6">{{ t("pay_date") }}</label>
          <v-date-picker v-model="formData.pay_date" locale="th-TH" :format="formatDatePicker">
            <template v-slot="{ togglePopover }">
              <input class="custom-input bg-white" :value="formatDatePicker(formData.pay_date)" @click="togglePopover"
                readonly :placeholder="t('selectDate')" style="width: 100%; cursor: pointer" />
            </template>
          </v-date-picker>
        </div>
      </div>
      <div class="mb-5 div-for-formControl-textarea">
        <label class="col-sm-6 col-md-6 label-textarea">{{
          t("quotationRemark")
        }}</label>
        <div class="text-editor">
          <textarea v-model="formData.remark" class="form-control" maxlength="105" rows="3"
            :placeholder="t('remarkPlaceholder')" @input="onInput"></textarea>
          <p>
            {{ 105 - (formData.remark ? formData.remark.length : 0) }}
            {{ t("characters") }}
          </p>
        </div>
      </div>
    </div>
    <div class="modal-footer">
      <!-- ✅ Add mode: Save button -->
      <button v-if="!isEditMode" :disabled="isLoading" class="btn btn-primary me-3" @click="saveBilling">
        <span v-if="isLoading" class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
        <span v-else>{{ t("buttonSave") }}</span>
      </button>
      <!-- Edit mode: Edit button -->
      <button v-else :disabled="isLoading" class="btn btn-primary me-3" @click="editBilling">
        <span v-if="isLoading" class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
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
    <Popup :isOpen="isDeleteConfirmPopupOpen" :closePopup="closeDeleteConfirmPopup">
      <div class="mb-5">
        <a>{{ t("deleteConfirmSentence") }}</a>
      </div>
      <div class="modal-footer mb-5">
        <button class="btn btn-danger me-2" @click="deleteBilling">
          {{ t("buttonDelete") }}
        </button>
        <button class="btn btn-secondary" @click="closeDeleteConfirmPopup">
          {{ t("buttonCancel") }}
        </button>
      </div>
    </Popup>
  </div>
  <div class="delete-popup">
    <Popup :isOpen="isAllowConfirmPopupOpen" :closePopup="closeAllowConfirmPopup">
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
  <div class="delete-popup">
    <Popup :isOpen="isCutStockConfirmPopupOpen" :closePopup="closeCutStockConfirmPopup">
      <div v-if="formData.deleted_at === '' || formData.deleted_at === null" class="mb-5">
        <a>{{ t("CutStockConfirmSentence") }}</a>
      </div>
      <div v-else-if="formData.deleted_at !== ''" class="mb-5">
        <a>{{ t("CancalCutStockConfirmSentence") }}</a>
      </div>
      <div class="modal-footer mb-5">
        <button class="btn btn-danger me-2" @click="cutStock">
          {{ t("buttonConfirm") }}
        </button>
        <button class="btn btn-secondary" @click="closeCutStockConfirmPopup">
          {{ t("buttonCancel") }}
        </button>
      </div>
    </Popup>
  </div>

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
        <button type="button" class="btn-close" aria-label="Close" @click="closeErrorPopup"
          style="color: #9f9999"></button>
      </div>
      <div class="popup-content-error2">
        <a>{{ popupMessage_error }}</a>
      </div>
    </div>
  </div>
  <div v-if="isLoading" class="d-flex justify-content-center align-items-center">
    <div class="spinner-border text-primary" role="status">
      <span class="visually-hidden">Loading...</span>
    </div>
  </div>
  <transition name="fade">
    <div v-if="openPopupAllow" class="popup-overlay">
      <div class="popup-content-custome alert alert-light" role="alert">
        <span v-if="formData.deleted_at === '' || formData.deleted_at === null">
          <i class="fa-solid fa-check"></i> {{ t("popupCutBilling") }}
        </span>
        <span v-else-if="formData.deleted_at !== ''">
          <i class="fa-solid fa-ban"></i> {{ t("popupCancalCutBilling") }}
        </span>
      </div>
    </div>
  </transition>
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
</template>

<script>
import Navigate from "../components/Navigation.vue";
import BillingList from "../components/tableList.vue";
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
  name: "BillingNote",
  components: {
    Navigate,
    BillingList,
    Popup,
    DatePicker,
  },
  setup() {
    const { t } = useI18n();
    const { locale } = useI18n();
    // ✅ สร้าง computed property ให้ใช้งาน t("lang") ได้ตลอด
    const lang = computed(() => t("lang"));
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

    const documentName = computed(() => t("dontHaveBilling"));

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
      errorMessages: [],
      shortcutAllow: false,
      isCutStockConfirmPopupOpen: false,
      isAllowConfirmPopupOpen: false,
      expandedItems: new Set(), // ใช้ Set() แทน isExpanded,
      AllBanks: [],
      pdfUrl: null,
      isDeleteConfirmPopupOpen: false,
      Billings: [],
      isLoading: false,
      isPopupVisible_error: false,
      inputError: false,
      isPopupVisible: false,
      isPopupOpen: false,
      isPopupPDFOpen: false,
      Business: [],
      Products: [],
      Employees: [],
      Customers: [], // ✅ Added for customer selection
      productForms: [],
      isEditMode: false, // ✅ Added to differentiate Add vs Edit mode
      selectedCusName: "", // ✅ Added for customer dropdown
      popupMessage: "", // ✅ Added for success messages
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
        cusPurchasePhone: "",
        cusPurchaseEmail: "",
        cusPurchaseRemark: "",
        sale_qty: "",
        sale_number: "",
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
        billing_id: "",
        billing_number: "",
        billing_status: "",
        billing_date: "",
        payments: "Cash", // ✅ Set default payment method
        pay_bank: "",
        pay_number: "",
        pay_branch: "",
        pay_date: "",
        status: "",
        pdfname: "",
        file: "",
        vatType: "VATexcluding", // ✅ Added VAT type
      },
      searchQuery: "",
      isEmpty: { // ✅ Added for form validation
        cus_name: false,
        employeeID: false,
        billing_date: false,
        productForms: false,
        cus_address: false,
        cus_tel: false,
        cus_email: false,
        cus_purchase: false,
        cus_tax: false,
        pay_bank: false,
      },
      showCustomerDropdown: false,
    };
  },
  computed: {
    filteredCustomers() {
      if (!this.selectedCusName) return this.Customers;
      return this.Customers.filter(c =>
        c.cus_name.toLowerCase().includes(this.selectedCusName.toLowerCase())
      );
    },
    allExpanded() {
      return this.expandedItems.size === this.Billings.length; // ถ้าทั้งหมดขยาย ให้ return true
    },
    //for calculate data
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
    //send header data to table, this for update switch lang
    tableHeaders() {
      return [
        //key of lang, key of data that get from API
        // { label: this.t("billingStatusHeaderTable"), key: "billing_status" },
        { label: this.t("billingNumberHeaderTable"), key: "billing_number" },
        { label: this.t("employeeNameHeaderTable"), key: "employeeName" },
        { label: this.t("cusNameHeaderTable"), key: "cus_name" },
        { label: this.t("cusAddressHeaderTable"), key: "cus_address" },
        { label: this.t("cusTelHeaderTable"), key: "cus_tel" },
        { label: this.t("cusEmailHeaderTable"), key: "cus_email" },
        { label: this.t("cusTaxHeaderTable"), key: "cus_tax" },
        { label: this.t("cusPurchaseHeaderTable"), key: "cus_purchase" },
        { label: this.t("saleTotalpriceHeaderTable"), key: "sale_totalprice" },
        { label: this.t("netpriceHeaderTable"), key: "sale_totalprice" },
        // { label: this.t("invoiceDateHeaderTable"), key: "invoice_date" },
        { label: this.t("remarkHeaderTable"), key: "remark" },
        { label: this.t("billingDateHeaderTable"), key: "billing_date" },
        { label: this.t("paymentsHeaderTable"), key: "payments" },
      ];
    },
    filteredBill() {
      let filteredInvoices = [...this.Billings];
      // กรองข้อมูล Categories โดยเปรียบเทียบกับ searchQuery
      if (this.searchQuery.trim()) {
        filteredInvoices = filteredInvoices.filter((bll) =>
          bll["billing_number"]
            .toLowerCase()
            .includes(this.searchQuery.toLowerCase())
        );
      }

      if (this.t("headerLang") === "TH") {
        filteredInvoices = filteredInvoices.map((inv) => {
          const {
            tax_invoice_number,
            sale_number,
            invoice_number,
            quotation_num,
            bus_id,
            created_at,
            updated_at,
            ...rest
          } = inv;
          return {
            ...rest,
            showAllowButton: true,
            // statusds: inv.deleted_at,
            billing_status:
              inv.billing_status === "Complete"
                ? this.t("CompleteLG")
                : inv.billing_status, // ถ้าไม่ตรงเงื่อนไขใด ๆ ใช้ค่าเดิม

            payments:
              inv.payments === "Cash"
                ? this.t("CashLG")
                : inv.payments === "Card"
                  ? this.t("CardLG")
                  : inv.payments === "MobileBank"
                    ? this.t("MobileBankLG")
                    : inv.payments, // ถ้าไม่ตรงเงื่อนไขใด ๆ ใช้ค่าเดิม
          };
        });

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

        // แปลง invoice_date ให้เป็นชื่อเดือนภาษาไทย
        filteredInvoices = filteredInvoices.map((sale) => ({
          ...sale,
          billing_date: String(sale.billing_date)
            .replace(
              /\b(Jan|Feb|Mar|Apr|May|Jun|Jul|Aug|Sep|Oct|Nov|Dec)\b/g,
              (match) => monthMapping[match]
            )
            .replace(/(\d{4})/, (match) => (parseInt(match) + 543).toString()), // ✅ เพิ่มปี ค.ศ. + 543 (เป็น พ.ศ.),
        }));
      }

      return filteredInvoices; // ถ้าไม่มีการค้นหาแสดงทั้งหมด
    },
    // ✅ Added for product filtering
    filteredProducts() {
      return this.Products.filter(
        (product) =>
          !product.Status || // If Status is null/undefined, include it
          product.Status === "Active" ||
          product.Status === "OpenSale"
      );
    },

    // ✅ Computed properties for automatic price calculation
    calculatedTotalPrice() {
      return this.productForms.reduce((total, form) => {
        console.log(form);
        return total + (parseFloat(form.price) || 0) * (parseFloat(form.sale_qty) || 0);
      }, 0);
    },

    calculatedTotalDiscount() {
      return this.productForms.reduce((total, form) => {
        const qty = parseFloat(form.sale_qty) || 0;
        const price = parseFloat(form.price) || 0;
        const discount = parseFloat(form.sale_discount) || 0;

        if (form.discounttype === "percent") {
          return total + (qty * price * discount) / 100;
        } else {
          return total + discount;
        }
      }, 0);
    },

    calculatedNetPrice() {
      return this.calculatedTotalPrice - this.calculatedTotalDiscount;
    },

    calculatedVat() {
      if (this.formData.vatType === "VATexcluding") {
        return (this.calculatedNetPrice * 7) / 100;
      } else {
        // VATincluding
        return (this.calculatedNetPrice * 7) / 107;
      }
    },

    calculatedFinalPrice() {
      if (this.formData.vatType === "VATexcluding") {
        return this.calculatedNetPrice + this.calculatedVat;
      } else {
        return this.calculatedNetPrice;
      }
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
      this.shortcutAllow = false;
      this.isPopupVisible_error = false;
    },
    resetError(field) {
      if (field && this.isEmpty[field] !== undefined) {
        this.isEmpty[field] = false;
      }
      this.isPopupVisible_error = false;
    },
    closeCutStockConfirmPopup() {
      this.isCutStockConfirmPopupOpen = false;
      this.shortcutAllow = false;
      this.isPopupVisible_error = false;
    },
    async handleAllow(row) {
      this.shortcutAllow = true;
      await this.handleEdit(row);
      this.isCutStockConfirmPopupOpen = true;
    },
    async cutStock() {
      this.errorMessages = [];
      this.isLoading = true;
      this.shortcutAllow = false;

      const isAlreadyCut = !!this.formData.deleted_at;
      const transactionType = isAlreadyCut ? "Receive" : "Issue";

      try {
        // ✅ STEP 1: ตรวจสอบทั้งหมดก่อน
        for (const form of this.productForms) {
          const productData = this.Products.find(
            (p) => p.productID === form.productID
          );

          if (!productData) {
            this.errorMessages.push(`ไม่พบข้อมูลสินค้า ${form.productID}`);
            return;
          }

          if (!form.sale_qty || form.sale_qty <= 0) {
            this.errorMessages.push(
              `กรุณาระบุจำนวนของสินค้า "${productData.productname}" ให้มากกว่า 0`
            );
            return;
          }
          if (
            transactionType === "Issue" &&
            form.sale_qty > productData.Amount
          ) {
            this.errorMessages.push(
              `ไม่สามารถตัดสต็อกสินค้า "${productData.productname}" ได้\nจำนวนขาย (${form.sale_qty}) มากกว่าจำนวนในคลัง (${productData.Amount})`
            );
            return;
          }
        }

        if (this.errorMessages.length > 0) {
          this.showPopup_validate(this.errorMessages);
          return;
        }

        // ✅ STEP 2: ถ้าผ่านหมด → ค่อยส่ง API
        for (const form of this.productForms) {
          const response = await fetch(
            `${API_CALL}/product/cut_strock_product`,
            {
              method: "PUT",
              headers: {
                "Content-Type": "application/json",
              },
              body: JSON.stringify({
                id: form.productID,
                transactionType,
                quantity: form.sale_qty,
                billing_number: this.formData.billing_number,
              }),
            }
          );

          const json = await response.json();
          console.log(json);

          if (json.statusCode !== 200) {
            this.errorMessages.push(
              `ไม่สามารถดำเนินการกับสินค้า ${form.productID} ได้: ${json.message || "เกิดข้อผิดพลาด"
              }`
            );
            return;
          }
        }

        if (this.errorMessages.length > 0) {
          this.showPopup_validate(this.errorMessages);
          return;
        }

        // 🎉 ทุกอย่างผ่าน
        await this.getBilling();
        await this.getProduct();
        this.isCutStockConfirmPopupOpen = false;
        this.openPopupAllow = true;
        setTimeout(() => {
          this.openPopupAllow = false;
        }, 3000);
      } catch (error) {
        console.error("Error fetching data:", error);
        this.errorMessages.push("เกิดข้อผิดพลาดในการติดต่อกับเซิร์ฟเวอร์");
        this.showPopup_validate(this.errorMessages);
      } finally {
        this.isLoading = false;
        this.shortcutAllow = false;
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

    closeAllowConfirmPopup() {
      this.isAllowConfirmPopupOpen = false;
    },
    toggleAll() {
      if (this.allExpanded) {
        this.expandedItems.clear(); // ถ้าทุกอันถูกขยาย → ย่อทั้งหมด
      } else {
        this.expandedItems = new Set(this.Billings.map((q) => q.sale_id)); // ถ้ายังขยายไม่หมด → ขยายทั้งหมด
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

    // Custom Dropdown Logic
    onCustomerInput() {
      this.showCustomerDropdown = true;
      // You can trigger extra logic here if needed, but getDetailCustomer is on selection now
      this.getDetailCustomer(); // Keep existing behavior of detail fetching on input? 
      // If getDetailCustomer relies on exact match, it might be better to call it only on select, 
      // BUT if the user types an exact name without clicking, we still want it.
    },
    selectCustomer(customer) {
      this.selectedCusName = customer.cus_name;
      this.showCustomerDropdown = false;
      this.getDetailCustomer();
    },
    closeCustomerList() {
      // Delay closing to allow click event to register
      setTimeout(() => {
        this.showCustomerDropdown = false;
      }, 200);
    },

    getFilteredProducts(query) {
      if (!query) return this.Products;
      return this.Products.filter(p =>
        p.productname.toLowerCase().includes(query.toLowerCase())
      );
    },
    onProductInput(form, index) {
      form.showProductDropdown = true;
      this.getDetailProduct(form, index); // Keep strict match check if user types exact name
    },
    selectProduct(form, product, index) {
      form.productName = product.productname;
      form.showProductDropdown = false;
      this.getDetailProduct(form, index);
    },
    closeProductList(form) {
      setTimeout(() => {
        form.showProductDropdown = false;
      }, 200);
    },

    // ✅ ========== NEW METHODS FOR DIRECT BILLING CREATION ==========

    // Open popup for creating new billing
    async openPopup() {
      this.isEditMode = false;
      this.isPopupOpen = true;
      this.selectedCusName = "";
      this.productForms = [];

      // Get employee info from localStorage
      const employeeID = localStorage.getItem("user_id");
      const employeeName = localStorage.getItem("user_name");

      console.log("Employee from localStorage:", { employeeID, employeeName });

      // Reset formData
      this.formData = {
        bus_id: localStorage.getItem("@bus_id"),
        cus_id: "",
        employeeID: employeeID || "",
        employeeName: employeeName || "",
        cus_name: "",
        cus_address: "",
        cus_tel: "",
        cus_email: "",
        cus_tax: "",
        cus_purchase: "",
        cusPurchasePhone: "",
        cusPurchaseEmail: "",
        cusPurchaseRemark: "",
        sale_totalprice: 0,
        total_price: 0,
        total_discount: 0,
        Net_price: 0,
        vat: 0,
        remark: "",
        billing_number: "",
        billing_date: new Date(),
        payments: "Cash",
        pay_bank: "",
        pay_number: "",
        pay_branch: "",
        pay_date: new Date(),
        vatType: "VATexcluding",
      };

      // Reset validation
      this.isEmpty = {
        cus_name: false,
        employeeID: false,
        billing_date: false,
        productForms: false,
        cus_address: false,
        cus_tel: false,
        cus_email: false,
        cus_purchase: false,
        cus_tax: false,
        pay_bank: false,
      };

      await this.checkLatestBillingNumber();
      await this.getCustomer();
      // ✅ Ensure products are loaded
      if (this.Products.length === 0) {
        await this.getProduct();
      }
      // ✅ Ensure employees are loaded
      if (this.Employees.length === 0) {
        await this.getEmployee();
      }
      console.log("Products loaded:", this.Products.length);
      console.log("Filtered products:", this.filteredProducts.length);

      // ✅ Initialize price calculations
      this.updateTotalDiscount();
    },

    // Handle employee selection
    getDetailEmployee() {
      const selectedEmployee = this.Employees.find(
        (emp) => emp.Name === this.formData.employeeName
      );

      if (selectedEmployee) {
        this.formData.employeeID = selectedEmployee.employeeID;
      }
    },

    // Close popup
    closePopup() {
      this.isPopupOpen = false;
      this.isEditMode = false;
      this.selectedCusName = "";
      this.productForms = [];
    },

    // Check latest billing number and generate new one
    async checkLatestBillingNumber() {
      const accessToken = localStorage.getItem("@accessToken");
      try {
        const response = await fetch(
          `${API_CALL}/quotation/checkLatestBilling`,
          {
            method: "GET",
            headers: {
              Authorization: `Bearer ${accessToken}`,
              "Content-Type": "application/json",
            },
          }
        );
        const json = await response.json();

        if (json.statusCode === 200 && json.data) {
          const latestNumber = json.data.billing_number;
          const today = new Date();
          const year = today.getFullYear();
          const month = String(today.getMonth() + 1).padStart(2, "0");
          const day = String(today.getDate()).padStart(2, "0");
          const todayStr = `${year}${month}${day}`;

          // Parse latest number: HDyyyyMMdd-xx (removed BL- prefix)
          const match = latestNumber.match(/HD(\d{8})-(\d{2})/);

          if (match) {
            const lastDate = match[1];
            const lastSeq = parseInt(match[2], 10);

            if (lastDate === todayStr) {
              // Same day, increment sequence
              const newSeq = String(lastSeq + 1).padStart(2, "0");
              this.formData.billing_number = `HD${todayStr}-${newSeq}`;
            } else {
              // Different day, start from 01
              this.formData.billing_number = `HD${todayStr}-01`;
            }
          } else {
            // Invalid format or first billing
            this.formData.billing_number = `HD${todayStr}-01`;
          }
        } else {
          // No billing found, start from 01
          const today = new Date();
          const year = today.getFullYear();
          const month = String(today.getMonth() + 1).padStart(2, "0");
          const day = String(today.getDate()).padStart(2, "0");
          this.formData.billing_number = `HD${year}${month}${day}-01`;
        }
      } catch (error) {
        console.error("Error checking latest billing:", error);
        // Fallback to default
        const today = new Date();
        const year = today.getFullYear();
        const month = String(today.getMonth() + 1).padStart(2, "0");
        const day = String(today.getDate()).padStart(2, "0");
        this.formData.billing_number = `HD${year}${month}${day}-01`;
      }
    },

    // Get customer list
    async getCustomer() {
      const accessToken = localStorage.getItem("@accessToken");
      try {
        const response = await fetch(`${API_CALL}/quotation/getCustomer`, {
          method: "GET",
          headers: {
            Authorization: `Bearer ${accessToken}`,
            "Content-Type": "application/json",
          },
        });
        const json = await response.json();

        if (json.statusCode === 200) {
          const data = json.data.filter((item) => item.Status !== "not active");
          this.Customers = data;
        }
      } catch (error) {
        console.error("Error fetching customers:", error);
      }
    },

    // Handle customer selection
    getDetailCustomer() {
      const selectedCustomer = this.Customers.find(
        (cus) => cus.cus_name === this.selectedCusName
      );

      if (selectedCustomer) {
        this.formData.cus_id = selectedCustomer.cus_id;
        this.formData.cus_name = selectedCustomer.cus_name;
        this.formData.cus_address = selectedCustomer.cus_address;
        this.formData.cus_tel = selectedCustomer.cus_tel;
        this.formData.cus_email = selectedCustomer.cus_email;
        this.formData.cus_tax = selectedCustomer.cus_tax;
        this.formData.cus_purchase = selectedCustomer.cus_purchase;
        this.formData.cusPurchasePhone = selectedCustomer.cusPurchasePhone || "";
        this.formData.cusPurchaseEmail = selectedCustomer.cusPurchaseEmail || "";
        this.formData.cusPurchaseRemark = selectedCustomer.cusPurchaseRemark || "";
      }
    },

    // Add product form
    showingAddProduct() {
      this.productForms.push({
        productID: "",
        productName: "",
        productname: "",
        price: 0,
        sale_qty: 1,
        sale_discount: 0,
        discounttype: "amount",
        sale_price: 0,
        product_detail: "",
        pro_unti: "",
        showDetails: false,
        isReadonly2: true,
        isDisabled2: true,
        showProductDropdown: false,
      });
    },

    // Remove product form
    closingProduct(index) {
      this.productForms.splice(index, 1);
      this.updateTotalDiscount();
    },

    // Toggle product detail textarea
    toggleProductDetail(form) {
      form.showDetails = !form.showDetails;
    },

    // Get product details when selected
    getDetailProduct(form, index) {
      const selectedProduct = this.Products.find(
        (product) => product.productname === form.productName
      );

      if (selectedProduct) {
        // ✅ Product found: Auto-fill but allow editing
        console.log("✅ Selected product:", selectedProduct);
        form.productID = selectedProduct.productID;
        form.productname = selectedProduct.productname;
        form.price = selectedProduct.price;
        form.pro_unti = selectedProduct.pro_unti || "";
        form.product_detail = selectedProduct.Detail || "";

        // Allow editing all fields except total price
        form.isReadonly2 = false;
        form.isDisabled2 = false;

        // Calculate initial price
        this.updatePrice2(form, index);
        // Force update total
        this.$nextTick(() => {
          this.updateTotalDiscount();
        });
      } else {
        // ✅ Product not found: Allow manual input
        console.log("⚠️ Product not found, allowing manual input:", form.productName);

        // Clear auto-filled data but keep user input
        form.productID = "";
        // Keep form.productName as user typed
        // Keep form.price, form.pro_unti, form.product_detail if user entered

        // Allow editing all fields
        form.isReadonly2 = false;
        form.isDisabled2 = false;

        // Recalculate if there's already price and quantity
        if (form.price && form.sale_qty) {
          this.updatePrice2(form, index);
          this.$nextTick(() => {
            this.updateTotalDiscount();
          });
        }
      }
    },

    // Update price for individual product
    updatePrice2(form, index) {
      console.log("💵 updatePrice2 called for product:", form.productname);
      const qty = parseFloat(form.sale_qty) || 0;
      const price = parseFloat(form.price) || 0;
      const discount = parseFloat(form.sale_discount) || 0;

      let totalPrice = qty * price;

      if (form.discounttype === "percent") {
        totalPrice = totalPrice - (totalPrice * discount) / 100;
      } else {
        totalPrice = totalPrice - discount;
      }

      form.sale_price = Math.max(0, totalPrice);
      console.log("💰 Product sale_price:", form.sale_price);

      console.log("🔍 About to call updateTotalDiscount...");
      try {
        this.updateTotalDiscount();
        console.log("✅ updateTotalDiscount called successfully");
      } catch (error) {
        console.error("❌ Error calling updateTotalDiscount:", error);
      }
    },

    // Limit discount to valid range
    limitDiscount(form) {
      if (form.discounttype === "percent") {
        if (form.sale_discount > 100) {
          form.sale_discount = 100;
        } else if (form.sale_discount < 0) {
          form.sale_discount = 0;
        }
      } else {
        const maxDiscount = form.price * form.sale_qty;
        if (form.sale_discount > maxDiscount) {
          form.sale_discount = maxDiscount;
        } else if (form.sale_discount < 0) {
          form.sale_discount = 0;
        }
      }
    },

    // Update total discount and prices
    updateTotalDiscount() {
      console.log("=== ENTERING updateTotalDiscount ===");
      console.log("🔄 updateTotalDiscount called");
      console.log("📦 productForms:", this.productForms);
      console.log("📋 formData:", this.formData);

      // Calculate total before discount
      this.formData.total_price = this.productForms.reduce((total, form) => {
        return total + (parseFloat(form.price) || 0) * (parseFloat(form.sale_qty) || 0);
      }, 0);
      console.log("💰 total_price:", this.formData.total_price);

      // Calculate total discount
      this.formData.total_discount = this.productForms.reduce((total, form) => {
        const qty = parseFloat(form.sale_qty) || 0;
        const price = parseFloat(form.price) || 0;
        const discount = parseFloat(form.sale_discount) || 0;

        if (form.discounttype === "percent") {
          return total + (qty * price * discount) / 100;
        } else {
          return total + discount;
        }
      }, 0);
      console.log("🎫 total_discount:", this.formData.total_discount);

      // Calculate net price (before VAT)
      this.formData.Net_price = this.formData.total_price - this.formData.total_discount;
      console.log("💵 Net_price:", this.formData.Net_price);

      // Calculate VAT
      if (this.formData.vatType === "VATexcluding") {
        this.formData.vat = (this.formData.Net_price * 7) / 100;
        this.formData.sale_totalprice = this.formData.Net_price + this.formData.vat;
      } else {
        // VATincluding
        this.formData.sale_totalprice = this.formData.Net_price;
        this.formData.vat = (this.formData.sale_totalprice * 7) / 107;
        this.formData.Net_price = this.formData.sale_totalprice - this.formData.vat;
      }
      console.log("=== EXITING updateTotalDiscount ===");
    },

    // Handle VAT type change
    vatTypeChange() {
      this.updateTotalDiscount();
    },

    // Validate input (numbers only)
    validateInput(event) {
      const charCode = event.which ? event.which : event.keyCode;
      if (charCode > 31 && (charCode < 48 || charCode > 57)) {
        event.preventDefault();
      }
    },

    // Handle input for remark
    onInput(event) {
      const maxLength = 105;
      if (event.target.value.length > maxLength) {
        this.formData.remark = event.target.value.substring(0, maxLength);
      }
    },

    // Validate form before saving
    validateForm() {
      let isValid = true;
      this.isEmpty = {
        cus_name: false,
        employeeID: false,
        billing_date: false,
        productForms: false,
        cus_address: false,
        cus_tel: false,
        cus_email: false,
        cus_purchase: false,
        cus_tax: false,
      };

      if (!this.formData.cus_name) {
        this.isEmpty.cus_name = true;
        isValid = false;
      }

      if (!this.formData.employeeID) {
        this.isEmpty.employeeID = true;
        isValid = false;
      }

      if (!this.formData.billing_date) {
        this.isEmpty.billing_date = true;
        isValid = false;
      }

      if (!this.formData.cus_name) {
        this.isEmpty.cus_name = true;
        isValid = false;
        this.errorMessages.push(this.t("validation.cus_name"));
      }

      if (!this.formData.cus_address) {
        this.isEmpty.cus_address = true;
        this.errorMessages.push(this.t("validation.cus_address"));
      }

      if (!this.formData.cus_tel) {
        this.isEmpty.cus_tel = true;
        this.errorMessages.push(this.t("validation.cus_tel"));
      }

      if (!this.formData.cus_email) {
        this.isEmpty.cus_email = true;
        this.errorMessages.push(this.t("validation.cus_email"));
      }

      if (!this.formData.cus_purchase) {
        this.isEmpty.cus_purchase = true;
        this.errorMessages.push(this.t("validation.cus_purchase"));
      }

      if (!this.formData.cus_tax) {
        this.isEmpty.cus_tax = true;
        this.errorMessages.push(this.t("validation.cus_tax"));
      }

      if (this.productForms.length === 0) {
        this.isEmpty.productForms = true;
        this.errorMessages.push(this.t("validation.productForms"));
      }

      // Validate each product has quantity > 0
      for (let i = 0; i < this.productForms.length; i++) {
        if (!this.productForms[i].sale_qty || this.productForms[i].sale_qty <= 0) {
          this.errorMessages.push(`${this.t("validation.productForms")} ${i + 1}: ${this.t("validation.quantity")}`);
          break;
        }
      }

      if (this.errorMessages.length > 0) {
        this.showPopup_validate(this.errorMessages);
        isValid = false;
      }

      return isValid;
    },

    // Save new billing
    async saveBilling() {
      if (!this.validateForm()) {
        return;
      }

      this.isLoading = true;
      const accessToken = localStorage.getItem("@accessToken");

      try {
        // Prepare products data
        const products = this.productForms.map((form) => ({
          productID: form.productID,
          sale_price: form.sale_price,
          sale_discount: form.sale_discount,
          discounttype: form.discounttype,
          sale_qty: form.sale_qty,
          product_detail: form.product_detail || "",
          pro_unti: form.pro_unti || "",
        }));

        const requestBody = {
          billing_number: this.formData.billing_number,
          billing_date: this.formData.billing_date,
          cus_id: this.formData.cus_id,
          employeeID: this.formData.employeeID,
          payments: this.formData.payments,
          pay_bank: this.formData.pay_bank,
          pay_number: this.formData.pay_number,
          pay_branch: this.formData.pay_branch,
          pay_date: this.formData.pay_date ? new Date(this.formData.pay_date).toISOString().split('T')[0] : "",
          remark: this.formData.remark,
          sale_totalprice: this.formData.sale_totalprice,
          total_discount: this.formData.total_discount,
          vatType: this.formData.vatType,
          products: products,
        };

        const response = await fetch(
          `${API_CALL}/quotation/addDirectBilling`,
          {
            method: "POST",
            headers: {
              Authorization: `Bearer ${accessToken}`,
              "Content-Type": "application/json",
            },
            body: JSON.stringify(requestBody),
          }
        );

        const json = await response.json();

        if (json.statusCode === 200) {
          this.popupMessage = this.t("popupCreatedBilling");
          this.isPopupVisible = true;
          setTimeout(() => {
            this.isPopupVisible = false;
          }, 3000);

          this.closePopup();
          await this.getBilling();
        } else {
          this.showPopup_error(json.message || "Error creating billing");
        }
      } catch (error) {
        console.error("Error saving billing:", error);
        this.showPopup_error("Error creating billing");
      } finally {
        this.isLoading = false;
      }
    },

    // Show error popup
    showPopup_error(message) {
      this.isPopupVisible_error = true;
      this.errorMessages = Array.isArray(message) ? message : [message];
      setTimeout(() => {
        this.isPopupVisible_error = false;
      }, 5000);
    },

    // ✅ ========== END OF NEW METHODS ==========

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
    //popup error message (set message where call this function)
    showPopup_error(message) {
      this.popupMessage_error = message;
      this.isPopupVisible = false;
      this.isPopupVisible_error = true;
      // setTimeout(() => {
      //   this.isPopupVisible = false;
      //   this.isPopupVisible_error = false;
      // }, 2000); // 2 seconds
    },
    //popup success message (set message where call this function)
    showPopup(message) {
      this.popupMessage = message;
      this.isPopupVisible = true;
      setTimeout(() => {
        this.isPopupVisible = false;
      }, 2000); // 2 seconds
    },
    //function for edit data
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
    async handleEdit(row) {
      this.getBilling();
      console.log("Edit item:", row);
      if (this.shortcutAllow === false) {
        this.isPopupOpen = true;
      }
      this.isAddingMode = false;
      this.isEditMode = true;
      //set date format
      const formatDateForPicker = (date) => {
        if (!date) return null;
        const d = new Date(date);
        if (isNaN(d.getTime())) return null; // ตรวจสอบว่าเป็นวันที่ถูกต้อง
        return d;
      };
      let formattedBill; // ประกาศตัวแปรก่อน

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
        if (row.billing_date && typeof row.billing_date === "string") {
          const match = row.billing_date.match(
            /(\d{1,2})\s(ม.ค.|ก.พ.|มี.ค.|เม.ย.|พ.ค.|มิ.ย.|ก.ค.|ส.ค.|ก.ย.|ต.ค.|พ.ย.|ธ.ค.)\s(\d{4})/
          );

          if (match) {
            let [_, day, thaiMonth, year] = match;
            let month = monthMapping[thaiMonth];
            formattedBill = new Date(year, month, day);
          } else {
            formattedBill = null; // ถ้าตรงกับรูปแบบไม่ได้ ให้เป็น null
          }
        } else {
          formattedBill = null; // ถ้า invoice_date ไม่มีค่าหรือไม่ใช่ string ให้เป็น null
        }
      } else {
        formattedBill = formatDateForPicker(row.billing_date); // ใช้ฟังก์ชันที่กำหนดเอง
      }
      // const formattedBill = formatDateForPicker(row.billing_date);

      const quotationData = await this.getQuotationByID(row.sale_id);

      const filteredInvoice = this.Billings.filter(
        (inv) => inv.billing_number === row.billing_number
      );
      console.log("this.Billings->>", this.Billings);
      // filteredInvoice[0].payments

      this.formData = {
        deleted_at: row.deleted_at,
        billing_number: row.billing_number,
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
        billing_id: row.ID,
        billing_status: row.billing_status,
        // billing_date: formattedBill,
        billing_date: formatDateForPicker(filteredInvoice[0].billing_date),
        payments: filteredInvoice[0].payments,
        pay_bank: filteredInvoice[0].pay_bank,
        pay_number: filteredInvoice[0].pay_number,
        pay_branch: filteredInvoice[0].pay_branch,
        pay_date: filteredInvoice[0].pay_date,
        discount_quotation: quotationData.discount_quotation,
        vatType: quotationData.vatType,
      };
      console.log("console.log(row.productForms)", this.Products);
      //loop of product
      this.productForms = (row.productForms || []).map((detail) => {
        const selectedProduct = this.Products.find(
          (product) => product.productID === detail.productID
        );
        let price = 0;
        let productname = "";
        if (selectedProduct) {
          price = this.formatDecimal(
            parseFloat(selectedProduct.price.toFixed(2))
          );
          productname = selectedProduct.productname;
        }
        const currentPrice = String(price || "0").replace(/,/g, "");
        const salePrice = detail.sale_qty * parseFloat(currentPrice);
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
          discounttype: detail.discounttype,
          product_detail: detail.product_detail,
          pro_unti: detail.pro_unti,
        };
      });
      this.updateTotalDiscount();
      this.totalNetPrice();
      // this.vat_price();
      this.total_pricesale();
      this.total_priceBeforeDiscount();

      if (this.formData.vatType === "VATincluding") {
        const netPriceRaw = String(this.formData.Net_price || "0").replace(/,/g, "");
        const netVal = parseFloat(netPriceRaw);

        this.formData.sale_totalprice = this.formatDecimal(netVal / 1.07);
        const saleTotalPriceRaw = String(this.formData.sale_totalprice || "0").replace(/,/g, "");

        this.formData.vat = this.formatDecimal(netVal - parseFloat(saleTotalPriceRaw));
      } else {
        const netPriceRaw = String(this.formData.Net_price || "0").replace(/,/g, "");
        const netVal = parseFloat(netPriceRaw);

        this.formData.vat = this.formatDecimal(netVal * 0.07);
        const vatRaw = String(this.formData.vat || "0").replace(/,/g, "");

        this.formData.sale_totalprice = this.formatDecimal(netVal + parseFloat(vatRaw));
      }
      this.calculateNat(quotationData.discount_quotation);
    },
    calculateNat(discount) {
      const totalNet = this.productForms.reduce((total, form) => {
        const salePriceRaw = String(form.sale_price || "0").replace(/,/g, "");
        const salePrice = parseFloat(salePriceRaw); // แปลง formatDecimal เป็นตัวเลข
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
    //function preview pdf call in table, the data in this function will send to pdfviewer for preview pdf
    handlePreview(row) {
      console.log("Preview item:", row);
      // this.isPopupPDFOpen = true;
      this.isAddingMode = false;
      this.isEditMode = true;
      const formatDateForPicker = (date) => {
        if (!date) return null;
        const d = new Date(date);
        if (isNaN(d.getTime())) return null; // ตรวจสอบว่าเป็นวันที่ถูกต้อง
        return d;
      };
      const formattedBilling = formatDateForPicker(row.billing_date);
      this.formData = {
        billing_number: row.billing_number,
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
        billing_id: row.ID,
        billing_status: row.billing_status,
        billing_date: formattedBilling,
        payments: row.payments,
        pay_bank: row.pay_bank,
        pay_number: row.pay_number,
        pay_branch: row.pay_branch,
        pay_date: row.pay_date,
        vatType: row.vatType || "VATexcluding",
      };
      //loop of product
      this.productForms = (row.productForms || row.details || []).map((detail) => {
        const selectedProduct = this.Products.find(
          (product) => product.productID === detail.productID || product.productname === detail.productID
        );
        let price = 0;
        if (selectedProduct) {
          price = this.formatDecimal(
            parseFloat(selectedProduct.price.toFixed(2))
          );
        } else if (detail.price) {
          price = detail.price;
        }

        const currentPrice = String(price || "0").replace(/,/g, "");
        const qty = parseFloat(detail.sale_qty) || 0;
        const salePrice = qty * parseFloat(currentPrice);
        let saleDiscount = parseFloat(detail.sale_discount) || 0;
        if (detail.discounttype === "percent") {
          saleDiscount = (saleDiscount / 100) * salePrice;
        }
        return {
          productID: detail.productID,
          price: price,
          sale_qty: qty,
          sale_price: this.formatDecimal(salePrice - saleDiscount),
          sale_discount: detail.sale_discount,
          discounttype: detail.discounttype || "amount",
        };
      });
      this.updateTotalDiscount();
      this.totalNetPrice();
      this.vat_price();
      this.total_pricesale();
      this.total_priceBeforeDiscount();

      // IMPORTANT: Call PDF generation AFTER state is updated
      this.viewformpdf("view", row);
    },
    //function download pdf call in table, the data in this function will send to pdfviewer for download
    handleDownload(row) {
      console.log("Edit item:", row);
      const formatDateForPicker = (date) => {
        if (!date) return null;
        const d = new Date(date);
        if (isNaN(d.getTime())) return null; // ตรวจสอบว่าเป็นวันที่ถูกต้อง
        return d;
      };
      this.formData = {
        billing_number: row.billing_number,
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
        billing_id: row.ID,
        billing_status: row.billing_status,
        billing_date: row.billing_date,
        payments: row.payments,
        pay_bank: row.pay_bank,
        pay_number: row.pay_number,
        pay_branch: row.pay_branch,
        pay_date: row.pay_date,
        vatType: row.vatType || "VATexcluding",
      };
      //loop of product
      this.productForms = (row.details || row.productForms || []).map((detail) => {
        const selectedProduct = this.Products.find(
          (product) => product.productID === detail.productID || product.productname === detail.productID
        );
        let price = 0;
        if (selectedProduct) {
          price = this.formatDecimal(
            parseFloat(selectedProduct.price.toFixed(2))
          );
        } else if (detail.price) {
          price = detail.price;
        }

        const currentPrice = String(price || "0").replace(/,/g, "");
        const qty = parseFloat(detail.sale_qty) || 0;
        const salePrice = qty * parseFloat(currentPrice);
        let saleDiscount = parseFloat(detail.sale_discount) || 0;
        if (detail.discounttype === "percent") {
          saleDiscount = (saleDiscount / 100) * salePrice;
        }
        return {
          productID: detail.productID,
          price: price,
          sale_qty: qty,
          sale_price: this.formatDecimal(salePrice - saleDiscount),
          sale_discount: detail.sale_discount,
          discounttype: detail.discounttype || "amount",
        };
      });
      this.updateTotalDiscount();
      this.totalNetPrice();
      this.vat_price();
      this.total_pricesale();
      this.total_priceBeforeDiscount();

      // IMPORTANT: Call PDF generation AFTER state is updated
      this.viewformpdf("download", row);
    },
    //handle delete call in table, data in this will send to popup delete
    handleDelete(item) {
      this.isDeleteConfirmPopupOpen = true;
      this.formData = {
        billing_id: item.ID,
        invoice_id: item.invoice_id,
      };
    },
    closeDeleteConfirmPopup() {
      this.isDeleteConfirmPopupOpen = false;
    },
    //this function for set date format
    formatDate(date) {
      const d = new Date(date);
      const day = String(d.getDate()).padStart(2, "0");
      const month = String(d.getMonth() + 1).padStart(2, "0");
      const year = d.getFullYear();
      return `${day}/${month}/${year}`;
    },
    //this function for set number format
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
    //calculate function total discount
    updateTotalDiscount() {
      const totalDiscount = this.productForms.reduce((total, form) => {
        let sumdiscount = "";
        if (form.sale_discount === "") {
          return 0;
        }

        if (form.discounttype === "percent") {
          sumdiscount =
            (parseFloat(form.sale_discount) *
              (parseFloat(form.sale_qty) * parseFloat(form.price))) /
            100;
        } else {
          sumdiscount = parseFloat(form.sale_discount);
        }

        const saleDiscount = parseFloat(sumdiscount);
        return total + saleDiscount;
      }, 0);
      this.formData.total_discount = this.formatDecimal(totalDiscount);
    },
    //calculate function Net
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
    //calculate function vat
    vat_price() {
      // this.formData.vat = this.formatDecimal(
      //   (7 / 100) * parseFloat(this.formData.Net_price.replace(/,/g, ""))
      // );
    },
    //calculate function total price not include discount
    total_priceBeforeDiscount() {
      const totalNet = this.productForms.reduce((total, form) => {
        const salePrice = parseFloat(form.sale_price.replace(/,/g, "")); // แปลง formatDecimal เป็นตัวเลข
        return total + salePrice;
      }, 0);
      this.formData.total_price = this.formatDecimal(totalNet);
    },
    //calculate function total price
    total_pricesale() {
      // const vat = parseFloat(this.formData.vat.replace(/,/g, ""));
      // const net = parseFloat(this.formData.Net_price.replace(/,/g, ""));
      // this.formData.sale_totalprice = this.formatDecimal(net + vat);
    },
    //function for set pdf view
    // ✅ New modular PDF generation method to match the requested design
    async viewformpdf(action, row) {
      const doc = new jsPDF();
      const orangeColor = [255, 165, 0];
      const lightGray = [240, 240, 240];
      const blueColor = [0, 87, 183]; // Added blueColor definition

      // Load fonts
      doc.addFileToVFS("Prompt-Bold.ttf", PromptBold);
      doc.addFont("Prompt-Bold.ttf", "PromptBold", "bold");
      doc.addFileToVFS("Prompt-Regular.ttf", PromptRegular);
      doc.addFont("Prompt-Regular.ttf", "PromptRegular", "normal");
      doc.addFileToVFS("Prompt-Light.ttf", PromptRegularLight);
      doc.addFont("Prompt-Light.ttf", "PromptLight", "normal");

      const quotationData = await this.getQuotationByID(row.sale_id);
      const filteredInvoicesPDF = this.Billings.filter(i => i.sale_id === row.sale_id);

      // Date formatting
      let billingDate;
      if (row.billing_date) {
        billingDate = new Date(row.billing_date);
        // Check if date is valid
        if (isNaN(billingDate.getTime())) {
          billingDate = new Date(); // Fallback to current date
        }
      } else {
        billingDate = new Date();
      }

      const day = billingDate.getDate().toString().padStart(2, "0");
      const month = (billingDate.getMonth() + 1).toString().padStart(2, "0");
      const buddhistYear = billingDate.getFullYear() + 543;
      const displayDate = `${day}/${month}/${buddhistYear}`;

      // Helper function for Thai Baht Text
      const toThaiBaht = (number) => {
        if (!number || isNaN(number)) return "ศูนย์บาทถ้วน";
        const numbers = ["ศูนย์", "หนึ่ง", "สอง", "สาม", "สี่", "ห้า", "หก", "เจ็ด", "แปด", "เก้า"];
        const scales = ["", "สิบ", "ร้อย", "พัน", "หมื่น", "แสน", "ล้าน"];
        let [integer, decimals] = number.toFixed(2).split(".");
        let result = "";

        // Handle millions if number is very large
        const handlePart = (numStr) => {
          let partResult = "";
          for (let i = 0; i < numStr.length; i++) {
            let n = parseInt(numStr[i]);
            let scaleIndex = numStr.length - i - 1;
            if (n !== 0) {
              if (scaleIndex === 0 && n === 1 && numStr.length > 1) partResult += "เอ็ด";
              else if (scaleIndex === 1 && n === 2) partResult += "ยี่";
              else if (scaleIndex === 1 && n === 1) partResult += "";
              else partResult += numbers[n];
              partResult += scales[scaleIndex];
            }
          }
          return partResult;
        };

        if (parseInt(integer) === 0) result = numbers[0];
        else {
          let parts = [];
          while (integer.length > 0) {
            parts.push(integer.substring(Math.max(0, integer.length - 6)));
            integer = integer.substring(0, Math.max(0, integer.length - 6));
          }
          for (let i = parts.length - 1; i >= 0; i--) {
            result += handlePart(parts[i]);
            if (i > 0) result += "ล้าน";
          }
        }
        result += "บาท";

        if (parseInt(decimals) === 0) result += "ถ้วน";
        else {
          result += handlePart(decimals) + "สตางค์";
        }
        return result;
      };

      // TABLE DATA
      const productForms = row.productForms || [];
      const tableData = productForms.map((item, index) => {
        const prod = this.Products.find(p => p.productID === item.productID) || {};
        return [
          index + 1,
          (prod.productname || item.productname || "") + (item.product_detail ? "\n" + item.product_detail : ""),
          parseFloat(item.sale_qty).toLocaleString(),
          this.formatDecimal(item.price || prod.price || 0),
          this.formatDecimal(item.sale_price || 0)
        ];
      });

      // RENDER LOGO & HEADER
      const renderHeader = () => {
        // Logo
        if (this.Business.bus_logo) {
          doc.addImage(this.Business.bus_logo, "JPEG", 10, 10, 25, 25);
        }

        // Company Info
        doc.setFont("PromptBold", "bold");
        doc.setFontSize(16);
        doc.text(this.Business.bus_name || "บริษัท เอช แอนด์ ดี อิงค์ เจ็ท จำกัด", 40, 18);

        doc.setFont("PromptLight", "normal");
        doc.setFontSize(9);
        const addressLines = doc.splitTextToSize(this.Business.bus_address || "", 100);
        doc.text(addressLines, 10, 40);
        doc.text(`เลขประจำตัวผู้เสียภาษี  ${this.Business.bus_tax || "0105569006906"}`, 10, 48);
        doc.text(`โทร  ${this.Business.bus_tel || "098-9315128"}`, 10, 52);

        // Title Box (Top Right)
        doc.setDrawColor(orangeColor[0], orangeColor[1], orangeColor[2]);
        doc.setLineWidth(0.5);
        doc.rect(140, 10, 60, 20); // Title frame
        doc.setFont("PromptRegular", "normal");
        doc.setFontSize(11);
        doc.text("ใบกำกับภาษี/ใบเสร็จรับเงิน", 170, 22, { align: "center" });
      };

      // RENDER SECTION BOXES
      const renderSectionBoxes = () => {
        // Customer Info Box
        doc.setDrawColor(orangeColor[0], orangeColor[1], orangeColor[2]);
        doc.rect(10, 60, 130, 35);

        doc.setFont("PromptRegular", "normal");
        doc.setFontSize(9);
        doc.text("ชื่อลูกค้า / Customers:", 12, 65);
        doc.text("ที่อยู่ / Address:", 12, 75);
        doc.text("เบอร์โทรศัพท์", 12, 85);
        doc.text("เลขประจำตัวผู้เสียภาษี", 12, 90);

        // Customer Values
        doc.setFont("PromptLight", "normal");
        doc.text(row.cus_name || "", 45, 65);
        const cusAddressLines = doc.splitTextToSize(row.cus_address || "", 90);
        doc.text(cusAddressLines, 45, 75);
        doc.text(row.cus_tel || "", 45, 85);
        doc.text(row.cus_tax || "", 45, 90);

        // Billing Info Box
        doc.rect(142, 60, 58, 35);
        doc.setFont("PromptRegular", "normal");
        doc.text("เลขที่ / No.", 145, 78);
        doc.text("วันที่ / Date", 145, 88);

        doc.setFont("PromptLight", "normal");
        doc.text(row.billing_number || "", 170, 78);
        doc.text(displayDate, 175, 88);
      };

      // RENDER PAYMENT & SUMMARY
      const renderFooter = () => {
        const finalY = doc.lastAutoTable.finalY + 2;

        // Final Prices Box (Right)
        const summaryX = 142;
        const summaryW = 58;
        const totalAmount = parseFloat(row.sale_totalprice) || 0;

        const netPrice = this.calculatedNetPrice;
        const vatPrice = this.calculatedVat;
        const finalPrice = this.calculatedFinalPrice;

        doc.setDrawColor(orangeColor[0], orangeColor[1], orangeColor[2]);
        doc.setLineWidth(0.2);

        // TOTAL Row
        doc.rect(summaryX, finalY, summaryW, 10);
        doc.setFont("PromptRegular", "normal");
        doc.text("รวมเงิน", summaryX + 2, finalY + 4);
        doc.text("TOTAL", summaryX + 2, finalY + 8);
        doc.text(this.formatDecimal(this.calculatedTotalPrice), summaryX + summaryW - 2, finalY + 7, { align: "right" });

        // VAT Row
        doc.rect(summaryX, finalY + 10, summaryW, 10);
        doc.text("VAT", summaryX + 2, finalY + 14);
        doc.text("7%", summaryX + 2, finalY + 18);
        doc.text(this.formatDecimal(vatPrice), summaryX + summaryW - 2, finalY + 17, { align: "right" });

        // NET AMOUNT Row (Colored)
        doc.setFillColor(255, 235, 204); // Light orange fill
        doc.rect(summaryX, finalY + 20, 30, 15, "F");
        doc.rect(summaryX, finalY + 20, summaryW, 15); // Frame

        doc.setFont("PromptBold", "bold");
        doc.text("ยอดเงินสุทธิ", summaryX + 2, finalY + 26);
        doc.text("NET AMOUNT", summaryX + 2, finalY + 32);
        doc.setFontSize(12);
        doc.text(this.formatDecimal(finalPrice), 198, finalY + 28, { align: "right" });

        // Payment Info (Left)
        doc.setFontSize(9);
        doc.text("รายการรับชำระเงิน :", 10, finalY + 5);

        const drawCheckbox = (x, y, label, checked = false) => {
          doc.setDrawColor(0, 0, 0);
          doc.rect(x, y - 3.5, 4, 4);
          if (checked) {
            doc.line(x + 0.5, y - 1.5, x + 1.5, y + 0.5);
            doc.line(x + 1.5, y + 0.5, x + 3.5, y - 3.5);
          }
          doc.text(label, x + 6, y);
        };

        drawCheckbox(45, finalY + 5, "เงินสด", row.payments === "Cash" || row.payments === "เงินสด");
        drawCheckbox(85, finalY + 5, "เงินโอน", row.payments === "MobileBank" || row.payments === "เงินโอน");
        drawCheckbox(125, finalY + 5, "เช็ค", row.payments === "Cheque" || row.payments === "เช็ค");

        const bankText = row.pay_bank || "";
        const payNum = row.pay_number || "";
        const branch = row.pay_branch || "";
        let displayPayDate = "";

        if (row.pay_date) {
          const pDate = new Date(row.pay_date);
          const pDay = pDate.getDate().toString().padStart(2, "0");
          const pMonth = (pDate.getMonth() + 1).toString().padStart(2, "0");
          const pYear = pDate.getFullYear() + 543;
          displayPayDate = `${pDay}/${pMonth}/${pYear}`;
        }

        doc.setFont("PromptLight", "normal");
        // Row 1: Bank and Chq No
        doc.text("ธนาคาร/Bank", 12, finalY + 12);
        doc.text(bankText, 38, finalY + 12);
        doc.text("...............................................", 38, finalY + 12.5);

        doc.text("เลขที่/ Chq #", 75, finalY + 12);
        doc.text(payNum, 98, finalY + 12);
        doc.text("...............................................", 98, finalY + 12.5);

        // Row 2: Branch and Date
        doc.text("สาขา/Branch", 12, finalY + 22);
        doc.text(branch, 38, finalY + 22);
        doc.text("...............................................", 38, finalY + 22.5);

        doc.text("วันที่ทำรายการ/Date", 75, finalY + 22);
        doc.text(displayPayDate, 108, finalY + 22);
        doc.text("...............................................", 108, finalY + 22.5);

        // Amount section (Top)
        doc.setFont("PromptRegular", "normal");
        doc.setFontSize(9);
        doc.text("จำนวนเงิน /Amount", 10, finalY + 33);
        doc.text(this.formatDecimal(finalPrice), 55, finalY + 33);

        // Thai Baht Text Box (Bottom)
        doc.setDrawColor(orangeColor[0], orangeColor[1], orangeColor[2]);
        doc.rect(40, finalY + 38, 100, 10);
        doc.text("ตัวอักษร", 10, finalY + 44);
        doc.setTextColor(blueColor[0], blueColor[1], blueColor[2]);
        doc.text(`( ${toThaiBaht(finalPrice)} )`, 90, finalY + 44, { align: "center" });
        doc.setTextColor(0, 0, 0);

        // Signatures
        const sigY = finalY + 55;
        doc.rect(10, sigY, 188, 30);
        doc.line(72, sigY, 72, sigY + 30);
        doc.line(135, sigY, 135, sigY + 30);

        doc.setFont("PromptRegular", "normal");
        doc.text("ลูกค้า/ผู้อนุมัติ", 41, sigY + 5, { align: "center" });
        doc.text("ผู้ผลิต", 103, sigY + 5, { align: "center" });
        doc.text(this.Business.bus_name || "เอช แอนด์ ดี อิงค์เจ็ท", 166, sigY + 5, { align: "center" });

        doc.text("วันที่ ...........................................", 25, sigY + 25);
        doc.text("วันที่ ...........................................", 85, sigY + 25);
        doc.text("ผู้มีอำนาจลงนาม", 166, sigY + 25, { align: "center" });

        // Remarks Footer
        doc.setTextColor(255, 0, 0);
        doc.setFont("PromptBold", "bold");
        doc.text('**หมายเหตุ : Remark', 10, sigY + 35);

        // Items 1-3 in black
        doc.setTextColor(0, 0, 0);
        doc.setFont("PromptLight", "normal");
        doc.setFontSize(8);
        doc.text("1. ราคาที่เสนอเป็นเงินบาทไทย และไม่มีภาษีมูลค่าเพิ่ม", 10, sigY + 40);
        doc.text("2. มัดจำค่าสินค้า 60% ส่วนที่เหลือชำระทั้งหมดในวันที่ส่งของ หรือ ติดตั้งแล้วเสร็จ", 10, sigY + 44);
        doc.text("3. ระยะเวลาดำเนินการ 1-2 วัน หลังจากได้รับ มัดจำ หรือระยะเวลาขึ้น อยู่กับปริมาณสินค้า ที่ลูกค้าสั่งซื้อ หรือตามเงื่อนไขอื่นๆ ตามที่ตกลงกัน", 10, sigY + 48);

        // Item 4 - text in black, account number in red
        console.log("Business data:", this.Business);
        const bankInfo = this.Business.banks && this.Business.banks.length > 0 ? this.Business.banks[0] : {};
        const bankName = bankInfo.bank_name || "";
        const bankAccount = bankInfo.bank_account || "";
        const bankNumber = bankInfo.bank_number || "";
        console.log("Bank info:", { bankName, bankAccount, bankNumber });

        const textBeforeNumber = `4. โอนเงินชำระค่าสินค้าที่ ${bankName} ( ${bankAccount} ) `;
        doc.text(textBeforeNumber, 10, sigY + 52);

        // Calculate position for bank number dynamically
        const textWidth = doc.getTextWidth(textBeforeNumber);
        const numberX = 10 + textWidth;

        doc.setTextColor(255, 0, 0);
        doc.text(bankNumber, numberX, sigY + 52);
        doc.setTextColor(0, 0, 0); // Reset to black
      };

      // EXECUTE RENDERING
      renderHeader();
      renderSectionBoxes();

      doc.autoTable({
        startY: 100,
        head: [["ลำดับที่\nItem", "รายการ\nDescription", "จำนวน\nQuantity", "ราคา/หน่วย\nUnit Price", "จำนวนเงิน\nAmount"]],
        body: tableData,
        theme: 'grid',
        styles: { font: "PromptLight", fontSize: 9, cellPadding: 2, halign: 'center', valign: 'middle' },
        headStyles: { fillColor: [255, 255, 255], textColor: [0, 0, 0], lineWidth: 0.1, halign: 'center', font: "PromptRegular" },
        columnStyles: {
          0: { cellWidth: 15 },
          1: { cellWidth: 95, halign: 'left' },
          2: { cellWidth: 20 },
          3: { cellWidth: 28, halign: 'right' },
          4: { cellWidth: 30, halign: 'right' },
        },
        margin: { left: 10, right: 10 },
        tableLineColor: blueColor,
        tableLineWidth: 0.1,
      });

      renderFooter();

      // OUTPUT ACTION
      if (action === "view") {
        const pdfBlob = doc.output("blob");
        const pdfUrl = URL.createObjectURL(pdfBlob);
        window.open(pdfUrl, "_blank");
      } else if (action === "download") {
        doc.save(`Receipt-${row.billing_number || "Draft"}.pdf`);
      }
    },

    // async viewformpdf(action, row) {
    //   console.log("Preview Item: ", row);
    //   const productForms = row.productForms || [];
    //   const doc = new jsPDF();
    //   //set date format
    //   const formatDate = { day: "2-digit", month: "short", year: "numeric" };
    //   const Qdate = new Date(row.billing_date);
    //   const Bill_date = Qdate.toLocaleDateString("en-GB", formatDate);

    //   const Expdate = new Date(row.credit_expired_date);
    //   const ExpiredDate = Expdate.toLocaleDateString("en-GB", formatDate);

    //   const quotationData = await this.getQuotationByID(row.sale_id);
    //   //set header table of product in pdf form
    //   const headerText = [
    //     "No.",
    //     "Image",
    //     "Description",
    //     "Qty",
    //     "Unit price",
    //     "Discount",
    //     "Amount",
    //   ];
    //   //set data in table of product in pdf form
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
    //   //set scale of product table
    //   const startY = 77;
    //   const margin = 10;
    //   const lineHeight = 15;
    //   const pageSize = 6; //row of product in each page
    //   const pageCount = Math.ceil(tableData.length / pageSize); //number of page

    //   //loop for set page
    //   for (let i = 0; i < pageCount; i++) {
    //     const currentPageData = tableData.slice(
    //       i * pageSize,
    //       (i + 1) * pageSize
    //     );
    //     if (i > 0) {
    //       // ตรวจสอบหน้า PDF ที่ไม่ใช่หน้าแรกหรือ action เป็น 'download'
    //       doc.addPage();
    //     }
    //     //set style text
    //     doc.setFont("helvetica", "bold");
    //     doc.setFontSize(18);
    //     doc.setTextColor(0, 0, 0);

    //     // เพิ่มฟอนต์
    //     doc.addFileToVFS("THSarabunNew-normal.ttf", thSarabunFont);
    //     doc.addFont("THSarabunNew-normal.ttf", "THSarabunNew", "normal");

    //     // ตั้งค่าฟอนต์ที่ต้องการ
    //     doc.setFont("THSarabunNew", "normal");
    //     //set text (text will get style that set before set text)
    //     doc.text(`${this.Business.bus_name}`, 10, 15);
    //     doc.text("Receipt", 160, 40);

    //     const bank_detail = this.AllBanks.filter(
    //       (bank) => bank.bank_id === this.Business.bank_id
    //     );

    //     const FormBank = [
    //       `${bank_detail[0].bank_name}`,
    //       `${bank_detail[0].bank_account}`,
    //       `${bank_detail[0].bank_number}`,
    //     ];

    //     // text group
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

    //     //set style text
    //     doc.setFont("helvetica", "normal");
    //     doc.setFontSize(12);
    //     doc.setTextColor(0, 0, 0);
    //     // เพิ่มฟอนต์
    //     doc.addFileToVFS("THSarabunNew-normal.ttf", thSarabunFont);
    //     doc.addFont("THSarabunNew-normal.ttf", "THSarabunNew", "normal");

    //     // ตั้งค่าฟอนต์ที่ต้องการ
    //     doc.setFont("THSarabunNew", "normal");
    //     //set text
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

    //     //set line style and position
    //     doc.setLineWidth(0.2); //size of line
    //     doc.line(130, 265, 200, 265); //(startX, startY, endX, endY)
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
    //     doc.text(`Number: ${row.billing_number}`, 150, 45);
    //     doc.text(`Address: `, 10, 50);
    //     doc.text(`Date: ${Bill_date}`, 150, 50);
    //     doc.text(`Contact: `, 10, 55);
    //     doc.text(`E-mail: `, 10, 60);
    //     doc.text(`Contact No: `, 10, 65);
    //     doc.text(`Tax ID: `, 10, 70);

    //     //set text group
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

    //     //calculate total price
    //     const total_price = productForms.reduce((total, form) => {
    //       return total + parseFloat(form.sale_price);
    //     }, 0);

    //     //calculate total discount
    //     const total_discount = productForms.reduce((total, form) => {
    //       return total + parseFloat(form.sale_discount || 0);
    //     }, 0);

    //     //calculate Net
    //     const net_price = total_price - total_discount;

    //     //calculate Vat
    //     const vat = (7 / 100) * net_price;

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
    //     doc.text("Bill Receiver", 100, 255);
    //     doc.text("Buyer", 10, 40);
    //     doc.text(`Salesperson`, 10, 215);

    //     // เพิ่มฟอนต์
    //     doc.addFileToVFS("THSarabunNew-normal.ttf", thSarabunFont);
    //     doc.addFont("THSarabunNew-normal.ttf", "THSarabunNew", "normal");

    //     // ตั้งค่าฟอนต์ที่ต้องการ
    //     doc.setFont("THSarabunNew", "normal");

    //     //setImage(Image data, file type, X,Y,width,height)
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
    //     //draw table
    //     this.drawHeader(doc, headerText, startY, margin);
    //     this.drawTable(doc, currentPageData, startY, margin, lineHeight);
    //   }

    //   //set action for view and download
    //   if (action === "view") {
    //     const pdfBlob = doc.output("blob");
    //     const pdfUrl = URL.createObjectURL(pdfBlob);
    //     this.pdfUrl = pdfUrl;
    //     // this.isPopupPDFOpen = true;
    //     // เปิดในแท็บใหม่
    //     window.open(pdfUrl, "_blank");
    //   } else if (action === "download") {
    //     doc.save(`receipt-${row.cus_name}-${row.billing_number}.pdf`);
    //   }
    // },
    //draw header table
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
    //draw table
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
    //Close Pdf view
    ClosePDFview() {
      this.isPopupPDFOpen = false;
    },
    //close popup Billing
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
        total_price: 0, //cal all product
        total_discount: 0,
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
      this.getBilling();
      this.isPopupVisible_error = false;
    },
    //get billing data
    async getBilling() {
      const accessToken = localStorage.getItem("@accessToken");
      try {
        const response = await fetch(`${API_CALL}/quotation/getBilling`, {
          headers: {
            Authorization: `Bearer ${accessToken}`,
          },
        });
        const json = await response.json();
        console.log("--->", json.data);

        if (json.statusCode === 200) {
          console.log("---->", json.data);
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

            const formatNumber = (number) => {
              return new Intl.NumberFormat("en-US", {
                minimumFractionDigits: 2,
                maximumFractionDigits: 2,
              }).format(number);
            };
            let total_before_vat;
            let vat_in;

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
              billing_number: item.billing,
              tax_invoice_number: item.tax_invoice_number,
              invoice_number: item.invoice_number,
              quotation_num: item.quotation_num,
              // billing_status: item.billing_status,
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
              invoice_id: item.invoice_id,
              remark: item.remark || "",
              ID: item.billing_id,
              billing_date: BillingDate,
              payments: item.payments,
              pay_bank: item.pay_bank,
              pay_number: item.pay_number,
              pay_branch: item.pay_branch,
              pay_date: item.pay_date,
              deleted_at: item.deleted_at,
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
    //edit billing data
    async editBilling() {
      const accessToken = localStorage.getItem("@accessToken");
      console.log("sale price: ", this.formData.remark);
      // const date_billing_date = new Date(this.formData.billing_date); // ค่าที่ได้
      // date_billing_date.setFullYear(date_billing_date.getFullYear() - 543); // ลบ 543 ปี

      if (this.formData.billing_date === "" || this.formData.payments === "") {
        this.inputError = true;
        this.showPopup_error("Please fill all data");
        this.result = false;
      } else {
        console.log("Sale number: ", this.formData.billing_number);
        console.log("ID number: ", this.formData.billing_id);
        try {
          const in_id = this.formData.billing_id;
          // alert(in_id);
          const response = await fetch(
            `${API_CALL}/quotation/editBilling/${in_id}`,
            {
              method: "PUT",
              headers: {
                "Content-Type": "application/json",
                Authorization: `Bearer ${accessToken}`,
              },
              body: JSON.stringify({
                billing_date: this.formData.billing_date,
                billing_number: this.formData.billing_number,
                payments: this.formData.payments,
                remark: this.formData.remark,
                pay_bank: this.formData.pay_bank,
                pay_number: this.formData.pay_number,
                pay_branch: this.formData.pay_branch,
                pay_date: this.formData.pay_date ? new Date(this.formData.pay_date).toISOString().split('T')[0] : "",
              }),
            }
          );
          const json = await response.json();

          if (json.statusCode === 200) {
            this.getBilling();
            this.showPopup(this.$t("validation.EditSucc"));
            this.closePopup();
          } else {
            this.showPopup_error(json.data);
            console.log("Employee ", json);
          }
        } catch (error) {
          console.error("Error fetching data:", error);
        }
      }
    },
    //delete billing data
    async deleteBilling() {
      const accessToken = localStorage.getItem("@accessToken");
      this.isLoading = true;
      const qty_id = this.formData.invoice_id;
      // alert(qty_id);
      // return false;

      try {
        const response = await fetch(
          `${API_CALL}/quotation/deleteBilling/${qty_id}`,
          {
            method: "DELETE",
            headers: {
              "Content-Type": "application/json",
              Authorization: `Bearer ${accessToken}`,
            },
            body: JSON.stringify({
              billing_id: qty_id,
            }),
          }
        );
        const json = await response.json();
        console.log(json);

        if (json.statusCode === 200) {
          this.getBilling();
          this.showPopup(this.$t("validation.DelateSucc"));
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
    //get Employee
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
      console.log("🔍 getProduct() called");
      try {
        const response = await fetch(`${API_CALL}/product/getProduct`, {
          headers: {
            Authorization: `Bearer ${accessToken}`,
          },
        });
        const json = await response.json();
        console.log("📦 API Response:", json);

        if (json.statusCode === 200) {
          console.log("✅ Products from API:", json.data.length);
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
              Status: item.Status || "Active", // ✅ Added Status field
              pro_unti: item.pro_unti || "", // ✅ Added unit field
            };
          });
          console.log("✅ Products array after mapping:", this.Products);
        } else {
          this.showPopup_error(json.data);
          console.log("❌ Product API Error: ", json);
        }
      } catch (error) {
        console.error("❌ Error fetching products:", error);
      } finally {
        this.isLoading = false;
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
  },
  created() {
    this.getBilling();
    this.getBusiness();
    this.getProduct();
    this.getEmployee();
    this.getBanks();
  },
};
</script>

<style scoped>
/* ปรับขนาดของแต่ละคอลัมน์ */
.product-name-column {
  width: 30%;
  /* กำหนดพื้นที่มากที่สุดสำหรับชื่อสินค้า */
  min-width: 150px !important;
}

.price-column {
  width: 15%;
  /* รองรับข้อมูลยาวสำหรับ Price */
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
  width: auto;
  /* ให้ขนาดปรับตามเนื้อหา */
  white-space: nowrap;
  /* บังคับให้ข้อความอยู่บรรทัดเดียว */
  text-align: center;
}

.total-price-column {
  width: 15%;
  /* รองรับข้อมูลยาวสำหรับ Price */
  min-width: 130px !important;
}

.action-column {
  width: 10%;
  /* คอลัมน์ปุ่มลบ */
  min-width: 10px !important;
}

.show-only-desktop table thead tr th:nth-child(7),
.show-only-desktop table tbody tr td:nth-child(7) {
  display: block !important;
}

/* Fix v-date-picker blue highlight issue - More aggressive approach */
.vc-day-content {
  background-color: transparent !important;
}

.vc-day.is-not-in-month .vc-day-content {
  opacity: 0.4;
}

.vc-day:hover .vc-day-content {
  background-color: #e5e7eb !important;
}

.vc-day.is-today .vc-day-content {
  background-color: #3b82f6 !important;
  color: white !important;
}

.vc-highlight {
  display: none !important;
}

.vc-highlights {
  display: none !important;
}

.add-product-btn {
  width: 100px;
  background-color: #007bff;
  color: white;
  border-radius: 8px;
  border: 1px solid #007bff;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.add-product-btn:hover {
  background-color: white;
  color: #007bff;
  border: 1px solid #007bff;
}

.add-product-btn:active {
  background-color: white;
  color: #007bff;
  border: 1px solid #007bff;
}
</style>
