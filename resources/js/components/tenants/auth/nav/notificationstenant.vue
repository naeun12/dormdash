<template>
    <Loader ref="loader" />

    <div class="notifications-wrapper py-5">
        <div class="container">
            <div class="glass-header d-flex align-items-center justify-content-between p-4 mb-5 shadow-sm">
                <div class="d-flex align-items-center">
                    <div class="modern-bell-icon me-3">
                        <i class="bi bi-bell-fill"></i>
                        <span v-if="notifications.some(n => !n.readAt)" class="pulse-dot"></span>
                    </div>
                    <div>
                        <h2 class="fw-black mb-0">Notifications</h2>
                        <small class="text-muted text-uppercase tracking-wider">Stay updated with DormDash</small>
                    </div>
                </div>
                <div class="d-flex gap-3">
                    <button class="btn btn-modern-outline" @click="markAllAsRead"
                        :disabled="notifications.length === 0">
                        <i class="bi bi-check-all me-1"></i> Mark All
                    </button>
                    <button class="btn btn-modern-danger" @click="clearAllNotifications"
                        :disabled="notifications.length === 0">
                        <i class="bi bi-trash3"></i>
                    </button>
                </div>
            </div>

            <div class="notification-feed">
                <div v-if="notifications.length === 0" class="empty-state-modern py-5">
                    <div class="empty-icon-wrapper mb-3">
                        <i class="bi bi-chat-dots-fill"></i>
                    </div>
                    <h5 class="fw-bold">All caught up!</h5>
                    <p class="text-muted">No new notifications at the moment.</p>
                </div>

                <div v-for="notification in notifications" :key="notification.id"
                    class="notification-card-modern mb-3 transition-all" :class="{ 'is-unread': !notification.readAt }">
                    <div class="card-body p-4 d-flex align-items-center">
                        <div class="indicator-column me-4">
                            <div class="status-ring" :class="notification.readAt ? 'read' : 'unread'"></div>
                        </div>

                        <div class="flex-grow-1">
                            <div class="d-flex align-items-center mb-1">
                                <span class="sender-name fw-bold me-2">
                                    {{ notification.sender?.firstname }} {{ notification.sender?.lastname }}
                                </span>
                                <span class="action-pill shadow-sm">{{ notification.action }}</span>
                            </div>
                            <p class="mb-0 text-secondary message-preview">{{ notification.message }}</p>
                            <div class="mt-2 d-flex align-items-center gap-3">
                                <small class="timestamp-modern"><i class="bi bi-clock me-1"></i> {{
                                    notification.created_at }}</small>
                                <small v-if="notification.readAt" class="text-success fw-bold"><i
                                        class="bi bi-check2-circle"></i> Seen</small>
                            </div>
                        </div>

                        <div class="action-column ms-3">
                            <button class="btn btn-view-modern shadow-sm"
                                @click="markNotificationAsRead(notification.id)">
                                View <i class="bi bi-chevron-right ms-1"></i>
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div v-if="ismodalviewNotifications" class="modern-modal-overlay">
            <div class="modal-dialog modal-dialog-centered">
                <div
                    class="modal-content border-0 shadow-2xl rounded-5 overflow-hidden animate__animated animate__zoomIn animate__faster">
                    <div class="modal-header-blue p-4 text-white d-flex justify-content-between align-items-center">
                        <h5 class="fw-bold mb-0">{{ selectedNotification.title || "Activity Details" }}</h5>
                        <button type="button" class="btn-close-custom" @click="ismodalviewNotifications = false">
                            <i class="bi bi-x-lg"></i>
                        </button>
                    </div>
                    <div class="modal-body p-4 bg-white">
                        <div class="p-4 rounded-4 bg-light mb-4">
                            <p class="fs-5 text-dark mb-0">{{ selectedNotification.message }}</p>
                        </div>
                        <div class="d-flex justify-content-end">
                            <button class="btn btn-orange-rounded px-5 py-2"
                                @click="ismodalviewNotifications = false">Dismiss</button>
                        </div>
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
            tenant_id: '',
            ismodalviewNotifications: false,
            selectedNotification: {},
        };
    },
    methods: {
        subscribeToNotifications() {
            if (this.hasSubscribed) return;
            this.hasSubscribed = true;

            this.receiverID = this.tenant_id;
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
                const response = await axios.get(`/get/notifications/tenant/${this.tenant_id}`);
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

                const res = await axios.post(`/mark/read/tenant/${id}`);
                this.ismodalviewNotifications = true;
                this.selectedNotification = res.data.notification;
                this.getNotificationsList();
            } catch (error) {
                console.error(error);
            }
            finally {
                this.$refs.loader.loading = false;
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
                await axios.post(`/notifications/mark-all-as-read/tenant`, { tenant_id: this.tenant_id }, {
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
                await axios.post(`/clear/notifications/tenant`, { tenant_id: this.tenant_id }, {
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

        }
    },
    mounted() {
        const el = document.getElementById('notificationsTenant');
        if (el) {
            this.tenant_id = el.getAttribute('data-tenant-id');
        }

        this.subscribeToNotifications();
        this.getNotificationsList();
    }
};
</script>
<style src="../../../../../css/tenant/notifications.css"></style>