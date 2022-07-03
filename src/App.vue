<template>
  <a-layout class="layout" style="padding: 0 24px; background: #fff">
    <UploadComponent @upload-success="reload" />
    <a-table
      :row-key="(record) => record.id"
      :columns="columns"
      :data-source="dataSource && dataSource.items"
      :pagination="pagination"
      :loading="loading"
      @change="handleTableChange"
    >
      <template #bodyCell="{ column, text, record }">
        <template
          v-if="['empId', 'firstName', 'lastName'].includes(column.dataIndex)"
        >
          <div>
            <a-input
              v-if="editableData[record.id]"
              v-model:value="editableData[record.id][column.dataIndex]"
              style="margin: -5px 0"
            />
            <template v-else>
              {{ text }}
            </template>
          </div>
        </template>
        <template v-else-if="column.dataIndex === 'dateOfBirthFormatted'">
          <div>
            <a-date-picker
              v-if="editableData[record.id]"
              v-model:value="editableData[record.id]['dateOfBirth']"
              format="DD MMM YYYY"
            />
            <!-- <a-input
              v-if="editableData[record.id]"
              v-model:value="editableData[record.id][column.dataIndex]"
              style="margin: -5px 0"
            /> -->
            <template v-else>
              {{ text }}
            </template>
          </div>
        </template>
        <template v-else-if="column.dataIndex === 'operation'">
          <div class="editable-row-operations">
            <a-space v-if="editableData[record.id]">
              <a-button type="primary" @click="save(record.id)">
                <template #icon><SaveOutlined /></template>
                Save</a-button
              >
              <a-popconfirm
                title="Sure to cancel?"
                @confirm="cancel(record.id)"
              >
                <a-button type="secondary">
                  <template #icon><StopOutlined /></template>
                  Cancel</a-button
                >
              </a-popconfirm>
            </a-space>
            <a-space v-else>
              <a-button @click="edit(record.id)">
                <template #icon><EditOutlined /></template>
                Edit
              </a-button>
            </a-space>
          </div>
        </template>
      </template>
    </a-table>
  </a-layout>
</template>

<script>
import { cloneDeep } from "lodash-es";
import { computed, reactive, defineComponent, ref } from "@vue/runtime-core";
import { usePagination } from "vue-request";
import { message } from "ant-design-vue";
import moment from "moment";
import dayjs from "dayjs";
import UploadComponent from "./components/UploadComponent.vue";
import {
  EditOutlined,
  SaveOutlined,
  StopOutlined,
} from "@ant-design/icons-vue";
import { API_URL } from "./commons/api";

const UPDATE_EMPLOYEE_URL = `${API_URL}Employee`;

const columns = [
  {
    title: "Employee Id",
    dataIndex: "empId",
    key: "empId",
  },
  {
    title: "First Name",
    dataIndex: "firstName",
    key: "firstName",
  },
  {
    title: "Last Name",
    dataIndex: "lastName",
    key: "lastName",
  },
  {
    title: "Date Of Birth",
    dataIndex: "dateOfBirthFormatted",
    key: "dateOfBirthFormatted",
  },
  {
    title: "operation",
    dataIndex: "operation",
  },
];

const queryData = async ({ pageIndex, pageSize }) => {
  let result = await fetch(
    `${API_URL}Employee?PageIndex=${pageIndex}&PageSize=${pageSize}`
  );
  let data = await result.json();
  return data;
};

export default defineComponent({
  components: {
    EditOutlined,
    SaveOutlined,
    StopOutlined,
    UploadComponent,
  },
  setup() {
    const { data, run, loading, current, pageSize, totalPage } = usePagination(
      queryData,
      {
        formatResult: (res) => {
          res.results.forEach((item) => {
            item.dateOfBirthFormatted = item.dateOfBirth
              ? moment(item.dateOfBirth).format("DD MMM YYYY")
              : "";
            item.dateOfBirth = dayjs(item.dateOfBirth);
          });

          return {
            total: res.total,
            items: res.results,
          };
        },
        pagination: {
          currentKey: "pageIndex",
          pageSizeKey: "pageSize",
        },
      }
    );

    const dataSource = ref(data);

    const pagination = computed(() => ({
      total: totalPage.value * pageSize.value,
      pageIndex: current.value,
      pageSize: pageSize.value,
    }));

    const handleTableChange = (pag) => {
      run({
        pageIndex: pag.current,
        pageSize: pag.pageSize,
      });
    };

    const reload = () => {
      run({
        pageIndex: 1,
        pageSize: pageSize.value,
      });
    };

    const editableData = reactive({});

    const edit = (key) => {
      editableData[key] = cloneDeep(
        dataSource.value.items.filter((item) => key === item.id)[0]
      );
    };

    const save = (key) => {
      const newObject = editableData[key];
      fetch(UPDATE_EMPLOYEE_URL, {
        method: "PUT",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(newObject),
      })
        .then((res) => res.json())
        .then(({ success }) => {
          if (success) {
            message.success("Employee updated successfully");
            Object.assign(
              dataSource.value.items.filter((item) => key === item.id)[0],
              newObject
            );
            delete editableData[key];
          } else {
            message.error("Employee update failed");
          }
        });
    };

    const cancel = (key) => {
      delete editableData[key];
    };

    return {
      loading,
      dataSource,
      columns,
      pagination,
      reload,
      handleTableChange,
      editableData,
      edit,
      save,
      cancel,
    };
  },
});
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
