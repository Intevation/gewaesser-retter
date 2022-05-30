<template>
  <v-card v-if="item" elevation="5" class="infomenu-card">
    <h4>{{ item.aktionsname }}
      <v-icon @click.stop="zoomTo()" class="zoom-button">
          mdi-target
      </v-icon>
      <a :href="item.placelink" class="share-button">
        <v-icon>
            mdi-share-variant
        </v-icon>
      </a>
    </h4>
    <p v-if="item.veranstalter"><i>{{ item.veranstalter }} </i></p>

  <p v-if="item.muell">Gesamtmenge: {{ item.muell }}</p>
  <p>
   <span v-if="item.type==='campaign'">Am</span>
   <span v-if="item.type!='campaign'">Gesammelt am</span>
    {{ item.datum }} <span v-if="item.uhrzeit"> um {{ item.uhrzeit}}
    Uhr</span> <br />
  </p>
   <p v-if="item.beschreibung">
    <b> Beschreibung: </b><br />
    {{ item.beschreibung }}
   </p>
    <p>
        <a :href="getItemUrl(item)" target="_blank"> Zusatzinfos</a>
    </p>
    <!-- TODO: Detail button for closed and findings -->
    <!-- TODO better separation in past and present -->
  </v-card>
</template>
<script>
// This is the "popup" for a leaflet map item clicked on Map.vue. It has the
// infoItem as a property, and should offer short information. The url property
// might contain external data on a different server (privacy reasons) and is
// currently included in an iframe (TODO: check if that works on mobile + all
// browsers)

export default {
  props: {
    infoItem: Object
  },
  data: () => ({
    detailUrl: process.env.VUE_APP_detailUrl,
    detailParams: process.env.VUE_APP_detailParams
  }),
  computed: {
    item: {
      get() {
        if (!this.infoItem) { return {} }
        let p = this.infoItem.properties;
        p.placelink = `?uuid=${p.uuidPublic}`
        return p;
      }
    }
  },

  /**
   * emits a "I want to zoom to my position" event to the parent map
  */
  methods: {
    zoomTo() {
      this.$emit('update:zoom', this.infoItem.geometry.coordinates);
    },
    getItemUrl(item){
      return `${this.detailUrl}&uuidPublic=${item.uuidPublic}${this.detailParams}`;
    }
  }
};
</script>

<style>
.info-iframe {
  width: 100%;
  height: 100%;
  border: none;
}
.infomenu-card {
  max-width: 25vw;
  max-height: 70vH;
  overflow-y: auto;
  border-radius: 10px !important;
  padding: 5px;
}
.zoom-button {
  text-decoration: none;
}
.share-button {
  text-decoration: none;
}
</style>