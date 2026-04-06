<template>
    <Loader ref="loader" />
    <NotificationList ref="toastRef" />

    <div class="bookings-wrapper py-5">
        <div class="container">
            <div class="booking-glass-header d-flex align-items-center justify-content-between p-4 mb-4 shadow-sm">
                <div>
                    <h2 class="fw-black mb-0 text-dark">My Bookings</h2>
                    <p class="text-muted mb-0 small">Manage your stay and track reservation status</p>
                </div>
                <button @click="viewPayments" class="btn btn-orange-premium px-4 py-2 rounded-pill shadow-sm">
                    <i class="bi bi-credit-card-2-back me-2"></i>View Payments
                </button>
            </div>

            <div v-if="bookings.length === 0" class="empty-bookings-card text-center p-5 bg-white rounded-5 shadow-sm">
                <div class="empty-icon-bg mb-3 mx-auto">
                    <i class="bi bi-calendar-x text-orange"></i>
                </div>
                <h4 class="fw-bold text-dark">No bookings found</h4>
                <p class="text-muted mb-4">Explore dormitories and find your perfect room today!</p>
                <button class="btn btn-blue-premium px-4 rounded-pill">Find a Room</button>
            </div>

            <div class="booking-feed">
                <div class="booking-card-modern mb-4 overflow-hidden shadow-sm"
                    v-for="(booking, index) in bookings.slice(0, showCount)" :key="index">

                    <div class="row g-0">
                        <div class="col-lg-3 position-relative">
                            <img :src="booking.pictureID" alt="Dorm Image" class="booking-img-main" />
                            <div class="status-overlay">
                                <statusMap :status="booking.status" role="tenant" />
                            </div>
                        </div>

                        <div class="col-lg-7 p-4">
                            <div class="d-flex justify-content-between align-items-start mb-3">
                                <div>
                                    <h5 class="fw-bold text-dark mb-1">{{ booking.room?.dorm.dormName }}</h5>
                                    <p class="text-muted small mb-0"><i
                                            class="bi bi-geo-alt-fill text-orange me-1"></i>{{
                                        booking.room?.dorm.address }}</p>
                                </div>
                                <div class="room-badge shadow-sm">
                                    <small class="fw-bold">Room {{ booking.room?.roomNumber }}</small>
                                </div>
                            </div>

                            <div class="row g-3">
                                <div class="col-md-6">
                                    <div class="info-item d-flex align-items-center gap-2">
                                        <i class="bi bi-person-circle text-blue"></i>
                                        <div>
                                            <small class="d-block text-muted">Tenant</small>
                                            <span class="fw-semibold">{{ booking.firstname }} {{ booking.lastname
                                                }}</span>
                                        </div>
                                    </div>
                                </div>
                                <div class="col-md-6">
                                    <div class="info-item d-flex align-items-center gap-2">
                                        <i class="bi bi-calendar-check text-blue"></i>
                                        <div>
                                            <small class="d-block text-muted">Check-in Date</small>
                                            <span class="fw-semibold">{{ booking.moveOutDate }}</span>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div
                            class="col-lg-2 d-flex align-items-center justify-content-center p-3 border-start bg-light-soft">
                            <button @click="viewBooking(booking)" class="btn btn-view-booking w-100 py-3 rounded-4">
                                <span>Details</span>
                                <i class="bi bi-arrow-right-short ms-1"></i>
                            </button>
                        </div>
                    </div>
                </div>
            </div>

            <div class="text-center mt-5" v-if="bookings.length > 3">
                <button class="btn btn-show-more shadow-sm" @click="toggleShow">
                    {{ showAll ? 'Show Less' : 'Show More Bookings' }}
                </button>
            </div>
        </div>
    </div>
</template>
<script>
import axios from 'axios';
import Loader from '@/components/loader.vue';
import BookingStatus from '@/components/BookingStatusAlert.vue';
import statusMap from '@/components/statusmap.vue';
import NotificationList from '@/components/notifications.vue';

export default {
    components: {
        Loader,
        BookingStatus,
        statusMap,
        NotificationList,

    },
    data() {
        return {
            bookings: [],
            tenantid: '',
            showAll: false, // default is false
            showCount: 3,   // show 3 items by default
            viewBookingModal: false,
            notifications: [],
            receiverID: '',
            notifications: [],
            receiverID: '',
        };
    },
    methods: {
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
        viewPayments() {
            window.location.href = `/view/payment/${this.tenantid}`;
        },
        viewBooking(booking) {
            window.location.href = `/view/booking/details/${this.tenantid}/${booking.bookingID}`;
        },
        async myBookingList() {
            try {
                this.$refs.loader.loading = true;

                const response = await axios.get(`/tenant/my-bookings/${this.tenantid}`);
                this.bookings = response.data;
            } catch (error) {
                console.error("Error fetching booking list:", error);
            }
            finally {
                this.$refs.loader.loading = false;

            }
        },
        toggleShow() {
            if (this.showAll) {
                this.showCount = 3;
            } else {
                this.showCount = this.bookings.length;
            }
            this.showAll = !this.showAll;
        }
    },
    mounted() {
        const el = document.getElementById('viewBooking');
        this.tenantid = el.getAttribute('tenant_id')?.trim();
        this.myBookingList();
        this.subscribeToNotifications();




    },
};
</script>

<style scoped src="/resources/css/tenant/booking.css">

</style>
