<script setup>
definePageMeta({ middleware: ['sanctum:auth'] })

const { user, logout } = useSanctumAuth()

const profile = computed(() => user.value || null)
const loading = computed(() => !profile.value)

function initials(name) {
  if (!name) return 'U'
  const parts = name.trim().split(/\s+/)
  return (parts[0]?.[0] || 'U') + (parts[1]?.[0] || '')
}

function formatDate(iso) {
  if (!iso) return '—'
  try {
    return new Date(iso).toLocaleString('en-US', {
      timeZone: 'Africa/Kampala',
      year: 'numeric',
      month: 'short',
      day: '2-digit',
      hour: '2-digit',
      minute: '2-digit'
    })
  } catch (e) {
    return iso
  }
}

const copied = ref(false)
async function copy(text) {
  try {
    await navigator.clipboard.writeText(String(text))
    copied.value = true
    setTimeout(() => (copied.value = false), 1500)
  } catch (e) {}
}

async function signOut() {
  await logout()
  return navigateTo('/login')
}
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br from-emerald-600 via-teal-600 to-slate-900 text-slate-100">
    <div class="mx-auto max-w-3xl px-6 py-14">
      <div class="rounded-2xl border border-white/10 bg-white/10 backdrop-blur-xl shadow-2xl overflow-hidden">
        <!-- Header -->
        <div class="relative">
          <div class="h-28 bg-gradient-to-r from-emerald-500/80 to-teal-500/80"></div>
          <div class="-mt-12 flex items-end gap-4 px-6">
            <div class="h-24 w-24 shrink-0 rounded-2xl bg-white/90 ring-4 ring-white/40 text-slate-800 grid place-items-center text-3xl font-bold">
              {{ initials(profile?.name) }}
            </div>
            <div class="pb-3">
              <h1 class="text-2xl font-semibold">
                <span v-if="!loading">{{ profile?.name }}</span>
                <span v-else class="inline-block h-7 w-48 animate-pulse rounded bg-white/30"></span>
              </h1>
              <p class="text-white/80">
                <span v-if="!loading">{{ profile?.email }}</span>
                <span v-else class="mt-2 inline-block h-5 w-56 animate-pulse rounded bg-white/20"></span>
              </p>
            </div>

            <div class="ml-auto pb-3">
              <span v-if="!loading" class="inline-flex items-center gap-2 rounded-full bg-white/15 px-3 py-1 text-sm">
                <svg v-if="profile?.email_verified_at" xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" viewBox="0 0 24 24" fill="currentColor"><path d="M9 16.2 4.8 12l1.4-1.4L9 13.4l8.8-8.8L19.2 6z"/></svg>
                <svg v-else xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" viewBox="0 0 24 24" fill="currentColor"><path d="M11 15h2v2h-2v-2zm0-8h2v6h-2V7zm1-7C5.48 0 1 4.48 1 10s4.48 10 10 10 10-4.48 10-10S16.52 0 12 0zm0 18c-4.41 0-8-3.59-8-8s3.59-8 8-8 8 3.59 8 8-3.59 8-8 8z"/></svg>
                <span>{{ profile?.email_verified_at ? 'Verified' : 'Not verified' }}</span>
              </span>
            </div>
          </div>
        </div>

        <!-- Body -->
        <div class="px-6 pb-6">
          <div class="mt-6 grid grid-cols-1 gap-4 sm:grid-cols-2">
            <div class="rounded-xl border border-white/10 bg-white/5 p-4">
              <p class="text-xs uppercase tracking-wider text-white/70">User ID</p>
              <div class="mt-1 flex items-center gap-2">
                <code class="truncate text-white/95">{{ loading ? '••••••••••' : profile?.id }}</code>
                <button
                  v-if="!loading"
                  @click="copy(profile?.id)"
                  class="ml-auto inline-flex items-center gap-1 rounded-md bg-white/15 px-2 py-1 text-xs hover:bg-white/25 transition">
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-3.5 w-3.5" viewBox="0 0 24 24" fill="currentColor"><path d="M16 1H4c-1.1 0-2 .9-2 2v12h2V3h12V1zm3 4H8c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h11c1.1 0 2-.9 2-2V7c0-1.1-.9-2-2-2zm0 16H8V7h11v14z"/></svg>
                  <span>{{ copied ? 'Copied' : 'Copy' }}</span>
                </button>
              </div>
            </div>

            <div class="rounded-xl border border-white/10 bg-white/5 p-4">
              <p class="text-xs uppercase tracking-wider text-white/70">Created</p>
              <p class="mt-1 text-white/95">
                <span v-if="!loading">{{ formatDate(profile?.created_at) }}</span>
                <span v-else class="inline-block h-5 w-40 animate-pulse rounded bg-white/20"></span>
              </p>
            </div>

            <div class="rounded-xl border border-white/10 bg-white/5 p-4">
              <p class="text-xs uppercase tracking-wider text-white/70">Updated</p>
              <p class="mt-1 text-white/95">
                <span v-if="!loading">{{ formatDate(profile?.updated_at) }}</span>
                <span v-else class="inline-block h-5 w-40 animate-pulse rounded bg-white/20"></span>
              </p>
            </div>

            <div class="rounded-xl border border-white/10 bg-white/5 p-4">
              <p class="text-xs uppercase tracking-wider text-white/70">Email Verified At</p>
              <p class="mt-1 text-white/95">
                <span v-if="!loading">{{ formatDate(profile?.email_verified_at) }}</span>
                <span v-else class="inline-block h-5 w-40 animate-pulse rounded bg-white/20"></span>
              </p>
            </div>
          </div>

          <div class="mt-8 flex flex-wrap items-center gap-3">
            <NuxtLink
              to="/"
              class="inline-flex items-center justify-center rounded-xl bg-white text-slate-900 px-4 py-2 font-medium shadow hover:shadow-md transition">
              Go to Dashboard
            </NuxtLink>

            <button
              @click="signOut"
              class="inline-flex items-center justify-center rounded-xl border border-white/20 bg-white/10 px-4 py-2 font-medium text-white hover:bg-white/20 transition">
              Sign out
            </button>
          </div>
        </div>
      </div>

      <p class="mt-6 text-center text-sm text-white/70">
        Session secured via Sanctum • Times shown in Africa/Kampala
      </p>
    </div>
  </div>
</template>
