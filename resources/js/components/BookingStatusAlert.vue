<template>
    <div v-if="statusData"
        class="status-container p-4 rounded-4 shadow-sm border-0 mb-4 position-relative overflow-hidden"
        :class="statusData.themeClass">

        <div class="status-bg-icon">
            <i :class="statusData.bootstrapIcon"></i>
        </div>

        <div class="d-flex align-items-start position-relative">
            <div class="icon-box d-flex align-items-center justify-content-center rounded-circle shadow-sm me-3">
                <i :class="statusData.bootstrapIcon" class="fs-4"></i>
            </div>

            <div class="flex-grow-1">
                <h6 class="mb-1 fw-bold text-uppercase ls-1 small opacity-75">
                    Booking Status
                </h6>
                <h5 class="mb-2 fw-extrabold d-flex align-items-center">
                    {{ statusData.title }}
                </h5>
                <p class="mb-0 fw-medium status-message">
                    {{ statusData.message }}
                </p>

                <div v-if="statusData.note"
                    class="mt-3 p-2 px-3 rounded-3 bg-white bg-opacity-50 border border-white small fst-italic">
                    <i class="bi bi-info-circle me-1"></i> {{ statusData.note }}
                </div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        status: { type: String, required: true },
        role: {
            type: String,
            required: true,
            validator: (value) => ['tenant', 'landlord'].includes(value)
        }
    },
    computed: {
        statusData() {
            const config = {
                approved: {
                    themeClass: 'theme-success',
                    bootstrapIcon: 'bi bi-check-all',
                    title: 'Booking Approved',
                    message: this.role === 'tenant'
                        ? 'Congrats! You can now view your room details under "My Room".'
                        : 'Great! You have successfully approved this tenant\'s request.',
                },
                pending: {
                    themeClass: 'theme-warning',
                    bootstrapIcon: 'bi bi-hourglass-split',
                    title: 'Review in Progress',
                    message: this.role === 'tenant'
                        ? 'Please wait while the landlord reviews your booking request.'
                        : 'Action Required: You have a new booking request to review.',
                },
                confirmed: {
                    themeClass: 'theme-secondary',
                    bootstrapIcon: 'bi bi-credit-card-2-front',
                    title: 'Awaiting Payment',
                    message: this.role === 'tenant'
                        ? 'Your booking is confirmed! Please proceed with the payment.'
                        : 'The booking is confirmed. Waiting for the tenant to upload proof of payment.',
                    note: this.role === 'tenant'
                        ? 'Finalize your payment and select a move-in date to secure your slot.'
                        : null
                },
                rejected: {
                    themeClass: 'theme-danger',
                    bootstrapIcon: 'bi bi-x-circle',
                    title: 'Booking Declined',
                    message: this.role === 'tenant'
                        ? 'We regret to inform you that your booking was not approved.'
                        : 'You have declined this booking request.',
                },
                cancelled: {
                    themeClass: 'theme-danger',
                    bootstrapIcon: 'bi bi-slash-circle',
                    title: 'Booking Cancelled',
                    message: this.role === 'tenant'
                        ? 'You have successfully cancelled your booking.'
                        : 'This booking has been cancelled by the tenant.',
                },
                expired: {
                    themeClass: 'theme-dark',
                    bootstrapIcon: 'bi bi-calendar-x',
                    title: 'Booking Expired',
                    message: this.role === 'tenant'
                        ? 'Sorry, the time limit for this booking has passed.'
                        : 'The tenant failed to complete the booking within the given timeframe.',
                },
                paid: {
                    themeClass: 'theme-info',
                    bootstrapIcon: 'bi bi-shield-check',
                    title: 'Verification Needed',
                    message: this.role === 'tenant'
                        ? 'Payment received! Hang tight while the landlord verifies your deposit.'
                        : 'The tenant has uploaded a payment receipt. Please verify the transaction.',
                }
            };

            return config[this.status] || null;
        }
    }
};
</script>

<style scoped>
.status-container {
    transition: transform 0.3s ease;
    background-color: #fff;
}

.ls-1 {
    letter-spacing: 1px;
}

.fw-extrabold {
    font-weight: 800;
}

.icon-box {
    width: 48px;
    height: 48px;
    background: white;
    flex-shrink: 0;
}

.status-bg-icon {
    position: absolute;
    right: -20px;
    top: -20px;
    font-size: 7rem;
    opacity: 0.07;
    transform: rotate(15deg);
    pointer-events: none;
}

/* Theme Styling - Using soft colors with bold accents */
.theme-success {
    background-color: #f0fdf4;
    border-left: 6px solid #22c55e !important;
    color: #166534;
}

.theme-success .icon-box {
    color: #22c55e;
}

.theme-warning {
    background-color: #fffbeb;
    border-left: 6px solid #f59e0b !important;
    color: #92400e;
}

.theme-warning .icon-box {
    color: #f59e0b;
}

.theme-secondary {
    background-color: #fefce8;
    border-left: 6px solid #FC7D07 !important;
    color: #9a3412;
}

/* Brand Orange Accent */
.theme-secondary .icon-box {
    color: #FC7D07;
}

.theme-danger {
    background-color: #fef2f2;
    border-left: 6px solid #ef4444 !important;
    color: #991b1b;
}

.theme-danger .icon-box {
    color: #ef4444;
}

.theme-info {
    background-color: #f0f9ff;
    border-left: 6px solid #003C87 !important;
    color: #075985;
}

/* Brand Blue Accent */
.theme-info .icon-box {
    color: #003C87;
}

.theme-dark {
    background-color: #f8fafc;
    border-left: 6px solid #64748b !important;
    color: #334155;
}

.theme-dark .icon-box {
    color: #64748b;
}

.status-message {
    line-height: 1.5;
    font-size: 1rem;
}
</style>