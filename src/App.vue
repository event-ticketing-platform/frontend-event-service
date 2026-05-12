<script setup>
import { computed, onMounted, ref } from 'vue';
import EventList from './components/EventList.vue';

const ticketTypes = [
  { id: 10, name: 'Standard ticket', price: 25, note: 'Entry to the main hall' },
  { id: 11, name: 'VIP ticket', price: 45, note: 'Front rows and welcome drink' }
];

const selectedTicketId = ref(ticketTypes[0].id);
const quantity = ref(1);
const bookings = ref([]);
const status = ref('Choose a ticket type and book it.');
const state = ref('idle');
const loadingBookings = ref(false);
const activeTab = ref('booking');

const navItems = [
  { key: 'booking', label: 'Book event' },
  { key: 'bookings', label: 'All bookings' },
  { key: 'events', label: 'Events' }
];

const selectedTicket = computed(() => ticketTypes.find((ticket) => ticket.id === selectedTicketId.value) ?? ticketTypes[0]);
const total = computed(() => selectedTicket.value.price * quantity.value);

async function fetchJson(url, options) {
  const response = await fetch(url, options);

  if (!response.ok) {
    throw new Error(`${url} returned HTTP ${response.status}`);
  }

  return response.json();
}

async function loadBookings() {
  loadingBookings.value = true;

  try {
    const payload = await fetchJson('/api/demo/booking-table-data');
    bookings.value = payload.rows ?? [];
  } finally {
    loadingBookings.value = false;
  }
}

async function bookTicket() {
  state.value = 'loading';
  status.value = 'Creating booking first, then starting payment...';

  const request = {
    customerId: 1,
    eventId: 100,
    currency: 'EUR',
    items: [
      {
        ticketTypeId: selectedTicket.value.id,
        quantity: Number(quantity.value),
        unitPrice: selectedTicket.value.price
      }
    ]
  };

  try {
    const payload = await fetchJson('/api/demo/book-selected', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(request)
    });

    bookings.value = payload.rows ?? [];
    state.value = 'success';
    status.value = `Booked ${selectedTicket.value.name}. Booking #${payload.createdBooking.bookingId}, payment #${payload.createdBooking.paymentId}.`;
  } catch (error) {
    state.value = 'error';
    status.value = `Booking failed: ${error.message}`;
  }
}

onMounted(async () => {
  try {
    await loadBookings();
  } catch {
    status.value = 'Bookings will appear here after the first successful booking.';
  }
});
</script>

<template>
  <div class="app">
    <header class="appbar">
      <div>
        <strong>Event Ticketing and Venue Management Service</strong>
      </div>
    </header>

    <nav class="tabs" aria-label="Demo sections">
      <button
        v-for="item in navItems"
        :key="item.key"
        type="button"
        :class="{ active: activeTab === item.key }"
        @click="activeTab = item.key"
      >
        {{ item.label }}
      </button>
    </nav>

    <main v-if="activeTab === 'booking'" class="content">
      <section class="event-panel">
        <h1>Tartu Tech Night 2026</h1>
        <p class="details">Delta Centre, Friday 19:00</p>

        <h2>Choose ticket type</h2>

        <div class="tickets">
          <label
            v-for="ticket in ticketTypes"
            :key="ticket.id"
            :class="{ selected: selectedTicketId === ticket.id }"
          >
            <input
              v-model="selectedTicketId"
              type="radio"
              :value="ticket.id"
            />

            <span>
              <strong>{{ ticket.name }}</strong>
              <small>{{ ticket.note }}</small>
            </span>

            <b>{{ ticket.price }} EUR</b>
          </label>
        </div>

        <label class="quantity">
          Quantity

          <input
            v-model="quantity"
            type="number"
            min="1"
            max="8"
          />
        </label>

        <div class="summary">
          <span>Total</span>
          <strong>{{ total }} EUR</strong>
        </div>

        <button
          class="book-button"
          type="button"
          :disabled="state === 'loading'"
          @click="bookTicket"
        >
          {{ state === 'loading' ? 'Booking...' : 'Book Ticket' }}
        </button>

        <p class="status" :class="state">
          {{ status }}
        </p>
      </section>

      <section class="bookings-panel">
        <div class="panel-title">
          <div>
            <h2>All bookings</h2>
            <p>Loaded from the booking and payment services.</p>
          </div>

          <strong>{{ bookings.length }}</strong>
        </div>

        <button
          class="refresh-button"
          type="button"
          :disabled="loadingBookings"
          @click="loadBookings"
        >
          {{ loadingBookings ? 'Loading...' : 'Refresh bookings' }}
        </button>

        <div v-if="!bookings.length" class="empty">
          No bookings loaded yet.
        </div>

        <div
          v-for="booking in bookings"
          :key="booking.bookingId"
          class="booking-card"
        >
          <div>
            <strong>Booking #{{ booking.bookingId }}</strong>

            <span>
              {{ booking.totalAmount }}
              {{ booking.currency }}
              ·
              {{ booking.bookingStatus }}
            </span>
          </div>

          <div>
            <small>Payment</small>

            <b>
              #{{ booking.paymentId }}
              ·
              {{ booking.paymentStatus }}
            </b>
          </div>
        </div>
      </section>
    </main>

    <main
      v-else-if="activeTab === 'bookings'"
      class="single-content"
    >
      <section class="bookings-panel">
        <div class="panel-title">
          <div>
            <h1>All bookings</h1>
            <p>Every row comes from the current database.</p>
          </div>

          <strong>{{ bookings.length }}</strong>
        </div>

        <button
          class="refresh-button"
          type="button"
          :disabled="loadingBookings"
          @click="loadBookings"
        >
          {{ loadingBookings ? 'Loading...' : 'Refresh bookings' }}
        </button>

        <div v-if="!bookings.length" class="empty">
          No bookings loaded yet.
        </div>

        <div
          v-for="booking in bookings"
          :key="booking.bookingId"
          class="booking-card"
        >
          <div>
            <strong>Booking #{{ booking.bookingId }}</strong>

            <span>
              {{ booking.totalAmount }}
              {{ booking.currency }}
              ·
              {{ booking.bookingStatus }}
            </span>
          </div>

          <div>
            <small>Payment</small>

            <b>
              #{{ booking.paymentId }}
              ·
              {{ booking.paymentStatus }}
            </b>
          </div>
        </div>
      </section>
    </main>

    <main
      v-else-if="activeTab === 'events'"
      class="single-content"
    >
      <EventList />
    </main>
  </div>
</template>