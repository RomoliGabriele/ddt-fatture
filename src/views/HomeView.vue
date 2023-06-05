<template>
  <table-lite
    :is-loading="table.isLoading"
    :columns="table.columns"
    :rows="table.rows"
    :total="table.totalRecordCount"
    :sortable="table.sortable"
    :page="table.page"
    @do-search="doSearch"
    @VnodeMounted="initTable"
    @is-finished="table.isLoading = false"
  ></table-lite>
</template>

<script>
import { defineComponent, reactive, ref, computed, createApp, h } from "vue";
import TableLite from "vue3-table-lite";
import axios from "axios";

const id = ref(""); // Search text
const is_read = ref(""); // select
const type = ref(""); // select
const from = ref(); // date
const to = ref(); // date

var rows = [];
var customer_id = 1;

export default defineComponent({
  name: "App",
  components: { TableLite },
  data() {
    return {
      rows: [],
      table: {},
    };
  },
  setup() {
    // Table config
    const table = reactive({
      isLoading: false,
      columns: [
        {
          label: "STATO DOC",
          field: "is_read",
          width: "10%",
        },
        {
          label: "TIPO DOC",
          field: "type",
          width: "5%",
          sortable: true,
        },
        {
          label: "NOME DOCUMENTO",
          field: "name",
          width: "20%",
        },
        {
          label: "NUMERO DOCUMENTO",
          field: "id",
          width: "3%",
          sortable: true,
          isKey: true,
        },
        {
          label: "DATA DOC",
          field: "date",
          width: "10%",
          sortable: true,
          display: function (row) {
            let objectDate = new Date(row.date);
            let day = objectDate.getDate();
            let month = objectDate.getMonth();
            let year = objectDate.getFullYear();
            return day + "/" + month + "/" + year;
          },
        },
        {
          label: "CLIENTE COD",
          field: "customer_id",
          width: "5%",
        },
        {
          label: "VISIBILE FINO AL",
          field: "due_date",
          width: "5%",
          display: function (row) {
            let objectDate = new Date(row.due_date);
            let day = objectDate.getDate();
            let month = objectDate.getMonth();
            let year = objectDate.getFullYear();
            return day + "/" + month + "/" + year;
          },
        },
        {
          label: "TIPO FILE",
          field: "file_type",
          width: "5%",
        },
        {
          label: "FILE (KB)",
          field: "file_size",
          width: "5%",
        },
      ],
      rows: [],
      totalRecordCount: 0,
      sortable: {
        order: "id",
        sort: "asc",
      },
      page: 1,
    });

    const doSearch = async (offset, limit, order, sort) => {
      table.isLoading = true;
      // Start use axios to get data from Server
      let url = "";
      if (customer_id != 0) {
        url =
          "http://localhost:8010/api/documents/" +
          customer_id +
          "?offset=" +
          offset +
          "&limit=" +
          limit +
          "&order=" +
          order +
          "&sort=" +
          sort;
      } else {
        url =
          "http://localhost:8010/api/documents?offset=" +
          offset +
          "&limit=" +
          limit +
          "&order=" +
          order +
          "&sort=" +
          sort;
      }
      let response = await axios.get(url);
      console.log(response);
      if (response.data.message) {
        table.rows = [];
        table.totalRecordCount = 0;
        table.isLoading = false;
        alert(response.data.message);
        return;
      } else {
        table.rows = computed(() => {
          if (!from.value || !to.value) {
            return response.data.rows.filter(
              (x) =>
                String(x.id).toLowerCase().includes(id.value.toLowerCase()) &&
                String(x.is_read)
                  .toLowerCase()
                  .includes(is_read.value.toLowerCase()) &&
                String(x.type).toLowerCase().includes(type.value.toLowerCase())
            );
          } else {
            return response.data.rows.filter(
              (x) =>
                String(x.id).toLowerCase().includes(id.value.toLowerCase()) &&
                String(x.is_read)
                  .toLowerCase()
                  .includes(is_read.value.toLowerCase()) &&
                String(x.type)
                  .toLowerCase()
                  .includes(type.value.toLowerCase()) &&
                Date.parse(x.date) >= Date.parse(from.value) &&
                Date.parse(x.date) <= Date.parse(to.value)
            );
          }
        });
        table.totalRecordCount = response.data.total;
        table.sortable.order = order;
        table.sortable.sort = sort;
        table.isLoading = false;
        table.page = offset / limit + 1;
        console.log(table.page);
      }
      // End use axios to get data from Server
    };

    doSearch(0, 10, "id", "asc");
    /**
     * Trigger after initail component
     */
    const initTable = ({ el: tableComponent }) => {
      let headerTr = tableComponent.getElementsByClassName("vtl-thead-tr");
      if (!headerTr[0]) {
        return;
      }
      let cloneTr = headerTr[0].cloneNode(true); // Clone first <tr>
      let childTh = cloneTr.getElementsByClassName("vtl-thead-th");
      for (let i = 0; i < childTh.length; i++) {
        // Clear <th>'s HTML
        childTh[i].innerHTML = "";
      }
      createApp(
        defineComponent({
          setup() {
            return () =>
              h(
                "select",
                {
                  onInput: (e) => {
                    is_read.value = e.target.value;
                  },
                },
                [
                  h("option", { value: "" }, "Tutti"),
                  h("option", { value: "LETTO" }, "Letto"),
                  h("option", { value: "DA LEGGERE" }, "Da leggere"),
                ]
              );
          },
        })
      ).mount(childTh[0]);

      createApp(
        defineComponent({
          setup() {
            return () =>
              h(
                "select",
                {
                  onInput: (e) => {
                    type.value = e.target.value;
                  },
                },
                [
                  h("option", { value: "" }, "Tutti"),
                  h("option", { value: "FAT" }, "FAT"),
                  h("option", { value: "DDT" }, "DDT"),
                ]
              );
          },
        })
      ).mount(childTh[1]);
      // Create a input element and append to first <th>
      createApp(
        defineComponent({
          setup() {
            return () =>
              h("input", {
                type: "number",
                value: id.value,
                onInput: (e) => {
                  id.value = e.target.value;
                },
              });
          },
        })
      ).mount(childTh[3]);
      // append cloned element to the header after first <tr>
      headerTr[0].after(cloneTr);

      createApp(
        defineComponent({
          setup() {
            return () => [
              h("label", "DA: "),
              h("input", {
                type: "date",
                value: from.value,
                onInput: (e) => {
                  from.value = e.target.value;
                },
              }),
              h("br"),
              h("label", "A: "),
              h("input", {
                type: "date",
                value: to.value,
                onInput: (e) => {
                  to.value = e.target.value;
                },
              }),
            ];
          },
        })
      ).mount(childTh[4]);
    };

    return {
      id,
      table,
      initTable,
      doSearch,
    };
  },
});
</script>
