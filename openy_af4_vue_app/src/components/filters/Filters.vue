<template>
  <div class="filters-component">
    <div class="filters-header" :class="headerClasses">
      <span class="text-uppercase">
        <strong>{{ 'Filter' | t }}</strong>
      </span>
      <a role="button" @click="clearFilters">{{ 'Clear Filters' | t }}</a>
    </div>
    <div class="filters">
      <Fieldset
        :label="'Schedules' | t"
        :collapse-id="id + '-toggle-schedules'"
        :collapsed="fieldsetCollapseState('schedule')"
        :counter="scheduleFiltersCount"
        :hide-counter="true"
      >
        <div class="schedules-filter-component">
          <AgesFilter
            :id="id + '-ages-filter'"
            v-model="selectedAges"
            :ages="ages"
            :max-ages="maxAges"
            :facets="data.facets.static_age_filter ? data.facets.static_age_filter : []"
          />
          <DaysFilter
            v-if="legacyMode && !weeksFilter"
            :id="id + '-days-filter'"
            v-model="selectedDays"
            :days="days"
            :facets="data.facets.days_of_week"
          />
          <DaysTimesFilter
            v-if="!legacyMode && !weeksFilter"
            :id="id + '-days-times-filter'"
            v-model="selectedDaysTimes"
            :days-times="daysTimes"
            :facets="data.facets.af_weekdays_parts_of_day"
          />
          <WeeksFilter
            v-if="weeksFilter"
            :id="id + '-weeks-filter'"
            v-model="selectedWeeks"
            :weeks="weeks"
            :facets="data.facets.static_weeks_filter"
          />
        </div>
      </Fieldset>

      <Fieldset
        :label="'Activities' | t"
        :collapse-id="id + '-toggle-activities'"
        :collapsed="fieldsetCollapseState('category')"
        :counter="activityFiltersCount"
        :hide-counter="true"
      >
        <ActivitiesFilter
          :id="id + '-activities-filter'"
          v-model="selectedActivities"
          :activities="activities"
          :facets="data.facets.field_activity_category"
          :multiple="!daxko"
          :exclude-by-category="excludeByCategory"
        />
      </Fieldset>

      <Fieldset
        :label="'Locations' | t"
        :collapse-id="id + '-toggle-locations'"
        :collapsed="fieldsetCollapseState('locations')"
        :counter="locationFiltersCount"
        :hide-counter="true"
      >
        <LocationsFilter
          :id="id + '-locations-filter'"
          v-model="selectedLocations"
          :locations="locations"
          :facets="data.facets.locations"
          :exclude-by-location="excludeByLocation"
        />
      </Fieldset>
    </div>
    <div class="filters-footer" :class="footerClasses">
      <div class="buttons">
        <div class="separator"></div>
        <button v-if="hasChanges" type="button" class="btn btn-lg btn-apply" @click="applyFilters">
          {{ 'Apply' | t }}
        </button>
        <button v-else type="button" class="btn btn-lg btn-clear" @click="clearFilters">
          {{ 'Clear filters' | t }}
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import Fieldset from '@/components/Fieldset.vue'
import AgesFilter from '@/components/filters/Ages.vue'
import DaysFilter from '@/components/filters/Days.vue'
import DaysTimesFilter from '@/components/filters/DaysTimes.vue'
import WeeksFilter from '@/components/filters/Weeks.vue'
import LocationsFilter from '@/components/filters/Locations.vue'
import ActivitiesFilter from '@/components/filters/Activities.vue'

export default {
  name: 'Filters',
  components: {
    Fieldset,
    AgesFilter,
    DaysFilter,
    DaysTimesFilter,
    WeeksFilter,
    LocationsFilter,
    ActivitiesFilter
  },
  props: {
    id: {
      type: String,
      default: 'mobile-filters'
    },
    data: {
      type: Object,
      required: true
    },
    ages: {
      type: Array,
      required: true
    },
    days: {
      type: Array,
      required: true
    },
    daysTimes: {
      type: Array,
      required: true
    },
    weeks: {
      type: Array,
      required: true
    },
    locations: {
      type: Array,
      required: true
    },
    activities: {
      type: Array,
      required: true
    },
    initialAges: {
      type: Array,
      required: true
    },
    initialDays: {
      type: Array,
      required: true
    },
    initialDaysTimes: {
      type: Array,
      required: true
    },
    initialWeeks: {
      type: Array,
      required: true
    },
    initialLocations: {
      type: Array,
      required: true
    },
    initialActivities: {
      type: Array,
      required: true
    },
    maxAges: {
      type: Number,
      required: true
    },
    legacyMode: {
      type: Boolean,
      required: true
    },
    weeksFilter: {
      type: Boolean,
      required: true
    },
    filtersMode: {
      type: String,
      default: 'accumulative' // 'instant'
    },
    filtersSectionConfig: {
      type: Object,
      required: true
    },
    daxko: {
      type: Boolean,
      default: false
    },
    excludeByCategory: {
      type: Array,
      required: true
    },
    excludeByLocation: {
      type: Array,
      required: true
    },
    bsVersion: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      selectedAges: this.initialAges,
      selectedDays: this.initialDays,
      selectedDaysTimes: this.initialDaysTimes,
      selectedWeeks: this.initialWeeks,
      selectedLocations: this.initialLocations,
      selectedActivities: this.initialActivities
    }
  },
  computed: {
    scheduleFiltersCount() {
      return (
        this.selectedAges.length +
        this.selectedDays.length +
        this.selectedDaysTimes.length +
        this.selectedWeeks.length
      )
    },
    activityFiltersCount() {
      return this.selectedActivities.length
    },
    locationFiltersCount() {
      return this.selectedLocations.length
    },
    hasChanges() {
      if (this.filtersMode === 'instant') {
        return false
      }

      return (
        !this.isEqual(this.selectedAges, this.initialAges) ||
        !this.isEqual(this.selectedDays, this.initialDays) ||
        !this.isEqual(this.selectedDaysTimes, this.initialDaysTimes) ||
        !this.isEqual(this.selectedWeeks, this.initialWeeks) ||
        !this.isEqual(this.selectedLocations, this.initialLocations) ||
        !this.isEqual(this.selectedActivities, this.initialActivities)
      )
    },
    headerClasses() {
      return this.bsVersion === 4 ? 'd-none d-lg-flex' : 'hidden-xs hidden-sm'
    },
    footerClasses() {
      return this.bsVersion === 4 ? ['d-lg-none'] : ['hidden-md', 'hidden-lg']
    }
  },
  watch: {
    initialAges() {
      this.selectedAges = this.initialAges
    },
    initialDays() {
      this.selectedDays = this.initialDays
    },
    initialDaysTimes() {
      this.selectedDaysTimes = this.initialDaysTimes
    },
    initialWeeks() {
      this.selectedWeeks = this.initialWeeks
    },
    initialLocations() {
      this.selectedLocations = this.initialLocations
    },
    initialActivities() {
      this.selectedActivities = this.initialActivities
    },
    selectedAges() {
      this.filterChange({ filter: 'selectedAges', value: this.selectedAges })
    },
    selectedDays() {
      this.filterChange({ filter: 'selectedDays', value: this.selectedDays })
    },
    selectedDaysTimes() {
      this.filterChange({ filter: 'selectedDaysTimes', value: this.selectedDaysTimes })
    },
    selectedWeeks() {
      this.filterChange({ filter: 'selectedWeeks', value: this.selectedWeeks })
    },
    selectedLocations() {
      this.filterChange({ filter: 'selectedLocations', value: this.selectedLocations })
    },
    selectedActivities() {
      this.filterChange({ filter: 'selectedActivities', value: this.selectedActivities })
    }
  },
  methods: {
    clearFilters() {
      this.$emit('clearFilters')
    },
    filterChange(filter) {
      if (this.filtersMode === 'instant') {
        this.$emit('filterChange', filter)
      }
    },
    applyFilters() {
      for (let key of ['Ages', 'Days', 'DaysTimes', 'Weeks', 'Locations', 'Activities']) {
        if (!this.isEqual(this['selected' + key], this['initial' + key])) {
          this.$emit('filterChange', { filter: 'selected' + key, value: this['selected' + key] })
        }
      }
    },
    isEqual(array1, array2) {
      return array1.length === array2.length && array1.every(value => array2.includes(value))
    },
    fieldsetCollapseState(type) {
      return this.filtersSectionConfig[type]
    }
  }
}
</script>

<style lang="scss">
.filters-component {
  .filters-header {
    display: flex;
    justify-content: space-between;
    margin-bottom: 20px;
  }

  .filters-footer {
    height: 140px;

    .buttons {
      position: fixed;
      bottom: 0;
      margin-left: -10px;
      margin-right: -10px;
      width: calc(100% - 2px);
      max-width: 358px;

      .separator {
        height: 5px;
        background-color: $white;
        opacity: 0.1;
      }

      .btn {
        width: 100%;
        font-weight: bold;
        font-size: 18px;
        line-height: 46px;
        padding: 0;

        &.btn-clear {
          background-color: $white;
          color: $af-blue;
          border: 2px solid $af-blue;
        }

        &.btn-apply {
          background-color: $af-violet;
          color: $white;
          border: none;
          line-height: 50px;
        }
      }
    }
  }
}
</style>
