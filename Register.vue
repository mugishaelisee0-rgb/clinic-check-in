<!-- src/components/Register.vue - Complete Patient Registration Component -->
<template>
    <div class="register-container">
        <!-- Dynamic UI Rendering: Toggle between Registration Form and Success View (v-if) -->
        <div v-if="!isRegistered" class="card-modern">
            <div class="register-header">
                <h3 class="fw-bold mb-2">
                    <i class="fas fa-user-plus me-2 text-success"></i>
                    Patient Registration
                </h3>
                <p class="text-muted">Please complete the form below to register at our clinic</p>
            </div>

            <!-- Registration Form with v-model two-way binding -->
            <form @submit.prevent="submitRegistration">
                <!-- Personal Information Section -->
                <div class="row g-3 mb-4">
                    <div class="col-md-6">
                        <label class="form-label fw-semibold">
                            <i class="fas fa-user me-1 text-muted"></i> Full Name *
                        </label>
                        <input 
                            type="text" 
                            class="form-control" 
                            v-model="formData.name" 
                            placeholder="Enter your full name"
                            :class="{ 'is-invalid': validationErrors.name }"
                            @input="clearFieldError('name')"
                        >
                        <div class="invalid-feedback" v-if="validationErrors.name">
                            {{ validationErrors.name }}
                        </div>
                    </div>

                    <div class="col-md-6">
                        <label class="form-label fw-semibold">
                            <i class="fas fa-id-card me-1 text-muted"></i> National ID *
                        </label>
                        <input 
                            type="text" 
                            class="form-control" 
                            v-model="formData.nationalID" 
                            placeholder="6-12 digits only"
                            :class="{ 'is-invalid': validationErrors.nationalID }"
                            @input="clearFieldError('nationalID')"
                        >
                        <div class="invalid-feedback" v-if="validationErrors.nationalID">
                            {{ validationErrors.nationalID }}
                        </div>
                        <div class="form-text">Format: 6-12 digits (numbers only, e.g., 12345678)</div>
                    </div>
                </div>

                <!-- Contact Information -->
                <div class="row g-3 mb-4">
                    <div class="col-md-6">
                        <label class="form-label fw-semibold">
                            <i class="fas fa-phone me-1 text-muted"></i> Phone Number
                        </label>
                        <input 
                            type="tel" 
                            class="form-control" 
                            v-model="formData.phone" 
                            placeholder="e.g., 0712345678"
                        >
                    </div>

                    <div class="col-md-6">
                        <label class="form-label fw-semibold">
                            <i class="fas fa-envelope me-1 text-muted"></i> Email Address
                        </label>
                        <input 
                            type="email" 
                            class="form-control" 
                            v-model="formData.email" 
                            placeholder="patient@example.com"
                        >
                    </div>
                </div>

                <!-- Date of Birth & Gender -->
                <div class="row g-3 mb-4">
                    <div class="col-md-6">
                        <label class="form-label fw-semibold">
                            <i class="fas fa-calendar-alt me-1 text-muted"></i> Date of Birth
                        </label>
                        <input 
                            type="date" 
                            class="form-control" 
                            v-model="formData.dob"
                        >
                    </div>

                    <div class="col-md-6">
                        <label class="form-label fw-semibold">
                            <i class="fas fa-venus-mars me-1 text-muted"></i> Gender
                        </label>
                        <select class="form-select" v-model="formData.gender">
                            <option value="">Select gender</option>
                            <option value="Male">Male</option>
                            <option value="Female">Female</option>
                            <option value="Other">Other</option>
                            <option value="Prefer not to say">Prefer not to say</option>
                        </select>
                    </div>
                </div>

                <!-- Symptom Collection - Array binding with multiple checkboxes (Array Storage) -->
                <div class="mb-4">
                    <label class="form-label fw-semibold">
                        <i class="fas fa-stethoscope me-1 text-muted"></i> Symptoms (Select all that apply)
                    </label>
                    <div class="symptom-grid">
                        <div 
                            v-for="symptom in symptomOptions" 
                            :key="symptom.value"
                            class="symptom-checkbox"
                        >
                            <div class="form-check">
                                <input 
                                    class="form-check-input" 
                                    type="checkbox" 
                                    :value="symptom.value" 
                                    v-model="formData.symptoms"
                                    :id="'sym-' + symptom.value"
                                >
                                <label class="form-check-label" :for="'sym-' + symptom.value">
                                    <i :class="symptom.icon"></i> {{ symptom.label }}
                                </label>
                            </div>
                        </div>
                    </div>
                    <div class="selected-symptoms mt-2" v-if="formData.symptoms.length">
                        <small class="text-muted">
                            <i class="fas fa-check-circle text-success"></i> 
                            Selected: {{ formData.symptoms.join(', ') }}
                        </small>
                    </div>
                </div>

                <!-- Additional Notes -->
                <div class="mb-4">
                    <label class="form-label fw-semibold">
                        <i class="fas fa-notes-medical me-1 text-muted"></i> Additional Notes / Medical History
                    </label>
                    <textarea 
                        class="form-control" 
                        rows="3" 
                        v-model="formData.notes"
                        placeholder="Any allergies, chronic conditions, or additional information..."
                    ></textarea>
                </div>

                <!-- Agreement Checkbox -->
                <div class="mb-4">
                    <div class="form-check">
                        <input 
                            class="form-check-input" 
                            type="checkbox" 
                            v-model="formData.agreeToTerms"
                            :class="{ 'is-invalid': validationErrors.agreeToTerms }"
                        >
                        <label class="form-check-label">
                            I confirm that the information provided is accurate and agree to the clinic's 
                            <a href="#" @click.prevent="showTerms">terms and conditions</a>
                        </label>
                        <div class="invalid-feedback" v-if="validationErrors.agreeToTerms">
                            {{ validationErrors.agreeToTerms }}
                        </div>
                    </div>
                </div>

                <!-- Form Action Buttons -->
                <div class="d-flex gap-3">
                    <button type="submit" class="btn btn-primary-custom" :disabled="isSubmitting">
                        <i v-if="isSubmitting" class="fas fa-spinner fa-spin me-1"></i>
                        <i v-else class="fas fa-check-circle me-1"></i>
                        {{ isSubmitting ? 'Registering...' : 'Register Patient' }}
                    </button>
                    <button type="button" class="btn btn-outline-custom" @click="resetForm">
                        <i class="fas fa-redo-alt me-1"></i> Reset Form
                    </button>
                </div>

                <!-- Alert Messages - Conditional Success/Failure -->
                <div v-if="alertMessage" :class="['alert', alertType === 'success' ? 'alert-custom-success' : 'alert-custom-danger', 'mt-4']">
                    <i :class="alertType === 'success' ? 'fas fa-check-circle' : 'fas fa-exclamation-triangle'"></i>
                    {{ alertMessage }}
                    <button type="button" class="btn-close float-end" @click="clearAlert"></button>
                </div>
            </form>
        </div>

        <!-- Success View - Shown after successful registration -->
        <div v-else class="card-modern text-center">
            <div class="success-animation">
                <i class="fas fa-check-circle text-success success-icon"></i>
            </div>
            
            <h3 class="fw-bold mt-3">Registration Successful! 🎉</h3>
            <p class="text-muted">Welcome to Health Clinic Center</p>

            <!-- Display registered patient info -->
            <div class="registration-summary text-start mt-4">
                <h5 class="fw-semibold mb-3">
                    <i class="fas fa-id-card me-2 text-primary"></i>
                    Your Registration Details
                </h5>
                
                <div class="row">
                    <div class="col-md-6">
                        <div class="info-row">
                            <span class="info-label">Queue Number:</span>
                            <span class="info-value badge-queue fs-6">{{ currentQueueNumber }}</span>
                        </div>
                        <div class="info-row">
                            <span class="info-label">Full Name:</span>
                            <span class="info-value">{{ registeredPatient.name }}</span>
                        </div>
                        <div class="info-row">
                            <span class="info-label">National ID:</span>
                            <span class="info-value">{{ registeredPatient.nationalID }}</span>
                        </div>
                        <div class="info-row">
                            <span class="info-label">Phone:</span>
                            <span class="info-value">{{ registeredPatient.phone || 'Not provided' }}</span>
                        </div>
                    </div>
                    <div class="col-md-6">
                        <div class="info-row">
                            <span class="info-label">Symptoms:</span>
                            <span class="info-value">
                                {{ registeredPatient.symptoms?.length ? registeredPatient.symptoms.join(', ') : 'None reported' }}
                            </span>
                        </div>
                        <div class="info-row">
                            <span class="info-label">Registration Time:</span>
                            <span class="info-value">{{ formatDateTime(registeredPatient.registeredAt) }}</span>
                        </div>
                        <div class="info-row">
                            <span class="info-label">Estimated Wait:</span>
                            <span class="info-value">~{{ estimatedWaitTime }} minutes</span>
                        </div>
                    </div>
                </div>
            </div>

            <div class="mt-4 d-flex gap-3 justify-content-center">
                <button class="btn btn-primary-custom" @click="viewQueue">
                    <i class="fas fa-list-ol me-1"></i> View Full Queue
                </button>
                <button class="btn btn-outline-custom" @click="resetToNewRegistration">
                    <i class="fas fa-user-plus me-1"></i> Register Another Patient
                </button>
            </div>
        </div>

        <!-- Terms Modal -->
        <div v-if="showTermsModal" class="modal-overlay" @click.self="showTermsModal = false">
            <div class="modal-content-custom">
                <div class="modal-header-custom">
                    <h5>Clinic Terms & Conditions</h5>
                    <button class="btn-close" @click="showTermsModal = false"></button>
                </div>
                <div class="modal-body-custom">
                    <p>1. Patients must provide accurate personal and medical information.</p>
                    <p>2. The clinic follows strict confidentiality protocols.</p>
                    <p>3. Queue numbers are assigned on first-come-first-served basis.</p>
                    <p>4. Please arrive 10 minutes before your turn.</p>
                    <p>5. Emergency cases will be prioritized appropriately.</p>
                </div>
                <div class="modal-footer-custom">
                    <button class="btn btn-primary-custom btn-sm" @click="showTermsModal = false">Close</button>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { reactive, ref, computed, onMounted } from 'vue'
import { useClinicStore } from '../stores/clinicStore'

// Pinia Store for state management
const clinicStore = useClinicStore()

// Local reactive state
const isRegistered = ref(false)
const isSubmitting = ref(false)
const registeredPatient = ref({})
const currentQueueNumber = ref(null)
const alertMessage = ref('')
const alertType = ref('success')
const showTermsModal = ref(false)

// Form Data with reactive variables
const formData = reactive({
    name: '',
    nationalID: '',
    phone: '',
    email: '',
    dob: '',
    gender: '',
    symptoms: [],
    notes: '',
    agreeToTerms: false
})

// Validation errors object
const validationErrors = reactive({
    name: '',
    nationalID: '',
    agreeToTerms: ''
})

// Symptom options array for checkbox binding
const symptomOptions = ref([
    { label: 'Fever', value: 'Fever', icon: 'fas fa-temperature-high' },
    { label: 'Cough', value: 'Cough', icon: 'fas fa-lungs' },
    { label: 'Headache', value: 'Headache', icon: 'fas fa-brain' },
    { label: 'Fatigue', value: 'Fatigue', icon: 'fas fa-bed' },
    { label: 'Shortness of Breath', value: 'Shortness of breath', icon: 'fas fa-lungs' },
    { label: 'Chest Pain', value: 'Chest Pain', icon: 'fas fa-heartbeat' },
    { label: 'Nausea', value: 'Nausea', icon: 'fas fa-stomach' },
    { label: 'Muscle Aches', value: 'Muscle aches', icon: 'fas fa-dumbbell' },
    { label: 'Sore Throat', value: 'Sore throat', icon: 'fas fa-comment-dots' },
    { label: 'Runny Nose', value: 'Runny nose', icon: 'fas fa-tint' }
])

// Computed property for estimated wait time
const estimatedWaitTime = computed(() => {
    const avgTimePerPatient = 15 // minutes
    const position = currentQueueNumber.value || 1
    return position * avgTimePerPatient
})

// Validation functions using conditional statements
const validateForm = () => {
    let isValid = true
    
    // Reset validation errors
    validationErrors.name = ''
    validationErrors.nationalID = ''
    validationErrors.agreeToTerms = ''
    
    // Conditional validation for Name
    if (!formData.name.trim()) {
        validationErrors.name = 'Full name is required'
        isValid = false
    } else if (formData.name.trim().length < 3) {
        validationErrors.name = 'Name must be at least 3 characters'
        isValid = false
    }
    
    // Conditional validation for National ID
    const idRegex = /^\d{6,12}$/
    if (!formData.nationalID.trim()) {
        validationErrors.nationalID = 'National ID is required'
        isValid = false
    } else if (!idRegex.test(formData.nationalID.trim())) {
        validationErrors.nationalID = 'National ID must be 6-12 digits only'
        isValid = false
    } else {
        // Check for duplicate ID in store
        const existing = clinicStore.getPatientByNationalID(formData.nationalID.trim())
        if (existing) {
            validationErrors.nationalID = 'A patient with this ID is already registered'
            isValid = false
        }
    }
    
    // Conditional validation for Terms Agreement
    if (!formData.agreeToTerms) {
        validationErrors.agreeToTerms = 'You must agree to the terms and conditions'
        isValid = false
    }
    
    return isValid
}

// Clear specific field error
const clearFieldError = (field) => {
    if (validationErrors[field]) {
        validationErrors[field] = ''
    }
}

// Submit registration - Main function with event handling
const submitRegistration = async () => {
    // Clear previous alerts
    clearAlert()
    
    // Validate form using conditional statements
    if (!validateForm()) {
        alertMessage.value = 'Please fix the validation errors before submitting'
        alertType.value = 'danger'
        return
    }
    
    isSubmitting.value = true
    
    try {
        // Simulate async registration (e.g., API call)
        await new Promise(resolve => setTimeout(resolve, 1000))
        
        // Add patient to store (State Mutation)
        const newPatient = clinicStore.addPatient({
            name: formData.name.trim(),
            nationalID: formData.nationalID.trim(),
            symptoms: [...formData.symptoms],
            phone: formData.phone,
            email: formData.email,
            dob: formData.dob,
            gender: formData.gender,
            notes: formData.notes,
            registeredAt: new Date().toISOString()
        })
        
        // Update local state
        registeredPatient.value = newPatient
        currentQueueNumber.value = newPatient.queueNumber
        isRegistered.value = true
        
        // Show success message
        alertMessage.value = `✅ Registration successful! Your queue number is ${newPatient.queueNumber}`
        alertType.value = 'success'
        
        // Auto-clear alert after 5 seconds
        setTimeout(() => {
            if (alertMessage.value) clearAlert()
        }, 5000)
        
    } catch (error) {
        alertMessage.value = 'Registration failed. Please try again.'
        alertType.value = 'danger'
    } finally {
        isSubmitting.value = false
    }
}

// Reset form function
const resetForm = () => {
    formData.name = ''
    formData.nationalID = ''
    formData.phone = ''
    formData.email = ''
    formData.dob = ''
    formData.gender = ''
    formData.symptoms = []
    formData.notes = ''
    formData.agreeToTerms = false
    
    // Clear validation errors
    validationErrors.name = ''
    validationErrors.nationalID = ''
    validationErrors.agreeToTerms = ''
    
    clearAlert()
}

// Reset to new registration (from success view)
const resetToNewRegistration = () => {
    isRegistered.value = false
    resetForm()
    registeredPatient.value = {}
    currentQueueNumber.value = null
}

// View queue function
const viewQueue = () => {
    // Emit event to parent to change view
    emit('view-queue')
}

// Clear alert
const clearAlert = () => {
    alertMessage.value = ''
    alertType.value = 'success'
}

// Show terms modal
const showTerms = () => {
    showTermsModal.value = true
}

// Format date time
const formatDateTime = (dateString) => {
    if (!dateString) return 'N/A'
    const date = new Date(dateString)
    return date.toLocaleString()
}

// Emit events for parent communication
const emit = defineEmits(['view-queue', 'registration-complete'])

// Lifecycle hook - mounted
onMounted(() => {
    console.log('Register component mounted - Ready for patient registration')
    
    // Optional: Pre-fill demo data for testing
    // This shows lifecycle method initialization
    const demoMode = false
    if (demoMode) {
        formData.name = 'Demo Patient'
        formData.nationalID = '12345678'
    }
})

// Expose methods for parent component if needed
defineExpose({
    resetForm,
    isRegistered
})
</script>

<style scoped>
.register-container {
    width: 100%;
}

.card-modern {
    border: none;
    border-radius: 28px;
    background: white;
    box-shadow: 0 10px 25px rgba(0,0,0,0.05);
    padding: 2rem;
}

.register-header {
    margin-bottom: 1.5rem;
    padding-bottom: 1rem;
    border-bottom: 2px solid #eef5f0;
}

.symptom-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 0.75rem;
    background: #f9fafb;
    padding: 1rem;
    border-radius: 20px;
}

.symptom-checkbox {
    padding: 0.25rem 0;
}

.selected-symptoms {
    padding: 0.5rem 1rem;
    background: #d9efe3;
    border-radius: 50px;
    display: inline-block;
}

.btn-primary-custom {
    background: #2c7a4b;
    border: none;
    padding: 10px 28px;
    border-radius: 50px;
    color: white;
    font-weight: 500;
    transition: all 0.2s;
}

.btn-primary-custom:hover:not(:disabled) {
    background: #1e5f3a;
    transform: translateY(-2px);
}

.btn-primary-custom:disabled {
    opacity: 0.7;
    cursor: not-allowed;
}

.btn-outline-custom {
    border: 1px solid #2c7a4b;
    background: transparent;
    padding: 10px 28px;
    border-radius: 50px;
    color: #2c7a4b;
    font-weight: 500;
    transition: all 0.2s;
}

.btn-outline-custom:hover {
    background: #2c7a4b;
    color: white;
}

.success-icon {
    font-size: 5rem;
    animation: scaleIn 0.5s ease;
}

.success-animation {
    animation: bounceIn 0.6s ease;
}

.registration-summary {
    background: #f8f9fa;
    border-radius: 20px;
    padding: 1.5rem;
}

.info-row {
    display: flex;
    justify-content: space-between;
    padding: 0.75rem 0;
    border-bottom: 1px solid #e0e0e0;
}

.info-row:last-child {
    border-bottom: none;
}

.info-label {
    font-weight: 600;
    color: #6c757d;
}

.info-value {
    color: #2c3e50;
    font-weight: 500;
}

.badge-queue {
    background: #2b7a4b;
    padding: 6px 14px;
    border-radius: 50px;
    font-size: 1rem;
    color: white;
    display: inline-block;
}

.alert-custom-success {
    background: #d9efe3;
    border: 1px solid #b8dec9;
    color: #0a3622;
    border-radius: 16px;
    padding: 1rem;
}

.alert-custom-danger {
    background: #f8d7da;
    border: 1px solid #f5c2c7;
    color: #842029;
    border-radius: 16px;
    padding: 1rem;
}

/* Modal styles */
.modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0,0,0,0.5);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 1000;
}

.modal-content-custom {
    background: white;
    border-radius: 24px;
    max-width: 500px;
    width: 90%;
    max-height: 80vh;
    overflow: auto;
    animation: slideUp 0.3s ease;
}

.modal-header-custom {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem 1.5rem;
    border-bottom: 1px solid #e0e0e0;
}

.modal-body-custom {
    padding: 1.5rem;
}

.modal-footer-custom {
    padding: 1rem 1.5rem;
    border-top: 1px solid #e0e0e0;
    text-align: right;
}

/* Animations */
@keyframes scaleIn {
    from {
        transform: scale(0);
        opacity: 0;
    }
    to {
        transform: scale(1);
        opacity: 1;
    }
}

@keyframes bounceIn {
    0% {
        transform: scale(0.3);
        opacity: 0;
    }
    50% {
        transform: scale(1.05);
    }
    70% {
        transform: scale(0.9);
    }
    100% {
        transform: scale(1);
        opacity: 1;
    }
}

@keyframes slideUp {
    from {
        transform: translateY(50px);
        opacity: 0;
    }
    to {
        transform: translateY(0);
        opacity: 1;
    }
}

@media (max-width: 768px) {
    .card-modern {
        padding: 1.25rem;
    }
    
    .symptom-grid {
        grid-template-columns: 1fr;
    }
    
    .info-row {
        flex-direction: column;
        gap: 0.25rem;
    }
}
</style>