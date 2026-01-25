<template>
  <div class="main-page">
    <div class="h-screen pt-8 px-2" style="margin-top: 60px;">
      <!-- Section Header -->
      <div class="row justify-content-center mb-4">
        <div class="col-12 col-lg-10">
           <h2 class="section-title">เมนูลัด</h2>
        </div>
      </div>

      <!-- Shortcuts Cards -->
      <div class="row justify-content-center mx-2">
        <!-- Billing Note Card -->
        <div class="col-md-6 col-lg-6 col-xl-6 mb-4 lg:mx-2">
           <div class="shortcut-card card-billing"  @click="accessBillingNote">
              <div class="icon-wrapper">
                 <span class="mdi mdi-receipt-text-outline"></span>
              </div>
              <div class="card-info">
                 <h3 class="card-title">{{ t("headerBilling") }}</h3>
                 <span class="card-desc">{{ t("descBilling") }}</span>
              </div>
              <div class="card-decoration"></div>
           </div>
        </div>

        <!-- Product Card -->
        <div class="col-md-6 col-lg-6 col-xl-6 mb-4 lg:mx-2">
           <div class="shortcut-card card-product" @click="accessProduct">
              <div class="icon-wrapper">
                 <span class="mdi mdi-cube-outline"></span>
              </div>
              <div class="card-info">
                 <h3 class="card-title">{{ t("headerProduct") }}</h3>
                 <span class="card-desc">{{ t("descProduct") }}</span>
              </div>
              <div class="card-decoration"></div>
           </div>
        </div>

        <!-- Customer Card -->
        <div class="col-md-6 col-lg-6 col-xl-6 mb-4 lg:mx-2">
           <div class="shortcut-card card-customer" @click="accessCustomer">
              <div class="icon-wrapper">
                 <span class="mdi mdi-account-group-outline"></span>
              </div>
              <div class="card-info">
                 <h3 class="card-title">{{ t("headerCustomer") }}</h3>
                 <span class="card-desc">{{ t("descCustomer") }}</span>
              </div>
              <div class="card-decoration"></div>
           </div>
        </div>

        <!-- About Business Card -->
        <div class="col-md-6 col-lg-6 col-xl-6 mb-4 lg:mx-2">
           <div class="shortcut-card card-about" @click="accessAbout">
              <div class="icon-wrapper">
                 <span class="mdi mdi-domain"></span>
              </div>
              <div class="card-info">
                 <h3 class="card-title">{{ t("headerAbout") }}</h3>
                 <span class="card-desc">{{ t("descAbout") }}</span>
              </div>
              <div class="card-decoration"></div>
           </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { config } from "../../constant.js";
import { useI18n } from "vue-i18n";

const API_CALL = config["url"];

export default {
  name: "HomePage",
  components: {},
  setup() {
    const { t } = useI18n();
    return {
      t,
    };
  },
  methods: {
    accessProduct() {
      this.$router.push("/product");
    },
    accessBillingNote() {
      this.$router.push("/billingnote");
    },
    accessCustomer() {
      this.$router.push("/customer");
    },
    accessAbout() {
      this.$router.push("/aboutcompany");
    },
    TestToken() {
      const token = localStorage.getItem("@accessToken");
      const userRole = localStorage.getItem("RoleName");
      const userID = localStorage.getItem("user_id");
      if (token) {
        console.log("Token in Home page", token);
        console.log("User Role : ", userRole);
        console.log("User ID : ", userID);
      } else {
        console.log("No token found in Home page");
      }
    },
  },
  created() {
    this.TestToken();
  },
};
</script>

<style scoped>
.section-title {
  font-size: 28px;
  font-weight: 700;
  color: #495057;
  border-left: 6px solid #4facfe;
  padding-left: 15px;
  margin-bottom: 20px;
}

.shortcut-card {
  background: white;
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);
  padding: 30px;
  display: flex;
  align-items: center;
  position: relative;
  overflow: hidden;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
  height: 100%;
}

.shortcut-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.12);
}

.icon-wrapper {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 25px;
  z-index: 2;
  flex-shrink: 0;
}

.icon-wrapper .mdi {
  font-size: 36px;
  color: white;
}

.card-info {
  display: flex;
  flex-direction: column;
  z-index: 2;
}

.card-title {
  font-size: 22px;
  font-weight: 700;
  color: #2c3e50;
  margin: 0 0 8px 0;
}

.card-desc {
  font-size: 15px;
  color: #7f8c8d;
}

/* Specific Card Styles */
/* Billing Card - Blue/Purple Gradient */
.card-billing .icon-wrapper {
  background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
  box-shadow: 0 8px 16px rgba(79, 172, 254, 0.3);
}

.card-billing .card-decoration {
  position: absolute;
  top: -20px;
  right: -20px;
  width: 120px;
  height: 120px;
  background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
  opacity: 0.1;
  border-radius: 50%;
  z-index: 1;
}

.card-billing:hover .card-title {
  color: #007bff;
}

/* Product Card - Orange/Pink Gradient */
.card-product .icon-wrapper {
  background: linear-gradient(135deg, #ff9a9e 0%, #fecfef 99%, #fecfef 100%); 
  /* Using a more vibrant orange/red gradient based on popular dashboard trends */
  background: linear-gradient(135deg, #FF8008 0%, #FFC837 100%);
  box-shadow: 0 8px 16px rgba(255, 128, 8, 0.3);
}

.card-product .card-decoration {
  position: absolute;
  top: -20px;
  right: -20px;
  width: 120px;
  height: 120px;
  background: linear-gradient(135deg, #FF8008 0%, #FFC837 100%);
  opacity: 0.1;
  border-radius: 50%;
  z-index: 1;
}

.card-product:hover .card-title {
  color: #e67e22;
}

/* Customer Card - Green Gradient */
.card-customer .icon-wrapper {
  background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);
  box-shadow: 0 8px 16px rgba(17, 153, 142, 0.3);
}

.card-customer .card-decoration {
  position: absolute;
  top: -20px;
  right: -20px;
  width: 120px;
  height: 120px;
  background: linear-gradient(135deg, #11998e 0%, #38ef7d 100%);
  opacity: 0.1;
  border-radius: 50%;
  z-index: 1;
}

.card-customer:hover .card-title {
  color: #11998e;
}

/* About Card - Purple Gradient */
.card-about .icon-wrapper {
  background: linear-gradient(135deg, #8E2DE2 0%, #4A00E0 100%);
  box-shadow: 0 8px 16px rgba(142, 45, 226, 0.3);
}

.card-about .card-decoration {
  position: absolute;
  top: -20px;
  right: -20px;
  width: 120px;
  height: 120px;
  background: linear-gradient(135deg, #8E2DE2 0%, #4A00E0 100%);
  opacity: 0.1;
  border-radius: 50%;
  z-index: 1;
}

.card-about:hover .card-title {
  color: #8E2DE2;
}

@media (max-width: 768px) {
  .shortcut-card {
    padding: 20px;
  }
  .icon-wrapper {
    width: 60px;
    height: 60px;
    margin-right: 15px;
  }
  .icon-wrapper .mdi {
    font-size: 28px;
  }
  .card-title {
    font-size: 18px;
  }
}
</style>
