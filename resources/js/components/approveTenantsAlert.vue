<template>
    <div v-if="status" class="status-card p-4 rounded-4 shadow-sm border-0 position-relative overflow-hidden mb-4"
        :class="alertClass">

        <div class="status-icon-bg">
            <i :class="statusIcon"></i>
        </div>

        <div class="d-flex align-items-center position-relative">
            <div
                class="status-icon-container d-flex align-items-center justify-content-center rounded-circle me-3 shadow-sm">
                <i :class="statusIcon" class="fs-4"></i>
            </div>

            <div class="text-start">
                <div class="d-flex align-items-center gap-2 mb-1">
                    <span class="text-uppercase ls-wide fw-bold small opacity-75">Account Status</span>
                    <span class="badge rounded-pill px-3 py-1 status-badge" :class="badgeClass">
                        {{ formattedStatus }}
                    </span>
                </div>
                <p class="mb-0 fw-medium status-text">{{ statusMessage }}</p>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'StatusAlert',
    props: {
        status: { type: String, required: true },
        role: { type: String, default: 'Tenant' }
    },
    computed: {
        // Dynamic Icons based on status
        statusIcon() {
            const icons = {
                active: 'bi bi-check-circle-fill',
                moved_out: 'bi bi-house-door-fill',
                terminated: 'bi bi-x-circle-fill',
                pending_moveout: 'bi bi-clock-history',
                transferring: 'bi bi-arrow-left-right',
                suspended: 'bi bi-exclamation-octagon-fill'
            };
            return icons[this.status] || 'bi bi-info-circle-fill';
        },
        alertClass() {
            return `status-theme-${this.status.replace('_', '-')}`;
        },
        badgeClass() {
            return `bg-custom-${this.status}`;
        },
        formattedStatus() {
            return this.status.replace('_', ' ').toUpperCase();
        },
        statusMessage() {
            const messages = {
                Tenant: {
                    active: 'You are currently residing in the property. All your records are up to date.',
                    moved_out: 'You have successfully moved out from the unit. Thank you for staying with us.',
                    terminated: 'Your tenancy has been terminated. Please contact administration.',
                    pending_moveout: 'Your request to move out is currently being processed.',
                    transferring: 'You are currently being transferred to another unit.',
                    suspended: 'Your account has been suspended. Please resolve this with the landlord.'
                },
                Landlord: {
                    active: 'The tenant is currently active and occupying the unit.',
                    moved_out: 'The tenant has moved out and vacated the unit.',
                    terminated: 'The tenant’s lease has been terminated due to valid reasons.',
                    pending_moveout: 'The tenant has initiated a move-out request.',
                    transferring: 'The tenant is in the process of transferring units.',
                    suspended: 'The tenant’s access is currently suspended pending resolution.'
                }
            };
            return messages[this.role]?.[this.status] || 'Status information is being reviewed.';
        }
    }
};
</script>

<style scoped>
/* Modern Theme Variables & Base Styles */
.status-card {
    transition: all 0.3s ease;
    background: #ffffff;
}

.ls-wide {
    letter-spacing: 1px;
}

/* Status Icon Containers */
.status-icon-container {
    width: 50px;
    height: 50px;
    flex-shrink: 0;
    background: rgba(255, 255, 255, 0.9);
}

.status-icon-bg {
    position: absolute;
    right: -20px;
    bottom: -30px;
    font-size: 8rem;
    opacity: 0.05;
    transform: rotate(-15deg);
    pointer-events: none;
}

/* Theme Colors - Custom Palettes */
.status-theme-active {
    background: #f0fdf4;
    border-left: 5px solid #22c55e !important;
    color: #166534;
}

.status-theme-active .status-icon-container {
    color: #22c55e;
}

.bg-custom-active {
    background-color: #22c55e;
}

.status-theme-moved-out {
    background: #f8fafc;
    border-left: 5px solid #64748b !important;
    color: #334155;
}

.status-theme-moved-out .status-icon-container {
    color: #64748b;
}

.bg-custom-moved_out {
    background-color: #64748b;
}

.status-theme-terminated {
    background: #fef2f2;
    border-left: 5px solid #ef4444 !important;
    color: #991b1b;
}

.status-theme-terminated .status-icon-container {
    color: #ef4444;
}

.bg-custom-terminated {
    background-color: #ef4444;
}

.status-theme-pending-moveout {
    background: #fffbeb;
    border-left: 5px solid #f59e0b !important;
    color: #92400e;
}

.status-theme-pending-moveout .status-icon-container {
    color: #f59e0b;
}

.bg-custom-pending_moveout {
    background-color: #f59e0b;
}

.status-theme-transferring {
    background: #f0f9ff;
    border-left: 5px solid #0ea5e9 !important;
    color: #075985;
}

.status-theme-transferring .status-icon-container {
    color: #0ea5e9;
}

.bg-custom-transferring {
    background-color: #0ea5e9;
}

.status-theme-suspended {
    background: #fafafa;
    border-left: 5px solid #171717 !important;
    color: #262626;
}

.status-theme-suspended .status-icon-container {
    color: #171717;
}

.bg-custom-suspended {
    background-color: #171717;
}

.status-badge {
    font-size: 0.75rem;
    color: white;
}

.status-text {
    font-size: 0.95rem;
    line-height: 1.4;
}
</style>