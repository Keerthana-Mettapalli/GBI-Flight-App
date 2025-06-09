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
            @change="validateSelection"
            class="w-full p-2 border border-gray-300 rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500 text-gray-700 bg-white"
          >
            <option disabled value="">Select Source</option>
            <option v-for="loc in locations" :key="`source-${loc}`" :value="loc">
              {{ loc }}
            </option>
          </select>
        </div>

        <!-- Swap Button -->
        <div class="flex justify-center">
          <button
            @click="interchange"
            class="w-10 h-10 flex items-center justify-center rounded-full bg-blue-400 text-white text-xl shadow-md hover:bg-blue-600 transition"
            title="Swap Source & Destination"
          >
            🔁
          </button>
        </div>

        <!-- To Dropdown -->
        <div>
          <label class="block mb-2 text-sm font-semibold text-gray-800">To</label>
          <select
            v-model="selectedDestination"
            @change="validateSelection"
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

      <div v-if="error" class="text-red-600 text-center font-medium mt-2">
        {{ error }}
      </div>

      <div class="mt-4 text-end" v-if="filteredFlights.length > 0">
        <label class="inline-flex items-center space-x-2">
          <input
            type="checkbox"
            v-model="sortByDuration"
            class="accent-blue-600"
            :disabled="!!error"
          />
          <span class="text-sm text-gray-700">Sort by shortest duration</span>
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
export default {
  data() {
    return {
      flights: [],
      locations: [],
      selectedSource: '',
      selectedDestination: '',
      error: '',
      sortByDuration: false,
    }
  },
  computed: {
    filteredFlights() {
      let filtered = this.flights.filter(
        (flight) =>
          flight.segment[0].origin === this.selectedSource &&
          flight.segment[flight.segment.length - 1].destination === this.selectedDestination
      )

      if (this.sortByDuration) {
        filtered = [...filtered].sort((a, b) => a.duration - b.duration)
      }

      return filtered
    },
  },
  methods: {
    async fetchFlights() {
      const res = await fetch('/flights.json')
      const data = await res.json()
      this.flights = data

      const allLocations = new Set()
      data.forEach((flight) =>
        flight.segment.forEach((seg) => {
          allLocations.add(seg.origin)
          allLocations.add(seg.destination)
        })
      )
      this.locations = Array.from(allLocations)
    },
    validateSelection() {
      if (
        this.selectedSource &&
        this.selectedDestination &&
        this.selectedSource === this.selectedDestination
      ) {
        this.error = 'Source and destination cannot be the same.'
        setTimeout(() => {
          this.selectedSource = ''
          this.selectedDestination = ''
          this.error = ''
        }, 2000)
      } else {
        this.error = ''
      }
    },
    formatDate(dateStr) {
      const options = { dateStyle: 'medium', timeStyle: 'short' }
      return new Date(dateStr).toLocaleString(undefined, options)
    },
    interchange() {
      if (this.selectedSource && this.selectedDestination) {
        const temp = this.selectedSource
        this.selectedSource = this.selectedDestination
        this.selectedDestination = temp
        this.validateSelection()
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
