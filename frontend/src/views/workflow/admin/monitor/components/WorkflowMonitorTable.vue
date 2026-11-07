<script setup lang="ts">
import { computed, toRef } from 'vue';

import { IconifyIcon } from '@vben/icons';

import { Button, Space } from 'ant-design-vue';

import type { AdminMonitorInstanceInfo } from '#/api/workflow';
import type { TablePaginationConfig } from '#/composables/Table';

import { BaseTable } from '#/components/BaseTable';
import { permissionCodes } from '#/constants/permissions';

import { getWorkflowMonitorColumns } from '../hooks/useWorkflowMonitorColumns';

interface Props {
  activeFilters: Record<string, any>;
  archivingByQuery?: boolean;
  archivingSelected?: boolean;
  dataSource: AdminMonitorInstanceInfo[];
  loading: boolean;
  pagination: TablePaginationConfig;
  scopeTitle: string;
  selectedRowKeys: (number | string)[];
}

const props = defineProps<Props>();
const emit = defineEmits<{
  archive: [record: AdminMonitorInstanceInfo];
  archiveByQuery: [];
  archiveSelected: [];
  change: [pag: any, filters: any, sorter: any];
  detail: [record: AdminMonitorInstanceInfo];
  reassign: [record: AdminMonitorInstanceInfo];
  resendNotice: [record: AdminMonitorInstanceInfo];
  selectChange: [keys: (number | string)[]];
  terminate: [record: AdminMonitorInstanceInfo];
}>();

const filterStateRef = toRef(props, 'activeFilters');
const tableConfig = computed(() =>
  getWorkflowMonitorColumns(filterStateRef, emit, props.pagination),
);
const rowSelection = computed(() => ({
  getCheckboxProps: (record: AdminMonitorInstanceInfo) => ({
    disabled: !record.canArchive,
  }),
  onChange: (keys: (number | string)[]) => emit('selectChange', keys),
  selectedRowKeys: props.selectedRowKeys,
}));
</script>

<template>
  <BaseTable
    class="workflow-monitor-table"
    :columns="tableConfig.columns"
    :data-source="dataSource"
    :loading="loading"
    :pagination="pagination"
    :row-selection="rowSelection"
    :scroll="tableConfig.scroll"
    :show-toolbar="true"
    row-key="id"
    @change="(pag, tableFilters, sorter) => $emit('change', pag, tableFilters, sorter)"
  >
    <template #toolbar>
      <div class="workflow-monitor-table__header">
        <div class="workflow-monitor-table__title">
          {{ scopeTitle }}
        </div>
        <Space>
          <Button
            v-access:code="permissionCodes.workflowMonitor.manage"
            :disabled="selectedRowKeys.length === 0"
            :loading="archivingSelected"
            type="primary"
            @click="$emit('archiveSelected')"
          >
            <template #icon>
              <IconifyIcon icon="lucide:archive" />
            </template>
            批量归档
          </Button>
          <Button
            v-access:code="permissionCodes.workflowMonitor.manage"
            :loading="archivingByQuery"
            @click="$emit('archiveByQuery')"
          >
            <template #icon>
              <IconifyIcon icon="lucide:list-checks" />
            </template>
            按查询归档
          </Button>
        </Space>
      </div>
    </template>
  </BaseTable>
</template>

<style scoped>
.workflow-monitor-table {
  height: 100%;
  overflow: hidden;
}

.workflow-monitor-table :deep(.ant-card-body) {
  display: flex;
  flex-direction: column;
  height: 100%;
  min-height: 0;
}

.workflow-monitor-table__title {
  flex: 0 0 auto;
  color: var(--ant-color-text, rgb(0 0 0 / 88%));
  font-size: 15px;
  font-weight: 500;
}

.workflow-monitor-table__header {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.workflow-monitor-table :deep(.base-table-toolbar) {
  align-items: flex-start;
}

.workflow-monitor-table :deep(.base-table-toolbar-main) {
  align-items: flex-start;
}

.workflow-monitor-table :deep(.ant-table-wrapper) {
  flex: 0 0 auto;
  min-height: 0;
}

.workflow-monitor-table :deep(.ant-spin-nested-loading),
.workflow-monitor-table :deep(.ant-spin-container) {
  display: block;
  min-height: 0;
}

.workflow-monitor-table :deep(.ant-table) {
  flex: none;
}
</style>
