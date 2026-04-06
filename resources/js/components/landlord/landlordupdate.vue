<template>
    <Loader ref="loader" />
    <NotificationList ref="toastRef" />
    <div class="p-4 bg-white rounded-4 shadow-sm">
        <div class="text-center mb-5">
            <div class="position-relative d-inline-block">
                <div class="profile-container p-1 rounded-circle shadow"
                    style="background: linear-gradient(45deg, #003C87, #FC7D07);">
                    <img :src="landlord.previewPicUrl || (landlord.profilePicUrl ? '/' + landlord.profilePicUrl : '/default-avatar.png')"
                        alt="Profile Preview" class="rounded-circle border border-4 border-white"
                        style="width: 140px; height: 140px; object-fit: cover;">
                </div>

                <label for="profilePic"
                    class="btn btn-sm position-absolute bottom-0 end-0 rounded-circle shadow-sm d-flex align-items-center justify-content-center"
                    style="width: 42px; height: 42px; cursor: pointer; background-color: #FC7D07; border: 3px solid white; color: white;">
                    <i class="bi bi-camera-fill"></i>
                </label>
            </div>
            <input type="file" id="profilePic" name="profilePicUrl" accept="image/*" class="d-none"
                @change="previewImage">

            <div class="mt-3">
                <h4 class="fw-bold text-dark mb-1">{{ landlord.firstname }} {{ landlord.lastname }}</h4>
                <span v-if="landlord.isVerified" class="badge rounded-pill px-3 py-2 shadow-sm"
                    style="background-color: #003C87;">
                    <i class="bi bi-patch-check-fill me-1 text-info"></i> Verified Landlord
                </span>
                <span v-else class="badge bg-danger rounded-pill px-3 py-2 shadow-sm">
                    <i class="bi bi-x-circle-fill me-1"></i> Not Verified
                </span>
            </div>
        </div>

        <div class="row g-4">
            <div class="col-md-6">
                <label class="form-label small fw-bold text-muted text-uppercase mb-1">First Name</label>
                <div class="input-group shadow-sm">
                    <span class="input-group-text bg-white border-end-0 text-muted"><i class="bi bi-person"></i></span>
                    <input type="text" class="form-control border-start-0 rounded-end-3" v-model="landlord.firstname"
                        placeholder="First Name">
                </div>
                <p class="text-danger small fst-italic mt-1" v-if="error.firstname">{{ error.firstname[0] }}</p>
            </div>

            <div class="col-md-6">
                <label class="form-label small fw-bold text-muted text-uppercase mb-1">Last Name</label>
                <div class="input-group shadow-sm">
                    <span class="input-group-text bg-white border-end-0 text-muted"><i class="bi bi-person"></i></span>
                    <input type="text" class="form-control border-start-0 rounded-end-3" v-model="landlord.lastname"
                        placeholder="Last Name">
                </div>
                <p class="text-danger small fst-italic mt-1" v-if="error.lastname">{{ error.lastname[0] }}</p>
            </div>

            <div class="col-md-6">
                <label class="form-label small fw-bold text-muted text-uppercase mb-1">Email Address</label>
                <input type="email" class="form-control bg-light border-0 rounded-3 shadow-none p-2 ps-3"
                    v-model="landlord.email" readonly>
            </div>

            <div class="col-md-6">
                <label class="form-label small fw-bold text-muted text-uppercase mb-1">Phone Number</label>
                <input type="tel" class="form-control bg-light border-0 rounded-3 shadow-none p-2 ps-3"
                    v-model="landlord.phoneNumber" readonly>
            </div>

            <div class="col-md-6">
                <label class="form-label small fw-bold text-muted text-uppercase mb-1">Gender</label>
                <select class="form-select rounded-3 shadow-sm border-light" v-model="landlord.gender">
                    <option disabled value="">Select gender</option>
                    <option value="Male">Male</option>
                    <option value="Female">Female</option>
                </select>
            </div>

            <div class="col-md-6 d-flex align-items-end">
                <button @click="btnclickUpdateDocument()"
                    class="btn btn-outline-dark fw-bold w-100 rounded-3 py-2 border-2">
                    <i class="bi bi-files me-2"></i> Update Documents
                </button>
            </div>
        </div>

        <div v-if="clickUpdateDocument" class="modal fade show d-block"
            style="background: rgba(0,0,0,0.6); backdrop-filter: blur(4px);" tabindex="-1">
            <div class="modal-dialog modal-lg modal-dialog-centered">
                <div class="modal-content shadow-lg rounded-4 border-0">
                    <div class="modal-header text-white py-3" style="background-color: #003C87;">
                        <h5 class="modal-title fw-bold d-flex align-items-center">
                            <i class="bi bi-shield-lock-fill me-2 text-warning"></i> Verification Documents
                        </h5>
                        <button type="button" class="btn-close btn-close-white"
                            @click="clickUpdateDocument = false"></button>
                    </div>

                    <div class="modal-body p-4 bg-light">
                        <div class="row g-4">
                            <div class="col-md-6">
                                <div class="card h-100 border-0 shadow-sm rounded-3">
                                    <div class="card-body p-3">
                                        <h6 class="fw-bold text-dark mb-3 small text-uppercase">Business Permit</h6>
                                        <div class="position-relative overflow-hidden rounded-3 border-dashed border-2 p-2"
                                            style="border-color: #dee2e6;">
                                            <img :src="landlord.businessPermitPreview || (landlord.businessPermit ? '/' + landlord.businessPermit : '/images/no-file.png')"
                                                class="img-fluid rounded shadow-sm w-100"
                                                style="height: 180px; object-fit: cover;">

                                            <label for="businessPermit"
                                                class="position-absolute top-50 start-50 translate-middle btn btn-light btn-sm shadow rounded-pill px-3 fw-bold border-0">
                                                <i class="bi bi-cloud-upload me-1 text-primary"></i> Change Photo
                                            </label>
                                        </div>
                                        <input type="file" id="businessPermit" class="d-none"
                                            @change="previewBusinessPermit">
                                    </div>
                                </div>
                            </div>

                            <div class="col-md-6">
                                <div class="card h-100 border-0 shadow-sm rounded-3">
                                    <div class="card-body p-3">
                                        <h6 class="fw-bold text-dark mb-3 small text-uppercase">Government ID</h6>
                                        <div class="position-relative overflow-hidden rounded-3 border-dashed border-2 p-2"
                                            style="border-color: #dee2e6;">
                                            <img :src="landlord.governmentIDPreview || (landlord.govermentID ? '/' + landlord.govermentID : '/images/no-file.png')"
                                                class="img-fluid rounded shadow-sm w-100"
                                                style="height: 180px; object-fit: cover;">

                                            <label for="governmentID"
                                                class="position-absolute top-50 start-50 translate-middle btn btn-light btn-sm shadow rounded-pill px-3 fw-bold border-0">
                                                <i class="bi bi-cloud-upload me-1 text-success"></i> Change Photo
                                            </label>
                                        </div>
                                        <input type="file" id="governmentID" class="d-none"
                                            @change="previewGovernmentID">
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="alert alert-warning mt-4 border-0 rounded-3 small py-2">
                            <i class="bi bi-info-circle-fill me-2"></i> Only JPG and PNG formats are allowed. Maximum
                            file size is 2MB.
                        </div>
                    </div>

                    <div class="modal-footer border-0 p-3 bg-white">
                        <button class="btn btn-lg w-100 text-white fw-bold rounded-3" style="background-color: #003C87;"
                            @click="updateDocuments">
                            <i class="bi bi-save2 me-2"></i> Save Verified Documents
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <div class="mt-5">
            <button @click="updateLandlordAccount"
                class="btn btn-lg fw-bold px-5 rounded-pill shadow w-100 transition-all hover-lift"
                style="background-color: #FC7D07; color: white;">
                <i class="bi bi-person-check-fill me-2"></i> Save Profile Updates
            </button>
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
export default {
    components: {
        Toastcomponents,
        Loader,
        Modalconfirmation,
        NotificationList,
    },
    name: 'LandlordUpdate',
    data() { 
        return {
            landlord: {
                landlordID: '',
                firstname: '',
                lastname: '',
                email: '',
                gender: '',
                phoneNumber: '',
                profilePicUrl: '',
                previewPicUrl: '',
                profilePicFile: '',  
                isVerified: '',
                businessPermit: '',
                businessPermitPreview: '',
                businessPermitFile: '',
                govermentID: '',
                governmentIDPreview: '',
                governmentIDFile : '',
            },
            error: {},
            landlord_id: '',
            clickUpdateDocument: false,

        }
    },
    methods: {
        fetchLandlordData() {
            this.$refs.loader.loading = true;

            fetch('/get/landlord/data/' + this.landlord_id)
                .then(response => response.json())
                .then(data => {
                    if (data.landlord) {
                            this.landlord = {
                                firstname: data.landlord.firstname ?? '',
                                lastname: data.landlord.lastname ?? '',
                                email: data.landlord.email ?? '',
                                gender: data.landlord.gender ?? '',
                                phoneNumber: data.landlord.phoneNumber ?? '',
                                landlordID: data.landlord.landlordID ?? '',
                                profilePicUrl: data.landlord.profilePicUrl ?? '',
                                isVerified: data.landlord.isVerified ?? '',
                                govermentID: data.landlord.govermentID ?? '',
                                businessPermit: data.landlord.businessPermit ?? '',

                            };
                         this.$refs.loader.loading = false;

                    } else {
                        console.error('Landlord data not found');
                        this.$refs.loader.loading = false;

                    }
                })
                .catch(error => console.error('Error fetching landlord data:', error));
        },
         btnclickUpdateDocument() { 
            this.clickUpdateDocument = true;
        },
        async updateLandlordAccount() { 
            try {
                const confirmed = await this.$refs.modal.show({
                    title: 'Update Account',
                    message: `Confirm update to this account information?`,
                    functionName: 'Update Account',
                });
                if (!confirmed) {
                    return;
                }
                this.$refs.loader.loading = true;
                const formdata = new FormData();
                formdata.append('firstname', this.landlord.firstname);
                formdata.append('lastname', this.landlord.lastname);
                formdata.append('gender', this.landlord.gender);
                if (this.landlord.profilePicFile) {
                    formdata.append('profilePicUrl', this.landlord.profilePicFile);
                }
                const response = await axios.post(`/update/landlord/data/${this.landlord.landlordID}`, formdata, {
                    headers: { 'Content-Type': 'multipart/form-data' }
                });        
                if (response.data.status === 'success') { 
                    this.error = {};
                    this.landlord = response.data.landlord;
                    this.$refs.toast.showToast(response.data.message, 'success');
                }
                    
            }
            catch (error) {
                    if (error.response && error.response.status === 422) {
                        // Laravel validation errors
                        this.error = error.response.data.errors;
                    } else {
                        console.log(error);
                    }
                }

            
            finally { 
                this.$refs.loader.loading = false;

            }
        },
        async updateDocuments() {
            try {
                this.$refs.loader.loading = true;

                const formdata = new FormData();

                // Business Permit
                if (this.landlord.businessPermitFile) {
                    formdata.append('businessPermit', this.landlord.businessPermitFile);
                }

                // Government ID
                if (this.landlord.governmentIDFile) {
                    formdata.append('governmentID', this.landlord.governmentIDFile);
                }

                // Send to backend
                const response = await axios.post(
                    `/update/landlord/documents/${this.landlord.landlordID}`,
                    formdata,
                    { headers: { "Content-Type": "multipart/form-data" } }
                );

                if (response.data.status === "success") {
                    this.landlord = response.data.landlord; 
                    this.clickUpdateDocument = false;
                    this.$refs.toast.showToast(response.data.message, "success");
                }
            }
            catch (error) {
                if (error.response && error.response.status === 422) {
                    this.error = error.response.data.errors; // validation errors
                } else {
                    console.error(error);
                    this.$refs.toast.showToast("Something went wrong. Try again.", "error");
                }
            }
            finally {
                console.log("Update documents request finished.");
                this.$refs.loader.loading = false;

            }
        },

        previewImage(event) {
            const file = event.target.files[0];
            if (file) {
                this.landlord.previewPicUrl = URL.createObjectURL(file);
                this.landlord.profilePicFile = file; // ✅ diri siya gi-assign
            }
        },
        previewBusinessPermit(event) {
            const file = event.target.files[0];
            if (file) {
                this.landlord.businessPermitPreview = URL.createObjectURL(file);
                this.landlord.businessPermitFile = file; // keep file for upload
            }
        },
        previewGovernmentID(event) {
            const file = event.target.files[0];
            if (file) {
                this.landlord.governmentIDPreview = URL.createObjectURL(file);
                this.landlord.governmentIDFile = file; // keep file for upload
            }
        },
    },
    mounted() {
        const element = document.getElementById('landlordaccountUpdated');
        this.landlord_id = element.dataset.landlordId;
        this.fetchLandlordData();
        // window.vueInstance = this;


        }
}
</script>
<style scoped>
.hover-lift:hover {
    transform: translateY(-3px);
    box-shadow: 0 10px 20px rgba(252, 125, 7, 0.3) !important;
}

.border-dashed {
    border-style: dashed !important;
}

.transition-all {
    transition: all 0.3s ease;
}
</style>