<script setup>
import { computed, reactive, ref, watch } from 'vue'

const PROFILE_KEY = 'foodieapp-profile'
const VISITS_KEY = 'foodieapp-visits'

const loadStoredValue = (key, fallback) => {
  try {
    const storedValue = localStorage.getItem(key)
    return storedValue ? JSON.parse(storedValue) : fallback
  } catch {
    return fallback
  }
}

const profile = reactive({
  firstName: '',
  lastName: '',
  email: '',
  favoriteFood: '',
  ...loadStoredValue(PROFILE_KEY, {}),
})

const profileSaved = ref(Boolean(profile.firstName && profile.email))
const visits = ref(loadStoredValue(VISITS_KEY, []))
const editingVisitId = ref(null)

const visitForm = reactive({
  name: '',
  category: 'American',
  favoriteDish: '',
  rating: 5,
  visitedAt: new Date().toISOString().slice(0, 10),
  notes: '',
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

const averageRating = computed(() => {
  if (!visits.value.length) return '0.0'

  const total = visits.value.reduce((sum, visit) => sum + Number(visit.rating || 0), 0)
  return (total / visits.value.length).toFixed(1)
})

const topCategory = computed(() => {
  if (!visits.value.length) return 'None'

  const categoryCounts = visits.value.reduce((counts, visit) => {
    counts[visit.category] = (counts[visit.category] || 0) + 1
    return counts
  }, {})

  return Object.entries(categoryCounts).sort((first, second) => second[1] - first[1])[0][0]
})

watch(
  profile,
  () => {
    localStorage.setItem(PROFILE_KEY, JSON.stringify(profile))
  },
  { deep: true },
)

watch(
  visits,
  () => {
    localStorage.setItem(VISITS_KEY, JSON.stringify(visits.value))
  },
  { deep: true },
)

const saveProfile = () => {
  profileSaved.value = true
}

const resetVisitForm = () => {
  editingVisitId.value = null
  visitForm.name = ''
  visitForm.category = 'American'
  visitForm.favoriteDish = ''
  visitForm.rating = 5
  visitForm.visitedAt = new Date().toISOString().slice(0, 10)
  visitForm.notes = ''
}

const saveVisit = () => {
  const visitDetails = {
    name: visitForm.name.trim(),
    category: visitForm.category,
    favoriteDish: visitForm.favoriteDish.trim(),
    rating: Number(visitForm.rating),
    visitedAt: visitForm.visitedAt,
    notes: visitForm.notes.trim(),
  }

  if (editingVisitId.value) {
    visits.value = visits.value.map((visit) => {
      return visit.id === editingVisitId.value ? { ...visit, ...visitDetails } : visit
    })
  } else {
    visits.value.unshift({
      id: Date.now(),
      ...visitDetails,
    })
  }

  resetVisitForm()
}

const editVisit = (visit) => {
  editingVisitId.value = visit.id
  visitForm.name = visit.name
  visitForm.category = visit.category
  visitForm.favoriteDish = visit.favoriteDish
  visitForm.rating = visit.rating
  visitForm.visitedAt = visit.visitedAt
  visitForm.notes = visit.notes
}

const deleteVisit = (visitId) => {
  if (editingVisitId.value === visitId) {
    resetVisitForm()
  }

  visits.value = visits.value.filter((visit) => visit.id !== visitId)
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
          <p class="mt-1 text-sm text-slate-600">
            {{ visits.length }} visits captured in this session.
          </p>
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
          <h2 class="section-title">
            {{ editingVisitId ? 'Edit saved visit' : 'Restaurant and dish form' }}
          </h2>
          <p class="section-copy">
            {{
              editingVisitId
                ? 'Update the saved details for this local food visit.'
                : 'Save the local restaurant, favorite dish, rating, and notes from a visit.'
            }}
          </p>

          <form
            class="mt-6 space-y-4"
            @submit.prevent="saveVisit"
          >
            <label class="form-field">
              <span>Restaurant or Place</span>
              <input
                v-model="visitForm.name"
                type="text"
                required
                placeholder="Example: Taco Fiesta"
              />
            </label>

            <div class="grid gap-4 sm:grid-cols-2">
              <label class="form-field">
                <span>Category</span>
                <select v-model="visitForm.category">
                  <option>American</option>
                  <option>Italian</option>
                  <option>Mexican</option>
                  <option>Japanese</option>
                  <option>Chinese</option>
                  <option>Dessert</option>
                  <option>Coffee</option>
                  <option>Other</option>
                </select>
              </label>

              <label class="form-field">
                <span>Rating</span>
                <select v-model="visitForm.rating">
                  <option :value="5">5 - Loved it</option>
                  <option :value="4">4 - Great</option>
                  <option :value="3">3 - Good</option>
                  <option :value="2">2 - Okay</option>
                  <option :value="1">1 - Skip next time</option>
                </select>
              </label>
            </div>

            <label class="form-field">
              <span>Favorite Dish</span>
              <input
                v-model="visitForm.favoriteDish"
                type="text"
                required
                placeholder="Example: Street tacos"
              />
            </label>

            <label class="form-field">
              <span>Date Visited</span>
              <input
                v-model="visitForm.visitedAt"
                type="date"
              />
            </label>

            <label class="form-field">
              <span>Notes</span>
              <textarea
                v-model="visitForm.notes"
                rows="4"
                placeholder="What should you remember about this place?"
              ></textarea>
            </label>

            <button
              type="submit"
              class="primary-button"
            >
              {{ editingVisitId ? 'Save Changes' : 'Add Visit' }}
            </button>

            <button
              v-if="editingVisitId"
              type="button"
              class="secondary-button w-full"
              @click="resetVisitForm"
            >
              Cancel Edit
            </button>
          </form>
        </section>
      </div>

      <section class="app-panel">
        <p class="section-label">Dashboard</p>
        <h2 class="section-title">Saved restaurant visits</h2>
        <div class="mt-6 grid gap-4 sm:grid-cols-3">
          <div class="stat-card">
            <span>{{ visits.length }}</span>
            <p>Restaurants</p>
          </div>
          <div class="stat-card">
            <span>{{ averageRating }}</span>
            <p>Average Rating</p>
          </div>
          <div class="stat-card">
            <span>{{ topCategory }}</span>
            <p>Top Category</p>
          </div>
        </div>

        <div
          v-if="!visits.length"
          class="mt-6 rounded-2xl border border-dashed border-orange-200 bg-orange-50 p-6 text-sm font-medium text-orange-800"
        >
          Saved restaurant cards will appear here after users add local food spots.
        </div>

        <div
          v-else
          class="mt-6 space-y-4"
        >
          <article
            v-for="visit in visits"
            :key="visit.id"
            class="visit-card"
          >
            <div class="flex flex-col gap-3 sm:flex-row sm:items-start sm:justify-between">
              <div>
                <div class="flex flex-wrap items-center gap-2">
                  <h3 class="text-xl font-bold text-slate-950">{{ visit.name }}</h3>
                  <span class="category-pill">{{ visit.category }}</span>
                </div>
                <p class="mt-2 text-sm font-semibold text-slate-600">
                  {{ visit.favoriteDish }} · {{ visit.rating }}/5 · {{ visit.visitedAt }}
                </p>
                <p
                  v-if="visit.notes"
                  class="mt-3 text-sm leading-relaxed text-slate-500"
                >
                  {{ visit.notes }}
                </p>
              </div>

              <div class="flex gap-2">
                <button
                  type="button"
                  class="small-action"
                  @click="editVisit(visit)"
                >
                  Edit
                </button>
                <button
                  type="button"
                  class="small-action danger"
                  @click="deleteVisit(visit.id)"
                >
                  Delete
                </button>
              </div>
            </div>
          </article>
        </div>
      </section>
    </section>
  </main>
</template>
