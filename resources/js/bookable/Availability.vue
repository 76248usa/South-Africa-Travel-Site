<template>
  <div>
    <h6 class="text-uppercase text-secondary font-weight-bolder">Check Availability</h6>
    <span v-if="noAvailability" class="text-danger">(NOT AVAILABLE)</span>
    <span v-if="hasAvailability" class="text-success">(AVAILABLE)</span>
    <div class="form-row">
      <div class="form-group col-md-6">
        <label for="from">From</label>
        <input
          type="text"
          name="from"
          class="form-control form-control-sm"
          placeholder="Start Date"
          v-model="from"
          @keyup.enter="check"
          :class="[{'is-invalid': this.errorFor('from')}]"
        />
        <div
          class="invalid-feedback"
          v-for="(error, index) in this.errorFor('from')"
          :key="'from' + index"
        >{{ error }}</div>
      </div>

      <div class="form-group col-md-6">
        <label for="to">To</label>
        <input
          type="text"
          name="to"
          class="form-control form-control-sm"
          placeholder="End Date"
          v-model="to"
          @keyup.enter="check"
          :class="[{'is-invalid': this.errorFor('to')}]"
        />
        <div
          class="invalid-feedback"
          v-for="(error, index) in this.errorFor('to')"
          :key="'to' + index"
        >{{ error }}</div>
      </div>

      <button class="btn btn-secondary btn-block" @click="check" :disabled="loading">Check!</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      loading: false,
      from: null,
      to: null,
      status: null,
      errors: null
    };
  },
  methods: {
    check() {
      this.loading = true;
      this.errors = null;
      axios
        .get(
          `/api/bookables/${this.$route.params.id}/availability?from=${this.from}&to=${this.to}`
        )
        .then(response => {
          this.status = response.status;
        })
        .catch(error => {
          if (422 == error.response.status) {
            this.errors = error.response.data.errors;
          }
          this.status = error.response.status;
        })
        .then(() => (this.loading = false));
    },
    errorFor(field) {
      return this.hasErrors && this.errors[field] ? this.errors[field] : null;
    }
  },
  computed: {
    hasErrors() {
      return 422 === this.status && this.errors !== null;
    },
    hasAvailability() {
      return 200 === this.status;
    },
    noAvailability() {
      return 404 === this.status;
    }
  }
};
</script>

<style scoped>
label {
  font-size: 0.7em;
  text-transform: uppercase;
  color: gray;
  font-weight: bolder;
}
</style>
