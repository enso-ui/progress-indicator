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
import Gradient from 'javascript-color-gradient';
import { faCircleCheck, faCircleExclamation } from '@fortawesome/free-solid-svg-icons';
import ProgressIndicator from './ProgressIndicator.vue';

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
        gradient: new Gradient(),
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

            this.gradient.setMidpoint(100)
                .setGradient(...colors);

            this.color = this.inThreshold
                ? this.thresholdColor
                : this.gradient.getColor(this.gradientIndex);
        },
    },
};
</script>
