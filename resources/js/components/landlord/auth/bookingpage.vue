<template>
    <Loader ref="loader" />
    <NotificationList ref="toastRef" />

    <div class="p-4 mt-4">
        <div class="filter-bar-container p-2 bg-white shadow-sm rounded-4 border mb-4">
            <div class="d-flex flex-column flex-md-row align-items-center gap-2">

                <div class="input-group flex-grow-1 border-0 bg-light rounded-pill px-3 py-1">
                    <span class="input-group-text bg-transparent border-0 pe-1">
                        <i class="bi bi-search text-primary"></i>
                    </span>
                    <input type="text" class="form-control border-0 bg-transparent shadow-none"
                        placeholder="Search Tenant Name..." v-model="searchTerm" />
                </div>

                <div class="d-none d-md-block border-end h-100 mx-1" style="height: 30px !important;"></div>

                <div class="filter-dropdown">
                    <div class="d-flex align-items-center gap-2 px-2">
                        <i class="bi bi-building text-info"></i>
                        <select class="form-select border-0 shadow-none bg-transparent fw-600 py-1"
                            v-model="selectedDormId" @change="filterDorms"
                            style="min-width: 140px; font-size: 0.85rem;">
                            <option disabled value="">Select Dorm</option>
                            <option value="all">All Dorms</option>
                            <option v-for="dorm in dorms" :key="dorm.dormID" :value="dorm.dormID">
                                {{ dorm.dormName }}
                            </option>
                        </select>
                    </div>
                </div>

                <div class="filter-dropdown">
                    <div class="d-flex align-items-center gap-2 px-2">
                        <i class="bi bi-door-closed text-success"></i>
                        <select class="form-select border-0 shadow-none bg-transparent fw-600 py-1"
                            v-model="selectedroomNumber" @change="filterroomNumber"
                            style="min-width: 140px; font-size: 0.85rem;">
                            <option disabled value="">Select Room</option>
                            <option value="all">All Rooms</option>
                            <option v-for="room in uniqueRooms" :key="room.fkroomID" :value="room.room?.roomNumber">
                                Room {{ room.room?.roomNumber }}
                            </option>
                        </select>
                    </div>
                </div>

                <div class="filter-dropdown">
                    <div class="d-flex align-items-center gap-2 px-2">
                        <i class="bi bi-funnel text-warning"></i>
                        <select class="form-select border-0 shadow-none bg-transparent fw-600 py-1"
                            v-model="selectedapplicationStatus" @change="filterApplicationStatus"
                            style="min-width: 150px; font-size: 0.85rem;">
                            <option disabled value="">Select Status</option>
                            <option value="all">All Status</option>
                            <option value="pending">Pending</option>
                            <option value="confirmed">Confirmed</option>
                            <option value="approved">Approved</option>
                            <option value="paid">Paid</option>
                            <option value="rejected">Rejected</option>
                        </select>
                    </div>
                </div>

            </div>
        </div>
        <div v-if="!tenants.length"
            class="empty-state-container d-flex flex-column justify-content-center align-items-center py-5">
            <div class="empty-icon-wrapper mb-4">
                <div class="blob-bg"></div>
                <i class="bi bi-folder2-open"></i>
            </div>

            <div class="text-center">
                <h5 class="fw-800 text-dark mb-1">No Bookings Found</h5>
                <p class="text-muted small px-4">We couldn't find any records matching your current filters. <br> Try
                    adjusting your search or selection.</p>

                <button v-if="searchTerm || selectedDormId !== 'all'"
                    class="btn btn-sm btn-outline-primary rounded-pill mt-2 px-4" @click="resetFilters">
                    Clear all filters
                </button>
            </div>
        </div>
        <div class="row row-cols-1 row-cols-md-2 row-cols-lg-3 g-4">
            <div class="col" v-for="booking in tenants" :key="booking.bookingID">
                <div class="card h-100 border-0 shadow-sm rounded-4 overflow-hidden"
                    style="border-top: 5px solid #FC7D07 !important;">

                    <div class="d-flex justify-content-between align-items-center card-header border-0 p-3"
                        style="background-color: #003C87;">
                        <h6 class="fw-bold text-white mb-0">
                            <span class="opacity-75 small fw-light">ID:</span> #{{ booking.bookingID }}
                        </h6>

                        <button class="btn btn-link text-white p-0 opacity-75 text-decoration-none"
                            @click="deleteBooking(booking.bookingID)" title="Delete Booking">
                            <i class="bi bi-trash3-fill"></i>
                        </button>
                    </div>

                    <div class="card-body p-4">
                        <div class="mb-3">
                            <statusMap :status="booking.status" />
                        </div>

                        <h5 class="card-title fw-bold text-dark mb-1">
                            {{ booking.firstname }} {{ booking.lastname }}
                        </h5>
                        <p class="small text-muted mb-4 border-bottom pb-3">
                            <i class="bi bi-envelope me-1"></i> {{ booking.contactEmail }}
                        </p>

                        <div class="row g-3">
                            <div class="col-6">
                                <label class="text-uppercase x-small fw-bold text-muted d-block"
                                    style="font-size: 0.7rem;">Dormitory</label>
                                <span class="text-dark fw-medium"><i class="bi bi-building me-1"
                                        style="color: #FC7D07;"></i> {{ booking.room?.dorm?.dormName ?? 'N/A' }}</span>
                            </div>
                            <div class="col-6">
                                <label class="text-uppercase x-small fw-bold text-muted d-block"
                                    style="font-size: 0.7rem;">Unit</label>
                                <span class="text-dark fw-medium"><i class="bi bi-door-closed me-1"
                                        style="color: #FC7D07;"></i> {{ booking.room?.roomNumber ?? 'N/A' }}</span>
                            </div>

                            <div class="col-12 mt-3">
                                <div class="d-flex align-items-center mb-2">
                                    <i class="bi bi-calendar3 me-2" style="color: #003C87;"></i>
                                    <span class="text-secondary">Move-in: <strong class="text-dark">{{
                                            formatDate(booking.moveInDate) }}</strong></span>
                                </div>
                                <div class="d-flex align-items-center">
                                    <i class="bi bi-credit-card-2-front me-2" style="color: #003C87;"></i>
                                    <span class="text-secondary">Method:
                                        <strong class="text-dark" v-if="booking.payment.length">
                                            <span v-for="(pay, index) in booking.payment" :key="index">
                                                {{ pay.paymentType }}<span v-if="index !== booking.payment.length - 1">,
                                                </span>
                                            </span>
                                        </strong>
                                        <strong class="text-danger" v-else>Pending</strong>
                                    </span>
                                </div>
                            </div>
                        </div>

                        <div class="mt-4">
                            <button class="btn w-100 fw-bold py-2 rounded-3 text-white"
                                style="background-color: #003C87; transition: 0.3s;"
                                @click="openTenant(booking.bookingID)">
                                VIEW DETAILS
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        
        </div>

        <div v-if="lastPage > 1" class="d-flex justify-content-center my-4">
            <nav>
                <ul class="pagination pagination-sm shadow-sm">
                    <li :class="['page-item', { disabled: currentPage === 1 }]">
                        <button class="page-link" @click="handlePagination(currentPage - 1)"
                            :disabled="currentPage === 1">
                            &laquo; Prev
                        </button>
                    </li>
                    <li class="page-item disabled">
                        <span class="page-link">Page {{ currentPage }} of {{ lastPage }}</span>
                    </li>
                    <li :class="['page-item', { disabled: currentPage === lastPage }]">
                        <button class="page-link" @click="handlePagination(currentPage + 1)"
                            :disabled="currentPage === lastPage">
                            Next &raquo;
                        </button>
                    </li>
                </ul>
            </nav>
        </div>

        <!--Modal Tenant Appoval-->
        <!-- Use v-if to render the modal only if needed -->
        <div v-if="VisibleModalApproval" class="modal fade show d-block"
            style="background: rgba(0, 30, 60, 0.6); backdrop-filter: blur(4px);" tabindex="-1">
            <div class="modal-dialog modal-lg modal-dialog-centered">
                <div class="modal-content shadow-lg rounded-4 border-0 overflow-hidden">

                    <div class="modal-header border-0 p-4" style="background-color: #003C87;">
                        <h5 class="modal-title text-white fw-bold d-flex align-items-center">
                            <i class="bi bi-person-badge me-2"></i> Tenant Profile Details
                        </h5>
                        <button type="button" class="btn-close btn-close-white"
                            @click="VisibleModalApproval = false"></button>
                    </div>

                    <div class="modal-body px-lg-5 py-4">

                        <div class="d-flex align-items-center mb-4 pb-3 border-bottom">
                            <div class="position-relative">
                                <img :src="selectedtenant.pictureID"
                                    class="rounded-circle border border-4 border-white shadow"
                                    style="width: 110px; height: 110px; object-fit: cover;" />
                                <div class="position-absolute bottom-0 end-0 mb-1">
                                    <statusMap :status="selectedtenant.status" />
                                </div>
                            </div>
                            <div class="ms-4">
                                <h3 class="fw-bold mb-0 text-dark">{{ selectedtenant.firstname }} {{
                                    selectedtenant.lastname }}</h3>
                                <p class="text-muted mb-0"><i class="bi bi-envelope-at me-1"></i> {{
                                    selectedtenant.contactEmail }}</p>
                            </div>
                        </div>

                        <div class="row g-4 mb-4">
                            <div class="col-md-6">
                                <div class="mb-3">
                                    <label class="small text-uppercase fw-bold text-muted mb-1 d-block">Personal
                                        Details</label>
                                    <div class="p-2 rounded-3 bg-light">
                                        <p class="mb-1"><strong>Age:</strong> {{ selectedtenant.age }}</p>
                                        <p class="mb-1"><strong>Gender:</strong> {{ selectedtenant.gender }}</p>
                                        <p class="mb-0"><strong>Contact:</strong> {{ selectedtenant.contactNumber }}</p>
                                    </div>
                                </div>
                                <div class="mb-3">
                                    <label class="small text-uppercase fw-bold text-muted mb-1 d-block">Stay
                                        Duration</label>
                                    <div class="p-2 rounded-3 bg-light border-start border-4"
                                        style="border-color: #FC7D07 !important;">
                                        <p class="mb-1"><strong>Move-in:</strong> {{
                                            formatDate(selectedtenant.moveInDate) }}</p>
                                        <p class="mb-0"><strong>Move-out:</strong> {{
                                            formatDate(selectedtenant?.moveOutDate) }}</p>
                                    </div>
                                </div>
                            </div>

                            <div class="col-md-6">
                                <div class="mb-3">
                                    <label class="small text-uppercase fw-bold text-muted mb-1 d-block">Housing
                                        Information</label>
                                    <div class="p-2 rounded-3 bg-light">
                                        <p class="mb-1"><strong>Dorm:</strong> {{ selectedtenant.room?.dorm?.dormName ||
                                            'N/A' }}</p>
                                        <p class="mb-1"><strong>Room Number:</strong> {{ selectedtenant.room?.roomNumber
                                            }}</p>
                                        <p class="mb-0 text-primary fw-bold"><strong>Monthly:</strong> ₱{{
                                            Number(selectedtenant.room?.price).toLocaleString(undefined, {
                                            minimumFractionDigits: 2 }) }}</p>
                                    </div>
                                </div>
                                <div>
                                    <label class="small text-uppercase fw-bold text-muted mb-1 d-block">Location</label>
                                    <p class="small text-dark px-2"><i class="bi bi-geo-alt-fill text-danger"></i> {{
                                        selectedtenant.room?.dorm?.address }}</p>
                                </div>
                            </div>
                        </div>

                        <div class="p-4 rounded-4 border" style="background-color: #f8fbff;">
                            <label class="fw-bold text-dark mb-3">Update Booking Status</label>
                            <div class="row align-items-center">
                                <div class="col-md-12 mb-3">
                                    <select class="form-select border-2" v-model="status"
                                        style="border-color: #dee2e6;">
                                        <option value="" disabled selected>Change status to...</option>
                                        <option value="pending"
                                            v-if="['cancelled', 'rejected'].includes(selectedtenant.status)">Pending -
                                            Awaiting Confirmation</option>
                                        <option value="confirmed"
                                            v-if="['cancelled', 'rejected', 'pending'].includes(selectedtenant.status)">
                                            Confirmed - Tenant To Pay</option>
                                        <option value="approved"
                                            v-if="['cancelled', 'rejected', 'paid'].includes(selectedtenant.status)">
                                            Approved - Booking Approved</option>
                                        <option value="rejected">Rejected - User Initiated</option>
                                    </select>
                                </div>
                            </div>

                            <div v-if="selectedtenant.status === 'paid' && selectedtenant.payment.length"
                                class="text-center pt-3 border-top mt-2">
                                <label class="small text-uppercase fw-bold text-muted mb-2 d-block">Payment
                                    Verification</label>
                                <div class="d-inline-block position-relative">
                                    <img :src="selectedtenant.payment[0].paymentImage"
                                        class="img-thumbnail rounded-3 shadow-sm"
                                        style="width: 100%; max-width: 300px; height: auto; cursor: pointer;"
                                        @click="showFullImage = true" />
                                    <div class="mt-2 text-dark fw-bold small">
                                        <i class="bi bi-wallet2 me-1"></i> {{ selectedtenant.payment[0].paymentType }}
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div class="mt-4">
                            <BookingStatus :status="selectedtenant.status" role="landlord" />
                        </div>
                    </div>

                    <div class="modal-footer bg-light border-0 px-4 py-3">
                        <button class="btn btn-outline-secondary fw-bold px-4 rounded-3"
                            @click="messagePage(selectedtenant.fktenantID)">
                            <i class="bi bi-chat-dots me-2"></i>MESSAGE
                        </button>
                        <button class="btn px-5 fw-bold rounded-3 text-white shadow-sm"
                            style="background-color: #FC7D07;" @click="handleBookingAction(selectedtenant.bookingID)">
                            UPDATE BOOKING
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <div v-if="showFullImage" class="modal fade show d-block"
        style="background: rgba(0, 0, 0, 0.9); backdrop-filter: blur(10px); z-index: 1060;"
        @click.self="showFullImage = false">

        <div class="modal-dialog modal-dialog-centered modal-xl">
            <div class="modal-content bg-transparent border-0">

                <div class="d-flex justify-content-between align-items-center mb-3 px-3">
                    <div>
                        <h5 class="text-white mb-0 fw-bold">
                            <i class="bi bi-receipt me-2" style="color: #FC7D07;"></i>
                            Payment Proof: {{ selectedtenant.firstname }} {{ selectedtenant.lastname }}
                        </h5>
                        <small class="text-white-50">Booking #{{ selectedtenant.bookingID }}</small>
                    </div>

                    <button type="button"
                        class="btn btn-light rounded-circle d-flex align-items-center justify-content-center shadow-lg"
                        style="width: 45px; height: 45px; transition: 0.3s;" @click="showFullImage = false">
                        <i class="bi bi-x-lg fs-5"></i>
                    </button>
                </div>

                <div class="modal-body p-0 position-relative text-center">
                    <img :src="selectedtenant.payment[0].paymentImage" class="img-fluid rounded-4 shadow-lg"
                        style="max-height: 80vh; border: 2px solid rgba(255,255,255,0.1); object-fit: contain;"
                        alt="Payment Receipt" />
                </div>

                <div class="mt-4 text-center">
                    <div class="d-inline-flex align-items-center bg-white px-4 py-2 rounded-pill shadow">
                        <span class="fw-bold me-3 text-dark">
                            <i class="bi bi-credit-card-2-back me-2" style="color: #003C87;"></i>
                            {{ selectedtenant.payment[0].paymentType }}
                        </span>
                        <div style="width: 1px; height: 20px; background: #dee2e6;" class="me-3"></div>
                        <button class="btn btn-link text-decoration-none p-0 fw-bold"
                            style="color: #003C87; font-size: 0.9rem;" @click="showFullImage = false">
                            DONE VIEWING
                        </button>
                    </div>
                </div>

            </div>
        </div>
    </div>
    <Modalconfirmation ref="modal" />
    <Toastcomponents ref="toast" />



</template>
<script>
import axios from 'axios';
import Toastcomponents from '@/components/Toastcomponents.vue';
import Loader from '@/components/loader.vue';
import Modalconfirmation from '@/components/modalconfirmation.vue';
import { debounce } from 'lodash';
import NotificationList from '@/components/notifications.vue';
import BookingStatus from '@/components/BookingStatusAlert.vue';
import statusMap from '@/components/statusmap.vue';

import { toHandlers } from 'vue';
export default {
    components: {
        Toastcomponents,
        Loader,
        Modalconfirmation,
        NotificationList,
        BookingStatus,
        statusMap
    },
    data() {
        return {
            landlord_id: null,
            VisibleModalApproval: false,
            tenants: [],
            filterMode: '',
            dorms: [],
            searchTerm: '',
            selectedDormId: '',
            selectedapplicationStatus: '',
            selectedroomNumber: '',
            selectedtenant: [],
            selectedBookingID: '',
            showFullImage: false,
            tenant: {},
            filteredTenants: [],
            currentPage: 1,
            lastPage: 1,
            notifications: [],
            receiverID: '',
            status: '',
            hasSubscribed: false,
        };
    },

    methods: {
        subscribeToNotifications() {
            if (this.hasSubscribed) return; // prevent multiple subscriptions
            this.hasSubscribed = true;

            this.receiverID = this.landlord_id;
            Echo.private(`notifications.${this.receiverID}`)
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
        async searchBooking(page = 1) {
            try {
                this.$refs.loader.loading = true;

                const response = await axios.get('/search-booking', {
                    params: {
                        search: this.searchTerm,
                        page: page
                    }
                });

                if (response.data.status === 'success') {
                    this.$refs.loader.loading = false;
                    this.selectedDormId = '';
                    this.selectedapplicationStatus = '';
                    this.selectedroomNumber = '';
                    this.tenants = response.data.booking.data;
                    this.lastPage = response.data.booking.last_page;
                } else {
                    this.$refs.toast.showToast(response.data.message, 'danger');
                }
            } catch (error) {
                console.error('Error searching bookings:', error);
                this.$refs.toast.showToast('Search failed.', 'danger');
            }
        },
        displaydorms() {
            axios.get('/api/dorms')
                .then(response => {
                    this.dorms = response.data;
                });
        },
        dormId(dorm) {
            this.selectedDorm = dorm;
        },
        filterDorms(page = 1) {
            this.$refs.loader.loading = true;
            if (this.selectedDormId === 'all') {
                this.bookingList();
                this.$refs.loader.loading = false;
                return;
            }

            axios.get(`/api/dorms/${this.selectedDormId}/tenants`, {
                params: { page }
            })
                .then(response => {
                    this.searchTerm = '';
                    this.selectedapplicationStatus = '';
                    this.selectedroomNumber = '';
                    this.tenants = response.data.data;
                    this.lastPage = response.data.last_page;
                })
                .catch(error => {
                    this.$refs.loader.loading = false;
                    console.error('Error fetching dorm tenants:', error);

                }).finally(() => {
                    this.$refs.loader.loading = false;
                });
        },

        filterApplicationStatus(page = 1) {
            this.$refs.loader.loading = true;

            if (this.selectedapplicationStatus === 'all') {
                this.bookingList();
                this.$refs.loader.loading = false;
                return;
            }

            axios.get('/api/applications/booking', {
                params: {
                    selectedapplicationStatus: this.selectedapplicationStatus,
                    page: page
                }
            })
                .then(response => {
                    this.$refs.loader.loading = false;
                    this.searchTerm = '';
                    this.selectedDormId = '';
                    this.selectedroomNumber = '';
                    this.tenants = response.data.data;
                    this.lastPage = response.data.last_page;

                })
                .catch(error => {
                    this.$refs.loader.loading = false;
                    console.error("Error filtering applications:", error);
                });
        },


        filterroomNumber(page = 1) {
            this.$refs.loader.loading = true;

            if (this.selectedroomNumber === 'all') {
                this.bookingList();
                this.$refs.loader.loading = false;
                return;
            }

            axios.get(`/api/roomnumber/booking`, {
                params: {
                    roomsNumber: this.selectedroomNumber,
                    page: page,
                }
            })
                .then(response => {
                    this.$refs.loader.loading = false;
                    this.searchTerm = '';
                    this.selectedDormId = '';
                    this.selectedapplicationStatus = '';
                    if (response.data.data) {
                        this.tenants = response.data.data;
                        this.lastPage = response.data.last_page;
                    } else {
                        this.tenants = response.data;
                    }

                })
                .catch(error => {
                    this.$refs.loader.loading = false;
                    console.error("Error fetching room data:", error);
                });
        },

        async bookingList(page = 1) {
            try {
                this.$refs.loader.loading = true;

                const response = await axios.get(`/booking-list?page=${this.currentPage}`);

                if (response.data.status === 'success') {
                    this.tenants = response.data.booking.data;
                    this.lastPage = response.data.booking.last_page;

                } else {
                    this.$refs.toast.showToast(response.data.message, 'danger');
                }
            } catch (error) {
                console.error('Error fetching bookings:', error);
                this.$refs.toast.showToast("Failed to fetch bookings", 'danger');
            }
            finally {
                this.$refs.loader.loading = false;

            }
        },
        async openTenant(booking_id) {
            this.selectedBookingID = booking_id;
            this.$refs.loader.loading = true;

            try {
                const response = await axios.get(`/booking-tenant-view/${this.selectedBookingID}`);
                this.selectedtenant = response.data.tenant;
                this.VisibleModalApproval = true;
                this.$refs.loader.loading = false;
            }
            catch (error) {
                console.error('Error fetching tenant data:', error);
            }
            finally {
                this.$refs.loader.loading = false;

            }
        },
        async handleBookingAction(booking_id) {
            try {
                const confirmed = await this.$refs.modal.show({
                    title: 'Update Booking',
                    message: `The tenant's status will be updated to '${this.status}' for this booking. Are you sure?`,
                    functionName: 'Update Booking',
                });

                if (!confirmed) {
                    return;
                }


                this.$refs.loader.loading = true;
                const formdata = new FormData();
                formdata.append('bookingID', booking_id);
                formdata.append('landlordID', this.landlord_id);
                formdata.append('status', this.status);
                
                const response = await axios.post('/handle-tenant-booking', formdata);
                if (response.data.status === 'success') {
                    this.$refs.toast.showToast(response.data.message, 'success');
                    this.selectedtenant = response.data.tenant;
                    this.errors = {};
                    this.VisibleModalApproval = false;
                    this.$refs.loader.loading = false;
                    this.bookingList();
                    return;
                }

            } catch (error) {
                this.$refs.loader.loading = false;

                if (error.response && error.response.status === 403) {

                    this.$refs.toast.showToast(error.response.data.message, 'danger'); // ✅ Show danger toast
                } else {
                    console.error('Error approving tenant:', error);
                    this.$refs.toast.showToast('Something went wrong while approving the tenant.', 'danger');
                }
            } finally {
                this.$refs.loader.loading = false;
                this.VisibleModalApproval = false;

            }
        },

        async deleteBooking(booking_id) {
            try {
                // Confirm user action
                const confirmed = await this.$refs.modal.show({
                    title: 'Delete Booking',
                    message: `Confirm delete to this Book’s information?`,
                    functionName: 'Delete Booking',
                });

                if (!confirmed) {
                    return;
                }

                // Show loader
                this.$refs.loader.loading = true;

                const response = await axios.delete(`/delete-booking/${booking_id}`);

                // ✅ Show success toast
                this.$refs.toast.showToast(response.data.message, 'success');

                // Optional: Refresh list or remove from UI
                this.bookingList();

            } catch (error) {
                const errorMsg = error.response?.data?.message || 'An error occurred while deleting.';
                this.$refs.toast.showToast(errorMsg, 'danger');
                console.error('Delete error:', error);
            } finally {
                this.$refs.loader.loading = false;
            }
        },

        handlePagination(page) {

            if (page < 1 || page > this.lastPage) return;
            this.currentPage = page;


            // 👇 Priority check (only one filter is expected at a time)
            if (this.selectedapplicationStatus && this.selectedapplicationStatus !== 'all') {
                this.filterApplicationStatus(page);
            } else if (this.selectedDormId && this.selectedDormId !== 'all') {
                this.filterDorms(page);
            } else if (this.selectedroomNumber && this.selectedroomNumber !== 'all') {
                this.filterroomNumber(page);
            } else if (this.searchTerm && this.searchTerm.trim() !== '') {
                this.searchBooking(page);
            } else {
                this.bookingList(page); // Default
            }
        },
        messagePage(fktenantID) {
            const url = `/api/select/landlord/conversations/${this.landlord_id}?tenant_id=${fktenantID}`;
            window.location.href = url;
        },
        formatDate(date) {
            if (!date) return "N/A";
            return new Date(date).toLocaleDateString('en-US', {
                year: 'numeric',
                month: 'long',
                day: 'numeric'
            });
        },




    },
    mounted() {
        const el = document.getElementById('BookingManagement');
        if (el) {
            this.landlord_id = el.getAttribute('landlord-id');
        }
        this.subscribeToNotifications();
        this.bookingList();
        this.displaydorms();

    },
    computed:
    {
        uniqueRooms() {
            const seen = new Set();
            return this.tenants.filter(tenant => {
                if (seen.has(tenant.fkroomID)) return false;
                seen.add(tenant.fkroomID);
                return true;
            });
        }
    },

    watch: {
        searchTerm: {
            handler: debounce(function (newVal) {
                if (newVal.trim() !== '') {
                    this.searchBooking();
                } else {
                    this.bookingList();
                }
            }, 300),
            immediate: false
        },
        selectedDormId(newVal) {
            if (newVal !== '') {
                this.selectedapplicationStatus = '';
                this.selectedroomNumber = '';
                this.searchTerm = '';
                this.filterMode = 'dorm';
                this.handlePagination(1);
            }
        },
        selectedapplicationStatus(newVal) {
            if (newVal !== '') {
                this.selectedDormId = '';
                this.selectedroomNumber = '';
                this.searchTerm = '';
                this.filterMode = 'status';
                this.handlePagination(1);
            }
        },
        selectedroomNumber(newVal) {
            if (newVal !== '') {
                this.selectedDormId = '';
                this.selectedapplicationStatus = '';
                this.searchTerm = '';
                this.filterMode = 'room';
                this.handlePagination(1);
            }
        }
    }



};
</script>
<style scoped src="../../../../css/landlord/booking.css">
</style>
