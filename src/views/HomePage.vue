<template>
  <ion-page>
    <ion-content :fullscreen="true" class="page-bg">

      <div class="wrapper">
        <div class="card">

          <!-- HEADER -->
          <div class="header">
         
              <img
                src="https://lsu-media-styles.sgp1.digitaloceanspaces.com/lsu-public-images/banners/logo/lsu-corporate-logo-white.png"
                class="logo"
                alt="LSU Logo"
              />
         
            <h1 class="header-title">IT Services Request</h1>
            <p class="header-subtitle">Get technical support quickly and easily</p>
          </div>

          <!-- PROGRESS INDICATOR -->
          <div class="progress-bar">
            <div
              class="progress-fill"
              :style="{ width: progressPercentage + '%' }"
            ></div>
          </div>

          <!-- SECTION -->
          <div class="section-header">
            <ion-icon name="person-circle-outline" class="section-icon"></ion-icon>
            <p class="section-title">Personal Information</p>
          </div>

          <!-- FULL NAME -->
<ion-item class="modern-input" lines="none">
  <ion-input
    v-model="info.requestor_fullname"
    label="Full Name"
    label-placement="stacked"
    placeholder="e.g. Juan Dela Cruz"
  >
  </ion-input>
</ion-item>
          <!-- EMAIL -->
          <ion-item class="modern-input" lines="none">
            <ion-input
              v-model="info.requestor_lsu_email"
              label="Email Address *"
              label-placement="stacked"
              type="email"
              placeholder="e.g. juan.delacruz@lsu.edu.ph"
            >
            </ion-input>
          </ion-item>
          <p class="helper-text">Use your official LSU email address</p>

          <!-- ROLE -->
          <ion-item class="modern-input" lines="none">
            <ion-select
              v-model="info.client_role"
              label="Role *"
              label-placement="stacked"
              interface="action-sheet"
              placeholder="Select your role"
            >
              <ion-select-option value="Student">Student</ion-select-option>
              <ion-select-option value="Faculty">Faculty</ion-select-option>
              <ion-select-option value="Staff">Staff</ion-select-option>
              <ion-select-option value="Admin">Admin</ion-select-option>
            </ion-select>
          </ion-item>

          <!-- SECTION -->
          <div class="section-header">
            <ion-icon name="clipboard-outline" class="section-icon"></ion-icon>
            <p class="section-title">Request Details</p>
          </div>

          <!-- CATEGORY -->
          <ion-item class="modern-input" lines="none">
            <ion-select
              v-model="info.issue_concern_request_category_type"
              label="Category *"
              label-placement="stacked"
              interface="action-sheet"
              placeholder="Select category"
            >
              <ion-select-option
                v-for="cat in CATEGORY_OPTIONS"
                :key="cat"
                :value="cat"
              >
                {{ cat }}
              </ion-select-option>
            </ion-select>
          </ion-item>

          <!-- SPECIFIC -->
          <ion-item
            v-if="info.issue_concern_request_category_type"
            class="modern-input animated-item"
            lines="none"
          >
            <ion-select
              v-model="info.issue_concern_request_item_type"
              label="Specific Concern"
              label-placement="stacked"
              interface="action-sheet"
              placeholder="Select specific concern"
            >
              <ion-select-option
                v-for="type in getItemOptions(info.issue_concern_request_category_type)"
                :key="type"
                :value="type"
              >
                {{ type }}
              </ion-select-option>
            </ion-select>
          </ion-item>

          <!-- DESCRIPTION -->
          <ion-item class="modern-input" lines="none">
            <ion-textarea
              v-model="info.issue_concern_request_details"
              label="Description *"
              label-placement="stacked"
              :rows="4"
              placeholder="Briefly describe your issue or concern..."
              auto-grow
            >
            </ion-textarea>
          </ion-item>

          <!-- OFFICE -->
          <ion-item
            v-if="requiresOffice"
            class="modern-input animated-item"
            lines="none"
          >
            <ion-select
              v-model="info.issue_concern_request_center_office_room"
              label="Office Location"
              label-placement="stacked"
              interface="action-sheet"
              placeholder="Select your office location"
            >
              <ion-select-option
                v-for="office in getLocationOptions"
                :key="office"
                :value="office"
              >
                {{ office }}
              </ion-select-option>
            </ion-select>
          </ion-item>

          <!-- VALIDATION MESSAGE -->
          <div v-if="showValidationError" class="validation-error animated-item">
            <ion-icon name="alert-circle-outline"></ion-icon>
            <span>Please fill in all required fields</span>
          </div>

          <!-- BUTTON -->
          <ion-button
            expand="block"
            size="large"
            class="submit-btn"
            @click="handleSubmitClick"
            :disabled="!isFormValid"
          >
            <ion-icon name="send-outline" slot="start"></ion-icon>
            SUBMIT
          </ion-button>

          <!-- FOOTER NOTE -->
          <p class="footer-note">
            <ion-icon name="information-circle-outline"></ion-icon>
            You will receive a confirmation email once your request is submitted
          </p>

        </div>
      </div>

      <ion-loading
        :is-open="modalLoading"
        message="Submitting your request..."
        spinner="crescent"
      />

      <ion-alert
        :is-open="showSuccess"
        header="✅ Success!"
        sub-header="Request Submitted"
        message="Your IT support request has been successfully submitted. We'll get back to you soon."
        :buttons="[{
          text: 'Done',
          role: 'confirm',
          handler: () => resetForm()
        }]"
      />

      <ion-toast
        :is-open="showToast"
        :message="toastMessage"
        :duration="3000"
        position="top"
        color="warning"
        @didDismiss="showToast = false"
      />

    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage, IonContent,
  IonInput, IonTextarea,
  IonSelect, IonSelectOption,
  IonButton, IonAlert, IonLoading,
  IonItem, IonIcon, IonToast
} from "@ionic/vue";

import { ref, computed, watch } from "vue";
import { Capacitor } from "@capacitor/core";
import itServiceConfig from "./it-service-config.json";

// API Configuration
const API_ENDPOINT = "https://apipdp.lsu.edu.ph/buang_ka_eyy/api/cits/request-ticket/create/";

const CATEGORY_OPTIONS = itServiceConfig.categoryOptions;
const ITEM_TYPE_OPTIONS_MAP = itServiceConfig.itemTypeOptionsMap as any;
const CENTER_OFFICE_ROOM_OPTIONS = itServiceConfig.centerOfficeRoomOptions;

const modalLoading = ref(false);
const showSuccess = ref(false);
const showValidationError = ref(false);
const showToast = ref(false);
const toastMessage = ref("");

const defaultForm = () => ({
  ticket_id: "TID" + Date.now(),
  requestor_fullname: "",
  requestor_lsu_email: "",
  issue_concern_request_details: "",
  issue_concern_request_category_type: "",
  issue_concern_request_item_type: "",
  issue_concern_request_center_office_room: "",
  client_role: "",
  owner_type: "LSU",
});

const info = ref(defaultForm());

const getItemOptions = (category: string) => {
  return ITEM_TYPE_OPTIONS_MAP[category] || [];
};

const getLocationOptions = computed(() => CENTER_OFFICE_ROOM_OPTIONS);

const requiresOffice = computed(() =>
  ["Admin", "Faculty", "Staff"].includes(info.value.client_role)
);

// Progress calculation
const progressPercentage = computed(() => {
  const fields = [
    info.value.requestor_fullname,
    info.value.requestor_lsu_email,
    info.value.client_role,
    info.value.issue_concern_request_category_type,
    info.value.issue_concern_request_details,
  ];
  const filled = fields.filter(f => f && f.trim()).length;
  return Math.round((filled / fields.length) * 100);
});

// Form validation
const isFormValid = computed(() => {
  return !!(
    info.value.requestor_fullname?.trim() &&
    info.value.requestor_lsu_email?.trim() &&
    info.value.client_role &&
    info.value.issue_concern_request_category_type
  );
});

// Reset validation error when form changes
watch(() => info.value, () => {
  if (showValidationError.value) {
    showValidationError.value = false;
  }
}, { deep: true });

const handleSubmitClick = async () => {
  if (!isFormValid.value) {
    showValidationError.value = true;
    toastMessage.value = "Please fill in all required fields marked with *";
    showToast.value = true;
    return;
  }

  modalLoading.value = true;

  // Normalize office field
  if (requiresOffice.value) {
    // Office is required and should be filled
  } else {
    // Auto-set to N/A for non-Admin/Faculty/Staff
    info.value.issue_concern_request_center_office_room = "N/A";
  }

  // Auto-set owner_type to LSU (default)
  info.value.owner_type = "LSU";

  // Create FormData for submission
  const formData = new FormData();
  formData.append("ticket_id", info.value.ticket_id);
  formData.append("requestor_fullname", info.value.requestor_fullname.trim());
  formData.append("requestor_lsu_email", info.value.requestor_lsu_email.trim());
  formData.append("technicians_assigned", JSON.stringify([]));
  formData.append("issue_concern_request_details", info.value.issue_concern_request_details?.trim() || "");
  formData.append("issue_concern_request_category_type", info.value.issue_concern_request_category_type);
  formData.append("issue_concern_request_item_type", info.value.issue_concern_request_item_type || "");
  formData.append("issue_concern_request_center_office_room", info.value.issue_concern_request_center_office_room);
  formData.append("owner_type", "LSU");
  formData.append("client_role", info.value.client_role);
  formData.append("buy_me_coffee", "No");
  formData.append("logs", JSON.stringify([
    {
      timestamp: new Date().toISOString(),
      status: "Pending",
      remarks: "Initial status",
      assigned_technician_name: "",
      assigned_technician_lsu_email: "",
    }
  ]));

  try {
    // Use native fetch - works in both web and native platforms
    const res = await fetch(API_ENDPOINT, {
      method: "POST",
      body: formData,
      // Add headers for better compatibility
      headers: {
        'Accept': 'application/json',
      },
    });

    // Check if response is ok
    if (!res.ok) {
      throw new Error(`HTTP error! status: ${res.status}`);
    }

    const data = await res.json();

    if (data.status === "created") {
      modalLoading.value = false;
      showSuccess.value = true;
    } else if (data.status === "errors") {
      modalLoading.value = false;
      console.error("Form errors:", data.errors);
      toastMessage.value = "Failed to submit: " + (data.errors ? JSON.stringify(data.errors) : "Unknown error");
      showToast.value = true;
    } else {
      modalLoading.value = false;
      toastMessage.value = "Failed to submit request. Please try again.";
      showToast.value = true;
    }
  } catch (err) {
    console.error("Failed to create ticket:", err);
    modalLoading.value = false;

    // Provide more specific error messages
    if (err instanceof TypeError && err.message.includes('Failed to fetch')) {
      toastMessage.value = "Network error. Please check your internet connection.";
    } else {
      toastMessage.value = "Error submitting request. Please try again.";
    }
    showToast.value = true;
  }
};

const resetForm = () => {
  showSuccess.value = false;
  info.value = defaultForm();
  showValidationError.value = false;
};
</script>

<style scoped>
.page-bg {
  --background: #f9fafb;
}

.wrapper {
  min-height: 100%;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 16px 12px;
}

.card {
  width: 100%;
  max-width: 440px;
  background: white;
  padding: 0;
  border-radius: 0;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

/* HEADER */
.header {
  text-align: center;
  background: #087830;
  color: white;
  padding: 20px 20px 16px;
  margin-bottom: 0;
}

.logo-wrapper {
  background: white;
  width: 56px;
  height: 56px;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 12px;
  padding: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.logo {
  width: 200px;
  height: auto;
}

.header-title {
  font-size: 16px;
  font-weight: 700;
  color: white;
  margin: 8px 0 4px;
  letter-spacing: 0.5px;
  text-transform: uppercase;
}

.header-subtitle {
  font-size: 12px;
  color: rgba(255, 255, 255, 0.9);
  margin: 0;
}

/* PROGRESS BAR */
.progress-bar {
  width: 100%;
  height: 4px;
  background: rgba(255, 255, 255, 0.3);
  overflow: hidden;
  margin: 0;
}

.progress-fill {
  height: 100%;
  background: rgba(255, 255, 255, 0.9);
  transition: width 0.3s ease;
}

/* TEXT */
.title {
  font-size: 16px;
  font-weight: 600;
  color: #111827;
  margin: 0 0 4px;
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 0 20px;
}

.title-icon {
  font-size: 20px;
  color: #087830;
}

.subtitle {
  font-size: 13px;
  color: #6b7280;
  margin: 0 0 20px;
  padding: 0 20px;
}

.section-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin: 0;
  padding: 16px 20px 12px;
  background: #f9fafb;
  border-top: 3px solid #087830;
  border-bottom: 1px solid #e5e7eb;
}

.section-icon {
  font-size: 16px;
  color: #087830;
}

.section-title {
  font-size: 13px;
  font-weight: 700;
  color: #000000;
  margin: 0;
  text-transform: uppercase;
  letter-spacing: 0.8px;
}

/* INPUT */
.modern-input {
  margin-bottom: 18px;
  --background: transparent;
  --padding-start: 20px;
  --padding-end: 20px;
    --border-width: 0;
  border-bottom: 1px solid #d1d5db; /* gray-300 */
    --color: #000000; /* selected text */
  --placeholder-color: #000000; /* placeholder text */
  color: #000000;
}

.modern-input:focus-within {
  border-bottom: 2px solid #16a34a; /* green */
}

.modern-input ion-label {
  color: #000000 !important;
}

.animated-item {
  animation: slideIn 0.3s ease-out;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* INPUT STYLING */
ion-input,
ion-textarea,
ion-select {
  --border-radius: 0;
  --background: transparent;
  --color: #111827;
  --placeholder-color: #c7c7c7;
  --border-color: #087830;
  --border-width: 0 0 1px 0;
  --padding-start: 0;
  --padding-end: 0;
  --padding-top: 8px;
  --padding-bottom: 8px;
  font-size: 15px;
}

ion-item {
  --border-color: transparent;
  --background: transparent;
  --padding-start: 20px;
  --padding-end: 20px;
}

/* Labels */
ion-input::part(label),
ion-textarea::part(label),
ion-select::part(label) {
  font-weight: 600;
  color: #000000;
  font-size: 14px;
}

/* FOCUS STATE */
ion-input:focus-within,
ion-textarea:focus-within,
ion-select:focus-within {
  --border-color: #087830;
  --border-width: 0 0 2px 0;
}

/* HELPER */
.helper-text {
  font-size: 11px;
  color: #6b7280;
  margin: -12px 20px 16px;
  display: block;
}

/* VALIDATION ERROR */
.validation-error {
  background: #fef2f2;
  border-left: 4px solid #dc2626;
  padding: 12px 20px;
  margin: 0 0 16px;
  display: flex;
  align-items: center;
  gap: 8px;
  color: #dc2626;
  font-size: 13px;
  font-weight: 500;
}

.validation-error ion-icon {
  font-size: 20px;
  flex-shrink: 0;
}

/* BUTTON */
.submit-btn {
  margin: 24px 20px 16px;
  --background: #126c32;
  --background-activated: #087830;
  --background-hover: #064b2a;
  --border-radius: 4px;
  --padding-top: 14px;
  --padding-bottom: 14px;
  font-weight: 700;
  letter-spacing: 1.5px;
  text-transform: uppercase;
  font-size: 13px;
  --box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  color: #fff;
}

.submit-btn:disabled {
  opacity: 0.4;
  --background: #d1d5db;
}

/* FOOTER NOTE */
.footer-note {
  text-align: center;
  font-size: 11px;
  color: #6b7280;
  margin: 0 20px 20px;
  padding: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  background: #f9fafb;
  border-radius: 4px;
  line-height: 1.4;
}

.footer-note ion-icon {
  font-size: 16px;
  flex-shrink: 0;
}
</style>