<template>
  <div class="min-h-screen bg-gradient-to-r from-blue-50 to-white p-4">
    <div class="max-w-3xl mx-auto bg-white rounded-lg shadow-lg p-6">
      <h1 class="text-2xl font-extrabold text-center text-blue-700 mb-6">✈️ Flight Search</h1>

      <div class="grid grid-cols-[1fr_auto_1fr] gap-6 items-end mb-6">
        <!-- From Dropdown -->
        <div>
          <label class="block mb-2 text-sm font-semibold text-gray-800">From</label>
          <select
            v-model="selectedSource"
            class="w-full p-2 border border-gray-300 rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500 text-gray-700 bg-white"
          >
            <option disabled value="">Select Source</option>
            <option
              v-for="loc in locations"
              :key="`source-${loc}`"
              :value="loc"
              :disabled="loc === selectedDestination"
            >
              {{ loc }}
            </option>
          </select>
        </div>

        <div class="flex justify-center">
          <button
            @click="interchange"
            class="w-10 h-10 flex items-center justify-center rounded-full bg-white text-black border border-gray-300 shadow-md hover:bg-gray-100 transition"
            title="Swap Source & Destination"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 512 512"
              fill="currentColor"
              class="w-5 h-5"
            >
              <path
                d="M503.7 345.4L392.6 456.5c-10 10-27.3 2.9-27.3-11.3v-70.1H80c-13.3 0-24-10.7-24-24v-16c0-13.3 10.7-24 24-24h285.3v-70.1c0-14.2 17.3-21.4 27.3-11.3l111.1 111.1c6.6 6.6 6.6 17.4 0 24zM8.3 166.6L119.4 55.5c10-10 27.3-2.9 27.3 11.3v70.1H432c13.3 0 24 10.7 24 24v16c0 13.3-10.7 24-24 24H146.7v70.1c0 14.2-17.3 21.4-27.3 11.3L8.3 190.6c-6.6-6.6-6.6-17.4 0-24z"
              />
            </svg>
          </button>
        </div>

        <!-- To Dropdown -->
        <div>
          <label class="block mb-2 text-sm font-semibold text-gray-800">To</label>
          <select
            v-model="selectedDestination"
            class="w-full p-2 border border-gray-300 rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500 text-gray-700 bg-white"
          >
            <option disabled value="">Select Destination</option>
            <option
              v-for="loc in locations"
              :key="`dest-${loc}`"
              :value="loc"
              :disabled="loc === selectedSource"
            >
              {{ loc }}
            </option>
          </select>
        </div>
      </div>

      <div class="mt-4 text-end" v-if="filteredFlights.length > 0">
        <label class="inline-flex items-center space-x-2">
          <input type="checkbox" v-model="sortByDuration" class="accent-blue-600" />
          <span class="text-sm text-gray-700">Sort by duration (ascending)</span>
        </label>
      </div>

      <div class="mt-6">
        <div
          v-if="filteredFlights.length === 0 && selectedSource && selectedDestination"
          class="text-center text-gray-500 text-lg"
        >
          🚫 No flights available for this route.
        </div>

        <div v-else>
          <p class="text-sm text-gray-600 mb-2">
            Showing {{ filteredFlights.length }} result{{ filteredFlights.length > 1 ? 's' : '' }}
          </p>
          <div class="grid gap-4">
            <div
              v-for="flight in filteredFlights"
              :key="flight.id"
              class="border border-gray-200 hover:shadow-md transition-shadow p-4 rounded-lg bg-white"
            >
              <h2 class="text-lg font-bold text-blue-600 mb-2">
                {{ flight.company }}
              </h2>
              <div v-for="(segment, index) in flight.segment" :key="index" class="text-sm mb-1">
                <p>
                  <strong>{{ segment.origin }}</strong> →
                  <strong>{{ segment.destination }}</strong>
                </p>
                <p class="text-gray-600">Departure: {{ formatDate(segment.departureTime) }}</p>
                <p class="text-gray-600">Arrival: {{ formatDate(segment.arrivalTime) }}</p>
              </div>
              <p class="mt-2 text-sm font-semibold text-gray-800">
                Total Duration: {{ flight.duration }} mins
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { DateTime } from 'luxon'
export default {
  data() {
    return {
      flights: [],
      locations: [],
      selectedSource: '',
      selectedDestination: '',
      sortByDuration: false,
    }
  },
  computed: {
    filteredFlights() {
      let filtered = this.flights.filter((flight) =>
        flight.segment.some(
          (seg) =>
            seg.origin === this.selectedSource && seg.destination === this.selectedDestination
        )
      )
      if (this.sortByDuration) {
        filtered = [...filtered].sort((a, b) => a.duration - b.duration)
      }
      console.log(filtered, 'filtered')
      return filtered
    },
  },
  methods: {
    async fetchFlights() {
      const res = await fetch('./flights.json')
      const data = await res.json()
      this.flights = data
      const allLocations = new Set()
      data.forEach((flight) => {
        flight.segment.forEach((seg) => {
          allLocations.add(seg.origin)
          allLocations.add(seg.destination)
        })
      })
      this.locations = Array.from(allLocations)
    },
    formatDate(dateStr) {
      return DateTime.fromISO(dateStr).toFormat('dd LLL yyyy, hh:mm a')
    },
    interchange() {
      if (this.selectedSource && this.selectedDestination) {
        const temp = this.selectedSource
        this.selectedSource = this.selectedDestination
        this.selectedDestination = temp
      }
    },
  },
  mounted() {
    this.fetchFlights()
  },
}
</script>

<style scoped>
@media (max-width: 640px) {
  select {
    font-size: 0.875rem;
    padding: 0.5rem;
  }

  option {
    font-size: 0.875rem;
    padding: 0.25rem 0.5rem;
  }
}
</style>
