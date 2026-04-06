<template>
    <NotificationList ref="toastRef" />

    <div class="container-fluid py-4 bg-light-gray min-vh-100">
        <div class="row g-4">
            <div class="col-lg-8">
                <div class="card shadow-sm border-0 rounded-5 overflow-hidden">
                    <div class="card-header bg-solid-blue p-4 border-0">
                        <div class="d-flex align-items-center justify-content-between">
                            <h4 class="mb-0 text-white fw-black">
                                <i class="bi bi-receipt-cutoff me-2"></i>Payment History
                            </h4>
                            <span class="badge bg-white text-primary rounded-pill px-3 py-2 fw-bold">
                                {{ allHistory.length }} Records
                            </span>
                        </div>
                    </div>

                    <div class="card-body p-4 bg-white">
                        <div class="history-scroll-container px-2">

                            <div v-for="(history, index) in allHistory" :key="index"
                                class="payment-item-card mb-3 p-3 d-flex align-items-center justify-content-between">

                                <div class="d-flex align-items-center">
                                    <div class="icon-circle me-3"
                                        :class="history.paymentType === 'online' ? 'bg-soft-blue' : 'bg-soft-orange'">
                                        <i class="bi"
                                            :class="history.paymentType === 'online' ? 'bi-globe text-primary' : 'bi-house-door text-orange'"></i>
                                    </div>

                                    <div>
                                        <h6 class="mb-0 fw-bold text-dark text-capitalize">
                                            {{ history.type }} Payment
                                            <small class="text-muted fw-normal ms-1">#{{
                                                history.reservation?.room?.roomID || history.booking?.room?.roomID ||
                                                'N/A' }}</small>
                                        </h6>
                                        <p class="mb-0 text-muted small">
                                            <i class="bi bi-clock me-1"></i>{{ formatDate(history.created_at) }}
                                        </p>
                                    </div>
                                </div>

                                <div class="text-end d-flex align-items-center gap-4">
                                    <div class="amount-display">
                                        <small class="text-muted d-block" style="font-size: 0.7rem;">Amount Paid</small>
                                        <span class="fw-black text-success fs-5">₱{{ formatAmount(history.amount)
                                            }}</span>
                                    </div>
                                    <div class="badge-method">
                                        <span class="badge-custom"
                                            :class="history.paymentType === 'online' ? 'online' : 'onsite'">
                                            {{ history.paymentType || 'N/A' }}
                                        </span>
                                    </div>
                                </div>
                            </div>
                            <div v-if="allHistory.length === 0" class="text-center py-5">
                                <i class="bi bi-inbox text-muted display-4"></i>
                                <p class="text-muted mt-2">No payment history found for this selection.</p>
                            </div>

                        </div>
                    </div>
                </div>
            </div>

            <div class="col-lg-4">
                <div class="card shadow-sm border-0 rounded-5 sticky-top" style="top: 2rem;">
                    <div class="card-header bg-solid-orange p-4 border-0">
                        <h5 class="mb-0 text-white fw-black text-center">
                            <i class="bi bi-graph-up-arrow me-2"></i>Payment Summary
                        </h5>
                    </div>

                    <div class="card-body p-4 bg-white">
                        <div class="filter-section mb-4">
                            <label class="form-label fw-bold text-dark small text-uppercase">Select History Date</label>
                            <div class="input-group">
                                <span class="input-group-text bg-light border-0"><i class="bi bi-calendar3"></i></span>
                                <input type="date" v-model="chooseDate" @change="paymentHistory()"
                                    class="form-control bg-light border-0 py-2 rounded-end shadow-none">
                            </div>
                        </div>

                        <div class="summary-stat mb-4">
                            <div class="d-flex justify-content-between align-items-end mb-2">
                                <label class="fw-bold text-dark small text-uppercase">Payment Progress</label>
                                <span class="fw-black text-primary">{{ Math.round(percent) }}%</span>
                            </div>
                            <div class="progress rounded-pill shadow-inner"
                                style="height: 12px; background-color: #e9ecef;">
                                <div class="progress-bar bg-solid-blue progress-bar-striped progress-bar-animated"
                                    role="progressbar" :style="{ width: percent + '%' }">
                                </div>
                            </div>
                        </div>

                        <div class="total-amount-card p-3 rounded-4 mb-4 text-center">
                            <small class="text-uppercase fw-bold text-muted" style="letter-spacing: 1px;">Monthly
                                Total</small>
                            <h2 class="fw-black text-dark mb-0">₱{{ formatAmount(totalAmount) }}</h2>
                        </div>

                        <label class="form-label fw-bold text-dark small text-uppercase mb-3">Filter By Method</label>
                        <div class="method-selector d-flex flex-column gap-2">
                            <label class="method-option p-3 rounded-4 border"
                                :class="{ 'active': paymentMethod === 'online' }">
                                <input class="d-none" type="radio" v-model="paymentMethod" value="online"
                                    @change="paymentHistory()">
                                <div class="d-flex align-items-center">
                                    <i class="bi bi-phone-vibrate fs-4 me-3"></i>
                                    <span class="fw-bold">Online Payment</span>
                                    <i v-if="paymentMethod === 'online'"
                                        class="bi bi-check-circle-fill ms-auto text-primary"></i>
                                </div>
                            </label>

                            <label class="method-option p-3 rounded-4 border"
                                :class="{ 'active': paymentMethod === 'onsite' }">
                                <input class="d-none" type="radio" v-model="paymentMethod" value="onsite"
                                    @change="paymentHistory()">
                                <div class="d-flex align-items-center">
                                    <i class="bi bi-cash-stack fs-4 me-3"></i>
                                    <span class="fw-bold">Onsite Payment</span>
                                    <i v-if="paymentMethod === 'onsite'"
                                        class="bi bi-check-circle-fill ms-auto text-primary"></i>
                                </div>
                            </label>
                        </div>

                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
import axios from 'axios';
import Loader from '@/components/loader.vue';
import NotificationList from '@/components/notifications.vue';
export default {
    components: {
        Loader,
        NotificationList,
    },
    data() {
        return {
            tenant_id: '',
            reservationHistory: [],
            bookingHistory: [],
            paymentModal: false,
            approveHistory: [],
            roomHistory: [],
            allHistory: [],
            totalAmount: 0.0,
            bookingpaid: 0,
            reservationpaid: 0,
            approvepaid: 0,
            percent: 0,
            paymentMethod: '',
            chooseDate: this.todayDate,
            todayDate: new Date().toISOString().split('T')[0],
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

        async paymentHistory() {
            try {
                console.log(this.paymentMethod);
                const response = await axios.get(`/tenant/payment/history/list/${this.tenant_id}`, {
                    params: {
                        chooseDate: this.chooseDate,
                        paymentMethod: this.paymentMethod
                    }
                });
                // Tag each with its source type
                this.approveHistory = (response.data.approve_bill || []).map(item => ({
                    ...item,
                    type: 'Extension Payment'
                }));
                this.reservationHistory = (response.data.reservation_bills || []).map(item => ({
                    ...item,
                    type: 'Reservation'
                }));

                this.bookingHistory = (response.data.booking_bills || []).map(item => ({
                    ...item,
                    type: 'Booking'
                }));
                // Combine all and sort by date
                this.allHistory = [
                    ...this.approveHistory,
                    ...this.reservationHistory,
                    ...this.bookingHistory,
                ].sort((a, b) => {
                    const dateA = new Date(a.payment?.created_at || a.created_at || 0);
                    const dateB = new Date(b.payment?.created_at || b.created_at || 0);
                    return dateB - dateA;
                });
                this.getTotalAmount();


            } catch (error) {
                console.error("Error fetching payment history:", error);

            } finally {

            }
        },
        getTotalAmount() {
            axios.get(`/api/total-amount/${this.tenant_id}`, {
                params: {
                    chooseDate: this.chooseDate,
                    method: this.paymentMethod
                }
            }).then(response => {
                this.bookingpaid = response.data.bookingTotal;
                this.reservationpaid = response.data.reservationTotal;
                this.approvepaid = response.data.reservationTotal;
                this.totalAmount = response.data.totalAmount;
                this.percent = Math.min((response.data.totalAmount / 10000) * 100, 100);
            });
        },
        formatDate(date) {
            if (!date) return 'N/A';
            const d = new Date(date);
            return d.toLocaleDateString('en-PH', { year: 'numeric', month: 'long', day: 'numeric' });
        },
        formatAmount(amount) {
            if (!amount) return '0.00';
            return parseFloat(amount).toFixed(2);
        },

    },
    mounted() {
        this.tenant_id = window.tenant_id;
        this.paymentHistory();
        this.subscribeToNotifications();
    },
    watch: {
        paymentMethod(newVal) {
            console.log("Selected method:", newVal);
        }
    }
}
</script>
<style scoped src="/resources/css/tenant/payment.css"></style>
