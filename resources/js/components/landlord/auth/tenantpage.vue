<template>
    <NotificationList ref="toastRef" />
    <Loader ref="loader" />
    <Toastcomponents ref="toast" />
    <div class="p-4 mt-4 mb-3">
        <!-- Search Bar -->
        <div class="p-4 bg-white rounded-4 shadow-sm border mb-4">
            <div class="row mb-4">
                <div class="col-12">
                    <div
                        class="search-container d-flex align-items-center ps-3 rounded-pill border bg-light shadow-sm transition-all">
                        <i class="bi bi-search text-primary fs-5"></i>
                        <input type="text" class="form-control border-0 bg-transparent py-2 shadow-none ms-2"
                            placeholder="Search tenant's name..." v-model="searchTerm" @input="searchTenants" />
                    </div>
                </div>
            </div>

            <div class="d-flex flex-column flex-lg-row gap-3 align-items-end justify-content-between">

                <div class="d-flex flex-wrap gap-2 flex-grow-1 w-100">
                    <div class="filter-box flex-grow-1">
                        <label class="small fw-bold text-muted mb-1 ms-1 text-uppercase"
                            style="font-size: 0.7rem;">Property</label>
                        <div class="input-group border rounded-3 bg-white shadow-sm overflow-hidden px-2">
                            <span class="input-group-text bg-transparent border-0"><i
                                    class="bi bi-building text-secondary"></i></span>
                            <select class="form-select border-0 shadow-none py-2" v-model="selectedDormId"
                                @change="filterDorms">
                                <option value="" disabled>Select Dorm</option>
                                <option value="all">All Dorms</option>
                                <option v-for="dorm in dorms" :key="dorm.dormID" :value="dorm.dormID">
                                    {{ dorm.dormName }}
                                </option>
                            </select>
                        </div>
                    </div>

                    <div class="filter-box" style="min-width: 180px;">
                        <label class="small fw-bold text-muted mb-1 ms-1 text-uppercase"
                            style="font-size: 0.7rem;">Filter Date</label>
                        <div class="input-group border rounded-3 bg-white shadow-sm overflow-hidden px-2">
                            <span class="input-group-text bg-transparent border-0"><i
                                    class="bi bi-calendar-event text-secondary"></i></span>
                            <input type="date" class="form-control border-0 shadow-none py-2" v-model="startDate"
                                @change="filterByDate" />
                        </div>
                    </div>
                </div>

                <div class="d-flex flex-wrap gap-2 justify-content-lg-end w-100 w-lg-auto">
                    <button class="btn btn-action btn-move-in shadow-sm fw-bold rounded-3 px-3 py-2"
                        @click="viewMoveInTenants">
                        <i class="bi bi-house-check-fill me-2"></i>Confirmed
                    </button>

                    <button class="btn btn-action btn-report shadow-sm fw-bold rounded-3 px-3 py-2"
                        @click="downloadReport">
                        <i class="bi bi-file-earmark-arrow-down-fill me-2"></i>Active Report
                    </button>

                    <button class="btn btn-action btn-payment shadow-sm fw-bold rounded-3 px-3 py-2"
                        @click="extensionReport">
                        <i class="bi bi-credit-card-fill me-2"></i>Extension Report
                    </button>
                </div>

            </div>
        </div>


        <!-- Confirmed Move-in Modal -->
        <div v-if="clickConfirmedMoveInTenantModal" class="modal d-block" tabindex="-1"
            style="background: rgba(0, 30, 60, 0.5); backdrop-filter: blur(4px);">
            <div class="modal-dialog modal-lg modal-dialog-centered">
                <div class="modal-content shadow-lg border-0 rounded-4 overflow-hidden">
                    <div class="modal-header border-0 p-4 text-white" style="background-color: #003C87;">
                        <h5 class="modal-title fw-bold d-flex align-items-center">
                            <div class="p-2 bg-white bg-opacity-25 rounded-3 me-3">
                                <i class="bi bi-house-check text-white fs-4"></i>
                            </div>
                            Confirmed Move-in Tenants
                        </h5>
                        <button type="button" class="btn-close btn-close-white"
                            @click="clickConfirmedMoveInTenantModal = false"></button>
                    </div>

                    <div class="modal-body p-4 bg-light">
                        <div
                            class="search-wrapper mb-4 shadow-sm rounded-pill overflow-hidden d-flex align-items-center bg-white px-3 border transition-all">
                            <i class="bi bi-search text-muted me-2"></i>
                            <input type="text" class="form-control border-0 shadow-none py-2"
                                placeholder="Search by tenant name..." v-model="searchMoveIn"
                                @input="searchMoveInTenant" />
                        </div>

                        <div v-if="moveInTenants.length > 0" class="tenant-grid">
                            <div v-for="moveInTenant in moveInTenants" :key="moveInTenant.approvedID"
                                class="card border-0 shadow-sm rounded-4 mb-3 overflow-hidden tenant-card">
                                <div class="row g-0">
                                    <div class="col-md-4 position-relative">
                                        <img :src="moveInTenant.pictureID"
                                            class="h-100 w-100 object-fit-cover shadow-inner"
                                            style="min-height: 180px;" />
                                        <span
                                            class="badge bg-success position-absolute top-0 start-0 m-3 rounded-pill shadow-sm">
                                            {{ moveInTenant.source_type }}
                                        </span>
                                    </div>

                                    <div class="col-md-8 p-3">
                                        <div class="d-flex justify-content-between align-items-start mb-2">
                                            <h5 class="fw-bold text-dark mb-0">
                                                {{ moveInTenant.firstname }} {{ moveInTenant.lastname }}
                                            </h5>
                                            <small
                                                class="text-primary fw-bold bg-primary bg-opacity-10 px-2 py-1 rounded">
                                                Room {{ moveInTenant.room?.roomNumber }}
                                            </small>
                                        </div>

                                        <div class="row g-2 mb-3">
                                            <div class="col-6">
                                                <small class="text-muted d-block"><i class="bi bi-envelope me-1"></i>
                                                    Email</small>
                                                <span class="small fw-medium text-truncate d-block">{{
                                                    moveInTenant.contactEmail }}</span>
                                            </div>
                                            <div class="col-6">
                                                <small class="text-muted d-block"><i class="bi bi-telephone me-1"></i>
                                                    Phone</small>
                                                <span class="small fw-medium d-block">{{ moveInTenant.contactNumber
                                                    }}</span>
                                            </div>
                                            <div class="col-6">
                                                <small class="text-muted d-block"><i class="bi bi-building me-1"></i>
                                                    Dorm</small>
                                                <span class="small fw-medium d-block text-truncate">{{
                                                    moveInTenant.room?.dorm?.dormName }}</span>
                                            </div>
                                            <div class="col-6">
                                                <small class="text-muted d-block"><i
                                                        class="bi bi-calendar-check me-1"></i> Move-in Date</small>
                                                <span class="small fw-bold text-success d-block">{{
                                                    formatDate(moveInTenant.moveInDate) }}</span>
                                            </div>
                                        </div>

                                        <button
                                            class="btn btn-success w-100 rounded-3 fw-bold py-2 shadow-sm transition-all"
                                            @click="moveInTenantBTN(moveInTenant.approvedID)">
                                            <i class="bi bi-box-arrow-in-right me-2"></i> Confirm Arrival / Move-in
                                        </button>
                                    </div>
                                </div>
                            </div>

                            <nav class="mt-4 d-flex justify-content-center">
                                <ul class="pagination pagination-custom gap-2">
                                    <li class="page-item shadow-sm" :class="{ disabled: currentPage === 1 }">
                                        <a class="page-link rounded-pill border-0 px-3 transition-all" href="#"
                                            @click.prevent="viewMoveInTenants(currentPage - 1)">
                                            <i class="bi bi-chevron-left small"></i>
                                        </a>
                                    </li>

                                    <li v-for="page in totalPages" :key="page" class="page-item shadow-sm"
                                        :class="{ active: currentPage === page }">
                                        <a class="page-link rounded-pill border-0 px-3 fw-bold transition-all" href="#"
                                            @click.prevent="viewMoveInTenants(page)">
                                            {{ page }}
                                        </a>
                                    </li>

                                    <li class="page-item shadow-sm" :class="{ disabled: currentPage === totalPages }">
                                        <a class="page-link rounded-pill border-0 px-3 transition-all" href="#"
                                            @click.prevent="viewMoveInTenants(currentPage + 1)">
                                            <i class="bi bi-chevron-right small"></i>
                                        </a>
                                    </li>
                                </ul>
                            </nav>
                        </div>

                        <div v-else class="text-center py-5">
                            <div class="mb-3 opacity-25">
                                <i class="bi bi-person-x" style="font-size: 4rem;"></i>
                            </div>
                            <h6 class="fw-bold text-secondary">No move-in tenants found</h6>
                            <p class="text-muted small">Try searching for another name or check back later.</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>


        <!-- No Tenants Message -->
        <div v-if="!tenants.length"
            class="empty-state shadow-sm border rounded-4 d-flex flex-column justify-content-center align-items-center bg-white mx-auto mt-4"
            style="height: 300px; max-width: 600px;">

            <div class="icon-circle mb-3 d-flex align-items-center justify-content-center">
                <i class="bi bi-people text-secondary opacity-50" style="font-size: 3.5rem;"></i>
            </div>

            <h5 class="fw-bold text-dark mb-1">No Tenants Found</h5>
            <p class="text-muted small px-5 text-center mb-4">
                We couldn't find any tenants matching your current search or filter criteria.
                Try adjusting your keywords.
            </p>

            <button v-if="searchTerm" @click="searchTerm = ''; searchTenants()"
                class="btn btn-sm px-4 rounded-pill fw-bold shadow-sm transition-all"
                style="background-color: #003C87; color: white;">
                Clear Search
            </button>
        </div>


        <!-- TENANT LIST - NOW RESPONSIVE -->
        <div v-else class="container-fluid bg-white rounded shadow-sm mt-4 px-1 px-md-3"
            style=" max-height: 700px; overflow-y: auto;">
            <!-- Desktop Header -->
            <div
                class="d-none d-md-flex fw-bold text-white text-center py-3 rounded-top shadow-sm align-items-center header-gradient">
                <div class="col-1 border-end border-white border-opacity-10 small-caps">ID</div>
                <div class="col-2 text-start ps-4 small-caps">
                    <i class="bi bi-person me-2"></i>Tenant Name
                </div>
                <div class="col-3 text-start ps-4 small-caps">
                    <i class="bi bi-envelope me-2"></i>Email Address
                </div>
                <div class="col-2 text-start ps-4 small-caps">
                    <i class="bi bi-building me-2"></i>Dormitory
                </div>
                <div class="col-1 border-start border-end border-white border-opacity-10 small-caps">Room</div>
                <div class="col-2 small-caps">Status</div>
                <div class="col-1 small-caps">Actions</div>
            </div>


            <div v-for="tenant in tenants" :key="tenant.approvedID" class="tenant-row-wrapper">
                <div class="d-none d-md-flex align-items-center py-3 px-2 border-bottom hover-bg transition-all">
                    <div class="col-1 text-center fw-bold text-muted small">#{{ tenant.approvedID }}</div>

                    <div class="col-2 text-start ps-3">
                        <div class="d-flex align-items-center">
                            <div class="avatar-sm me-2 bg-light text-primary rounded-circle d-flex align-items-center justify-content-center"
                                style="width: 32px; height: 32px; border: 1px solid #eee;">
                                <i class="bi bi-person-fill small"></i>
                            </div>
                            <span class="fw-bold text-dark text-truncate">{{ tenant.firstname }} {{ tenant.lastname
                                }}</span>
                        </div>
                    </div>

                    <div class="col-3 text-start ps-3">
                        <span class="text-secondary small d-block text-truncate">
                            <i class="bi bi-envelope me-1 text-primary"></i>{{ tenant.contactEmail }}
                        </span>
                    </div>

                    <div class="col-2 text-start ps-3">
                        <span class="badge bg-light text-dark border fw-medium px-2 py-1">
                            <i class="bi bi-building me-1 opacity-50"></i> {{ tenant.room?.dorm?.dormName ?? 'N/A' }}
                        </span>
                    </div>

                    <div class="col-1 text-center">
                        <span class="fw-bold" style="color: #FC7D07;">Rm {{ tenant.room?.roomNumber ?? 'N/A' }}</span>
                    </div>

                    <div class="col-2 text-center">
                        <span class="status-badge px-3 py-1 fw-bold rounded-pill shadow-xs" :class="{
                            'status-active': tenant.status === 'active',
                            'status-moved-out': tenant.status === 'moved_out',
                            'status-pending': tenant.status === 'pending_moveout',
                            'status-transfer': tenant.status === 'transferring',
                        }">
                            <i class="bi bi-dot fs-5"></i>{{ tenant.status?.replace('_', ' ').toUpperCase() }}
                        </span>
                    </div>

                    <div class="col-1 d-flex justify-content-center gap-2">
                        <button class="btn btn-icon btn-view shadow-sm"
                            @click="displaytenantInformation(tenant.approvedID)" title="View Details">
                            <i class="bi bi-eye-fill"></i>
                        </button>
                        <button class="btn btn-icon btn-delete shadow-sm" @click="softDelete(tenant)"
                            title="Delete Tenant">
                            <i class="bi bi-trash3-fill"></i>
                        </button>
                    </div>
                </div>

                <div class="d-md-none card border-0 rounded-4 mb-3 shadow-sm overflow-hidden">
                    <div
                        class="card-header border-0 bg-light d-flex justify-content-between align-items-center py-2 px-3">
                        <span class="fw-bold text-muted small">#{{ tenant.approvedID }}</span>
                        <span class="status-badge px-2 py-1 fw-bold rounded-pill" style="font-size: 0.65rem;" :class="{
                            'status-active': tenant.status === 'active',
                            'status-moved-out': tenant.status === 'moved_out',
                            'status-pending': tenant.status === 'pending_moveout',
                            'status-transfer': tenant.status === 'transferring',
                        }">
                            {{ tenant.status?.replace('_', ' ').toUpperCase() }}
                        </span>
                    </div>
                    <div class="card-body p-3">
                        <div class="d-flex align-items-center mb-3">
                            <div class="p-2 rounded-circle me-3" style="background-color: #fff3e6;">
                                <i class="bi bi-person-circle fs-3" style="color: #FC7D07;"></i>
                            </div>
                            <div>
                                <h6 class="mb-0 fw-bold text-dark">{{ tenant.firstname }} {{ tenant.lastname }}</h6>
                                <small class="text-secondary">{{ tenant.contactEmail }}</small>
                            </div>
                        </div>

                        <div class="row g-2 mb-3 bg-light rounded-3 p-2 mx-0">
                            <div class="col-6">
                                <small class="text-muted d-block uppercase-label">Dorm</small>
                                <span class="small fw-bold text-dark">{{ tenant.room?.dorm?.dormName ?? 'N/A' }}</span>
                            </div>
                            <div class="col-6 border-start ps-3">
                                <small class="text-muted d-block uppercase-label">Room</small>
                                <span class="small fw-bold" style="color: #003C87;">{{ tenant.room?.roomNumber ?? 'N/A'
                                    }}</span>
                            </div>
                        </div>

                        <div class="d-flex gap-2">
                            <button class="btn btn-sm flex-grow-1 fw-bold rounded-3 py-2 shadow-sm"
                                style="background-color: #003C87; color: white;"
                                @click="displaytenantInformation(tenant.approvedID)">
                                <i class="bi bi-eye-fill me-1"></i> View
                            </button>
                            <button class="btn btn-sm rounded-3 py-2 px-3 shadow-sm"
                                style="background-color: #fff0f0; color: #e03131; border: none;"
                                @click="softDelete(tenant)">
                                <i class="bi bi-trash3-fill"></i>
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        </div>


 


        <!-- Tenant Info Modal -->
        <div v-if="VisibleTenantModal" class="modal fade show d-block"
            style="background: rgba(0, 0, 0, 0.6); backdrop-filter: blur(4px);" tabindex="-1">
            <div class="modal-dialog modal-xl modal-dialog-centered">
                <div class="modal-content shadow-lg rounded-4 border-0 overflow-hidden">
                    <div class="modal-header border-bottom-0 py-3 px-4" style="background-color: #003C87;">
                        <h5 class="modal-title text-white fw-bold d-flex align-items-center">
                            <i class="bi bi-person-badge-fill me-2 text-warning"></i>
                            Tenants Information
                        </h5>
                        <button type="button" class="btn-close btn-close-white"
                            @click="VisibleTenantModal = false"></button>
                    </div>

                    <div class="modal-body px-4 px-md-5 py-4">
                        <div class="text-center mb-5">
                            <div class="position-relative d-inline-block">
                                <img :src="selectedtenant.pictureID"
                                    class="rounded-circle border border-4 border-white shadow"
                                    style="width: 130px; height: 130px; object-fit: cover;" />
                                <div class="position-absolute bottom-0 end-0 bg-white rounded-circle p-1 shadow-sm">
                                    <i class="bi bi-patch-check-fill text-primary fs-4"></i>
                                </div>
                            </div>
                            <h4 class="mt-3 fw-bold text-dark mb-1">{{ selectedtenant.firstname }} {{
                                selectedtenant.lastname }}</h4>
                            <span class="status-badge px-4 py-2 fw-bold rounded-pill shadow-xs mt-2 d-inline-block"
                                :class="{
                                    'status-active': tenantStatus === 'active',
                                    'status-moved-out': tenantStatus === 'moved_out',
                                    'status-pending': tenantStatus === 'pending_moveout',
                                    'status-transfer': tenantStatus === 'transferring',
                                }">
                                {{ tenantStatus?.replace('_', ' ').toUpperCase() }}
                            </span>
                        </div>

                        <div class="row g-4">
                            <div class="col-md-6 border-end-md">
                                <div class="section-title mb-3">
                                    <h6 class="text-uppercase fw-bold text-muted small px-1"
                                        style="letter-spacing: 1px;">Personal Details</h6>
                                </div>
                                <div class="mb-3 custom-input-group">
                                    <label class="form-label small fw-bold text-muted"><i
                                            class="bi bi-person me-2 color-orange"></i>First Name</label>
                                    <input type="text" class="form-control rounded-3"
                                        v-model="selectedtenant.firstname" />
                                    <p v-if="errors.firstname" class="text-danger tiny mt-1">{{ errors.firstname[0] }}
                                    </p>
                                </div>
                                <div class="row g-3">
                                    <div class="col-6 mb-3">
                                        <label class="form-label small fw-bold text-muted"><i
                                                class="bi bi-calendar-event me-2 color-orange"></i>Age</label>
                                        <input type="number" class="form-control rounded-3"
                                            v-model="selectedtenant.age" />
                                    </div>
                                    <div class="col-6 mb-3">
                                        <label class="form-label small fw-bold text-muted"><i
                                                class="bi bi-gender-ambiguous me-2 color-orange"></i>Gender</label>
                                        <select class="form-select rounded-3" v-model="selectedtenant.gender">
                                            <option value="Male">Male</option>
                                            <option value="Female">Female</option>
                                        </select>
                                    </div>
                                </div>
                                <div class="mb-3">
                                    <label class="form-label small fw-bold text-muted"><i
                                            class="bi bi-envelope me-2 color-orange"></i>Email Address</label>
                                    <input type="email" class="form-control rounded-3"
                                        v-model="selectedtenant.contactEmail" />
                                </div>
                                <div class="mb-3">
                                    <label class="form-label small fw-bold text-muted"><i
                                            class="bi bi-telephone me-2 color-orange"></i>Contact Number</label>
                                    <input type="text" class="form-control rounded-3"
                                        v-model="selectedtenant.contactNumber" />
                                </div>
                            </div>

                            <div class="col-md-6">
                                <div class="section-title mb-3">
                                    <h6 class="text-uppercase fw-bold text-muted small px-1"
                                        style="letter-spacing: 1px;">Lease & Room Info</h6>
                                </div>
                                <div class="mb-3 p-3 rounded-3 bg-light border border-dashed">
                                    <div class="d-flex justify-content-between mb-2">
                                        <span class="text-muted small fw-bold">Dormitory:</span>
                                        <span class="fw-bold text-dark">{{ selectedtenant.room?.dorm?.dormName ?? 'N/A'
                                            }}</span>
                                    </div>
                                    <div class="d-flex justify-content-between mb-2">
                                        <span class="text-muted small fw-bold">Room & Type:</span>
                                        <span class="fw-bold" style="color: #003C87;">{{
                                            selectedtenant?.room?.roomNumber }} ({{ selectedtenant?.room?.roomType
                                            }})</span>
                                    </div>
                                    <div class="d-flex justify-content-between">
                                        <span class="text-muted small fw-bold">Monthly Rent:</span>
                                        <span class="fw-bold text-success">₱ {{ selectedtenant.room?.price }}</span>
                                    </div>
                                </div>

                                <div class="mb-3">
                                    <label class="form-label small fw-bold text-muted"><i
                                            class="bi bi-building me-2 color-orange"></i>Change Dorm (Transfer)</label>
                                    <select v-model="selectedDormId" class="form-select rounded-3 border-orange-focus">
                                        <option disabled value="">Select Dorm</option>
                                        <option v-for="dorm in dorms" :key="dorm.dormID" :value="dorm.dormID">{{
                                            dorm.dormName }}</option>
                                    </select>
                                    <button v-if="tenantStatus === 'transferring'"
                                        class="btn btn-warning btn-sm w-100 mt-2 fw-bold text-dark rounded-3"
                                        @click="onDormChange()">
                                        <i class="bi bi-arrow-left-right me-1"></i> Confirm Transfer
                                    </button>
                                </div>

                                <div class="row g-3">
                                    <div class="col-6 mb-3">
                                        <label class="form-label small fw-bold text-muted"><i
                                                class="bi bi-calendar-check me-2 color-orange"></i>Move-In</label>
                                        <input type="text" class="form-control bg-light"
                                            :value="formatDate(selectedtenant.moveInDate)" readonly />
                                    </div>
                                    <div class="col-6 mb-3">
                                        <label class="form-label small fw-bold text-muted"><i
                                                class="bi bi-calendar-x me-2 color-orange"></i>Move-Out</label>
                                        <input type="text" class="form-control bg-light"
                                            :value="formatDate(selectedtenant.moveOutDate)" readonly />
                                    </div>
                                </div>

                                <div class="mb-3">
                                    <label class="form-label small fw-bold text-muted"><i
                                            class="bi bi-shield-check me-2 color-orange"></i>Update Status</label>
                                    <select v-model="selectedtenant.status"
                                        class="form-select rounded-3 shadow-sm border-0 bg-white">
                                        <option value="active">Active</option>
                                        <option value="moved_out">Moved Out</option>
                                        <option value="pending_moveout">Pending Move Out</option>
                                        <option value="transferring">Transferring</option>
                                    </select>
                                </div>
                            </div>
                        </div>
                        <div class="mt-4">
                            <StatusAlert :status="tenantStatus" role="Landlord" />
                        </div>
                    </div>

                    <div class="modal-footer border-top-0 bg-light p-4 gap-3">
                        <button class="btn btn-outline-dark fw-bold px-4 rounded-3"
                            @click="messagePage(selectedtenant)">
                            <i class="bi bi-chat-dots me-2"></i> Message
                        </button>

                        <div class="ms-md-auto d-flex gap-2 w-100 w-md-auto">
                            <button v-if="selectedtenant.notifyRent === 0" @click="notifyTenant(selectedtenant)"
                                class="btn btn-orange-action fw-bold px-4 rounded-3 flex-grow-1">
                                <i class="bi bi-bell-fill me-2"></i> Notify Rent Extension
                            </button>
                            <button v-else-if="selectedtenant.notifyRent === 1"
                                @click="btnViewTenantPaymentModal(selectedtenant)"
                                class="btn btn-primary-action fw-bold px-4 rounded-3 flex-grow-1">
                                <i class="bi bi-receipt me-2"></i> View Payments
                            </button>

                            <button class="btn btn-success fw-bold px-4 rounded-3 flex-grow-1"
                                @click="updateTenantInformation">
                                <i class="bi bi-check-circle me-2"></i> Save Changes
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        </div>


        <!-- Room Modal -->
        <div v-if="selectedRoomModal" class="modal fade show d-block"
            style="background: rgba(0, 0, 0, 0.6); backdrop-filter: blur(4px);" tabindex="-1">
            <div class="modal-dialog modal-lg modal-dialog-centered">
                <div class="modal-content shadow-lg rounded-4 border-0 overflow-hidden">
                    <div class="modal-header text-white border-0 py-3 px-4" style="background-color: #003C87;">
                        <h5 class="modal-title fw-bold d-flex align-items-center">
                            <i class="bi bi-door-open-fill me-2 text-warning"></i>
                            Room Details
                        </h5>
                        <button type="button" class="btn-close btn-close-white"
                            @click="selectedRoomModal = false"></button>
                    </div>

                    <div class="modal-body bg-light p-4">
                        <div class="mb-4 p-3 bg-white rounded-3 shadow-sm border-start border-4"
                            style="border-color: #FC7D07 !important;">
                            <label class="form-label fw-bold text-muted small text-uppercase">
                                <i class="bi bi-search me-2"></i>Select Room to View
                            </label>
                            <select class="form-select border-0 bg-light shadow-none" @change="onchangeRoomDetails"
                                v-model="selectedRoomId">
                                <option disabled value="">Select Room</option>
                                <option v-for="room in rooms" :key="room.roomID" :value="room.roomID">
                                    Room {{ room.roomNumber }} — {{ room.roomType }} ({{ room.availability }})
                                </option>
                            </select>
                        </div>

                        <div class="card shadow-sm border-0 rounded-3">
                            <div class="card-header bg-white py-3 border-bottom border-light">
                                <h6 class="mb-0 fw-bold d-flex align-items-center" style="color: #003C87;">
                                    <i class="bi bi-info-circle-fill me-2"></i> Specifications
                                </h6>
                            </div>
                            <div class="card-body p-4">
                                <div class="row g-4">
                                    <div class="col-md-6">
                                        <label class="form-label small fw-bold text-muted">Room ID</label>
                                        <div class="input-group">
                                            <span class="input-group-text bg-light border-0"><i
                                                    class="bi bi-hash"></i></span>
                                            <input type="text" class="form-control bg-light border-0"
                                                v-model="roomsdetails.roomID" readonly />
                                        </div>
                                    </div>
                                    <div class="col-md-6">
                                        <label class="form-label small fw-bold text-muted">Dorm ID</label>
                                        <div class="input-group">
                                            <span class="input-group-text bg-light border-0"><i
                                                    class="bi bi-building"></i></span>
                                            <input type="text" class="form-control bg-light border-0"
                                                v-model="roomsdetails.fkdormID" readonly />
                                        </div>
                                    </div>
                                    <div class="col-md-6">
                                        <label class="form-label small fw-bold text-muted">Room Number</label>
                                        <input type="text" class="form-control border-0 bg-light fw-bold"
                                            v-model="roomsdetails.roomNumber" readonly />
                                    </div>
                                    <div class="col-md-6">
                                        <label class="form-label small fw-bold text-muted">Room Type</label>
                                        <input type="text" class="form-control border-0 bg-light fw-bold text-primary"
                                            v-model="roomsdetails.roomType" readonly />
                                    </div>
                                    <div class="col-md-6">
                                        <label class="form-label small fw-bold text-muted">Monthly Rent</label>
                                        <div class="input-group">
                                            <span class="input-group-text border-0"
                                                style="background-color: #fff3e6; color: #FC7D07;">₱</span>
                                            <input type="text" class="form-control border-0"
                                                style="background-color: #fff3e6; font-weight: 700;"
                                                v-model="roomsdetails.price" />
                                        </div>
                                    </div>
                                    <div class="col-md-6">
                                        <label class="form-label small fw-bold text-muted">Status</label>
                                        <div class="pt-1">
                                            <span v-if="roomsdetails.availability === 'Available'"
                                                class="badge rounded-pill px-3 py-2 bg-success bg-opacity-10 text-success border border-success border-opacity-25 w-100">
                                                <i class="bi bi-check-circle-fill me-1"></i> Available
                                            </span>
                                            <span v-else
                                                class="badge rounded-pill px-3 py-2 bg-danger bg-opacity-10 text-danger border border-danger border-opacity-25 w-100">
                                                <i class="bi bi-x-circle-fill me-1"></i> Occupied
                                            </span>
                                        </div>
                                    </div>

                                    <div class="col-12">
                                        <hr class="opacity-0 my-1">
                                    </div>

                                    <div class="col-md-6">
                                        <label class="form-label small fw-bold text-muted">Furnishing</label>
                                        <input type="text" class="form-control border-0 bg-light"
                                            v-model="roomsdetails.furnishing_status" readonly />
                                    </div>
                                    <div class="col-md-6">
                                        <label class="form-label small fw-bold text-muted">Listing Type</label>
                                        <input type="text" class="form-control border-0 bg-light"
                                            v-model="roomsdetails.listingType" readonly />
                                    </div>
                                    <div class="col-md-4">
                                        <label class="form-label small fw-bold text-muted">Area (Sqm)</label>
                                        <input type="text" class="form-control border-0 bg-light"
                                            v-model="roomsdetails.areaSqm" readonly />
                                    </div>
                                    <div class="col-md-8">
                                        <label class="form-label small fw-bold text-muted">Gender Preference</label>
                                        <input type="text" class="form-control border-0 bg-light"
                                            v-model="roomsdetails.genderPreference" readonly />
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>

                    <div class="modal-footer border-0 bg-white p-3 px-4" v-if="isButtonChangingRoom">
                        <button type="button" class="btn fw-bold px-4 rounded-3 text-white transition-all shadow-sm"
                            style="background-color: #FC7D07;" @click="updateRoom(roomsdetails.roomID)">
                            <i class="bi bi-check2-circle me-2"></i> Confirm Room Change
                        </button>
                    </div>
                </div>
            </div>
        </div>


        <!-- Tenant Payment Modal -->
        <div v-if="viewtenantpaymentModal" class="modal d-block" tabindex="-1"
            style="background-color: rgba(15, 23, 42, 0.6); backdrop-filter: blur(8px);">
            <div class="modal-dialog modal-lg modal-dialog-centered">
                <div class="modal-content shadow-2xl border-0 rounded-5 overflow-hidden">

                    <div class="modal-header border-0 p-4 text-white" style="background: #0d6efd;">
                        <div class="d-flex align-items-center">
                            <div class="bg-white bg-opacity-20 p-2 rounded-3 me-3">
                                <i class="bi bi-shield-check fs-4"></i>
                            </div>
                            <div>
                                <h5 class="modal-title fw-bold mb-0 text-white">Payment Verification</h5>
                                <small class="opacity-75">Review and process tenant extension</small>
                            </div>
                        </div>
                        <button type="button" class="btn-close btn-close-white"
                            @click="viewtenantpaymentModal = false"></button>
                    </div>

                    <div class="modal-body p-4 bg-light">

                        <div v-if="tenantpayment.paymentOption === 'online'" class="fade-in">
                            <div class="row g-4">
                                <div class="col-md-5">
                                    <div class="card border-0 rounded-4 shadow-sm overflow-hidden h-100">
                                        <div class="bg-dark d-flex align-items-center justify-content-center p-2"
                                            style="min-height: 350px;">
                                            <img :src="tenantpayment?.payments?.[0]?.paymentImage || 'default-placeholder.png'"
                                                alt="Payment Proof" class="img-fluid rounded-3"
                                                style="max-height: 400px; object-fit: contain; cursor: zoom-in;" />
                                        </div>
                                        <div class="card-footer bg-white border-0 text-center py-3">
                                            <span class="badge rounded-pill px-3 py-2"
                                                style="background: #fff7ed; color: #fd7e14;">
                                                <i class="bi bi-image me-1"></i> GCash Receipt
                                            </span>
                                        </div>
                                    </div>
                                </div>

                                <div class="col-md-7">
                                    <div class="card border-0 rounded-4 shadow-sm mb-4">
                                        <div class="card-body p-4">
                                            <h6 class="text-uppercase fw-bold text-muted small mb-4 letter-spacing-1">
                                                Transaction Details</h6>

                                            <div class="d-flex align-items-center mb-4 p-3 rounded-4 bg-light">
                                                <div class="avatar-circle me-3">
                                                    {{ tenantpayment.firstname[0] }}{{ tenantpayment.lastname[0] }}
                                                </div>
                                                <div>
                                                    <p class="text-muted small mb-0">Tenant Name</p>
                                                    <h6 class="fw-bold mb-0 text-dark">{{ tenantpayment.firstname }} {{
                                                        tenantpayment.lastname }}</h6>
                                                </div>
                                            </div>

                                            <div class="row g-3 mb-4">
                                                <div class="col-6">
                                                    <div class="p-3 rounded-4 border">
                                                        <p class="text-muted small mb-1">Amount Paid</p>
                                                        <h5 class="fw-black mb-0" style="color: #0d6efd;">₱{{
                                                            tenantpayment?.payments?.[0]?.amount || '0' }}</h5>
                                                    </div>
                                                </div>
                                                <div class="col-6">
                                                    <div class="p-3 rounded-4 border">
                                                        <p class="text-muted small mb-1">Status</p>
                                                        <span
                                                            class="badge bg-warning bg-opacity-10 text-warning px-3 py-2">Pending</span>
                                                    </div>
                                                </div>
                                            </div>

                                            <div class="mb-4">
                                                <label class="form-label fw-bold text-dark mb-2">New Move-out
                                                    Date</label>
                                                <div class="input-group">
                                                    <span
                                                        class="input-group-text bg-white border-end-0 rounded-start-4">
                                                        <i class="bi bi-calendar-event text-primary"></i>
                                                    </span>
                                                    <input type="date" v-model="extensionDate"
                                                        :min="tenantpayment.moveOutDate.split(' ')[0]"
                                                        class="form-control border-start-0 rounded-end-4 py-2" />
                                                </div>
                                            </div>

                                            <div class="d-flex gap-2">
                                                <button class="btn btn-success flex-fill py-3 rounded-4 fw-bold"
                                                    @click="updateOnlinePayment(tenantpayment?.payments?.[0], tenantpayment.approvedID, 'approved')">
                                                    Approve
                                                </button>
                                                <button class="btn btn-outline-danger flex-fill py-3 rounded-4 fw-bold"
                                                    @click="updateOnlinePayment(tenantpayment?.payments?.[0], tenantpayment.approvedID, 'rejected')">
                                                    Reject
                                                </button>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div v-if="tenantpayment.paymentOption === 'onsite'" class="fade-in p-2">
                            <div class="card border-0 rounded-5 shadow-sm mb-4 overflow-hidden">
                                <div class="card-body p-5 text-center">
                                    <div class="mx-auto bg-success bg-opacity-10 rounded-circle d-flex align-items-center justify-content-center mb-4"
                                        style="width: 80px; height: 80px;">
                                        <i class="bi bi-cash-coin fs-1 text-success"></i>
                                    </div>
                                    <h4 class="fw-black text-dark mb-2">On-site Payment</h4>
                                    <p class="text-muted mb-4">Confirm receipt of cash payment from the tenant below.
                                    </p>

                                    <div class="row justify-content-center mb-4">
                                        <div class="col-md-8">
                                            <div class="p-4 bg-light rounded-4 border-dashed text-start">
                                                <div class="d-flex justify-content-between mb-3 border-bottom pb-2">
                                                    <span class="text-muted">Tenant</span>
                                                    <span class="fw-bold">{{ tenantpayment.firstname }} {{
                                                        tenantpayment.lastname }}</span>
                                                </div>
                                                <div class="d-flex justify-content-between mb-3 border-bottom pb-2">
                                                    <span class="text-muted">Monthly Rate</span>
                                                    <span class="fw-bold text-primary">₱{{ tenantpayment?.room?.price
                                                        }}</span>
                                                </div>
                                                <div class="d-flex justify-content-between">
                                                    <span class="text-muted">Current Move-out</span>
                                                    <span class="fw-bold text-danger">{{
                                                        formatDate(tenantpayment.moveOutDate) }}</span>
                                                </div>
                                            </div>
                                        </div>
                                    </div>

                                    <div class="col-md-6 mx-auto mb-4 text-start">
                                        <label class="form-label fw-bold small text-muted">EXTEND MOVE-OUT DATE
                                            TO:</label>
                                        <input type="date" v-model="extensionDate"
                                            :min="tenantpayment.moveOutDate.split(' ')[0]"
                                            class="form-control rounded-4 py-3 shadow-sm" />
                                    </div>

                                    <button class="btn btn-success px-5 py-3 rounded-pill fw-bold shadow-lg"
                                        @click="updateExtension(tenantpayment.approvedID)">
                                        Confirm & Update Extension <i class="bi bi-arrow-right ms-2"></i>
                                    </button>
                                </div>
                            </div>
                        </div>

                    </div>
                </div>
            </div>
        </div>
    </div>
    <Modalconfirmation ref="modal" />
</template>

<script>
import axios from 'axios';
import Toastcomponents from '@/components/Toastcomponents.vue';
import Loader from '@/components/loader.vue';
import Modalconfirmation from '@/components/modalconfirmation.vue';
import { debounce } from 'lodash';
import NotificationList from '@/components/notifications.vue';
import StatusAlert from '@/components/approveTenantsAlert.vue';
import { update } from 'lodash';
export default {
    components: {
        Toastcomponents,
        Loader,
        Modalconfirmation,
        NotificationList,
        StatusAlert

    },
    data() {
        return {
            searchTerm: '',
            selectedDormId: '',
            selectedRoomNumber: '',
            selectedRoomId: '',
            currentRoomID: '',
            currentTenantID: '',
            isButtonChangingRoom: false,
            isButtonChangingDorm: false,
            tenantStatus: '',
            selectedRoomModal: false,
            dorms: [],
            rooms: [],
            errors: {},
            roomsdetails: [],
            selectedroomNumber: '',
            tenants: [],
            selectedtenant: {
                firstname: "",
                lastname: "",
                gender: "",
                age: "",
                contactEmail: "",
                contactNumber: "",
                status: ""
            }, VisibleTenantModal: false,
            notifications: [],
            hasSubscribed: false,
            receiverID: '',
            landlord_id: '',
            searchTimeout: null,
            clickConfirmedMoveInTenantModal: false,
            moveInTenants: [],
            currentPage: 1,
            totalPages: 1,
            searchMoveInTenantTimeout: null,
            searchMoveIn: '',
            viewtenantpaymentModal: false,
            tenantpayment: [],
            extensionDate: '',
            startDate: this.getTodayDate(), // set default to today
        }
    },
    watch:
    {
        'selectedtenant.room.dorm.dormID'(newVal) {
            if (newVal) {
                this.selectedDormId = newVal;
            }
        },
        searchTerm(newValue) {
            clearTimeout(this.searchTimeout);
            this.searchTimeout = setTimeout(() => {
                this.searchTenants(newValue);
            }, 500); // 3 seconds delay
        }
    },
    methods: {
        subscribeToNotifications() {
            if (this.hasSubscribed) return;
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

        async getDormId() {
            const res = await axios.get(`/get-dorms/${this.landlord_id}`);
            this.dorms = res.data.dorms;
            console.log(this.dorms);
        },
        async getTenantList() {
            try {
                this.$refs.loader.loading = true;

                const response = await axios.get('/tenants-list', { withCredentials: true });
                this.tenants = response.data.tenant;
            } catch (error) {
                console.error("Error fetching tenant list:", error.response?.data || error.message);
            } finally {
                this.$refs.loader.loading = false;

            }
        },

        async displaytenantInformation(tenantID) {
            this.errors = {};

            try {
                this.$refs.loader.loading = true;

                const response = await axios.get(`/tenants-view/${tenantID}`);
                if (response.data.status === 'success') {
                    this.selectedtenant = response.data.tenant;
                    this.currentRoomID = response.data.tenant?.room.roomID;
                    this.currentTenantID = response.data.tenant.approvedID;
                    this.tenantStatus = response.data.tenant.status;
                    this.VisibleTenantModal = true;
                }

            } catch (error) {
                console.error('Error fetching reservation details:', error);
            }
            finally {
                this.$refs.loader.loading = false;

            }
        },
        async updateTenantInformation() {
            try {

                const confirmed = await this.$refs.modal.show({
                    title: 'Update Tenant Information',
                    message: `Are you sure you want to update this tenant's information?`,
                    functionName: 'Update Tenant Info (Optional)'
                });
                if (!confirmed) {
                    this.rules.pop();
                    return;
                }
                this.$refs.loader.loading = true;

                const response = await axios.put(`/tenants-update/${this.selectedtenant.approvedID}`, this.selectedtenant);
                if (response.data.status === 'success') {
                    this.VisibleTenantModal = false;
                    this.$refs.toast.showToast(response.data.message, 'success');
                    this.errors = {};
                    this.getTenantList();

                }
            } catch (error) {
                if (error.response && error.response.status === 422) {
                    // Laravel validation errors
                    this.errors = error.response.data.errors;
                } else {
                    this.$refs.toast.showToast("Something went wrong.", 'danger');
                }
                console.error('Error updating tenant information:', error);
            } finally {
                this.$refs.loader.loading = false;
            }
        },

        onDormChange() {
            this.$refs.loader.loading = true;
            this.isButtonChangingRoom = false;
            axios.get('/get/rooms', {
                params: { dormID: this.selectedDormId }
            })
                .then(response => {
                    if (response.data.status === 'success') {
                        this.rooms = response.data.data;
                        this.selectedRoomModal = true;
                        this.$refs.loader.loading = false;
                        this.roomsdetails = [];
                        this.selectedRoomId = '';


                    }
                })
                .catch(error => {
                    console.error('Error fetching rooms:', error);
                    this.$refs.loader.loading = false;

                });
        },
        async onchangeRoomDetails() {
            this.$refs.loader.loading = true;
            this.isButtonChangingRoom = true;
            axios.get('/get/roomsdetails', {
                params: { roomID: this.selectedRoomId }
            })
                .then(response => {
                    if (response.data.status === 'success') {
                        this.roomsdetails = response.data.data[0] || {};
                        this.$refs.loader.loading = false;

                    }
                })
                .catch(error => {
                    this.$refs.loader.loading = false;

                    console.error('Error fetching rooms:', error);
                });
        },
        async updateRoom(newRoomID) {
            this.errors = {}; // clear previous errors

            try {
                const confirmed = await this.$refs.modal.show({
                    title: 'Change Tenant Room',
                    message: `Are you sure you want to move this tenant to another room?`,
                    functionName: 'Change Tenant Room (Optional)'
                });
                if (!confirmed) {
                    this.rules.pop();
                    return;
                }
                this.$refs.loader.loading = true;
                const res = await axios.put(`/tenant/room/update/${newRoomID}`, {
                    current_room_id: this.currentRoomID,
                    tenant_id: this.currentTenantID
                });
                if (res.data.status === 'success') {
                    this.selectedRoomModal = false;
                    this.VisibleTenantModal = false;
                    this.$refs.toast.showToast(res.data.message, 'success');
                    this.getTenantList();
                }
                else if (res.data.status === 'error') {
                    this.$refs.toast.showToast(res.data.message, 'danger');
                    this.selectedRoomModal = false;
                    this.VisibleTenantModal = false;
                }


            } catch (error) {

            }
            finally {
                this.$refs.loader.loading = false;

            }
        },
        async searchTenants() {
            try {
                this.$refs.loader.loading = true;
                this.selectedDormId = '';
                if (this.searchTerm === '') {
                    this.getTenantList();
                    return; // stop execution kung walay gi‑type


                }

                const res = await axios.get(`/tenants/search`, {
                    params: { query: this.searchTerm }
                });
                this.tenants = res.data.tenants;

            } catch (error) {
                console.error("Error searching tenants:", error);
            }
            finally {
                this.$refs.loader.loading = false;

            }
        },
        async filterDorms() {
            try {
                this.$refs.loader.loading = true;
                // Kung "all" ang gipili, kuhaon tanan tenants
                if (this.selectedDormId === 'all') {
                    this.getTenantList();
                    return;
                }

                // Fetch tenants filtered by dorm ID
                const res = await axios.get(`/tenants/filter-by-dorm`, {
                    params: { dorm_id: this.selectedDormId }
                });

                if (res.data.status === 'success') {
                    this.tenants = res.data.tenants;
                    this.searchTerm = '';

                } else {
                    this.tenants = [];
                }
            } catch (error) {
                console.error("Error filtering tenants:", error);
            } finally {
                this.$refs.loader.loading = false;
            }
        },
        messagePage(selectedReservation) {
            const tenantID = selectedReservation.fktenantID;

            if (!tenantID) {
                alert('No tenant assigned to this reservation.');
                return;
            }

            const url = `/api/select/landlord/conversations/${this.landlord_id}?tenant_id=${tenantID}`;
            window.location.href = url;
        },
        async notifyTenant(selectedtenant) {
            try {
                const confirmed = await this.$refs.modal.show({
                    title: 'Confirm Rent Extension',
                    message: `Are you sure you want to notify this tenant about their rent extension?`,
                    functionName: 'Send Notification'
                });

                if (!confirmed) {
                    if (this.rules && this.rules.length > 0) {
                        this.rules.pop();
                    }
                    return;
                }
                this.$refs.loader.loading = true;
                const response = await axios.post('/notify/tenant', {
                    landlordID: this.landlord_id,
                    approveID: selectedtenant.approvedID
                });

                if (response.data.status === 'success') {
                    this.$refs.toast.showToast(response.data.message, 'success');
                    this.VisibleTenantModal = false;

                } else {
                    this.$refs.toast.showToast('Failed to send notification.', 'error');
                    this.VisibleTenantModal = false;

                }
            }
            catch (error) {
                console.error(error);
                this.$refs.toast.showToast('Something went wrong while sending the notification.', 'error');
            }
            finally {
                this.$refs.loader.loading = false;
            }
        },
        async viewMoveInTenants(page = 1) {
            try {
                this.$refs.loader.loading = true;

                const response = await axios.get('/add/movein/tenant', {
                    params: { page }
                });

                if (response.data.status === 'success') {
                    this.clickConfirmedMoveInTenantModal = true;
                    this.moveInTenants = response.data.moveInTenant.data; // tenants
                    this.currentPage = response.data.moveInTenant.current_page;
                    this.totalPages = response.data.moveInTenant.last_page;
                }
            } catch (error) {
                console.error(error);
            } finally {
                this.$refs.loader.loading = false;
            }
        },
        async searchMoveInTenant() {
            this.$refs.loader.loading = true;

            try {
                const response = await axios.get('/search/movein/tenant', {
                    params: {
                        searchMoveIn: this.searchMoveIn
                    }
                });

                if (response.data.status === 'success') {
                    this.moveInTenants = response.data.tenants.data;
                } else {
                    this.moveInTenants = [];
                }
            } catch (error) {
                console.error("Error searching move-in tenants:", error);
            } finally {
                this.$refs.loader.loading = false;
            }
        },
        async moveInTenantBTN(approvedID) {
            try {
                const confirmed = await this.$refs.modal.show({
                    title: 'Confirm Move-In',
                    message: `Are you sure you want to move in this tenant?`,
                    functionName: 'Move In Tenant'
                });
                if (confirmed === false) {
                    if (this.rules && this.rules.length > 0) {
                        this.rules.pop();
                    }
                    return;
                }
                this.$refs.loader.loading = true;
                const response = await axios.get('/movein/tenant', {
                    params: { approvedID }
                });
                if (response.data.status === 'success') {
                    this.$refs.toast.showToast(response.data.message, 'success');
                    this.clickConfirmedMoveInTenantModal = false;
                    this.getTenantList();
                } else if (response.data.status === 'error') {
                    this.$refs.toast.showToast(response.data.message, 'danger');
                    this.clickConfirmedMoveInTenantModal = false;
                } else {
                    this.$refs.toast.showToast('Failed to move in tenant.', 'danger');
                }

            } catch (error) {
                console.error("Error moving in tenant:", error);
            } finally {
                this.$refs.loader.loading = false;
            }
        },
        async btnViewTenantPaymentModal(selectedtenant) {
            try {
                const res = await axios.get(`/view/tenant/payment/${selectedtenant.approvedID}`);
                this.$refs.loader.loading = true;
                this.tenantpayment = res.data.tenant;
                this.extensionDate = this.tenantpayment.moveOutDate;
                this.viewtenantpaymentModal = true;
            } catch (error) {
                console.error("Error viewing tenant payment:", error);
            }
            finally {
                this.$refs.loader.loading = false;
            }

        },
        async updateExtension(approveID) {
            try {
                const confirmed = await this.$refs.modal.show({
                    title: 'Confirm Extension Update',
                    message: `Are you sure you want to update the extension date to ${this.extensionDate.split(' ')[0]}?`,
                    functionName: 'Update Extension'
                });

                if (confirmed) {
                    this.$refs.loader.loading = true;

                    const response = await axios.post(`/update/extension/${approveID}`, {
                        extensionDate: this.extensionDate
                    });

                    if (response.data.status === 'success') {
                        this.$refs.toast.showToast(response.data.message, 'success');
                        this.viewtenantpaymentModal = false;
                        this.VisibleTenantModal = false;

                        this.getTenantList();
                    } else {
                        this.$refs.toast.showToast('Failed to update extension.', 'error');
                    }
                } else {
                    if (this.rules && this.rules.length > 0) {
                        this.rules.pop();
                    }
                    return;
                }
            }
            catch (error) {
                console.error("Error updating extension:", error);
            }
            finally {
                this.$refs.loader.loading = false;
            }
        },
        async updateOnlinePayment(tenantPayment, approvedID, status) {
            try {
                const confirmed = await this.$refs.modal.show({
                    title: 'Confirm Payment {}'.replace('{}', status.charAt(0).toUpperCase() + status.slice(1)),
                    message: `Are you sure you want to ${status} this payment?`,
                    functionName: `${status.charAt(0).toUpperCase() + status.slice(1)} Payment`
                });

                if (confirmed) {
                    this.$refs.loader.loading = true;

                    const response = await axios.post(`/update/tenant/payment/extension/${approvedID}`, {
                        status: status,
                        paymentID: tenantPayment,
                        extensionDate: this.extensionDate

                    });

                    if (response.data.status === 'success') {
                        this.$refs.toast.showToast(response.data.message, 'success');
                        this.viewtenantpaymentModal = false;
                        this.VisibleTenantModal = false;

                        this.getTenantList();
                    } else {
                        this.$refs.toast.showToast('Failed to approve payment.', 'error');
                    }
                } else {
                    if (this.rules && this.rules.length > 0) {
                        this.rules.pop();
                    }
                    return;
                }
            }
            catch (error) {
                console.error("Error approving payment:", error);
            }
            finally {
                this.$refs.loader.loading = false;
            }
        },
        async softDelete(tenant) {
            const confirmed = await this.$refs.modal.show({
                title: 'Delete Tenant',
                message: `Are you sure you want to delete this tenant? This action cannot be undone.`,
                functionName: 'Delete Tenant'
            });
            try {
                if (confirmed) {
                    this.$refs.loader.loading = true;

                    const response = await axios.post(`/soft-delete/tenant/${tenant.approvedID}`);

                    if (response.data.status === 'success') {
                        this.$refs.toast.showToast(response.data.message, 'success');
                        this.getTenantList();

                    } else {
                        this.$refs.toast.showToast('Failed to delete tenant.', 'error');
                    }
                } else {
                    if (this.rules && this.rules.length > 0) {
                        this.rules.pop();
                    }
                    return;
                }
            }
            catch (error) {

            }
            finally {
                this.$refs.loader.loading = false;
            }

        },
        formatDate(date) {
            if (!date) return "N/A";
            return new Date(date).toLocaleDateString('en-US', {
                year: 'numeric',
                month: 'long',
                day: 'numeric'
            });
        },
        getTodayDate() {
            const today = new Date();
            const yyyy = today.getFullYear();
            const mm = String(today.getMonth() + 1).padStart(2, '0');
            const dd = String(today.getDate()).padStart(2, '0');
            return `${yyyy}-${mm}-${dd}`;
        },
        nextDay(date) {
            if (!date) return null;
            let d = new Date(date);
            d.setDate(d.getDate() + 1);
            return d.toISOString().split('T')[0]; // format YYYY-MM-DD
        },
        downloadReport() {
            const dateParam = this.startDate || this.getTodayDate(); // fallback to today
            window.open(`/report/active-tenants?date=${dateParam}`, '_blank');
        },
        extensionReport() {
            const dateParam = this.startDate || this.getTodayDate();
            window.open(`/report/extension-payments?date=${dateParam}`, '_blank');
        },


    },

    mounted() {
        this.landlord_id = document.getElementById('tenantpage').dataset.landlordId;

        this.subscribeToNotifications();
        this.getTenantList();
        if (this.selectedtenant?.room?.dorm?.dormID) {
            this.selectedDormId = this.selectedtenant.room.dorm.dormID;
        }

        this.getDormId();

    },
    computed:
    {

        selectedDorm() {
            return this.dorms.find(d => d.dormID === this.selectedDormId) || {};
        },

        filteredRooms() {
            const dorm = this.selectedDorm;
            return dorm ? dorm.rooms : [];
        },

    },


}
</script>
<style src="../../../../css/landlord/tenantpage.css"></style>