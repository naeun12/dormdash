<template>
    <div class="main-content w-100">


        <div class="dashboard-content px-2 px-md-4 py-3">
            <NotificationList ref="toastRef" />
            <Loader ref="loader" />


            <!-- Header Card -->
            <div class="stats-filter-card p-4 mb-4 shadow-sm border-0">
                <div class="d-flex flex-column flex-lg-row align-items-lg-center justify-content-between gap-4">

                    <div class="d-flex align-items-center gap-3">
                        <div class="avatar-icon-box">
                            <i class="bi bi-person-badge-fill text-white fs-3"></i>
                        </div>
                        <div>
                            <span class="nav-label d-block mb-1">PROPERTIES OF</span>
                            <h3 class="fw-800 text-dark mb-0">
                                {{ landlord.firstname }} {{ landlord.lastname }}
                            </h3>
                        </div>
                    </div>

                    <div class="d-flex flex-column flex-md-row gap-3 flex-grow-1 justify-content-lg-end">

                        <div class="filter-group">
                            <label class="nav-label">REPORT DATE</label>
                            <div class="input-with-icon">
                                <i class="bi bi-calendar3"></i>
                                <input type="date" class="form-control modern-input" v-model="newDate" :max="today">
                            </div>
                        </div>

                        <div class="filter-group">
                            <label class="nav-label">SELECT DORMITORY</label>
                            <div class="dropdown">
                                <button class="btn modern-dropdown-btn dropdown-toggle w-100" type="button"
                                    id="reportDropdown" data-bs-toggle="dropdown" aria-expanded="false">
                                    <i class="bi bi-building me-2"></i>
                                    {{ selectedDorm ? selectedDorm.dormName : 'All Dormitories' }}
                                </button>
                                <ul class="dropdown-menu dropdown-menu-end shadow border-0 rounded-3 mt-2"
                                    aria-labelledby="reportDropdown">
                                    <li v-for="dorm in dorms" :key="dorm.dormID">
                                        <a class="dropdown-item py-2" href="#" @click.prevent="selectDorm(dorm)">
                                            {{ dorm.dormName }}
                                        </a>
                                    </li>
                                </ul>
                            </div>
                        </div>

                        <div class="filter-group d-flex align-items-end">
                            <a :href="`/generate-full-report/${landlord_id}?date=${newDate}`" target="_blank"
                                class="btn download-btn w-100" :class="{ 'disabled': !newDate }">
                                <i class="bi bi-cloud-arrow-down-fill me-2"></i>
                                Export PDF Report
                            </a>
                        </div>

                    </div>
                </div>
            </div>

            <!-- Info Cards -->
            <div class="row g-4 mb-4">
                <div class="col-12 col-md-6">
                    <a :href="`/all-tenants-index/${landlord_id}`" class="stat-link-wrapper">
                        <div class="modern-stat-card bg-brand-blue">
                            <div class="glass-shine"></div>
                            <div class="card-inner">
                                <div class="stat-content">
                                    <span class="stat-category">MANAGEMENT</span>
                                    <h5 class="stat-title">Total Tenants</h5>
                                    <h2 class="stat-number">{{ totalTenants }}</h2>
                                </div>
                                <div class="stat-visual">
                                    <div class="icon-blob">
                                        <i class="bi bi-people-fill"></i>
                                    </div>
                                </div>
                            </div>
                            <div class="card-action-bar">
                                <span>View Directory</span>
                                <i class="bi bi-arrow-right"></i>
                            </div>
                        </div>
                    </a>
                </div>

                <div class="col-12 col-md-6">
                    <a :href="`/landlordRoomManagement/${landlord_id}`" class="stat-link-wrapper">
                        <div class="modern-stat-card bg-brand-orange">
                            <div class="glass-shine"></div>
                            <div class="card-inner">
                                <div class="stat-content">
                                    <span class="stat-category">AVAILABILITY</span>
                                    <h5 class="stat-title">Vacant Beds</h5>
                                    <h2 class="stat-number">{{ availableBeds }}</h2>
                                </div>
                                <div class="stat-visual">
                                    <div class="icon-blob">
                                        <i class="bi bi-door-open-fill"></i>
                                    </div>
                                </div>
                            </div>
                            <div class="card-action-bar">
                                <span>Manage Inventory</span>
                                <i class="bi bi-arrow-right"></i>
                            </div>
                        </div>
                    </a>
                </div>
            </div>


            <!-- Charts -->
            <div class="charts d-flex flex-wrap gap-4 mb-4">
                <div class="modern-chart-card flex-grow-1">
                    <div class="chart-header">
                        <div class="header-icon-box">
                            <i class="bi bi-graph-up-arrow text-primary"></i>
                        </div>
                        <div>
                            <span class="nav-label">REVENUE TRENDS</span>
                            <h6 class="fw-bold m-0">Highest Rooms Profits</h6>
                        </div>
                    </div>

                    <div class="chart-body pt-3">
                        <LineChart v-if="chartData" :chart-data="chartData" :chart-options="chartOptions"
                            style="max-height: 250px;" />
                    </div>
                </div>

                <div class="modern-chart-card flex-grow-1">
                    <div class="chart-header">
                        <div class="header-icon-box icon-orange">
                            <i class="bi bi-person-lines-fill text-primary"></i>
                        </div>
                        <div>
                            <span class="nav-label">DEMOGRAPHICS</span>
                            <h6 class="fw-bold m-0">Occupants by Gender</h6>
                        </div>
                    </div>

                    <div class="chart-body d-flex align-items-center gap-3 pt-3">
                        <div class="chart-wrapper" style="width: 140px;">
                            <DoughnutChart v-if="bookingChartData" :chart-data="bookingChartData"
                                :chart-options="bookingChartOptions" />
                        </div>

                        <div class="modern-legend flex-grow-1" v-if="bookingChartData?.labels?.length">
                            <div class="legend-row d-flex align-items-center justify-content-between mb-2 p-2 rounded-3"
                                v-for="(label, index) in bookingChartData.labels" :key="index">
                                <div class="d-flex align-items-center gap-2">
                                    <span class="legend-indicator"
                                        :style="{ backgroundColor: bookingChartData.datasets[0].backgroundColor[index] }"></span>
                                    <span class="label-text">{{ label }}</span>
                                </div>
                                <span class="label-value">
                                    {{ calculatePercentage(bookingChartData.datasets[0].data[index],
                                    bookingChartData.datasets[0].data) }}%
                                </span>
                            </div>
                        </div>
                    </div>
                </div>
            </div>


            <!-- Recent Bookings & Reservations -->
            <div class="row g-4">
                <div class="col-12 col-lg-6">
                    <a :href="`/booking-index/${landlord_id}`" class="table-card-link">
                        <div class="modern-table-card">
                            <div class="card-top-accent bg-brand-blue"></div>
                            <div class="p-4">
                                <div class="d-flex justify-content-between align-items-center mb-4">
                                    <div>
                                        <span class="nav-label">LOGISTICS</span>
                                        <h5 class="fw-bold m-0"><i
                                                class="bi bi-calendar-check-fill me-2 text-brand-blue"></i>Recent
                                            Bookings</h5>
                                    </div>
                                    <span class="status-pill pill-blue">Live Update</span>
                                </div>

                                <div class="table-responsive">
                                    <table class="table table-custom">
                                        <thead>
                                            <tr>
                                                <th>Tenant Name</th>
                                                <th>Move-In</th>
                                                <th class="text-end">Room</th>
                                            </tr>
                                        </thead>
                                        <tbody>
                                            <tr v-for="booking in bookings.slice(0, 3)" :key="booking.bookingID">
                                                <td class="fw-bold text-dark">{{ booking.firstname }} {{
                                                    booking.lastname }}</td>
                                                <td><span class="text-muted small">{{ booking.moveInDate }}</span></td>
                                                <td class="text-end">
                                                    <span class="room-tag">{{ booking.room?.roomNumber ?? 'N/A'
                                                        }}</span>
                                                </td>
                                            </tr>
                                        </tbody>
                                    </table>
                                </div>
                            </div>
                        </div>
                    </a>
                </div>

                <div class="col-12 col-lg-6">
                    <a :href="`/reservation-index/${landlord_id}`" class="table-card-link">
                        <div class="modern-table-card">
                            <div class="card-top-accent bg-brand-orange"></div>
                            <div class="p-4">
                                <div class="d-flex justify-content-between align-items-center mb-4">
                                    <div>
                                        <span class="nav-label">PENDING</span>
                                        <h5 class="fw-bold m-0"><i
                                                class="bi bi-person-plus-fill me-2 text-brand-orange"></i>Recent
                                            Reservations</h5>
                                    </div>
                                </div>

                                <div class="table-responsive">
                                    <table class="table table-custom">
                                        <thead>
                                            <tr>
                                                <th>Tenant Name</th>
                                                <th class="text-end">Room Assigned</th>
                                            </tr>
                                        </thead>
                                        <tbody>
                                            <tr v-for="tenant in reservations.slice(0, 3)" :key="tenant.reservationID">
                                                <td class="fw-bold text-dark">{{ tenant.firstname }} {{ tenant.lastname
                                                    }}</td>
                                                <td class="text-end">
                                                    <span class="room-tag orange-tag">{{ tenant.room?.roomNumber ??
                                                        'N/A' }}</span>
                                                </td>
                                            </tr>
                                        </tbody>
                                    </table>
                                </div>
                            </div>
                        </div>
                    </a>
                </div>
            </div>


        </div>
    </div>
</template>
<script>
import axios from 'axios';
import LineChart from './chart/LineChart.vue';
import DoughnutChart from './chart/DoughnuChart.vue';
import Loader from '@/components/loader.vue';
import NotificationList from '@/components/notifications.vue';
import { Title } from 'chart.js';
import { get } from 'lodash';



export default
    {
        components: {
            LineChart,
            DoughnutChart,
            Loader,
            NotificationList

        },
        data() {
            return {
                receiverID: '',
                hasSubscribed: false,
                landlord_id: '',
                landlord: [],
                reservations: [],
                notifications: [],
                rooms: [],
                dorms: [],
                selectedDorm: null,
                bookings: [],
                newDate: '',
                today: '',
                totalTenants: 0,
                availableBeds: 0,
                totalRoomProfit: 0,
                chartData: null,
                chartOptions: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        y: {
                            beginAtZero: true
                        }
                    }
                },
                bookingChartData: null,
                bookingChartOptions: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: {
                            display: false
                        }
                    }
                }
            }
        },

        mounted() {
            const element = document.getElementById('dashboard');
            this.landlord_id = element.dataset.landlordId;
            this.receiverID = this.landlord_id;  // set receiverID here, early
            this.subscribeToNotifications();
            this.today = this.getTodayDate();
            this.newDate = this.today; // default value to today
            this.getLandlord();

        },
        methods:
        {
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
            async getLandlord() {
                try {
                    this.$refs.loader.loading = true;

                    const response = await axios.get(`/get/landlord/${this.landlord_id}`);

                    this.landlord = response.data.landlord;
                    await Promise.all([
                        this.getTotalTenants(),
                        this.getAvailableBeds(),
                        this.getReservationList(),
                        this.getBookingList(),
                        this.getRoomProfits(),
                        this.getGenderDistribution(),
                        this.getDormID()
                    ]);

                }
                catch (error) {
                    console.log(error);
                }
                finally {
                    this.$refs.loader.loading = false;

                }

            },
            selectDorm(dorm) {
                this.selectedDorm = dorm;
                // Call your function to fetch room profits for this dorm
                this.getRoomProfits(dorm.dormID);
                this.getGenderDistribution(dorm.dormID);
                this.getTotalTenants(dorm.dormID);
                this.getAvailableBeds(dorm.dormID);
                this.getReservationList(dorm.dormID);
                this.getBookingList(dorm.dormID);
            },
            async getTotalTenants(dorm_id = null) {
                try {
                    // Build params object
                    const params = { date: this.newDate };
                    if (dorm_id) {
                        params.dorm_id = dorm_id;
                    }

                    const response = await axios.get(`/get/total-tenants/${this.landlord_id}`, { params });
                    this.totalTenants = response.data.total_tenants;
                } catch (error) {
                    console.error('Failed to fetch total tenants:', error);
                    this.totalTenants = 0;
                }
            },

            async getAvailableBeds(dorm_id = null) {
                try {
                    // Build params object
                    const params = { date: this.newDate };
                    if (dorm_id) {
                        params.dorm_id = dorm_id;
                    }

                    const response = await axios.get(`/get/available-beds/${this.landlord_id}`, { params });
                    this.availableBeds = response.data.available_beds;
                } catch (error) {
                    console.error('Failed to fetch available beds:', error);
                }
            },
            async getReservationList(dorm_id = null) {
                try {
                    const response = await axios.get(`/get/reservation-list/${this.landlord_id}`, {
                        params: { date: this.newDate, dorm_id }
                    });
                    this.reservations = response.data.reservations;
                } catch (error) {
                    console.error('Failed to fetch tenant list:', error);
                }
            },
            async getBookingList(dorm_id = null) {
                try {
                    const response = await axios.get(`/get/booking-list/${this.landlord_id}`, {
                        params: { date: this.newDate, dorm_id }
                    });
                    this.bookings = response.data.bookings;
                } catch (error) {
                    console.error('Failed to fetch booking list:', error);
                }
            },
            async getDormID() {
                try {
                    const response = await axios.get(`/get/dorm-id/${this.landlord_id}`);
                    this.dorms = response.data.dorms;
                } catch (error) {
                    console.error('Failed to fetch dorm IDs:', error);
                }
            },

            async getRoomProfits(dorm_id = null) {
                try {
                    const params = { date: this.newDate };
                    if (dorm_id) params.dorm_id = dorm_id;

                    const response = await axios.get(`/get/room-profits/${this.landlord_id}`, { params });
                    const rooms = response.data.data;

                    this.chartData = {
                        labels: rooms.map(r => r.roomNumber), // ← roomNumber, dili roomName
                        datasets: [
                            {
                                label: 'Room Profits',
                                data: rooms.map(r => r.profit), // ← profit field
                                borderColor: '#2196f3',
                                tension: 0.4,
                                fill: false
                            }
                        ]
                    };

                    this.totalRoomProfit = response.data.total_profit;
                } catch (error) {
                    console.error('Error fetching room profits:', error);
                }
            },
            async getGenderDistribution(dorm_id = null) {
                try {
                    this.$refs.loader.loading = true;

                    const response = await axios.get(`/get/gender-distribution/${this.landlord_id}`, {
                        params: {
                            date: this.newDate,
                            dorm_id: dorm_id
                        }
                    });

                    const genders = Array.isArray(response.data?.data) ? response.data.data : [];

                    if (genders.length === 0) {
                        this.bookingChartData = {
                            labels: ["No Data"],
                            datasets: [
                                {
                                    label: "Gender Distribution",
                                    data: [1],
                                    backgroundColor: ["#e0e0e0"],
                                    hoverOffset: 4
                                }
                            ]
                        };
                        return;
                    }

                    const labels = genders.map(item => item.gender || "Unknown");
                    const data = genders.map(item => item.count || 0);

                    const backgroundColors = ["#2196f3", "#e91e63", "#ff9800", "#4caf50", "#9c27b0"];

                    this.bookingChartData = {
                        labels,
                        datasets: [
                            {
                                label: "Gender Distribution",
                                data,
                                backgroundColor: backgroundColors.slice(0, data.length),
                                hoverOffset: 4
                            }
                        ]
                    };

                } catch (error) {
                    console.error("❌ Failed to fetch gender distribution:", error);
                    this.bookingChartData = { labels: [], datasets: [] };
                } finally {
                    this.$refs.loader.loading = false;
                }
            },

            calculatePercentage(value, dataArray) {
                const total = dataArray.reduce((sum, val) => sum + val, 0);
                if (total === 0) return 0;
                return ((value / total) * 100).toFixed(1);
            },
            getTodayDate() {
                const today = new Date();
                const yyyy = today.getFullYear();
                const mm = String(today.getMonth() + 1).padStart(2, '0');
                const dd = String(today.getDate()).padStart(2, '0');
                return `${yyyy}-${mm}-${dd}`;
            },
            formatDate(dateString) {
                if (!dateString) return '';
                const date = new Date(dateString);
                return date.toLocaleString();
            }


        },
        watch: {
            newDate(newVal) {
                if (newVal) {
                    this.getLandlord();
                }
            },
            landlord_id(newVal) {
                if (newVal) {
                    this.subscribeToNotifications();
                }
            }
        }

    }

</script>
<style scoped src="../../../../css/landlord/dashboard.css"></style>