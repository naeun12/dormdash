<template>
    <Loader ref="loader" />
    <Toastcomponents ref="toast" />
    <Modalconfirmation ref="modal" />
    <NotificationList ref="toastRef" />

    <div class="container-fluid py-4 px-lg-5 bg-light min-vh-100">

        <div class="d-flex justify-content-center mb-4">
            <div class="bg-white p-2 rounded-pill shadow-sm d-inline-flex gap-2 border">
                <button
                    :class="['btn rounded-pill px-4 fw-bold transition-all', chooseStatus === 'Available' ? 'btn-primary shadow' : 'btn-light text-muted']"
                    @click="ClickavailableRooms()">
                    <i class="bi bi-door-open-fill me-2"></i>Available
                </button>
                <button
                    :class="['btn rounded-pill px-4 fw-bold transition-all', chooseStatus === 'Occupied' ? 'btn-primary shadow' : 'btn-light text-muted']"
                    @click="ClickoccupiedRooms()">
                    <i class="bi bi-person-workspace me-2"></i>Occupied
                </button>
            </div>
        </div>

        <div class="row g-3 mb-5 justify-content-center">
            <div class="col-12 col-md-4 col-lg-3">
                <div class="input-group shadow-sm rounded-4 overflow-hidden border-0">
                    <span class="input-group-text bg-white border-0 text-primary"><i class="bi bi-tags-fill"></i></span>
                    <select class="form-select border-0 py-2 fw-medium" v-model="selectPriceRange"
                        @change="filterByPriceRange($event)">
                        <option value="" disabled>Price Range</option>
                        <option value="all">All Prices</option>
                        <option value="0-100">₱0 - ₱100</option>
                        <option value="101-300">₱101 - ₱300</option>
                        <option value="301-99999">₱300+</option>
                    </select>
                </div>
            </div>
            <div class="col-12 col-md-4 col-lg-3">
                <div class="input-group shadow-sm rounded-4 overflow-hidden border-0">
                    <span class="input-group-text bg-white border-0 text-primary"><i
                            class="bi bi-gender-ambiguous"></i></span>
                    <select class="form-select border-0 py-2 fw-medium" v-model="selectedGender"
                        @change="filterByGender($event)">
                        <option value="" disabled>Gender Preference</option>
                        <option value="all">All Genders</option>
                        <option value="Male Only">Male Only</option>
                        <option value="Female Only">Female Only</option>
                    </select>
                </div>
            </div>
        </div>

        <div v-if="!rooms.length"
            class="text-center py-5 bg-white rounded-5 shadow-sm border mx-auto animate__animated animate__fadeIn"
            style="max-width: 600px;">
            <div class="mb-3 text-muted display-1"><i class="bi bi-door-closed"></i></div>
            <h4 class="fw-bold">No Rooms Found</h4>
            <p class="text-muted px-4">We couldn't find any rooms matching your current filters. Try adjusting your
                preferences or check back later.</p>
        </div>

        <div class="row g-4">
            <div class="col-12" v-for="(room, index) in visibleRooms" :key="room.room_id">
                <div
                    class="card border-0 shadow-sm rounded-4 overflow-hidden hover-lift animate__animated animate__fadeInUp">
                    <div class="row g-0">
                        <div class="col-md-4 position-relative">
                            <img :src="room.roomImages" :alt="room.listingType" class="h-100 w-100"
                                style="object-fit: cover; min-height: 220px;" />
                            <div class="position-absolute top-0 start-0 m-3">
                                <span class="badge glass-effect text-white rounded-pill px-3 py-2 shadow-sm"
                                    style="background: rgba(0,0,0,0.4); backdrop-filter: blur(8px);">
                                    <i class="bi bi-camera me-1"></i> Featured
                                </span>
                            </div>
                        </div>

                        <div class="col-md-8 p-4 d-flex flex-column justify-content-between bg-white">
                            <div>
                                <div class="d-flex justify-content-between align-items-start mb-2">
                                    <h4 class="fw-bold text-dark mb-0">
                                        <i class="bi bi-house-heart text-primary me-2"></i>
                                        {{ room.listingType || 'Available Dorm' }}
                                    </h4>
                                    <h4 class="fw-bold text-success mb-0">₱{{ Number(room.price).toLocaleString()
                                        }}<small class="fs-6 text-muted">/head</small></h4>
                                </div>

                                <div class="d-flex flex-wrap gap-2 my-3">
                                    <span class="badge bg-light text-dark border rounded-pill px-3 py-2 fw-normal">
                                        <i class="bi bi-arrows-fullscreen text-primary me-1"></i> {{ room.areaSqm ||
                                        'N/A' }} sqm
                                    </span>
                                    <span class="badge bg-light text-dark border rounded-pill px-3 py-2 fw-normal">
                                        <i class="bi bi-person-check text-primary me-1"></i> {{ room.genderPreference }}
                                    </span>
                                    <span class="badge bg-light text-dark border rounded-pill px-3 py-2 fw-normal">
                                        <i class="bi bi-lamp text-primary me-1"></i> {{ room.furnishing_status }}
                                    </span>
                                </div>
                            </div>

                            <div class="d-flex justify-content-between align-items-center mt-3 pt-3 border-top">
                                <a @click="openRoomDetails(room.roomID)"
                                    class="btn btn-link text-primary text-decoration-none fw-bold p-0">
                                    <i class="bi bi-info-circle me-1"></i> Full Details
                                </a>
                                <div class="d-flex gap-2">
                                    <button v-if="chooseStatus === 'Occupied'" @click="openReservationModal(room)"
                                        class="btn btn-outline-primary rounded-pill px-4 fw-bold shadow-sm">
                                        <i class="bi bi-calendar-event me-2"></i>Reserve Room
                                    </button>
                                    <button v-if="chooseStatus === 'Available'" @click="bookRoom(room)"
                                        class="btn btn-primary rounded-pill px-4 fw-bold shadow">
                                        <i class="bi bi-lightning-fill me-1"></i>Book Now
                                    </button>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div class="text-center mt-5" v-if="rooms.length > 3">
            <button @click.prevent="toggleShowMore"
                class="btn btn-white border shadow-sm rounded-pill px-5 fw-bold text-primary">
                {{ showAll ? 'Show Less Rooms' : 'View More Available Rooms' }}
            </button>
        </div>

        <div v-if="openRoomDetailsModal" class="modal fade show d-block" tabindex="-1"
            style="background: rgba(0,0,0,0.7); backdrop-filter: blur(4px);">
            <div class="modal-dialog modal-lg modal-dialog-centered">
                <div
                    class="modal-content border-0 shadow-lg rounded-5 overflow-hidden animate__animated animate__zoomIn">
                    <div class="modal-header bg-dark text-white border-0 py-3 px-4">
                        <h5 class="modal-title fw-bold">Room Information</h5>
                        <button type="button" class="btn-close btn-close-white" @click="CloseRoomDetails()"></button>
                    </div>
                    <div class="modal-body p-4">
                        <div class="text-center mb-4">
                            <img :src="roomsDetail?.roomImages" class="img-fluid rounded-4 shadow w-100"
                                style="max-height: 350px; object-fit: cover;" />
                        </div>
                        <div class="row g-3">
                            <div class="col-md-6"
                                v-for="(val, label) in { 'Room Number': roomsDetail?.roomNumber, 'Room Type': roomsDetail?.roomType, 'Monthly Rate': '₱' + Number(roomsDetail?.price).toLocaleString() }"
                                :key="label">
                                <div class="p-3 bg-light rounded-4 border">
                                    <small class="text-muted d-block text-uppercase fw-bold">{{ label }}</small>
                                    <span class="fs-5 fw-bold text-dark">{{ val || 'N/A' }}</span>
                                </div>
                            </div>
                            <div class="col-md-6">
                                <div
                                    class="p-3 bg-light rounded-4 border h-100 d-flex flex-column justify-content-center">
                                    <small class="text-muted d-block text-uppercase fw-bold">Status</small>
                                    <span
                                        :class="['badge rounded-pill mt-1 py-2 fs-6', roomsDetail?.availability === 'Available' ? 'bg-success' : 'bg-danger']">
                                        {{ roomsDetail?.availability }}
                                    </span>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="modal-footer border-0 justify-content-center pb-4">
                        <button type="button" class="btn btn-dark rounded-pill px-5 fw-bold py-2"
                            @click="CloseRoomDetails()">Got it!</button>
                    </div>
                </div>
            </div>
        </div>

        <div v-if="reservationDetailsModal" class="modal fade show d-block" tabindex="-1"
            style="background: rgba(0,0,0,0.7); backdrop-filter: blur(4px);">
            <div class="modal-dialog modal-lg modal-dialog-centered">
                <div
                    class="modal-content border-0 shadow-lg rounded-5 overflow-hidden animate__animated animate__fadeInDown">
                    <div class="modal-header bg-primary text-white border-0 py-3 px-4">
                        <h5 class="modal-title fw-bold">Confirm Reservation</h5>
                        <button type="button" class="btn-close btn-close-white"
                            @click="reservationDetailsModal = false"></button>
                    </div>
                    <div class="modal-body p-4">
                        <div class="row g-4 align-items-center">
                            <div class="col-md-4">
                                <img :src="this.imageUrl" class="img-fluid rounded-4 shadow-sm" />
                            </div>
                            <div class="col-md-8">
                                <div class="card border-0 bg-light rounded-4 p-4">
                                    <h5 class="fw-bold mb-3 border-bottom pb-2">Customer Details</h5>
                                    <div class="row g-3">
                                        <div class="col-6"><small class="text-muted d-block">Name</small><strong
                                                class="text-dark">{{ firstname }} {{ lastname }}</strong></div>
                                        <div class="col-6"><small class="text-muted d-block">Gender</small><strong
                                                class="text-dark">{{ sex }}</strong></div>
                                        <div class="col-12"><small class="text-muted d-block">Email</small><strong
                                                class="text-dark text-break">{{ email }}</strong></div>
                                        <div class="col-6"><small class="text-muted d-block">Contact</small><strong
                                                class="text-dark">{{ contactInfo }}</strong></div>
                                        <div class="col-6"><small class="text-muted d-block">Age</small><strong
                                                class="text-dark">{{ age }} yrs old</strong></div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="modal-footer border-0 justify-content-center pb-4">
                        <button type="button" class="btn btn-primary rounded-pill px-5 py-3 fw-bold shadow"
                            @click="reserveRoom()">
                            <i class="bi bi-check-circle-fill me-2"></i>Confirm Reservation
                        </button>
                    </div>
                </div>
            </div>
        </div>

    </div>
</template>

<style scoped src="../../../../../css/tenant/roomselection.css">

</style>

<script>
import axios from 'axios'
import Toastcomponents from '@/components/Toastcomponents.vue';
import Loader from '@/components/loader.vue';
import Modalconfirmation from '@/components/modalconfirmation.vue';
import NotificationList from '@/components/notifications.vue';

import { toHandlers } from 'vue';
import { parse } from 'vue/compiler-sfc';
export default {
    components: {
        Toastcomponents,
        Loader,
        Modalconfirmation,
        NotificationList,
    },
    data() {
        return {
            isAvailable: 'btn-outline-primary',
            chooseStatus: '',
            isOccupied: 'btn-outline-danger',
            selectPriceRange: '',
            selectedGender: '',
            isGender: '',
            dormitory_id: '',
            rooms: [],
            showAll: false,
            roomsToShow: 3,
            room_id: '',
            tenant_id: '',
            openRoomDetailsModal: false,
            reservationDetailsModal: false,
            roomsDetail: '',
            selectedRoomId: '',
            firstname: '',
            lastname: '',
            email: '',
            contactInfo: '',
            age: '',
            sex: '',
            roomNu: '',
            roomType: '',
            price: '',
            idPicturePreview: '',
            imageUrl: null,
            selectedRoomId: '',
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
        getInformationData() {
            const data = {
                firstname: this.firstname,
                lastname: this.lastname,
                email: this.email,
                contactInfo: this.contactInfo,
                age: this.age,
                sex: this.sex,
                idPicturePreview: this.idPicturePreview,
                imageUrl: this.imageUrl,
                selectedRoomId: this.selectedRoomId,
            };
            localStorage.setItem('tenantInfo', JSON.stringify(data));

        },
        async availableRooms() {
            try {
                this.$refs.loader.loading = true;
                this.selectPriceRange = '';
                const response = await axios.get(`/available-room/${this.dormitory_id}`);
                this.rooms = response.data.rooms;
                this.isAvailable = 'btn-primary';
                this.isOccupied = 'btn-outline-danger';
                this.chooseStatus = 'Available';
                this.selectedGender = '';

            }
            catch (error) {

            }
            finally {
                this.$refs.loader.loading = false;

            }
        },
        async ClickavailableRooms() {
            this.availableRooms();
        },

        async ClickoccupiedRooms() {
            try {
                this.$refs.loader.loading = true;

                this.selectPriceRange = '';
                const response = await axios.get(`/occupied-room/${this.dormitory_id}`);
                this.rooms = response.data.rooms;
                this.isAvailable = 'btn-outline-primary';
                this.isOccupied = 'btn-danger';
                this.chooseStatus = 'Occupied';
            }
            catch (error) {
                this.$refs.loader.loading = false;

            }
            finally {
                this.$refs.loader.loading = false;

            }
        },
        async filterByPriceRange(event) {
            this.$refs.loader.loading = true;

            const range = event.target.value;
            if (range === 'all') {
                if (this.chooseStatus === 'Available') {
                    this.availableRooms();

                }
                else if (this.chooseStatus === 'Occupied') {
                    this.ClickoccupiedRooms();

                }
                return;
            }

            const [min, max] = range.split('-');

            try {
                const response = await axios.get(`/filter-price-range/${this.dormitory_id}?min=${min}&max=${max}&chooseStatus=${this.chooseStatus}`);
                this.rooms = response.data.rooms;
                this.selectedGender = '';

            } catch (error) {
                console.error('Error fetching price-filtered rooms', error);
            }
            finally {
                this.$refs.loader.loading = false;

            }
        },
        async filterByGender(event) {
            this.$refs.loader.loading = true;

            const gender = event.target.value;

            // If "Any Gender" is selected, reload all rooms by availability
            if (gender === 'all') {
                if (this.chooseStatus === 'Available') {
                    await this.availableRooms();
                } else if (this.chooseStatus === 'Occupied') {
                    await this.ClickoccupiedRooms();
                }
                return;
            }

            // Else fetch rooms filtered by gender and status
            try {
                const response = await axios.get(
                    `/filter-gender/${this.dormitory_id}?gender=${gender}&chooseStatus=${this.chooseStatus}`
                );
                this.rooms = response.data.rooms;
                this.selectPriceRange = '';

            } catch (error) {
                console.error('Error filtering by gender:', error);
            }
            finally {
                this.$refs.loader.loading = false;

            }
        },

        async openRoomDetails(roomId) {
            this.selectedRoomId = roomId;
            try {
                const response = await axios.get(`/view-room-details/${this.selectedRoomId}`);
                if (response.data.status === 'success') {
                    this.roomsDetail = response.data.room;
                    this.openRoomDetailsModal = true;
                    console.log(this.roomsDetail);

                }
            }
            catch (error) {

            }
        },
        openReservationModal(room) {
            this.room_id = room.roomID;
            this.roomNu = room.roomNumber;
            this.roomsex = room.genderPreference;
            this.dormsec = room.dorm?.occupancyType;
            this.reservationDetailsModal = true;
        },
        async reserveRoom() {
            try {

                const tenantSex = this.sex.toLowerCase(); // "male" / "female"

                // Normalize room preference
                let roomPref = this.roomsex?.toLowerCase();
                if (roomPref === "male only") roomPref = "male";
                if (roomPref === "female only") roomPref = "female";
                if (roomPref === "any gender") roomPref = "any";

                // Normalize occupancy type
                let occupancyType = this.dormsec?.toLowerCase();
                if (occupancyType === "male only") occupancyType = "male";
                if (occupancyType === "female only") occupancyType = "female";
                if (occupancyType === "any gender") occupancyType = "any";

                // ✅ Gender restriction check
                const genderAllowed =
                    roomPref === "any" || tenantSex === roomPref;

                // ✅ Occupancy restriction check
                const occupancyAllowed =
                    !occupancyType || occupancyType === "any" || tenantSex === occupancyType;

                if (!genderAllowed || !occupancyAllowed) {
                    this.$refs.toast.showToast('You are not eligible to reserve this room.', 'warning');
                    this.reservationDetails = false;
                    return;
                }

                // Confirm reservation
                const confirmed = await this.$refs.modal.show({
                    title: `Are you sure you want to reserve Room #${this.roomNu || this.room_id}?`,
                    message: `This will reserve Room #${this.roomNu || this.room_id} for you.`,
                    functionName: 'Confirm Reservation'
                });

                if (!confirmed) return;

                this.$refs.loader.loading = true;

                const formdata = new FormData();
                formdata.append('dormitory_id', this.dormitory_id);
                formdata.append('room_id', this.room_id);
                formdata.append('tenant_id', this.tenant_id);
                formdata.append('firstname', this.firstname);
                formdata.append('lastname', this.lastname);
                formdata.append('contact_number', this.contactInfo);
                formdata.append('email', this.email);
                formdata.append('age', this.age);
                formdata.append('gender', this.sex);
                formdata.append('studentpicture_id', this.imageUrl);
                const response = await axios.post('/reserved-room', formdata);

                if (response.data.status === 'success') {
                    this.$refs.toast.showToast(response.data.message, 'success');
                    this.reservationDetailsModal = false;
                    window.location.href = `/view/reservation/${this.tenant_id}`;
                } else if (response.data.status === 'error') {
                    this.$refs.toast.showToast(response.data.message, 'danger');
                    this.reservationDetailsModal = false;
                }

            } catch (error) {
                console.error('Reservation error:', error);

                this.$refs.toast.showToast(
                    error.response?.data?.message || 'Reservation failed. Please try again later.',
                    'error'
                );
            } finally {
                this.$refs.loader.loading = false;
                this.openRoomDetailsModal = false;

            }
        },


        CloseRoomDetails() {
            this.selectedRoomId = '';
            this.openRoomDetailsModal = false;

        },
        async bookRoom(room) {

            const tenantSex = this.sex.toLowerCase(); // "male" / "female"
            // Convert room values to simpler form
            let roomPref = room.genderPreference?.toLowerCase(); // "male only", "female only", "any gender"
            if (roomPref === "male only") roomPref = "male";
            if (roomPref === "female only") roomPref = "female";
            if (roomPref === "any gender") roomPref = "any";

            let occupancyType = room.dorm?.occupancyType?.toLowerCase();
            if (occupancyType === "male only") occupancyType = "male";
            if (occupancyType === "female only") occupancyType = "female";
            if (occupancyType === "any gender") occupancyType = "any";

            // ✅ Gender eligibility
            const genderAllowed =
                roomPref === "any" || tenantSex === roomPref;

            // ✅ Occupancy eligibility
            const occupancyAllowed =
                !occupancyType || occupancyType === "any" || tenantSex === occupancyType;

            if (genderAllowed && occupancyAllowed) {
                const confirmed = await this.$refs.modal.show({
                    title: `Are you sure you want to choose Room #${room.roomNumber || room.roomID}?`,
                    message: `This will book Room #${room.roomNumber || room.roomID} for you.`,
                    functionName: 'Confirm Room Selection'
                });

                if (!confirmed) return;

                this.room_id = room.roomID;
                this.getInformationData();
                window.location.href = `/booking-process/${this.room_id}/${this.tenant_id}`;
            } else {
                this.$refs.toast.showToast('You are not eligible to book this room.', 'warning');
            }
        },
        toggleShowMore() {
            this.showAll = !this.showAll;
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
        const element = document.getElementById('roomSelection');
        this.dormitory_id = element.dataset.dormId;
        this.tenant_id = element.dataset.tenantId;
        this.availableRooms();
        this.subscribeToNotifications();

    },
    computed: {
        visibleRooms() {
            return this.showAll ? this.rooms : this.rooms.slice(0, this.roomsToShow);
        },
    }
}
</script>