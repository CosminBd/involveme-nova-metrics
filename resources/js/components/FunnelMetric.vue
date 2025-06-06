<template>
  <LoadingCard :loading="loading" class="py-3 im-nova-metrics">
    <div class="h-6 flex items-center px-6 mb-4">
      <h3 class="mr-3 leading-tight text-sm font-bold">{{card.name}}</h3>
      <SelectControl
          v-if="hasRanges"
          v-model="selectedRangeKey"
          :aria-label="__('Select Ranges')"
          :options="card.ranges"
          class="ml-auto w-[9rem] flex-shrink-0"
          size="xxs"
          @selected="handleRangeSelected"
      />
    </div>


    <div class="flex flex-wrap items-center justify-start px-6">
      <div v-for="(metricData, idx) in metricsData" :key="idx" class="flex items-center justify-between h-14 w-1/4 ">
        <div class="flex justify-between flex-1 items-center space-x-3">
          <span class="text-4xl">{{ formatNumber(metricData.value) }}</span>
          <div class="flex flex-col flex-grow text-center">
            <a v-if="metricData.link" :href="metricData.link" :title="metricData.tooltip" target="_blank" class="link-default flex-grow">{{ metricData.name }}</a>
            <span v-else :title="metricData.tooltip" class="flex-grow">{{ metricData.name }}</span>
            <div>
              <span class="text-sm" title="Compared to previous">{{calculatePercentageTo(idx,idx-1)}}%</span>
              <span v-if="idx > 1" class="text-sm font-bold" title="Compared to first">&nbsp;/ {{calculatePercentageTo(idx,0)}}%</span>
            </div>
          </div>
        </div>
        <div v-if="idx !== (metricsData.length - 1)">
          <svg class="w-12 h-12" fill="none" stroke="currentColor" stroke-width="1.5" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path d="M8.25 4.5l7.5 7.5-7.5 7.5" stroke-linecap="round" stroke-linejoin="round"/></svg>
        </div>
      </div>
    </div>
  </LoadingCard>
</template>

<script>

import MetricBehavior from '../mixins/MetricBehavior'

export default {
  props: [
    'card',
  ],
  mixins: [MetricBehavior],
  data: () => ({
    metricsData: [],
    selectedRangeKey: null,
    loading: true,
  }),
  computed: {
    hasRanges() {
      return this.card.ranges.length > 0;
    },
    metricPayload() {
      const payload = {
        params: {
          timezone: this.userTimezone,
        },
      }
      if (this.hasRanges) {
        payload.params.range = this.selectedRangeKey
      }
      return payload
    },
  },
  created() {
    if (this.hasRanges) {
      this.selectedRangeKey =
          this.card.selectedRangeKey || this.card.ranges[0].value
    }
    this.fetch()
  },
  methods: {
    handleRangeSelected(range) {
      this.selectedRangeKey = range.value
      this.fetch()
    },
    handleFetchCallback() {
      return (response) => {
        this.metricsData = response.data.value;
        this.loading = false;
      }
    },
    calculatePercentageTo(idx, idy) {
      if(idx === 0) {
        return 100;
      }
      if(this.metricsData[idy].value === 0) {
        return 0;
      }
      return Math.round((this.metricsData[idx].value / this.metricsData[idy].value) * 10000) / 100;
    },
    formatNumber(number) {
      return new Intl.NumberFormat('en-US', {maximumFractionDigits: 0}).format(number);
    }
  }
};
</script>
