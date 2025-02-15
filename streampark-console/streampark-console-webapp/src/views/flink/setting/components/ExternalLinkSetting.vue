<!--
  Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      https://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->

<script lang="ts">
  import { defineComponent } from 'vue';

  import { useI18n } from '/@/hooks/web/useI18n';
  import { ExternalLink } from '/@/api/flink/setting/types/externalLink.type';
  import { useMessage } from '/@/hooks/web/useMessage';

  export default defineComponent({
    name: 'ExternalLinkSetting',
  });
</script>
<script lang="ts" setup name="ExternalLinkSetting">
  import { onMounted, ref } from 'vue';
  import { PlusOutlined } from '@ant-design/icons-vue';
  import { ColumnsType } from 'ant-design-vue/lib/table';
  import { useModal } from '/@/components/Modal';
  import ExternalLinkModal from './ExternalLinkModal.vue';
  import ExternalLinkBadge from './ExternalLinkBadge.vue';
  import { Table, Popconfirm } from 'ant-design-vue';
  import { fetchExternalLink, fetchExternalLinkDelete } from '/@/api/flink/setting/externalLink';

  const [registerLinkModal, { openModal: openLinkModal }] = useModal();
  const { Swal } = useMessage();
  const { t } = useI18n();

  const externalLinks = ref<ExternalLink[]>([]);

  /* Get external link list */
  async function getExternalLink() {
    const externalLinkList = await fetchExternalLink();
    externalLinks.value = externalLinkList;
  }

  onMounted(() => {
    getExternalLink();
  });

  /* delete external link */
  async function handleDeleteExternalLink(id: string) {
    try {
      await fetchExternalLinkDelete(id);
      Swal.fire({
        icon: 'success',
        title: 'Delete Alert Config  successful!',
        showConfirmButton: false,
        timer: 2000,
      });
      await getExternalLink();
    } catch (error: any) {
      console.log(error);
    }
  }

  /* Edit button */
  function handleEditExternalLink(item: ExternalLink) {
    openLinkModal(true, item);
  }
  const columns: ColumnsType<ExternalLink> = [
    { key: 'badgeColor', dataIndex: 'badgeColor' },
    { key: 'linkUrl', dataIndex: 'linkUrl', ellipsis: true },
    {
      key: 'action',
      dataIndex: 'action',
      fixed: 'right',
      align: 'right',
    },
  ];

  const ATable = Table;
  const APopconfirm = Popconfirm;
</script>
<template>
  <div v-auth="'externalLink:create'" style="margin-bottom: 20px">
    <a-button type="dashed" style="width: 100%; margin-top: 20px" @click="openLinkModal(true, {})">
      <plus-outlined />
      {{ t('common.add') }}
    </a-button>
  </div>
  <a-table :showHeader="false" :data-source="externalLinks" :columns="columns" :pagination="false">
    <template #bodyCell="{ column, record }">
      <template v-if="column.key === 'badgeColor'">
        <ExternalLinkBadge
          :openLink="false"
          :color="record.badgeColor"
          :label="record.badgeLabel"
          :redirect="record.linkUrl"
          :message="record.badgeName"
        />
      </template>
      <template v-else-if="column.key === 'action'">
        <span>
          <a-button
            v-auth="'externalLink:update'"
            type="link"
            @click="handleEditExternalLink(record)"
            >{{ t('common.edit') }}</a-button
          >
          <a-popconfirm
            :title="t('flink.setting.externalLink.confDeleteTitle')"
            @confirm="handleDeleteExternalLink(record.id)"
            placement="topRight"
          >
            <a-button v-auth="'externalLink:delete'" danger type="text">{{
              t('common.delText')
            }}</a-button>
          </a-popconfirm>
        </span>
      </template>
    </template>
  </a-table>
  <ExternalLinkModal @register="registerLinkModal" width="850px" @reload="getExternalLink" />
</template>
