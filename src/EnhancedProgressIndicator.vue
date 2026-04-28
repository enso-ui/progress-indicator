<template>
    <progress-indicator
        :value="value"
        :percentage="percentage"
        :average="average"
        :fill-color="color">
        <template #above
            v-if="emptyIndicator && percentage < 5">
            <div class="level-item">
                <span class="icon"
                    :style="{'color': color}">
                    <fa :icon="inverted ? faCircleCheck : faCircleExclamation"
                        size="sm"/>
                </span>
            </div>
        </template>
    </progress-indicator>
</template>

<script>
import { FontAwesomeIcon as Fa } from '@fortawesome/vue-fontawesome';
import { faCircleCheck, faCircleExclamation } from '@fortawesome/free-solid-svg-icons';
import ProgressIndicator from './ProgressIndicator.vue';

const hexToRgb = hex => {
    const value = hex.replace('#', '');
    const normalized = value.length === 3
        ? value.split('').map(char => char + char).join('')
        : value;
    return {
        r: Number.parseInt(normalized.slice(0, 2), 16),
        g: Number.parseInt(normalized.slice(2, 4), 16),
        b: Number.parseInt(normalized.slice(4, 6), 16),
    };
};

const hex = value => value.toString(16).padStart(2, '0');

const gradientColor = (from, to, percentage) => {
    const start = hexToRgb(from);
    const end = hexToRgb(to);
    const ratio = Math.min(Math.max(percentage, 1), 100) / 100;

    return `#${hex(Math.round(start.r + (end.r - start.r) * ratio))}`
        + `${hex(Math.round(start.g + (end.g - start.g) * ratio))}`
        + `${hex(Math.round(start.b + (end.b - start.b) * ratio))}`;
};

export default {
    name: 'EnhancedProgressIndicator',

    components: {
        Fa,
        ProgressIndicator,
    },

    props: {
        aboveThresholdColor: {
            default: '#0DFF00',
            type: String,
        },
        average: {
            default: null,
            type: Number,
            validator: value => value >= 0 && value <= 100,
        },
        belowThresholdColor: {
            default: '#FF0000',
            type: String,
        },
        emptyIndicator: {
            default: false,
            type: Boolean,
        },
        percentage: {
            required: true,
            type: Number,
            validator: value => value >= 0 && value <= 100,
        },
        inverted: {
            default: false,
            type: Boolean,
        },
        thresholdColor: {
            default: '#F7DD57',
            type: String,
        },
        tolerance: {
            required: true,
            type: Number,
            validator: value => value >= 0 && value <= 100,
        },
        value: {
            required: true,
            type: Number,
        },
    },

    data: () => ({
        color: null,
        faCircleCheck,
        faCircleExclamation,
    }),

    computed: {
        aboveAverage() {
            return this.percentage > (this.average + this.tolerance);
        },
        gradientIndex() {
            const index = this.aboveAverage
                ? Math.round((this.percentage - this.average) / (100 - this.average) * 100)
                : Math.round((this.average - this.percentage) / this.average * 100);

            return index || 1;
        },
        inThreshold() {
            return Math.abs(this.percentage - this.average) <= this.tolerance;
        },
    },

    created() {
        this.init();
    },

    methods: {
        init() {
            // eslint-disable-next-line no-bitwise
            const colors = this.aboveAverage ^ this.inverted
                ? [this.thresholdColor, this.aboveThresholdColor]
                : [this.thresholdColor, this.belowThresholdColor];

            this.color = this.inThreshold
                ? this.thresholdColor
                : gradientColor(...colors, this.gradientIndex);
        },
    },
};
</script>
