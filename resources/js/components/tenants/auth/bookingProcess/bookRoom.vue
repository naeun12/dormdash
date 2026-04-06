<template>
    <Toastcomponents ref="toast" />
    <Modalconfirmation ref="modal" />
    <Loader ref="loader" />
    <NotificationList ref="toastRef" />

    <div class="booking-container container py-5">
        <div class="booking-header mb-5 text-center">
            <h2 class="fw-bold">✨ Book Your Room Now</h2>
            <div class="header-line mx-auto"></div>
        </div>

        <div class="row g-4">
            <div class="col-lg-7">
                <div class="booking-card personal-info-card p-4 h-100">
                    <h5 class="section-title mb-4">
                        <i class="bi bi-person-badge-fill me-2 text-orange"></i>Personal Information
                    </h5>

                    <div class="row g-3">
                        <div class="col-md-6">
                            <label class="custom-label">Firstname</label>
                            <div class="info-box border-blue">{{ firstname || 'N/A' }}</div>
                        </div>
                        <div class="col-md-6">
                            <label class="custom-label">Lastname</label>
                            <div class="info-box border-blue">{{ lastname || 'N/A' }}</div>
                        </div>
                        <div class="col-12">
                            <label class="custom-label">Contact Number</label>
                            <div class="info-box border-blue">
                                <i class="bi bi-telephone-fill me-2 text-orange"></i>{{ contactInfo || 'N/A' }}
                            </div>
                        </div>
                        <div class="col-12">
                            <label class="custom-label">Email Address</label>
                            <div class="info-box border-blue">
                                <i class="bi bi-envelope-at-fill me-2 text-orange"></i>{{ email || 'N/A' }}
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <div class="col-lg-5">
                <div class="booking-card room-details-card p-4">
                    <h5 class="section-title mb-4">
                        <i class="bi bi-house-lock-fill me-2 text-orange"></i>Room Summary
                    </h5>

                    <div class="room-summary-box p-3 rounded-3 mb-4">
                        <div class="d-flex justify-content-between mb-2">
                            <span class="text-white-50">Room Number</span>
                            <span class="fw-bold text-white">#{{ roomsDetail?.roomNumber || 'N/A' }}</span>
                        </div>
                        <div class="d-flex justify-content-between mb-2">
                            <span class="text-white-50">Type</span>
                            <span class="badge bg-white text-orange rounded-pill px-3 fw-bold">{{ roomsDetail?.roomType
                                || 'N/A' }}</span>
                        </div>
                        <div class="d-flex justify-content-between pt-2 border-top border-orange-light">
                            <span class="text-white fw-bold">Monthly Rate</span>
                            <span class="fw-bold fs-5 text-white">₱{{ Number(roomsDetail?.price).toLocaleString()
                                }}</span>
                        </div>
                    </div>

                    <div class="date-section">
                        <div class="mb-3">
                            <label class="custom-label">Move-in Date</label>
                            <input type="date" class="form-control custom-input input-focus-orange" v-model="moveInDate"
                                @change="setMoveOutDate" :min="today" />
                            <small v-if="errors.moveInDate" class="text-danger mt-1 d-block fw-bold">
                                <i class="bi bi-exclamation-triangle-fill me-1"></i>{{ errors.moveInDate[0] }}
                            </small>
                        </div>
                        <div class="mb-4">
                            <label class="custom-label">Move-out Date (Auto)</label>
                            <input type="date" class="form-control custom-input bg-light" v-model="moveOutDate"
                                disabled />
                        </div>
                    </div>

                    <button type="submit" class="btn btn-book-now w-100 py-3 shadow" @click="bookRoom">
                        <i class="bi bi-bookmark-check-fill me-2"></i>Confirm & Book Now
                    </button>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import axios from 'axios'
import Toastcomponents from '@/components/Toastcomponents.vue';
import Loader from '@/components/loader.vue';
import Modalconfirmation from '@/components/modalconfirmation.vue';
import NotificationList from '@/components/notifications.vue';
export default {
    components: {
        Toastcomponents,
        Loader,
        Modalconfirmation,
        NotificationList,

    },
    data() {
        return {
            firstname: '',
            lastname: '',
            email: '',
            contactInfo: '',
            age: '',
            sex: '',
            tenant_id: '',
            room_id: '',
            dormitory_id: '',
            isPaymentImage: true,
            idPicturePreview: '',
            selectedRoomId: '',
            roomsDetail: {},
            errors: {},
            moveInDate: this.getTodayLocal(),
            moveOutDate: '',
            today: this.getTodayLocal(),
            paymentIcon: '/images/tenant/allimagesResouces/paymentIcon.jpg',
            id_picture: '/images/tenant/allimagesResouces/vector-id-card-icon.jpg',
            openPaymentModel: false,
            imageUrl: '',
            notifications: [],
            receiverID: '',
        }
    },
    methods: {
        subscribeToNotifications() {
            if (this.hasSubscribed) return;
            this.hasSubscribed = true;

            this.receiverID = this.tenant_id;
            Echo.private(`notifications.${this.tenant_id}`)
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
        async getRoomDetails() {
            try {
                this.$refs.loader.loading = true;
                const response = await axios.get(`/get-room-details/${this.room_id}`);
                this.roomsDetail = response.data.room;
                this.$refs.loader.loading = false;

            } catch (error) {
                this.$refs.loader.loading = false;

            }
        },
        tenantData() {
            this.room_id = '';
            this.tenant_id = '';
            this.firstname = '';
            this.lastname = '';
            this.contactInfo = '';
            this.email = '';
            this.age = '';
            this.sex = '';
            this.imageUrl = '';
            this.moveInDate = '';
            this.moveOutDate = '';
        },


        async bookRoom() {
            try {
                const confirmed = await this.$refs.modal.show({
                    title: `Confirm Booking`,
                    message: `Are you sure you want to book Room ${this.roomsDetail?.roomNumber || 'N/A'} (${this.roomsDetail?.roomType || 'Room'})?`,
                    functionName: 'Confirm Room Booking'
                });


                if (!confirmed) {
                    return;
                }
                
                const formdata = new FormData();
                formdata.append('room_id', this.room_id);
                formdata.append('tenant_id', this.tenant_id);
                formdata.append('firstname', this.firstname);
                formdata.append('lastname', this.lastname);
                formdata.append('contact_number', this.contactInfo);
                formdata.append('email', this.email);
                formdata.append('age', this.age);
                formdata.append('gender', this.sex);
                formdata.append('studentpicture_id', this.imageUrl);
                formdata.append('moveInDate', this.moveInDate);
                formdata.append('moveOutDate', this.moveOutDate);
                const response = await axios.post('/book-room', formdata);
                if (response.data.status === 'success') {
                    this.$refs.toast.showToast(response.data.message, 'success');
                    this.tenantData();
                    this.tenant_id = window.tenant_id;
                    window.location.href = `/view/booking/${this.tenant_id}`;

                }
            } catch (error) {
                if (error.response && error.response.status === 422) {
                    const validationErrors = error.response.data.errors;
                    this.errors = validationErrors;
                } else {
                    this.$refs.toast.showToast('Something went wrong. Please try again.', 'danger');
                }
            }


        },
        setMoveOutDate() {
            if (this.moveInDate) {
                // Parse YYYY-MM-DD manually
                const [year, month, day] = this.moveInDate.split('-').map(Number);
                const date = new Date(year, month - 1, day); // Local date

                // Add 1 month
                date.setMonth(date.getMonth() + 1);

                // Format back to YYYY-MM-DD
                const outYear = date.getFullYear();
                const outMonth = String(date.getMonth() + 1).padStart(2, '0');
                const outDay = String(date.getDate()).padStart(2, '0');

                this.moveOutDate = `${outYear}-${outMonth}-${outDay}`;
            } else {
                this.moveOutDate = '';
            }
        },

        getTodayLocal() {
            const now = new Date();
            const year = now.getFullYear();
            const month = String(now.getMonth() + 1).padStart(2, '0');
            const day = String(now.getDate()).padStart(2, '0');
            return `${year}-${month}-${day}`; // YYYY-MM-DD
        },
    },
    mounted() {
        const data = localStorage.getItem('tenantInfo');
        if (data) {
            const parsed = JSON.parse(data);
            this.firstname = parsed.firstname;
            this.lastname = parsed.lastname;
            this.email = parsed.email;
            this.contactInfo = parsed.contactInfo;
            this.age = parsed.age;
            this.sex = parsed.sex;
            this.idPicturePreview = parsed.idPicturePreview;
            this.imageUrl = parsed.imageUrl;
        }
        const element = document.getElementById('roomBook');
        this.dormitory_id = element.dataset.dormId;
        this.room_id = element.dataset.roomId;
        this.tenant_id = element.dataset.tenantId;
        this.getRoomDetails();
        this.setMoveOutDate();
        this.subscribeToNotifications();

    }
}
</script>
<style scoped src="../../../../../css/tenant/bookroom.css"></style>