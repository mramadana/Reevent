<template>
    <div class="container">

        <div class="custom-width with-auth">

            <h1 class="main-title bold lg mb-3">{{ $t("Auth.create_account") }}</h1>
            <h3 class="desc mb-4">{{ $t("Auth.enter_data_to_login") }}</h3>
            
            <!-- Sign up form -->
            <form @submit.prevent="signUp" ref="signUpForm">
                <div class="row">
                    <div class="col-12 col-md-8 mr-auto">

                        <!-- Profile image upload -->
                        <div class="position-relative single-input-upload mb-4">
                            <div class="main_input special-input without-edit">
                                <div class="d-flex align-items-center justify-content-center gap-2 flex-grow-1 gray">
                                    <span>{{ $t("Global.attach_photo") }}</span>
                                </div>
                            </div>
                            <GlobalImgUploader 
                                ref="profileImageUploader"
                                acceptedFiles="image/*" 
                                :IsMultible="false" 
                                :resetTrigger="resetImageTrigger"
                                :showValidation="showValidation"
                                :required="true"
                                :errorMessage="t('validation.image_required')"
                                @uploaded-images-updated="updateUploadedImages_1" />
                        </div>

                        <!-- Provider name input -->
                        <FormInput 
                            v-model:modelValue="formData.name"
                            name="name"
                            type="text"
                            :label="$t('Auth.provider_Name')"
                            :placeholder="$t('Auth.full_name')"
                            :validation-schema="validations.name"
                            :showErrors="showValidation"
                        />

                        <!-- Mobile number input with country dropdown -->
                        <div class="form-group">
                            <label class="label">
                                {{ $t('Auth.mobile_number') }}
                            </label>
                            <div class="with_cun_select">
                                <div class="main_input" :class="{ 'is-invalid': showValidation && getValidationError('phone') }">
                                    <input 
                                        type="number" 
                                        name="phone" 
                                        v-model="formData.phone" 
                                        :placeholder="$t('Auth.please_mobile_number')"
                                        class="custum-input-icon"
                                    >
                                </div>
                                <GlobalCountryDropdown
                                    v-model="selectedCountry"
                                    :placeholder="$t('Auth.select_country')"
                                />          
                            </div>
                            <div v-if="showValidation && getValidationError('phone')" class="text-danger mt-1" style="font-size: 12px;">
                                <span>{{ getValidationError('phone') }}</span>
                            </div>
                            <div v-if="showValidation && !selectedCountry" class="text-danger mt-1" style="font-size: 12px;">
                                <span>{{ t('validation.required_with_label', { field: t('Auth.select_country') }) }}</span>
                            </div>
                        </div>

                        <!-- Scope of work input -->
                        <FormInput 
                            v-model:modelValue="formData.scope_work"
                            name="scope_work"
                            type="text"
                            :label="$t('Auth.scope_work')"
                            :placeholder="$t('Auth.enter_scope_work')"
                            :validation-schema="validations.scope_work"
                            :showErrors="showValidation"
                        />

                        <!-- Service provider location input -->
                        <div class="form-group">
                            <label class="label">
                                {{ $t('Auth.service_provider_location') }}
                            </label>
                            <div class="main_input pointer" @click="openmodal" :class="{ 'is-invalid': showValidation && getValidationError('address') }">
                                <input 
                                    name="address" 
                                    v-model="formData.address" 
                                    class="custum-input-icon pointer" 
                                    readonly 
                                    :placeholder="$t('Auth.enter_service_provider_location')"
                                >
                                <button class="static-btn" type="button">
                                    <font-awesome-icon :icon="['fas', 'location-crosshairs']" class="icon" />
                                </button>
                            </div>
                            <div v-if="showValidation && getValidationError('address')" class="text-danger mt-1" style="font-size: 12px;">
                                <span>{{ getValidationError('address') }}</span>
                            </div>
                        </div>

                        <!-- Password input with visibility toggle -->
                        <div class="form-group">
                            <label class="label">
                                {{ $t('Auth.password') }}
                            </label>
                            <div class="main_input">
                                <input :type="inputType('definitelyNewPassword')" name="password" v-model="formData.password" class="custum-input-icon" :class="{ 'is-invalid': showValidation && getValidationError('password') }" :placeholder=" $t('Auth.please_enter_password') ">
                                <button class="static-btn with_eye" type="button" @click="togglePasswordVisibility('definitelyNewPassword')" :class="{ 'active_class': passwordVisible.definitelyNewPassword }">
                                    <i class="far fa-eye icon"></i>
                                </button>
                            </div>
                            <div v-if="showValidation && getValidationError('password')" class="text-danger mt-1" style="font-size: 12px;">
                                <span>{{ getValidationError('password') }}</span>
                            </div>
                        </div>

                        <!-- Shop images upload (optional) -->
                        <div class="position-relative single-input-upload mb-4">
                            <div class="main_input special-input without-edit">
                                <div class="d-flex align-items-center justify-content-center gap-2 flex-grow-1 gray">
                                    <span>{{ $t("Global.attach_photos") }}</span>
                                </div>
                            </div>
                            <GlobalImgUploader
                                ref="shopImagesUploader"
                                acceptedFiles="image/*, application/*" 
                                :IsMultible="true"
                                :resetTrigger="resetImageTrigger"
                                :showValidation="false"
                                :required="false"
                                @uploaded-images-updated="updateUploadedImages_2" 
                            />
                        </div>

                        <!-- Delivery options -->
                        <div class="mb-3">
                            <label class="form-label">{{ $t('Auth.delivery_options') }}</label>
                            <div class="row">
                                <div class="col-12 col-lg-6">
                                    <label class="inner-radio">
                                        <input type="radio" class="d-none radio-input" name="delivery" v-model="formData.delivery" value="is_delivery">
                                        <div class="check-inner">
                                            <p class="hint-inner">
                                                <span class="radio-body"></span>
                                            </p>
                                            <p class="hint-text">
                                                {{ $t('Auth.delivery_available') }}
                                            </p>
                                        </div>
                                    </label>
                                </div>
        
                                <div class="col-12 col-lg-6">
                                    <label class="inner-radio">
                                        <input type="radio" class="d-none radio-input" name="delivery" v-model="formData.delivery" value="not_delivery">
                                        <div class="check-inner">
                                            <p class="hint-inner">
                                                <span class="radio-body"></span>
                                            </p>
                                            <p class="hint-text">
                                                {{ $t('Auth.delivery_not_available') }}
                                            </p>
                                        </div>
                                    </label>
                                </div>
                            </div>
                            <div v-if="showValidation && getValidationError('delivery')" class="text-danger mt-1" style="font-size: 12px;">
                                <span>{{ getValidationError('delivery') }}</span>
                            </div>
                        </div>

                        <!-- Delivery Coverage -->
                        <div class="mb-3">
                            <label class="form-label">{{ $t('Auth.delivery_coverage') }}</label>
                            <div class="row">
                                <div class="col-12 col-lg-6">
                                    <label class="inner-radio">
                                        <input type="radio" class="d-none radio-input" name="delivery_country" value="in_iraq" v-model="formData.delivery_country">
                                        <div class="check-inner">
                                            <p class="hint-inner">
                                                <span class="radio-body"></span>
                                            </p>
                                            <p class="hint-text">
                                                {{ $t('Auth.all_over_Iraq') }}
                                            </p>
                                        </div>
                                    </label>
                                </div>
        
                                <div class="col-12 col-lg-6">
                                    <label class="inner-radio">
                                        <input type="radio" class="d-none radio-input" name="delivery_country" value="out_iraq" v-model="formData.delivery_country">
                                        <div class="check-inner">
                                            <p class="hint-inner">
                                                <span class="radio-body"></span>
                                            </p>
                                            <p class="hint-text">
                                                {{ $t('Auth.within_specific_governorates') }}
                                            </p>
                                        </div>
                                    </label>
                                </div>
                            </div>
                            <div v-if="showValidation && getValidationError('delivery_country')" class="text-danger mt-1" style="font-size: 12px;">
                                <span>{{ getValidationError('delivery_country') }}</span>
                            </div>
                        </div>
                        
                        <!-- Governorates selection for out-of-Iraq delivery -->
                        <div v-if="formData.delivery_country === 'out_iraq'">
                            <GlobalCustomDropdown 
                                v-model="formData.selectedCities" 
                                :options="cities"
                                option-value="code"
                                placeholder="اختر المحافظات" 
                                :label="$t('Auth.governorates_delivery')"
                                :showValidation="showValidation"
                            />
                        </div>
                        
                        <!-- Submit button -->
                        <button type="submit" class="custom-btn w-100 mr-auto" :disabled="loading">
                            {{ $t('Auth.confirmation') }}
                            <span class="spinner-border spinner-border-sm" v-if="loading" role="status"
                                    aria-hidden="true"></span>
                        </button>

                        <!-- Login link -->
                        <div class="new-sign mt-4">
                            {{ $t('Auth.already_have_account') }}
                            <nuxt-link to="/Auth/login" >{{ $t('Auth.login') }}</nuxt-link>
                        </div>
                    </div>
                </div>
            </form>
        </div>

        <!-- success modal -->
        <Dialog
            v-model:visible="successRegister"
            modal
            class="custum_dialog_width without-close"
            :draggable="false"
        >
            <div class="text-center">
                <h1 class="main-title bold mb-4 hint_success">
                    {{ $t('Auth.hint_success') }}
                </h1>
                <img
                    src="@/assets/images/check_img.svg"
                    alt="check-img"
                    class="check-img lg"
                    loading="lazy"
                />
            </div>
        </Dialog>

        <!-- global google map component -->
        <GlobalGoogleMap
            v-model:visible="visible"
            @closeModal="updateAddress"
            @updateAddress="handleUpdateAddress"
            @handleClose="handleClose"
            :show_inputs="show_inputs"
            :lat="location.lat"
            :lng="location.lng"
            :current_location="currentLocation"
            :isDraggable="true"
            :closeModal_btn="closeModal_btn"
            :current_location_button="true"
            :title= "$t('Global.current_location')"
        />
    </div>
</template>

<script setup>
import { useI18n } from 'vue-i18n';
import * as yup from 'yup';

// Composables and utilities
const { t } = useI18n({ useScope: 'global' });
const { successToast, errorToast } = toastMsg();
const { response } = responseApi();

// Validation setup
const showValidation = ref(false);
const { phoneNumber, fullName, required, radioButton } = useValidationSchema();
const { isFormValid, resetForm, scrollToFirstError } = useFormValidation();

// Form data
const formData = reactive({
    name: null,
    phone: null,
    scope_work: null,
    address: null,
    password: null,
    delivery: null,
    delivery_country: null,
    selectedCities: null
});

// Other refs
const successRegister = ref(false);
const { signUpHandler } = useAuthStore();
const store = useAuthStore();
const { lat, lng } = storeToRefs(store);
const signUpForm = ref(null);
const selectedCountry = ref(null);
const uploadedImage = ref([]);
const uploadedImage_2 = ref([]);
const loading = ref(false);
const resetImageTrigger = ref(0);

// Image uploader refs
const profileImageUploader = ref(null);
const shopImagesUploader = ref(null);

// Cities data
const cities = ref([
    { name: 'New York', code: 'NY' },
    { name: 'Rome', code: 'RM' },
    { name: 'London', code: 'LDN' },
    { name: 'Istanbul', code: 'IST' },
    { name: 'Paris', code: 'PRS' },
    { name: 'Tokyo', code: 'TKY' },
    { name: 'Beijing', code: 'BJG' },
    { name: 'Shanghai', code: 'SHG' },
]);

// Create password validation schema
const passwordSchema = () => {
    return yup
        .string()
        .required(t('validation.required_with_label', { field: t('Auth.password') }))
        .min(6, t('validation.password_min_length', { min: 6 }))
        .label(t('Auth.password'));
};

// Validation schemas
const validations = {
    name: fullName(),
    phone: phoneNumber(),
    scope_work: required('Auth.scope_work'),
    address: required('Auth.service_provider_location'),
    password: passwordSchema(),
    delivery: radioButton('Auth.delivery_options'),
    delivery_country: radioButton('Auth.delivery_coverage')
};

// Google map related
const closeModal_btn = ref(true);
const handleClose = () => {
    visible.value = false;
};
const handleUpdateAddress = (newAddress) => {
    location.value = newAddress;
};
const updateAddress = () => {
    formData.address = location.value.address;
    handleClose();
};
const currentLocation = ref(false);
const openmodal = () => {
    visible.value = true;
    setTimeout(() => {
        currentLocation.value = true;
    }, 300);
};
const location = ref({
    lat: lat.value,
    lng: lng.value
});
const show_inputs = ref(false);
const visible = ref(false);

// Password visibility
const passwordVisible = ref({
    definitelyNewPassword: false,
});

// Password visibility toggle
const togglePasswordVisibility = (input) => {
    passwordVisible.value[input] = !passwordVisible.value[input];
};
const inputType = (input) => {
    return passwordVisible.value[input] ? 'text' : 'password';
};

// Image upload handlers
const updateUploadedImages_1 = (images) => {
    uploadedImage.value = images;
};
const updateUploadedImages_2 = (images) => {
    uploadedImage_2.value = Array.isArray(images) ? images : [images];
};

// Function to get validation error for a specific field
const getValidationError = (field) => {
    if (!showValidation.value || !validations[field]) return null;
    try {
        validations[field].validateSync(formData[field]);
        return null;
    } catch (error) {
        return error.message;
    }
};

// Function to reset the form
const handleResetForm = () => {
    resetForm(formData, showValidation);
    uploadedImage.value = [];
    uploadedImage_2.value = [];
    selectedCountry.value = null;
    resetImageTrigger.value++;
};

// Sign up function with new validation
const signUp = async () => {
    showValidation.value = true;
    
    // Check profile image validation
    const profileImagesValid = profileImageUploader.value?.validate() || false;
    const isValid = isFormValid(formData, validations);
    const isCountrySelected = selectedCountry.value !== null;

    if (!isValid || !profileImagesValid || !isCountrySelected) {
        if (!profileImagesValid) {
            console.log("Profile image validation failed");
        } else if (!isCountrySelected) {
            console.log("Country selection is required");
            errorToast(t('validation.required_with_label', { field: t('Auth.select_country') }));
        } else if (!isValid) {
            scrollToFirstError(formData, validations);
        }
        return;
    }

    try {
        loading.value = true;
        
        // Create FormData and merge all data
        const fd = new FormData();
        
        // Add form data
        Object.entries(formData).forEach(([key, value]) => {
            if (value !== null) {
                fd.append(key, value);
            }
        });
        
        // Add country code
        if (selectedCountry.value?.key) {
            fd.append('country_code', selectedCountry.value.key);
        }
        
        // Add profile image
        if (uploadedImage.value.length > 0) {
            uploadedImage.value.forEach((file, index) => {
                fd.append(`profile_image[${index}]`, file);
            });
        }
        
        // Add shop images
        if (uploadedImage_2.value.length > 0) {
            uploadedImage_2.value.forEach((file, index) => {
                fd.append(`shop_images[${index}]`, file);
            });
        }
        
        const res = await signUpHandler(fd);
        
        if (res.status === "success") {
            successToast(res.msg);
            successRegister.value = true;
            handleResetForm();
        } else {
            errorToast(res.msg);
        }
        
        loading.value = false;
    } catch (error) {
        console.error('Form submission error:', error);
        errorToast('حدث خطأ أثناء التسجيل');
        loading.value = false;
    }
};

// Page meta
definePageMeta({
    name: "Auth.create_account",
    layout: 'auth'
});
</script>

<style lang="scss">
.single-input-upload {
    .special-input {
        +input {
            width: 100%;
            height: 45px;
            top: 0;
            position: absolute;
            opacity: 0;
            cursor: pointer;
            z-index: 100;
            overflow: hidden;
        }
    }

    .uploaded-block {
        position: relative;
        width: 100px;
        height: 100px;
        border-radius: 10px;
        transition: all 0.3s ease;
        will-change: transform, opacity;
        display: flex;
        align-items: center;
        justify-content: center;
        margin-top: 20px;

        img {
            width: 100%;
            height: 100%;
            object-fit: cover;
            border-radius: 10px;
        }
    }
}

.special-input {
    border: 1.5px dashed #C7C7C7;
}

.is-invalid {
    border: 1px solid #e74c3c !important;
    box-shadow: 0 0 5px rgba(231, 76, 60, 0.3) !important;
}

.error-message {
    font-size: 12px;
    color: #e74c3c;
    font-weight: 500;
    margin-top: 5px;
}
</style>