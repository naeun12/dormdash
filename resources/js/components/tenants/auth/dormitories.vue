<template>
    <Loader ref="loader" />
    <NotificationList ref="toastRef" />


    <div class="container-fluid py-4 bg-light min-vh-100 d-flex flex-column flex-lg-row gap-4">
        <!-- AI Question Sidebar -->
        <!-- Main Content -->
        <div class="flex-grow-1" style="overflow-x: hidden;">
            <div class="container-fluid mt-5 mb-2">
                <div class="header-content text-center animate__animated animate__fadeIn">
                    <span class="badge rounded-pill bg-blue-soft text-dash-blue mb-2 px-3 py-2">
                        <i class="bi bi-house-heart-fill me-1"></i> Verified Stays
                    </span>

                    <h2 class="display-6 fw-bold mb-3">
                        Find Your Ideal Dorm in <span class="text-dash-orange">Surigao City</span>
                    </h2>

                    <div class="d-flex justify-content-center">
                        <div class="title-divider"></div>
                    </div>

                    <p class="text-muted mt-3 lead-sm">
                        Browse through the best and most affordable dormitories across the city.
                    </p>
                </div>
            </div>


            <!-- Most Watched Dorms Horizontal Scroll -->
            <section class="most-watched-section mb-5 px-2">
                <div class="d-flex justify-content-between align-items-center mb-3 px-1">
                    <h5 class="fw-bold text-dash-blue mb-0">
                        <i class="bi bi-fire text-dash-orange me-2"></i>Most Watched Dormitories
                    </h5>

                    <div
                        class="swipe-indicator d-flex align-items-center gap-2 text-muted animate__animated animate__pulse animate__infinite">
                        <span class="x-small fw-medium italic">Swipe to view more</span>
                        <div class="swipe-icon">
                            <i class="bi bi-arrow-right x-small"></i>
                        </div>
                    </div>
                </div>

                <div class="horizontal-scroll-wrapper shadow-sm rounded-4 bg-white p-3">
                    <div class="d-flex flex-row gap-4 scroll-content">

                        <div class="card watched-dorm-card border-0 rounded-4 overflow-hidden shadow-sm"
                            v-for="(dorm, index) in mostwatchdorm" :key="index" @click="viewDormsDetails(dorm.dormID)">

                            <div class="position-relative card-image-wrap">
                                <img :src="dorm?.images?.mainImage || dorm?.mainImage || '/images/default-dorm.webp'"
                                    class="card-img-top" :alt="dorm.dormName" />

                                <div class="view-count-badge">
                                    <i class="bi bi-eye-fill me-1"></i> {{ dorm.views || 0 }}
                                </div>
                            </div>

                            <div class="card-body p-3 d-flex flex-column">
                                <h6 class="card-title fw-bold text-dark text-truncate mb-1">{{ dorm.dormName }}</h6>
                                <p class="card-text text-muted x-small mb-3 text-truncate">
                                    <i class="bi bi-geo-alt-fill text-dash-orange me-1"></i>
                                    {{ dorm.address || 'Surigao City' }}
                                </p>

                                <div class="mt-auto">
                                    <button class="btn btn-dash-blue-sm w-100 rounded-pill">
                                        View Details
                                    </button>
                                </div>
                            </div>
                        </div>

                    </div>
                </div>
            </section>


            <!-- Search Bar -->


            <!-- Filters -->
            <div class="py-4 px-2">
                <div
                    class="filter-header mb-4 p-4 rounded-4 shadow-sm bg-white border-start border-primary border-5 text-start">
                    <div class="d-flex align-items-center gap-3">
                        <div class="icon-box bg-primary-soft p-3 rounded-3">
                            <i class="bi bi-house-heart-fill fs-2 text-primary"></i>
                        </div>
                        <div>
                            <h4 class="fw-bold mb-0 text-dark">Find Dormitory Houses</h4>
                            <p class="text-muted small mb-0">Discover your next home in Surigao City with smart filters
                            </p>
                        </div>
                    </div>
                </div>

                <div class="search-wrapper mb-4">
                    <div
                        class="input-group shadow-sm rounded-pill overflow-hidden border-2 border-primary-soft px-3 bg-white">
                        <span class="input-group-text bg-transparent border-0">
                            <i class="bi bi-search text-primary fs-5"></i>
                        </span>
                        <input type="text" class="form-control border-0 shadow-none py-3"
                            placeholder="Search by location, street, or dorm name..." v-model="searchQuery"
                            @input="debouncedSearch" />
                    </div>
                </div>

                <div class="d-flex flex-wrap gap-2 mb-4 justify-content-center justify-content-md-start">
                    <button class="btn btn-filter-pill" :class="selectedButtons === 'All' ? 'active' : ''"
                        @click="btnAllFilter">
                        <i class="bi bi-grid-fill me-1"></i> All ({{ numberdorms }})
                    </button>

                    <button class="btn btn-filter-pill" :class="selectedButtons === 'Surigao' ? 'active' : ''"
                        @click="btnCityFilter('Surigao')">
                        Surigao City ({{ surigao_dorms || 0 }})
                    </button>
                </div>

                <div class="filter-grid-container p-3 rounded-4 bg-light border shadow-sm">
                    <p class="fw-bold text-muted small text-uppercase mb-3 px-2"><i
                            class="bi bi-sliders me-2"></i>Advanced Filters</p>

                    <div class="row g-3">
                        <div class="col-6 col-md-4 col-lg-3">
                            <div class="form-floating custom-floating">
                                <select class="form-select border-0 shadow-none" v-model="selectedPriceRange"
                                    @change="dropdownPriceRecommendations">
                                    <option value="all">All Prices</option>
                                    <option value="0-500">₱0 - ₱500</option>
                                    <option value="501-1000">₱501 - ₱1000</option>
                                    <option value="1001-1500">₱1001 - ₱1500</option>
                                    <option value="1501+">₱1501 & up</option>
                                </select>
                                <label><i class="bi bi-tag-fill me-1"></i>Price Range</label>
                            </div>
                        </div>

                        <div class="col-6 col-md-4 col-lg-3">
                            <div class="form-floating custom-floating">
                                <select class="form-select border-0 shadow-none" v-model="selectedOccupancyType"
                                    @change="dropdownGenderRecommdations">
                                    <option value="all">All Types</option>
                                    <option value="Male">Male Only</option>
                                    <option value="Female">Female Only</option>
                                    <option value="Mixed">Mixed/Co-ed</option>
                                </select>
                                <label><i class="bi bi-people-fill me-1"></i>Occupancy</label>
                            </div>
                        </div>

                        <div class="col-6 col-md-4 col-lg-3">
                            <div class="form-floating custom-floating">
                                <select class="form-select border-0 shadow-none" v-model="selectedAmenity"
                                    @change="dropdownAmenities">
                                    <option value="">Select Amenity</option>
                                    <option v-for="amenity in amenitiesList" :key="amenity.id" :value="amenity.id">
                                        {{ amenity.aminityName }}
                                    </option>
                                </select>
                                <label><i class="bi bi-wifi me-1"></i>Amenities</label>
                            </div>
                        </div>

                        <div class="col-6 col-md-4 col-lg-3">
                            <div class="form-floating custom-floating">
                                <select v-model="selectedAvailability" class="form-select border-0 shadow-none"
                                    @change="getAvailability">
                                    <option value="all">All Status</option>
                                    <option value="Available">Available Only</option>
                                    <option value="Not Available">Fully Booked</option>
                                </select>
                                <label><i class="bi bi-calendar-check me-1"></i>Availability</label>
                            </div>
                        </div>

                        <div class="col-6 col-md-4 col-lg-3">
                            <div class="form-floating custom-floating">
                                <select class="form-select border-0 shadow-none" v-model="selectedRating"
                                    @change="dropdownRate">
                                    <option value="all">Any Rating</option>
                                    <option value="5">★★★★★ (5 Stars)</option>
                                    <option value="4">★★★★☆ (4+ Stars)</option>
                                    <option value="3">★★★☆☆ (3+ Stars)</option>
                                </select>
                                <label><i class="bi bi-star-fill me-1"></i>Min. Rating</label>
                            </div>
                        </div>

                        <div class="col-6 col-md-4 col-lg-3">
                            <div class="form-floating custom-floating">
                                <select class="form-select border-0 shadow-none" v-model="sortBy"
                                    @change="sortDateDropDown">
                                    <option value="new-old">Newest Listing</option>
                                    <option value="old-new">Oldest Listing</option>
                                </select>
                                <label><i class="bi bi-sort-down me-1"></i>Sort By Date</label>
                            </div>
                        </div>
                    </div>
                </div>
            </div>


            <!-- Dorm Listings Grid -->
            <div class="container-fluid px-4">
                <div class="row g-4 mb-5">
                    <div class="col-12 col-sm-6 col-md-4 col-lg-3 animate__animated animate__fadeInUp"
                        v-for="(dorm, dormID) in dormitories" :key="dormID">

                        <div class="card h-100 shadow-sm border-0 rounded-4 overflow-hidden dorm-main-card">
                            <div class="position-relative">
                                <img :src="dorm?.images?.mainImage || dorm?.mainImage || '/images/default-dorm.webp'"
                                    class="card-img-top" :alt="dorm.dormName"
                                    style="height: 200px; object-fit: cover;" />

                                <div class="position-absolute top-0 start-0 m-2">
                                    <span class="badge rounded-pill shadow-sm px-3 py-2"
                                        :class="dorm.availability === 'Available' ? 'bg-success' : 'bg-secondary'">
                                        <i class="bi me-1"
                                            :class="dorm.availability === 'Available' ? 'bi-check-circle-fill' : 'bi-dash-circle-fill'"></i>
                                        {{ dorm.availability }}
                                    </span>
                                </div>

                                <div v-if="boolrate" class="position-absolute bottom-0 end-0 m-2">
                                    <div class="glass-rating px-2 py-1 rounded-3 text-white small">
                                        <i class="bi bi-star-fill text-warning me-1"></i> {{ dorm.rating_percentage }}%
                                    </div>
                                </div>
                            </div>

                            <div class="card-body d-flex flex-column p-4">
                                <div class="mb-3">
                                    <h5 class="fw-bold text-dark mb-1 text-truncate">{{ dorm.dormName }}</h5>
                                    <p class="text-muted small mb-0 text-truncate">
                                        <i class="bi bi-geo-alt-fill text-dash-orange me-1"></i> {{ dorm.address }}
                                    </p>
                                </div>

                                <div
                                    class="d-flex justify-content-between align-items-center mb-4 bg-light p-2 rounded-3">
                                    <div class="text-center flex-fill border-end">
                                        <small class="d-block text-muted x-small">TYPE</small>
                                        <span class="fw-bold small text-dash-blue">{{ dorm.occupancyType }}</span>
                                    </div>
                                    <div class="text-center flex-fill">
                                        <small class="d-block text-muted x-small">STARTING AT</small>
                                        <span class="fw-bold small text-success">₱{{ dorm.price }}</span>
                                    </div>
                                </div>

                                <div class="mt-auto">
                                    <button class="btn btn-dash-blue rounded-pill w-100 py-2 shadow-sm fw-bold"
                                        @click="viewDormsDetails(dorm.dormID)">
                                        Explore Details <i class="bi bi-arrow-right ms-1"></i>
                                    </button>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <nav aria-label="Page navigation" class="mt-5">
                    <ul class="pagination justify-content-center gap-2">
                        <li class="page-item" :class="{ disabled: currentPage === 1 }">
                            <a class="page-link border-0 shadow-sm rounded-circle p-pagination" href="#"
                                @click.prevent="goToPage(currentPage - 1)">
                                <i class="bi bi-chevron-left"></i>
                            </a>
                        </li>

                        <li class="page-item" v-for="page in totalPages" :key="page">
                            <a class="page-link border-0 shadow-sm rounded-4 px-3 py-2 fw-bold"
                                :class="currentPage === page ? 'bg-dash-blue text-white' : 'bg-white text-dark'"
                                href="#" @click.prevent="goToPage(page)">
                                {{ page }}
                            </a>
                        </li>

                        <li class="page-item" :class="{ disabled: currentPage === totalPages }">
                            <a class="page-link border-0 shadow-sm rounded-circle p-pagination" href="#"
                                @click.prevent="goToPage(currentPage + 1)">
                                <i class="bi bi-chevron-right"></i>
                            </a>
                        </li>
                    </ul>
                </nav>
            </div>


        </div>


    </div>
</template>


<script>
import axios from 'axios';
import _ from 'lodash';
import debounce from 'lodash/debounce';
import Loader from '@/components/loader.vue';
import NotificationList from '@/components/notifications.vue';


export default {
    components: {
        Loader,
        NotificationList,


    },
    name: 'ProductGrid',
    data() {
        return {
            dormitories: [],
            filteredDorms: [],
            searchQuery: '',
            selectedPriceRange: '',
            selectedOccupancyType: '',
            recommendations: [],
            recommendloading: false,
            isGenderBased: false,
            recommend: '',
            dorms: [],
            dormReccomend: [],
            question: '',
            chatresponse: '',
            mostwatchdorm: [],
            tenant_id: '',
            notifications: [],
            receiverID: '',
            //new filters
            lapulapu_dorms: [],
            rooms: [],
            numberdorms: [],
            currentFilter: null, // e.g. 'all', 'city', 'price', etc.
            currentFilterParams: {},
            mandaue_dorms: [],
            selectedAmenity: '',
            selectedRating: '',
            sortBy: '',
            selectedAvailability: '',
            currentPage: 1,
            totalPages: 1,
            itemsPerPage: 12,
            selectedButtons: '',
            amenitiesList: [],
            amenitieslength: false,
            boolrate: false,
        };
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
        //filter
        async btnAllFilter() {
            this.selectedAmenity = '';
            this.selectedRating = '';
            this.sortBy = '';
            this.selectedAvailability = '';
            this.selectedOccupancyType = '';
            this.selectedPriceRange = '';
            this.rooms = [];
            this.boolrate = false;
            this.amenitieslength = false;
            this.dormListingfetch();
        },
        async btnCityFilter(city, page = 1) {
            this.selectedButtons = city;

            this.currentFilter = 'city';
            this.currentFilterParams = { city };
            this.selectedAmenity = '';
            this.selectedRating = '';
            this.sortBy = '';
            this.selectedAvailability = '';
            this.selectedOccupancyType = '';
            this.selectedPriceRange = '';
            this.rooms = [];
            this.boolrate = false;
            this.amenitieslength = false;
            this.$refs.loader.loading = true;
            const response = await axios.get('/select-cities', {
                params: { city: this.selectedButtons, page: page, per_page: this.itemsPerPage }

            });
            this.currentPage = response.data.dorms.current_page;
            this.totalPages = response.data.dorms.last_page;
            this.dormitories = response.data.dorms.data;
            this.$refs.loader.loading = false;
        },


        async dropdownPriceRecommendations(page = 1) {
            this.selectedButtons = '';
            this.selectedAmenity = '';
            this.currentFilter = 'price';
            this.selectedRating = '';
            this.sortBy = '';
            this.selectedAvailability = '';
            this.selectedOccupancyType = '';
            this.$refs.loader.loading = true;
            this.rooms = [];
            this.boolrate = false;
            this.amenitieslength = false;


            const [min, max] = this.getPriceRange(this.selectedPriceRange);


            try {
                const response = await fetch('/pricerecommendations', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'Accept': 'application/json',
                        'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').getAttribute('content'),
                    },
                    body: JSON.stringify({
                        min_price: Number(min),
                        max_price: Number(max),
                        page: page,
                        per_page: this.itemsPerPage
                    })
                });


                const result = await response.json();


                if (result.status === 'success') {
                    this.dormitories = result.data.data;       // current page dorms
                    this.currentPage = result.data.current_page;
                    this.totalPages = result.data.last_page;
                    this.rooms = result.data.data;
                    console.log(this.rooms);
                }


            } catch (error) {
                console.error(error);
            } finally {
                this.$refs.loader.loading = false;
            }
        },


        getPriceRange(range) {
            switch (range) {
                case '0-500': return [0, 500];
                case '501-1000': return [501, 1000];
                case '1001-1500': return [1001, 1500];
                case '1501+': return [1501, 999999];
                default: return [0, 999999]; // fallback
            }
        },
        async dropdownGenderRecommdations(page = 1) {
            try {
                this.$refs.loader.loading = true;
                this.selectedButtons = '';
                this.currentFilter = 'gender';

                this.selectedPriceRange = '';
                this.selectedAmenity = '';
                this.selectedRating = '';
                this.sortBy = '';
                this.selectedAvailability = '';
                this.rooms = [];
                this.boolrate = false;
                this.amenitieslength = false;
                this.$refs.loader.loading = true;


                const response = await fetch('gender-recommendations', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'Accept': 'application/json',
                        'X-CSRF-TOKEN': document.querySelector('meta[name="csrf-token"]').getAttribute('content'),
                    },
                    body: JSON.stringify({
                        occupancy_type: this.selectedOccupancyType.toLowerCase(),
                        page: page,
                        per_page: this.itemsPerPage
                    }),

                });
                const result = await response.json();


                if (result.status === 'success') {
                    this.dormitories = result.recommendations.data; // only items
                    this.currentPage = result.recommendations.current_page;
                    this.totalPages = result.recommendations.last_page;
                }


            }
            catch (error) {
                this.$refs.loader.loading = false;
            }
            finally {
                this.$refs.loader.loading = false;


            }
        },
        async dropdownAmenities(page = 1) {
            try {
                this.$refs.loader.loading = true;
                this.selectedButtons = '';
                this.selectedRating = '';
                this.currentFilter = 'amenity';
                this.sortBy = '';
                this.selectedAvailability = '';
                this.selectedOccupancyType = '';
                this.selectedPriceRange = '';
                this.rooms = [];
                this.boolrate = false;
                this.$refs.loader.loading = true;
                const response = await axios.post('/get/amenities', {
                    amenities: Array.isArray(this.selectedAmenity) ? this.selectedAmenity : [this.selectedAmenity],
                    params: { page: page, per_page: this.itemsPerPage }
                });


                // Update dorms list and count
                this.dormitories = response.data.dorms.data;
                this.currentPage = response.data.dorms.current_page;
                this.totalPages = response.data.dorms.last_page;
                this.amenitieslength = true;


            } catch (error) {
                console.error("Error fetching amenities:", error);
            } finally {
                this.$refs.loader.loading = false;
            }
        },
        async dropdownRate(page = 1) {
            try {
                this.$refs.loader.loading = true;
                this.selectedButtons = '';
                this.selectedAmenity = '';
                this.sortBy = '';
                this.selectedAvailability = '';
                this.selectedOccupancyType = '';
                this.selectedPriceRange = '';
                this.rooms = [];
                this.amenitieslength = false;
                this.$refs.loader.loading = true;
                const response = await axios.get('/get/rate', {
                    params: { rating: this.selectedRating, page: page, per_page: this.itemsPerPage }
                });
                this.boolrate = true;
                this.dormitories = response.data.dorms.data;
                this.currentPage = response.data.dorms.current_page;
                this.totalPages = response.data.dorms.last_page;
            } catch (error) {
                console.error("Error fetching by rate:", error);
            } finally {
                this.$refs.loader.loading = false;
            }
        },
        async sortDateDropDown(page = 1) {
            try {
                this.$refs.loader.loading = true;
                this.currentFilter = 'sort';
                this.selectedButtons = '';
                this.selectedAmenity = '';
                this.selectedRating = '';
                this.selectedAvailability = '';
                this.selectedOccupancyType = '';
                this.selectedPriceRange = '';
                this.rooms = [];
                this.amenitieslength = false;
                this.boolrate = false;
                this.$refs.loader.loading = true;
                const response = await axios.get('/get/sortByDate', {
                    params: { sortBy: this.sortBy, page: page, per_page: this.itemsPerPage }
                });


                this.dormitories = response.data.dorms.data;
                this.currentPage = response.data.dorms.current_page;
                this.totalPages = response.data.dorms.last_page;


            } catch (error) {
                console.error("Error sorting by date:", error);
            } finally {
                this.$refs.loader.loading = false;
            }
        },
        async getAvailability(page = 1) {
            try {
                this.$refs.loader.loading = true;
                this.selectedButtons = '';
                this.currentFilter = 'availability';
                this.selectedAmenity = '';
                this.selectedRating = '';
                this.sortBy = '';
                this.selectedOccupancyType = '';
                this.selectedPriceRange = '';
                this.rooms = [];
                this.amenitieslength = false;
                this.boolrate = false;
                this.$refs.loader.loading = true;
                const response = await axios.get('/get/availability', {
                    params: { availability: this.selectedAvailability, page: page, per_page: this.itemsPerPage },


                });


                this.dormitories = response.data.dorms.data;
                this.currentPage = response.data.dorms.current_page;
                this.totalPages = response.data.dorms.last_page;


            } catch (error) {
                console.error("Error fetching by availability:", error);
            } finally {
                this.$refs.loader.loading = false;
            }
        },


        async dormListingfetch(page = 1) {
            try {
                this.selectedButtons = 'All';
                this.$refs.loader.loading = true;
                const response = await axios.get('/list-dorms', {
                    params: { page: page, per_page: this.itemsPerPage },
                });
                this.currentPage = response.data.dorms.current_page;
                this.totalPages = response.data.dorms.last_page;
                this.dormitories = response.data.dorms.data;
                this.numberdorms = response.data.total_dorms;
                this.lapulapu_dorms = response.data.lapulapu_dorms;
                this.mandaue_dorms = response.data.mandaue_dorms;
                this.rooms = [];
                this.$refs.loader.loading = false;




            } catch (error) {
                console.error("Error fetching dorms:", error);
                this.$refs.loader.loading = false;


            }
        },
        viewDormsDetails(dormitoryId) {
            this.tenant_id = window.tenant_id;
            window.location.href = `/room-details/${dormitoryId}/${this.tenant_id}`;
        },



        async goToPage(page) {
            if (page < 1 || page > this.totalPages) return;


            // Always call the right filter function based on currentFilter
            switch (this.currentFilter) {
                case 'city':
                    await this.btnCityFilter(this.currentFilterParams.city, page);
                    break;
                case 'price':
                    await this.dropdownPriceRecommendations(page);
                    break;
                case 'gender':
                    await this.dropdownGenderRecommdations(page);
                    break;
                case 'amenity':
                    await this.dropdownAmenities(page);
                    break;
                case 'rating':
                    await this.dropdownRate(page);
                    break;
                case 'sort':
                    await this.sortDateDropDown(page);
                    break;
                case 'availability':
                    await this.getAvailability(page);
                    break;
                case 'search':
                    await this.searchLocations(page);
                    break;
                default:
                    await this.dormListingfetch(page);
            }


            this.currentPage = page; // Update currentPage for UI
        },
        async searchLocations(page = 1) {
            try {
                this.$refs.loader.loading = true;
                this.selectedButtons = '';
                this.currentFilter = 'search';
                this.selectedAmenity = '';
                this.selectedRating = '';
                this.sortBy = '';
                this.selectedAvailability = '';
                this.selectedOccupancyType = '';
                this.selectedPriceRange = '';
                this.rooms = [];
                this.amenitieslength = false;
                this.boolrate = false;


                this.$refs.loader.loading = true;
                if (!this.searchQuery.trim()) {


                    await this.dormListingfetch();  // re-fetch all dorms
                    return;
                }


                const csrfToken = document.querySelector('meta[name="csrf-token"]').getAttribute('content');
                const response = await fetch('/search-locations', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'Accept': 'application/json',
                        'X-CSRF-TOKEN': csrfToken,
                    },
                    body: JSON.stringify({
                        location: this.searchQuery,
                        page: page,
                        per_page: this.itemsPerPage
                    }),
                });


                const result = await response.json();
                if (result.status === "success") {
                    this.dormitories = result.recommendations.data;
                    this.currentPage = result.recommendations.current_page; // must access from recommendations
                    this.totalPages = result.recommendations.last_page;    // also from recommendations
                    this.$refs.loader.loading = false;




                } else {
                    console.error('Server responded with error:', result.message);
                    this.dormitories = [];
                    this.$refs.loader.loading = false;




                }
            } catch (err) {
                console.error('Search failed:', err);
                this.filteredDorms = [];
                this.$refs.loader.loading = false;


            }
        },
        async aiQuestion() {
            try {
                this.$refs.loader.loading = true;


                // Send the user question to Laravel AI route
                const response = await axios.post('ai/question/reccomendations', { question: this.question });


                // AI text response
                this.chatresponse = response.data.message || 'Walay tubag gikan sa AI';


                // AI recommendations (dorms or rooms)
                const recs = response.data.recommendations || [];


                this.dormReccomend = (recs || []).map(d => ({
                    dormID: d.dormID || 0,
                    dormName: d.dormName || 'Unnamed Dorm',
                    address: d.address || 'No address provided',
                    occupancyType: d.occupancyType || 'Mixed',
                    amenities: d.amenities || '',
                    rules: d.rules || [],
                    rooms: (d.rooms || []).map(r => ({
                        roomNumber: r.roomNumber || 'N/A',
                        type: r.type || r.roomType || 'Standard',
                        price: r.price || 'Contact landlord',
                        availability: r.availability || 'Unknown',
                        features: r.features
                            ? (Array.isArray(r.features)
                                ? r.features
                                : r.features.split(',').map(f => f.trim()))
                            : []
                    }))
                }));


                console.log('AI Recommendations:', this.dormReccomend);
                console.log('AI Response:', this.chatresponse);


            } catch (error) {
                console.error('Error sending AI question:', error);
                this.chatresponse = 'Naa’y error sa pagkuha og recommendations.';
                this.dormReccomend = [];
            } finally {
                this.$refs.loader.loading = false;
            }
        },









        async mostWatchDorm() {
            try {
                this.tenant_id = window.tenant_id;  // siguro naa ni sa global js
                const response = await axios.get(`/most/watched/dorm/${this.tenant_id}`);
                this.mostwatchdorm = response.data.dorm;
                // I-update imong UI with this.mostwatchdorm
                console.log('Most watched dorm:', this.mostwatchdorm);
            } catch (error) {
                console.error('Error fetching most watched dorm:', error);
            }
        }




    },
    created() {
        this.debouncedSearch = debounce(this.searchLocations, 500);
    },
    mounted() {
        this.dormListingfetch();
        this.mostWatchDorm();
        this.tenant_id = window.tenant_id;
        this.subscribeToNotifications();
        axios.get('/fetch-amenities').then(response => {
            this.amenitiesList = response.data.amenities;
        });
    },
};
</script>
<style scoped src="../../../../css/tenant/dormitory.css"></style>