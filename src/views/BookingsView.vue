<script setup>
import { ref, onMounted, watch } from "vue";

const search = ref("");

watch(() => search.value, () => {
  loadAsyncData();
});

const data = ref([]);
const total = ref(0);
const loading = ref(false);
const sortField = ref("");
const sortOrder = ref("desc");
const page = ref(1);
const perPage = ref(20);

const loadAsyncData = () => {
  const params = [
    // "api_key=bb6f51bef07465653c3e553d6ab161a8",
    // "language=en-US",
    // "include_adult=false",
    // "include_video=false",
    `sort_by=${sortField.value}.${sortOrder.value}`,
    `email=${search.value}`,
    `page=${page.value}`,
  ].join("&");
  loading.value = true;
  fetch(`/api/bookings?${params}`)
    .then((response) => response.json())
    .then((result) => {
      perPage.value = result.perPage;
      total.value = result.total;
      data.value = result.bookings

      loading.value = false;
    })
};

/*
 * Handle page-change event
 */
const onPageChange = (p) => {
  page.value = p;
  loadAsyncData();
};

/*
 * Handle sort event
 */
const onSort = (column, order) => {
  sortField.value = column.field;
  sortOrder.value = order;
  loadAsyncData();
};

/*
 * Type style in relation to the value
 */
const type = (value) => {
  const number = parseFloat(value);
  if (number < 6) {
    return "is-danger";
  } else if (number >= 6 && number < 8) {
    return "is-warning";
  } else if (number >= 8) {
    return "is-success";
  }
};

onMounted(() => {
  loadAsyncData();
});
</script>

<template>

  <section>
    <input class="form-control" v-model="search" placeholder="Search..." />
    <o-table :data="data" :loading="loading" paginated backend-pagination :total="total" :per-page="perPage"
      backend-sorting :default-sort="[sortField, sortOrder]" aria-next-label="Next page"
      aria-previous-label="Previous page" aria-page-label="Page" aria-current-label="Current page"
      @page-change="onPageChange" @sort="onSort">
      <o-table-column v-slot="{ row }" field="email" label="Email" sortable>
        <RouterLink :to="`/booking/${row._id}`"> {{ row.email }} </RouterLink>
      </o-table-column>
      <o-table-column v-slot="{ row }" field="numTickets" label="NumTickets" numeric sortable>
        <span class="tag" :class="type(row.numTickets)">
          {{ row.numTickets }}
        </span>
      </o-table-column>
      <o-table-column v-slot="{ row }" field="superhero" label="Superhero" numeric sortable>
        {{ row.superhero }}
      </o-table-column>
      <o-table-column v-slot="{ row }" field="modified_at" label="Last Modified At" sortable centered>
        {{
          row.modified_at
            ? new Date(row.modified_at).toLocaleDateString()
            : "unknown"
        }}
      </o-table-column>
      <o-table-column v-slot="{ row }" label="Terms" width="500">
        {{ row.terms }}
      </o-table-column>
    </o-table>
  </section>

</template>
