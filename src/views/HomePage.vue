<script setup>
import { computed, reactive, ref } from 'vue'

const profileSaved = ref(false)

const profile = reactive({
  firstName: '',
  lastName: '',
  email: '',
  favoriteFood: '',
})

const profileName = computed(() => {
  const fullName = `${profile.firstName} ${profile.lastName}`.trim()
  return fullName || 'Foodie Guest'
})

const profileInitials = computed(() => {
  const firstInitial = profile.firstName.trim().charAt(0)
  const lastInitial = profile.lastName.trim().charAt(0)
  return `${firstInitial}${lastInitial}` || 'F'
})

const saveProfile = () => {
  profileSaved.value = true
}
</script>

<template>
  <main>
    <header class="border-b border-orange-100 bg-white">
      <div class="mx-auto flex max-w-6xl flex-col gap-4 px-6 py-6 sm:flex-row sm:items-center sm:justify-between lg:px-10">
        <div>
          <p class="text-sm font-semibold uppercase text-orange-600">FoodieApp</p>
          <h1 class="mt-2 text-4xl font-bold tracking-tight text-slate-950">Local food tracker</h1>
          <p class="mt-3 max-w-2xl text-slate-600">
            Save restaurants, favorite dishes, ratings, and notes so good local food spots are easy to remember.
          </p>
        </div>

        <div class="rounded-2xl border border-orange-100 bg-orange-50 px-5 py-4">
          <p class="text-sm font-semibold text-orange-700">MVP Loop</p>
          <p class="mt-1 text-sm text-slate-600">Add, review, edit, and save local food visits.</p>
        </div>
      </div>
    </header>

    <section class="mx-auto grid max-w-6xl gap-6 px-6 py-8 lg:grid-cols-[0.9fr_1.1fr] lg:px-10">
      <div class="space-y-6">
        <section
          v-if="!profileSaved"
          class="app-panel"
        >
          <p class="section-label">Profile</p>
          <h2 class="section-title">Create your Foodie profile</h2>
          <p class="section-copy">Start by saving the basic info that will personalize the dashboard.</p>

          <form
            class="mt-6 space-y-4"
            @submit.prevent="saveProfile"
          >
            <div class="grid gap-4 sm:grid-cols-2">
              <label class="form-field">
                <span>First Name</span>
                <input
                  v-model="profile.firstName"
                  type="text"
                  required
                  placeholder="Chris"
                />
              </label>

              <label class="form-field">
                <span>Last Name</span>
                <input
                  v-model="profile.lastName"
                  type="text"
                  placeholder="Contreras"
                />
              </label>
            </div>

            <label class="form-field">
              <span>Email</span>
              <input
                v-model="profile.email"
                type="email"
                required
                placeholder="grader@example.com"
              />
            </label>

            <label class="form-field">
              <span>Favorite Food</span>
              <input
                v-model="profile.favoriteFood"
                type="text"
                placeholder="Tacos, pizza, sushi..."
              />
            </label>

            <button
              type="submit"
              class="primary-button"
            >
              Save Profile
            </button>
          </form>
        </section>

        <section
          v-else
          class="app-panel"
        >
          <p class="section-label">Profile</p>
          <div class="mt-3 flex items-center gap-4">
            <div class="flex h-16 w-16 items-center justify-center rounded-full bg-orange-100 text-xl font-black text-orange-700">
              {{ profileInitials }}
            </div>
            <div>
              <h2 class="section-title">{{ profileName }}</h2>
              <p class="section-copy mt-1">
                Favorite food: {{ profile.favoriteFood || 'Still deciding' }}
              </p>
            </div>
          </div>

          <button
            type="button"
            class="secondary-button mt-6"
            @click="profileSaved = false"
          >
            Edit Profile
          </button>
        </section>

        <section class="app-panel">
          <p class="section-label">Add Restaurant</p>
          <h2 class="section-title">Restaurant and dish form</h2>
          <p class="section-copy">
            This section will let users add restaurant details, favorite dishes, ratings, visit dates, and notes.
          </p>
        </section>
      </div>

      <section class="app-panel">
        <p class="section-label">Dashboard</p>
        <h2 class="section-title">Saved restaurant visits</h2>
        <div class="mt-6 grid gap-4 sm:grid-cols-3">
          <div class="stat-card">
            <span>0</span>
            <p>Restaurants</p>
          </div>
          <div class="stat-card">
            <span>0.0</span>
            <p>Average Rating</p>
          </div>
          <div class="stat-card">
            <span>None</span>
            <p>Top Category</p>
          </div>
        </div>

        <div class="mt-6 rounded-2xl border border-dashed border-orange-200 bg-orange-50 p-6 text-sm font-medium text-orange-800">
          Saved restaurant cards will appear here after users add local food spots.
        </div>
      </section>
    </section>
  </main>
</template>
