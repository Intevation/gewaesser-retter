<template>
  <v-card>
    <v-data-table
      :headers="tableHeader"
      :items="currentData"
      item-key="uuidPublic"
      :options="tableOptions"
      :items-per-page="10"
      no-data-text="Keine Daten gefunden"
      @dblclick:row="onDblClick"
      show-expand
      :expanded.sync="expanded"
    >
      <template v-slot:item.extra="{ item }">
        <a :href="getItemUrl(item)" target="_blank"> Zusatzinfos </a>
      </template>
      <template v-slot:item.tableaction="{ item }">
        <v-icon @click.stop="onDblClick(null, {item})">
          mdi-target
        </v-icon>
      </template>
      <template v-slot:expanded-item="{ headers, item }">
        <td :colspan="headers.length">
        <v-card flat>
          <p v-if="item.beschreibung"><b> Beschreibung:</b> <br/>
            {{ item.beschreibung }}
          </p>
          <p v-if="item.muell">
          <b>Gesammelte Gesamtmenge</b><br/>
            {{item.muell}}
            <span v-if="item.muell-saecke"> ({{ item.muell-saecke }}) Säcke)</span>
          </p>
          <p v-if="item.funde">
            <b> Funde:</b> <br />
              <span v-for="(value, name, index) in item.funde" v-bind:value="value"
                v-bind:key="index">
                <span v-if="index">, </span>
                 {{name[0].toUpperCase() + name.substring(1) }} : {{ value }}
          </span>
          <p v-if="item.type==='campaign'">
            <a :href="getItemUrl(item)" target="_blank">
              Weitere Infos zu Treffpunkt und Kontakt
            </a>
          </p>
        </v-card>
        </td>
      </template>
    </v-data-table>
  </v-card>
</template>

<script>

export default {
  props: {
    trashData: Array
  },

  data: () => ({
    expanded: [],
    detailUrl: process.env.VUE_APP_detailUrl,
    detailParams: process.env.VUE_APP_detailParams,
    tableOptions: {
      sortBy: ["datum", "veranstalter", "aktionsname"],
      sortDesc: [false, false, false],
      mustSort: true
    },
    tableHeader: [
      {
        text: "",
        sortable: false,
        value:"tableaction",
      },
      {
        text: "Name",
        align: "left",
        value: "aktionsname",
        sortable: true,
      },
      {
        text: "Veranstalter",
        align: "left",
        value: "veranstalter",
        sortable: true,
      },
      {
        text: "Datum",
        align: "left",
        value: "datum",
        sortable: true,
        sort: (a, b) => {
          const as = a.split('.');
          const bs = b.split('.');
          return new Date(`${as[2]}-${as[1]}-${as[0]}`) - new Date(`${bs[2]}-${bs[1]}-${bs[0]}`)
        }
      },
      {
        text: "Gesammelte Menge",
        align: "left",
        value: "muell",
        sortable: false
      },
      {
        text: "",
        align: "left",
        value: "extra",
        sortable: false,
      },
      {
        text: "",
        value: "data-table-expand"
        },
    ],
    currentData: []
  }),
  watch: {
    // watcher that triggers as soon as trashData gets an update.
    trashData() {
      this.sortData();
    }
  },
  mounted() {
    if (!this.trashData || !this.trashData.length) {
      this.currentData = [];
    } else {
      this.sortData();
    }
  },
  methods: {
    sortData() {
      this.currentData = this.trashData.map(f => f.properties);
    },

    onDblClick(ev, ev2) {
      const item = this.trashData.find(t => t.properties === ev2.item);
      if (item) {
        this.$root.$emit("requestzoom", {latlng: {
          lat: item.geometry.coordinates[1],
          lng: item.geometry.coordinates[0]
      }});
        this.$emit("update:mapitem", item);
      }
    },
    getItemUrl(item){
      return `${this.detailUrl}&uuidPublic=${item.uuidPublic}${this.detailParams}`;
    }
  },
};
</script>
