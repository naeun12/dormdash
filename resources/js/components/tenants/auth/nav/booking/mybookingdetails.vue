<template>
    <Loader ref="loader" />
    <Toastcomponents ref="toast" />
    <Modalconfirmation ref="modal" />
    <NotificationList ref="toastRef" />

    <div class="container py-4">
        <div class="row g-0 shadow-lg rounded-5 overflow-hidden modern-booking-card animate-fade-in">

            <div class="col-md-4 p-4 text-white d-flex flex-column bg-solid-blue">
                <div class="profile-img-container shadow-sm mb-4">
                    <img v-if="booking.pictureID" :src="booking.pictureID" alt="Profile Image"
                        class="img-fluid rounded-4">
                    <div v-else class="no-image-placeholder text-white-50">
                        <i class="bi bi-person-bounding-box display-4"></i>
                        <p class="small mt-2">No Image</p>
                    </div>
                </div>

                <div class="status-badge-container mb-4">
                    <statusMap :status="this.ispayment" role="tenant" />
                </div>

                <div class="personal-info-list flex-grow-1">
                    <div class="info-row border-bottom border-white-opacity py-2 mb-2">
                        <small class="d-block text-white-50 text-uppercase tracking-wider">Full Name</small>
                        <span class="fw-bold fs-5">{{ booking.firstname }} {{ booking.lastname }}</span>
                    </div>
                    <div class="d-flex gap-3 border-bottom border-white-opacity py-2 mb-2">
                        <div class="flex-fill">
                            <small class="d-block text-white-50 text-uppercase tracking-wider">Age</small>
                            <span class="fw-bold">{{ booking.age }}</span>
                        </div>
                        <div class="flex-fill">
                            <small class="d-block text-white-50 text-uppercase tracking-wider">Gender</small>
                            <span class="fw-bold text-capitalize">{{ booking.gender }}</span>
                        </div>
                    </div>
                    <div class="info-row border-bottom border-white-opacity py-2 mb-2">
                        <small class="d-block text-white-50 text-uppercase tracking-wider">Contact</small>
                        <span class="fw-bold d-block small">{{ booking.contactNumber }}</span>
                        <span class="fw-bold d-block small opacity-75">{{ booking.contactEmail }}</span>
                    </div>
                    <div class="info-row border-bottom border-white-opacity py-2">
                        <small class="d-block text-white-50 text-uppercase tracking-wider">Stay Duration</small>
                        <span class="fw-bold small">{{ booking.moveInDate }} — {{ booking.moveOutDate }}</span>
                    </div>
                </div>

                <div v-if="booking.status === 'confirmed' || booking.status === 'pending'" class="mt-4">
                    <button class="btn btn-cancel-modern w-100 rounded-pill py-2 fw-bold"
                        @click="cancelBooking(booking.bookingID)">
                        <i class="bi bi-x-circle me-2"></i>Cancel Booking
                    </button>
                </div>
            </div>

            <div class="col-md-8 bg-white p-4 p-lg-5">
                <div class="dorm-header-img rounded-4 overflow-hidden mb-4 shadow-sm border position-relative">
                    <img :src="booking.room?.roomImages" alt="Dormitory Image" class="w-100 h-100 object-fit-cover"
                        v-if="booking.room?.roomImages" />
                    <div class="bg-light w-100 h-100 d-flex align-items-center justify-content-center" v-else>
                        <p class="text-muted">No Room Image Available</p>
                    </div>
                    <div class="price-tag-floating shadow-sm">
                        ₱{{ booking.room?.price }}<span class="fs-6 fw-normal">/mo</span>
                    </div>
                </div>

                <div class="row g-3 mb-4">
                    <div class="col-md-6" v-for="(val, label) in {
                        'Dormitory': booking.room?.dorm.dormName,
                        'Address': booking.room?.dorm.address,
                        'Room Number': booking.room?.roomNumber,
                        'Room Type': booking.room?.roomType
                    }" :key="label">
                        <div class="p-3 rounded-4 bg-soft-blue border border-blue-subtle h-100">
                            <small class="text-muted d-block fw-bold text-uppercase" style="font-size: 0.65rem;">{{
                                label }}</small>
                            <span class="fw-bold text-dark">{{ val }}</span>
                        </div>
                    </div>
                </div>

                <div class="mb-4">
                    <BookingStatus :status="this.ispayment" role="tenant" />
                </div>

                <div v-if="ispayment === 'confirmed'" class="payment-section mt-5">
                    <h5 class="fw-black mb-4 text-dark border-start-orange ps-3">Payment Settlement</h5>

                    <div class="row g-4">
                        <div class="col-md-5">
                            <div
                                class="gcash-card p-4 rounded-5 text-white shadow-sm position-relative overflow-hidden">
                                <div class="d-flex justify-content-between mb-4">
                                    <span class="fw-bold small tracking-widest">GCASH PORTAL</span>
                                    <i class="bi bi-qr-code-scan fs-4"></i>
                                </div>
                                <small class="d-block opacity-75">Merchant Number</small>
                                <h4 class="fw-black mb-0 tracking-wider">{{ booking.room?.dorm.gcashNumber }}</h4>
                                <div class="card-wave"></div>
                            </div>
                        </div>
                        <div class="col-md-7">
                            <div class="upload-zone rounded-5 border-dashed p-4 text-center h-100 d-flex flex-column justify-content-center"
                                @click="triggerPaymentImage" v-if="isPaymentImage">
                                <input ref="PaymentPicturesInput" class="d-none" type="file" accept="image/*"
                                    @change="handlePaymentPicture" />
                                <div class="upload-icon-circle bg-light mb-2 mx-auto">
                                    <i class="bi bi-cloud-arrow-up text-blue fs-3"></i>
                                </div>
                                <h6 class="fw-bold mb-1 text-dark">Upload Receipt</h6>
                                <p class="small text-muted mb-0">Browse GCash Screenshot</p>
                            </div>

                            <div v-if="PaymentPicturePreview"
                                class="preview-container mt-3 position-relative text-center">
                                <img :src="PaymentPicturePreview" class="img-fluid rounded-4 border shadow-sm"
                                    style="max-height: 200px;">
                                <button type="button" @click="removePaymentPicture"
                                    class="btn btn-danger btn-sm rounded-circle position-absolute top-0 end-0 m-2">
                                    <i class="bi bi-x"></i>
                                </button>
                            </div>
                        </div>
                    </div>

                    <div v-if="errors.payment_image"
                        class="alert alert-danger mt-3 rounded-4 py-2 small border-0 shadow-sm">
                        <i class="bi bi-exclamation-triangle-fill me-2"></i>{{ errors.payment_image[0] }}
                    </div>

                    <button type="submit" class="btn btn-orange-solid w-100 py-3 mt-4 rounded-pill fw-black shadow-sm"
                        @click="submitPayment">
                        SUBMIT PAYMENT RECEIPT
                    </button>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';
import Toastcomponents from '@/components/Toastcomponents.vue';
import Loader from '@/components/loader.vue';
import Modalconfirmation from '@/components/modalconfirmation.vue';
import BookingStatus from '@/components/BookingStatusAlert.vue';
import statusMap from '@/components/statusmap.vue';
import NotificationList from '@/components/notifications.vue';

export default {
    components: {
        Toastcomponents,
        Loader,
        Modalconfirmation,
        BookingStatus,
        statusMap,
        NotificationList,
    },
    data() {
        return {
            booking: [],
            ispayment: '',
            paymentIcon: '/images/tenant/allimagesResouces/paymentIcon.jpg',
            id_picture: '/images/tenant/allimagesResouces/vector-id-card-icon.jpg',
            errors: [],
            paymentIcon: '/images/tenant/allimagesResouces/paymentIcon.jpg',
            PaymentPicturePreview: '',
            PaymentPictureFile: null,
            isPaymentImage: true,
            bookingID: '',
            notifications: [],
            receiverID: '',
            payment_type: 'online',
            tenantid: '',

        };
    },
    methods: {
        getBookingDetails() {
            this.$refs.loader.loading = true;

            axios.get(`/my-bookings/details/${this.bookingID}`)
                .then(response => {
                    if (response.data.length > 0) {
                        this.booking = response.data[0];
                        this.ispayment = this.booking.status;
                        this.$refs.loader.loading = false;

                    }
                })
                .catch(error => {
                    console.error('Error fetching booking details:', error);
                    this.$refs.loader.loading = false;

                });
        },
        async cancelBooking(bookingID) {
            try {
                const confirmed = await this.$refs.modal.show({
                    title: `Cancel Booking`,
                    message: `Are you sure you want to Cancelled this booking?`,
                    functionName: 'Confirm Cancelled Booking'
                });

                if (!confirmed) {
                    return;
                }
                const response = await axios.get(`/cancel/booking/${bookingID}`);
                this.$refs.toast.showToast(response.data.message, 'success');

                this.getBookingDetails();
            } catch (error) {
                console.error(error);
                alert('Failed to cancel reservation.');
            }
        },

        handlePaymentPicture(event) {
            const file = event.target.files[0];
            if (file) {
                // Create object URL and revoke previous one if exists
                if (this.PaymentPicturePreview) {
                    URL.revokeObjectURL(this.PaymentPicturePreview);
                }
                this.PaymentPictureFile = file;
                this.isPaymentImage = false;

                this.PaymentPicturePreview = URL.createObjectURL(file);
            }
        },
        triggerPaymentImage() {
            if (this.$refs.PaymentPicturesInput) {
                this.$refs.PaymentPicturesInput.click();
            }
        },
        removePaymentPicture() {
            if (this.PaymentPicturePreview) {
                URL.revokeObjectURL(this.PaymentPicturePreview);
            }
            this.PaymentPicturePreview = null;
            // Add null check for safety
            if (this.$refs.PaymentPicturePreview) {
                this.$refs.PaymentPicturePreview.value = ''; // Reset file input
            }
            this.isPaymentImage = true;
        },
        subscribeToNotifications() {
            if (this.hasSubscribed) return;
            this.hasSubscribed = true;

            this.receiverID = this.tenantid;
            Echo.private(`notifications.${this.tenantid}`)
                .subscribed(() => {
                    console.log('✔ Subscribed!');
                })
                .listen('.NewNotificationEvent', (e) => {
                    this.notifications.unshift(e); // save for list
                    this.$refs.toastRef.pushNotification({
                        title: e.title || 'New Notification',
                        message: e.message,
                        color: 'success',
                    });
                });
        },
        async submitPayment() {
            this.errors = {};
            const confirmed = await this.$refs.modal.show({
                title: `Confirm Payment`,
                message: `Are you sure you want to proceed with the payment for this room?`,
                functionName: 'Confirm Payment'
            });

            if (!confirmed) {
                return;
            }

            this.$refs.loader.loading = true;
            const formData = new FormData();
            formData.append('fkbookingID', this.bookingID);
            formData.append('paymentType', this.payment_type);
            formData.append('amount', this.booking.room?.price || 0);
            if (this.PaymentPictureFile) {
                formData.append('paymentImage', this.PaymentPictureFile);
            }
            try {
                const response = await axios.post('/tenant/pay-room', formData, {
                    headers: {
                        'Content-Type': 'multipart/form-data'
                    }
                });
                this.PaymentPicturePreview = '';
                this.PaymentPictureFile = null;
                this.payment_type = '';
                this.$refs.loader.loading = false;
                this.isPaymentImage = true;
                this.$refs.toast.showToast(response.data.message, 'success');
                this.getBookingDetails();

            } catch (error) {
                if (error.response && error.response.status === 422) {
                    this.errors = error.response.data.errors || {};

                } else if (error.response && error.response.data.message) {
                    alert(error.response.data.message);
                } else {
                    alert('Something went wrong. Please try again.');
                }
            }
            finally {
                this.$refs.loader.loading = false;

            }

        },

    },

    mounted() {
        const el = document.getElementById('viewBookingDetails');
        this.bookingID = el.getAttribute('booking_id')?.trim();
        this.tenantid = el.getAttribute('tenant_id')?.trim();

        this.getBookingDetails();
        this.subscribeToNotifications();

    },
};
</script>

<style scoped src="/resources/css/tenant/mybooking.css">

</style>
