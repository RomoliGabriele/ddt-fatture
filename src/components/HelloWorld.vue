<template>
  <div class="hello">
    <table-lite
      :is-loading="table.isLoading"
      :columns="table.columns"
      :rows="table.rows"
      :total="table.totalRecordCount"
      :sortable="table.sortable"
      @do-search="doSearch"
      @is-finished="tableLoadingFinish"
    />
  </div>
</template>

<script>
import TableLite from "vue3-table-lite";
import axios from "axios";
import { defineComponent, reactive, ref, computed, createApp, h } from "vue";

const searchTerm = ref(""); // Search text
const searchTerm2 = ref(""); // Search text

export default {
  components: { TableLite },
  setup() {
    // Init Your table settings
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
          sortable: true,
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
          width: "5%",
          sortable: true,
        },
        {
          label: "CLIENTE COD",
          field: "client_id",
          width: "5%",
          sortable: true,
        },
        {
          label: "VISIBILE FINO AL",
          field: "due_date",
          width: "5%",
          sortable: true,
        },
        {
          label: "TIPO FILE",
          field: "file_type",
          width: "5%",
          sortable: false,
        },
        {
          label: "FILE (KB)",
          field: "file_size",
          width: "5%",
          sortable: true,
        },
      ],
      rows: computed(() => {
        return data.filter(
          (x) => x.name.toLowerCase().includes(searchTerm2.value.toLowerCase()) //&&
          //x.name.toLowerCase().includes(searchTerm.value.toLowerCase())
        );
      }),
      totalRecordCount: computed(() => {
        return table.rows.length;
      }),
      sortable: {
        order: "id",
        sort: "asc",
      },
    });

    /**
     * Table search event
     */
    const doSearch = (offset, limit, order, sort) => {
      table.isLoading = true;

      // Start use axios to get data from Server
      let url =
        "http://localhost/api/health" +
        offset +
        "&limit=" +
        limit +
        "&order=" +
        order +
        "&sort=" +
        sort;
      axios.get(url).then((response) => {
        // Point: your response is like it on this example.
        response = {
          rows: [
            {
              is_read: "DA LEGGERE",
              id: 5264627,
              name: "DDT20254692-01245",
              type: "FAT",
              date: "2021-01-01",
              client_id: 123456,
              file_type: "PDF",
              due_date: "2021-01-31",
              file_size: 123.8,
            },
            {
              is_read: "LETTO",
              id: 23952750,
              name: "DDT21254692-02403",
              type: "DDT",
              date: "2021-01-01",
              client_id: 123456,
              file_type: "PDF",
              due_date: "2021-01-31",
              file_size: 13.8,
            },
          ],
          count: 2,
        };

        // refresh table rows
        table.rows = response.rows;
        table.totalRecordCount = response.count;
        table.sortable.order = order;
        table.sortable.sort = sort;

        tableLoadingFinish();
      });
      // End use axios to get data from Server
    };

    /**
     * Table search finished event
     */
    const tableLoadingFinish = (elements) => {
      table.isLoading = false;
    };

    // Get data first
    doSearch(0, 10, "id", "asc");

    return {
      table,
      doSearch,
      tableLoadingFinish,
    };
  },
};
</script>
