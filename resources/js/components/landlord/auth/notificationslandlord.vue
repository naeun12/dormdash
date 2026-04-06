<template>
    <Loader ref="loader" />

    <div class="notifications-container p-4 bg-light min-vh-100">
        <div
            class="header d-flex align-items-center justify-content-between mb-4 pb-3 border-bottom bg-white p-3 rounded-4 shadow-sm">
            <div class="d-flex align-items-center">
                <div class="p-3 rounded-circle me-3" style="background-color: #003C87;">
                    <i class="bi bi-bell-fill text-white fs-4"></i>
                </div>
                <div>
                    <h3 class="fw-bold mb-0 text-dark">Notifications</h3>
                    <p class="text-muted mb-0 small">Manage your property alerts and updates</p>
                </div>
            </div>

            <div class="d-flex gap-2">
                <button class="btn btn-sm fw-bold px-3 rounded-pill transition-all border-2" @click="markAllAsRead"
                    :disabled="notifications.length === 0" style="color: #003C87; border-color: #003C87;">
                    <i class="bi bi-check2-all me-1"></i> Read All
                </button>

                <button class="btn btn-outline-danger btn-sm fw-bold px-3 rounded-pill border-2"
                    @click="clearAllNotifications" :disabled="notifications.length === 0">
                    <i class="bi bi-trash-fill me-1"></i> Clear
                </button>
            </div>
        </div>

        <div class="notification-list mx-auto" style="max-width: 900px;">
            <div v-if="notifications.length === 0" class="text-center py-5 bg-white rounded-4 shadow-sm border">
                <i class="bi bi-chat-left-dots display-1 d-block mb-3 opacity-10"></i>
                <h5 class="fw-bold text-secondary">All caught up!</h5>
                <p class="small text-muted">No new notifications at the moment.</p>
            </div>

            <div v-for="notification in notifications" :key="notification.id"
                class="card border-0 shadow-sm mb-3 rounded-4 overflow-hidden transition-all hover-shadow" :style="!notification.readAt
                    ? 'border-left: 6px solid #FC7D07 !important; background-color: #fdfbff;'
                    : 'border-left: 6px solid #dee2e6 !important;'">

                <div class="card-body p-3">
                    <div class="d-flex align-items-start">
                        <div class="me-3">
                            <div v-if="!notification.readAt" class="rounded-circle p-2"
                                style="background-color: #fff4e6;">
                                <i class="bi bi-lightning-fill" style="color: #FC7D07;"></i>
                            </div>
                            <div v-else class="rounded-circle p-2 bg-light">
                                <i class="bi bi-check-circle text-muted"></i>
                            </div>
                        </div>

                        <div class="flex-grow-1">
                            <div class="d-flex justify-content-between align-items-start">
                                <div>
                                    <h6 class="mb-0 fw-bold text-dark" style="font-size: 1.05rem;">
                                        {{ notification.sender?.firstname }} {{ notification.sender?.lastname }}
                                    </h6>
                                    <span class="badge text-uppercase mb-2 mt-1"
                                        :style="!notification.readAt ? 'background-color: #003C87;' : 'background-color: #6c757d;'"
                                        style="font-size: 0.65rem; letter-spacing: 0.5px;">
                                        {{ notification.action }}
                                    </span>
                                </div>
                                <small class="text-muted fw-medium" style="font-size: 0.75rem;">
                                    <i class="bi bi-clock-history me-1"></i> {{ formatDate(notification.created_at) }}
                                </small>
                            </div>

                            <p class="text-secondary mb-3 mt-1" style="font-size: 0.95rem; line-height: 1.5;">
                                {{ notification.message }}
                            </p>

                            <div class="d-flex justify-content-between align-items-center pt-2 border-top">
                                <div class="small">
                                    <span v-if="notification.readAt" class="text-success fw-bold">
                                        <i class="bi bi-check2-circle me-1"></i>Seen {{ formatDate(notification.readAt) }}
                                    </span>
                                    <span v-else class="text-warning fw-bold">
                                        <i class="bi bi-envelope-exclamation me-1"></i>New Update
                                    </span>
                                </div>

                                <button class="btn btn-sm fw-bold px-4 rounded-3 text-white shadow-sm"
                                    style="background-color: #003C87; border: none;"
                                    @click="markNotificationAsRead(notification.id)">
                                    VIEW DETAILS
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div v-if="ismodalviewNotifications" class="modal fade show d-block" tabindex="-1"
            style="background-color: rgba(0, 30, 60, 0.8); backdrop-filter: blur(5px);">
            <div class="modal-dialog modal-dialog-centered">
                <div class="modal-content border-0 shadow-lg rounded-4 overflow-hidden">
                    <div class="modal-header border-0 p-4 text-white" style="background-color: #003C87;">
                        <h5 class="modal-title fw-bold d-flex align-items-center">
                            <i class="bi bi-info-square-fill me-2" style="color: #FC7D07;"></i>
                            Notification Details
                        </h5>
                        <button type="button" class="btn-close btn-close-white"
                            @click="ismodalviewNotifications = false"></button>
                    </div>

                    <div class="modal-body p-4 bg-white">
                        <div class="p-3 rounded-4 mb-3 border-start border-4 border-primary bg-light">
                            <p class="mb-0 text-dark fw-medium" style="line-height: 1.6;">
                                {{ selectedNotification.message }}
                            </p>
                        </div>

                        <div class="d-flex align-items-center p-2 rounded-3 border">
                            <div class="p-2 rounded-circle me-2 bg-light">
                                <i
                                    :class="selectedNotification.readAt ? 'bi bi-check2-circle text-success' : 'bi bi-clock text-warning'"></i>
                            </div>
                            <small class="text-muted fw-bold">
                                Status: {{ formatDate(selectedNotification.readAt) ? 'Read on ' + formatDate(selectedNotification.readAt) :
                                "Unread Message" }}
                            </small>
                        </div>
                    </div>

                    <div class="modal-footer border-0 p-3 bg-light">
                        <button class="btn btn-dark fw-bold rounded-pill px-4 w-100"
                            @click="ismodalviewNotifications = false">CLOSE</button>
                    </div>
                </div>
            </div>
        </div>

        <Modalconfirmation ref="modal" />
    </div>
</template>
<script>
import axios from 'axios';
import NotificationList from '@/components/notifications.vue';
import Loader from '@/components/loader.vue';
import Modalconfirmation from '@/components/modalconfirmation.vue';
export default {
    components: {
        Loader,
        Modalconfirmation,
        NotificationList,
    },
    data() {
        return {
            // Notification Data
            notifications: [],
            landlord_id: '',
            ismodalviewNotifications: false,
            selectedNotification: {},
        };
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
        async getNotificationsList() {
            try {
                this.$refs.loader.loading = true;

                const response = await axios.get(`/get/notifications/landlord/${this.landlord_id}`);
                if (response.data.status === 'success') {
                    this.notifications = response.data.notifications;
                    this.$refs.loader.loading = false;

                }
            } catch (error) {
                console.error(error);
            }
        },
        async markNotificationAsRead(id) {
            try {
                this.$refs.loader.loading = true;

                const res = await axios.post(`/mark/read/landlord/${id}`);
                this.ismodalviewNotifications = true;
                this.$refs.loader.loading = false;

                this.selectedNotification = res.data.notification;
                this.getNotificationsList();
            } catch (error) {
                console.error(error);
            }
        },

        async markAllAsRead() {
            const confirmed = await this.$refs.modal.show({
                title: 'Mark Notifications',
                message: `Confirm to mark all notifications as read?`,
                functionName: 'Mark Notifications',
            });
            if (!confirmed) {
                return;
            }
            try {
                this.$refs.loader.loading = true;
                await axios.post(`/notifications/mark-all-as-read`, { landlord_id: this.landlord_id }, {
                    headers: {
                        "X-CSRF-TOKEN": document.querySelector('meta[name="csrf-token"]').getAttribute("content")
                    }

                });
                this.getNotificationsList();

            } catch (error) {

            }
            finally {
                this.$refs.loader.loading = false;

            }


        },
        async clearAllNotifications() {
            const confirmed = await this.$refs.modal.show({
                title: 'Clear Notifications',
                message: `Confirm to clear all notifications?`,
                functionName: 'Clear Notifications',
            });
            if (!confirmed) {
                return;
            }
            try {
                this.$refs.loader.loading = true;
                await axios.post(`/clear/notifications`, { landlord_id: this.landlord_id }, {
                    headers: {
                        "X-CSRF-TOKEN": document.querySelector('meta[name="csrf-token"]').getAttribute("content")
                    }

                });
                this.getNotificationsList();

            } catch (error) {

            }
            finally {
                this.$refs.loader.loading = false;

            }

        },
        formatDate(date) {
            if (!date) return "Not read yet";
            const options = {
                weekday: 'long',
                year: 'numeric',
                month: 'long',
                day: 'numeric',
                hour: '2-digit',
                minute: '2-digit',
                second: '2-digit'
            };
            return new Date(date).toLocaleDateString('en-US', options);
        }





    },
    mounted() {
        const el = document.getElementById('notificationsLandlord');
        if (el) {
            this.landlord_id = el.getAttribute('data-landlord-id');
        }
        this.subscribeToNotifications();
        this.getNotificationsList();
    }
};
</script>

