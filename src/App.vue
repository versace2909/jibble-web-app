<template>
  <UploadComponent @upload-success="reload"/>
  <a-table
    :row-key="(record) => record.empId"
    :columns="columns"
    :data-source="dataSource && dataSource.items"
    :pagination="pagination"
    :loading="loading"
    @change="handleTableChange"
  ></a-table>
</template>

<script>
import { computed, defineComponent } from "@vue/runtime-core";
import { usePagination } from "vue-request";
import UploadComponent from "./components/UploadComponent.vue";
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
    dataIndex: "doB",
    key: "doB",
  },
];

const queryData = async ({ pageIndex, pageSize }) => {
  let result = await fetch(
    `https://localhost:5001/api/Employee?PageIndex=${pageIndex}&PageSize=${pageSize}`
  );
  let data = await result.json();
  return data;
};

export default defineComponent({
  components: {
    UploadComponent,
  },
  setup() {
    const {
      data: dataSource,
      run,
      loading,
      current,
      pageSize,
      totalPage,
    } = usePagination(queryData, {
      formatResult: (res) => {
        return {
          total: res.total,
          items: res.results,
        };
      },
      pagination: {
        currentKey: "pageIndex",
        pageSizeKey: "pageSize",
      },
    });

    const pagination = computed(() => ({
      total: totalPage.value,
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
        pageSize: 10,
      });
    };

    return {
      loading,
      dataSource,
      columns,
      pagination,
      reload,
      handleTableChange,
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
