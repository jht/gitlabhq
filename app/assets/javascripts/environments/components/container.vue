<script>
import { GlLoadingIcon } from '@gitlab/ui';
import tablePagination from '../../vue_shared/components/table_pagination.vue';
import environmentTable from '../components/environments_table.vue';

export default {
  components: {
    environmentTable,
    tablePagination,
    GlLoadingIcon,
  },
  props: {
    isLoading: {
      type: Boolean,
      required: true,
    },
    environments: {
      type: Array,
      required: true,
    },
    pagination: {
      type: Object,
      required: true,
    },
    canCreateDeployment: {
      type: Boolean,
      required: true,
    },
    canReadEnvironment: {
      type: Boolean,
      required: true,
    },
  },
  methods: {
    onChangePage(page) {
      this.$emit('onChangePage', page);
    },
  },
};
</script>

<template>
  <div class="environments-container">

    <gl-loading-icon
      v-if="isLoading"
      :size="3"
      class="prepend-top-default"
      label="Loading environments"
    />

    <slot name="emptyState"></slot>

    <div
      v-if="!isLoading && environments.length > 0"
      class="table-holder">

      <environment-table
        :environments="environments"
        :can-create-deployment="canCreateDeployment"
        :can-read-environment="canReadEnvironment"
      />

      <table-pagination
        v-if="pagination && pagination.totalPages > 1"
        :change="onChangePage"
        :page-info="pagination"
      />
    </div>
  </div>
</template>
