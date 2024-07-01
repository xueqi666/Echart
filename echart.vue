<template>
    <div ref="chartContainer" :style="containerStyle"></div>
</template>

<script>
import * as echarts from 'echarts';
import _ from 'lodash';

export default {
    name: 'EChartComponent',
    props: {
        options: {
            type: Object,
            default: () => ({}),
            required: true
        },
        width: {
            type: String,
            default: '100%'
        },
        height: {
            type: String,
            default: '400px'
        }
    },
    data() {
        return {
            chart: null,
            initialized: false // 初始化标志
        };
    },
    computed: {
        containerStyle() {
            return {
                width: this.width,
                height: this.height
            };
        }
    },
    mounted() {
        this.$nextTick(() => {
            this.initChart();
            this.throttledResizeChart = _.throttle(this.resizeChart, 100, { leading: false, trailing: true });
            window.addEventListener('resize', this.throttledResizeChart);
        });
    },
    beforeUnmount() {
        window.removeEventListener('resize', this.throttledResizeChart);
        if (this.chart) {
            this.chart.dispose();
        }
    },
    methods: {
        initChart() {
            if (!this.initialized) {
                if (this.chart) {
                    this.chart.dispose();
                }
                if (this.$refs.chartContainer) {
                    this.chart = echarts.init(this.$refs.chartContainer);
                    if (this.options) {
                        this.chart.setOption(this.options,true);
                    } else {
                        console.error('Options are undefined');
                    }
                    this.initialized = true;
                } else {
                    console.error('Chart container is undefined');
                }
            }
        },
        resizeChart() {
            if (this.chart) {
                this.chart.resize();
            }
        }
    },
    watch: {
        options: {
            handler(newOptions) {
                if (this.chart && newOptions) {
                    this.chart.setOption(newOptions, true); // 合并新的配置项
                }
            },
            deep: true
        },
        width(newWidth) {
            this.resizeChart();
        },
        height(newHeight) {
            this.resizeChart();
        }
    }
};
</script>

<style scoped>
.chart-container {
    width: 100%;
    height: 400px;
    /* 这个初始值会被传入的props覆盖 */
}
</style>
