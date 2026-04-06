<template>
    <Loader ref="loader" />
    <NotificationList ref="toastRef" />

    <div class="d-flex flex-column flex-md-row bg-light overflow-hidden" style="min-height: 100vh;">

        <div class="p-4 p-lg-5 text-white shadow-lg m-3 m-md-4 rounded-4 d-flex flex-column justify-content-center"
            style="width: 100%; max-width: 400px; flex-shrink: 0; background-color: #003C87; position: relative;">

            <div class="position-absolute top-0 start-0 w-100 h-100 opacity-10"
                style="background-image: radial-gradient(circle, #fff 1px, transparent 1px); background-size: 20px 20px;">
            </div>

            <div class="position-relative">
                <div class="text-center mb-4">
                    <div class="d-inline-block p-3 rounded-circle bg-white bg-opacity-10 mb-3">
                        <i class="bi bi-patch-check-fill text-warning fs-1"></i>
                    </div>
                    <h3 class="fw-bold">Landlord Pro</h3>
                    <p class="text-white-50">Unlock the full potential of DormDash</p>
                </div>

                <div class="list-group list-group-flush bg-transparent mt-4">
                    <div class="d-flex align-items-start mb-4">
                        <div class="bg-white bg-opacity-25 rounded-circle p-2 me-3">
                            <i class="bi bi-house-add-fill text-warning"></i>
                        </div>
                        <div>
                            <h6 class="mb-0 fw-bold">Unlimited Listings</h6>
                            <small class="opacity-75">Upload and manage all your dorm properties.</small>
                        </div>
                    </div>
                    <div class="d-flex align-items-start mb-4">
                        <div class="bg-white bg-opacity-25 rounded-circle p-2 me-3">
                            <i class="bi bi-lightning-charge-fill text-warning"></i>
                        </div>
                        <div>
                            <h6 class="mb-0 fw-bold">Instant Visibility</h6>
                            <small class="opacity-75">Reach thousands of potential tenants instantly.</small>
                        </div>
                    </div>
                    <div class="d-flex align-items-start mb-4">
                        <div class="bg-white bg-opacity-25 rounded-circle p-2 me-3">
                            <i class="bi bi-shield-shaded text-warning"></i>
                        </div>
                        <div>
                            <h6 class="mb-0 fw-bold">Trust Badge</h6>
                            <small class="opacity-75">Get a verified badge to build tenant confidence.</small>
                        </div>
                    </div>
                </div>

                <div class="mt-5 pt-4 border-top border-white border-opacity-10 text-center">
                    <small class="opacity-50 fst-italic">DormDash Secure Gateway</small>
                </div>
            </div>
        </div>

        <div class="flex-fill d-flex align-items-center justify-content-center p-3 p-md-5">

            <div v-if="verified === 0" class="card shadow-lg border-0 rounded-4 p-4 p-md-5 w-100"
                style="max-width: 480px; background-color: #ffffff;">
                <div class="text-center mb-4">
                    <h2 class="fw-bold text-dark mb-1">Upgrade Account</h2>
                    <p class="text-muted">Start your journey as a verified partner</p>
                </div>

                <div class="mb-4">
                    <div class="p-3 rounded-4 bg-light d-flex align-items-center justify-content-between border">
                        <div class="d-flex align-items-center">
                            <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/5/5c/GCash_logo.svg/1280px-GCash_logo.svg.png"
                                alt="GCash" style="height: 20px;" class="me-2">
                            <span class="fw-bold text-secondary small">GCASH E-WALLET</span>
                        </div>
                        <span class="badge bg-success rounded-pill px-3">Fast Secure</span>
                    </div>
                </div>

                <div class="mb-4">
                    <label class="form-label fw-bold text-dark small">BILLING EMAIL</label>
                    <div class="input-group">
                        <span class="input-group-text bg-white border-end-0 text-muted">
                            <i class="bi bi-envelope"></i>
                        </span>
                        <input type="email" class="form-control form-control-lg border-start-0 ps-0"
                            placeholder="you@email.com" v-model="email" style="font-size: 1rem;" />
                    </div>
                    <p class="text-danger small mt-2 fw-medium" v-if="error.email">
                        <i class="bi bi-exclamation-circle me-1"></i>{{ error.email[0] }}
                    </p>
                </div>

                <div class="p-4 rounded-4 mb-4 text-center"
                    style="background-color: #f8faff; border: 2px dashed #003C8733;">
                    <small class="text-muted d-block mb-1">TOTAL AMOUNT</small>
                    <h1 class="fw-bold mb-0" style="color: #003C87;">₱{{ Number(amount).toLocaleString() }}</h1>
                </div>

                <button class="btn btn-lg w-100 py-3 rounded-4 shadow fw-bold text-white transition-all hover-scale"
                    style="background-color: #FC7D07; border: none;" @click="payWithGCash" :disabled="loading">
                    <span v-if="loading" class="spinner-border spinner-border-sm me-2"></span>
                    <i v-else class="bi bi-lock-fill me-2"></i>
                    {{ loading ? 'PROCESSING...' : 'CONFIRM & PAY NOW' }}
                </button>

                <p class="text-center text-muted small mt-4">
                    <i class="bi bi-shield-lock me-1"></i> Encrypted Payment Processing
                </p>
            </div>

            <div v-else class="card shadow-lg border-0 text-center p-5 rounded-4 animate__animated animate__fadeIn"
                style="max-width: 480px;">
                <div class="mb-4">
                    <div class="d-inline-flex p-4 rounded-circle bg-success bg-opacity-10">
                        <i class="bi bi-check-circle-fill text-success" style="font-size: 4rem;"></i>
                    </div>
                </div>
                <h2 class="fw-bold mb-3 text-dark">Verification Complete!</h2>
                <p class="text-secondary mb-4 px-3" style="line-height: 1.6;">
                    Excellent! Your landlord account is now fully verified. You can now access your dashboard and start
                    listing your properties to find new tenants.
                </p>
                <div class="pt-3">
                    <div class="p-3 bg-light rounded-4 border">
                        <div class="d-flex align-items-center justify-content-center text-success fw-bold">
                            <i class="bi bi-shield-fill-check me-2"></i> PRO LANDLORD FEATURES ENABLED
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>


<script>
import axios from "axios";
import Loader from '@/components/loader.vue';
import Modalconfirmation from '@/components/modalconfirmation.vue';
import NotificationList from '@/components/notifications.vue';
export default {
    components: {
        Loader,
        Modalconfirmation,
        NotificationList
    },
    data() {
        return {
            email: "",
            amount: 500,
            loading: false,
            error: {},
            verified: 0,
            landlord_id: '',
        };
    },
    methods: {
        async payWithGCash() {


            if (!this.email) {
                this.error.email = ['Email is required.'];
                return;
            }

            try {
                this.$refs.loader.loading = true;


                const res = await axios.post(
                    `/landlord/verify-payment`,
                    {
                        paymentEmail: this.email,
                        amount: this.amount,


                    }
                );
                window.location.href = res.data.checkout_url;
                this.error = {};


            } catch (err) {
                console.error(err.response?.data || err);
                alert("Something went wrong. Please try again.");
            } finally {
                this.$refs.loader.loading = false;
            }
        },
        async cancelAnimationFrame() {
            const response = await axios.get('/landlord/{landlord_id}/payment-cancel');
            if (response.data.status === 'cancelled') {
                alert('Payment was cancelled. Please try again.');
            }
        },
        async fetchLandlordData() {


            try {
                const response = await axios.get(`/get/landlord/data/${this.landlord_id}`);
                this.verified = response.data.landlord.isVerified;
                this.$refs.loader.loading = false;

            } catch (error) {
                console.error('Error fetching landlord data:', error);
                this.$refs.loader.loading = false;
            }
        }
    },
    mounted() {
        const element = document.getElementById('paymentLandlord');
        this.landlord_id = element.dataset.landlordId;
        this.fetchLandlordData();
        const params = new URLSearchParams(window.location.search);
        const status = params.get('status');


        if (status === 'success') {
            this.$refs.toast.show('✅ Payment Success! Your account is now verified.');
            // Optional: call backend to save payment info
        } else if (status === 'cancelled') {
            this.$refs.toast.show('⚠️ Payment was cancelled.');
        }

    }
};
</script>
