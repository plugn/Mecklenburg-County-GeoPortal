<template lang="html">
    <div v-if="privateState.results && sharedState.show.indexOf('parks') !== -1">
        <div class="mdl-typography--text-center">
            <div class="report-record-highlight">
                <i role="presentation" class="material-icons">nature_people</i>
                <h2>Your closest park is</h2>
                <h1>{{ privateState.results[0].name }}</h1>
                <h3>
                    <a href="javascript:void(0)"v-on:click="locationClick(0)">
                        {{ privateState.results[0].address }}, {{ privateState.results[0].city}}
                    </a>
                </h3>
                <h4>
                    {{ privateState.results[0].distance | distance }}
                </h4>
            </div>
            <table class="mdl-data-table mdl-js-data-table">
                <caption>Parks Nearby</caption>
                <thead>
                    <tr>
                        <th class="mdl-data-table__cell--non-numeric">Park</th>
                        <th class="mdl-data-table__cell--non-numeric">Address</th>
                        <th class="col-responsive">Distance</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(item, index) in privateState.results">
                        <td class="mdl-data-table__cell--non-numeric">
                            {{ item.name }}
                        </td>
                        <td class="mdl-data-table__cell--non-numeric">
                            <a href="javascript:void(0)" v-on:click="locationClick(index)">
                                {{ item.address }}, {{item.city}}
                            </a>
                        </td>
                        <td class="nowrap col-responsive">
                            {{ item.distance | distance }}
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
        <div class="report-moreinfo mdl-typography--text-left">
            <h5>For more information, please visit:</h5>
            <ul class="list-unstyled">
                <li><a href="http://charmeck.org/mecklenburg/county/ParkandRec/Pages/default.aspx">Mecklenburg County Park and Recreation</a></li>
                <li><a href="http://www.huntersville.org/Departments/ParksRecreation.aspx" target="_blank">Huntersville Parks and Recreation</a></li>
            </ul>
        </div>
    </div>
</template>

<script>
import axios from 'axios';
import format from 'format-number';

export default {
  name: 'parks',
  filters: {
      distance: function(dist) {
          return format({'truncate': 1, 'suffix': ' miles'})(dist / 5280);
      }
  },
  watch: {
      'sharedState.selected.lnglat': 'getResults',
      'sharedState.show': 'getResults'
  },
  mounted: function() {
      this.getResults();
  },
  methods: {
      getResults: function() {
          let _this = this;
          if (_this.sharedState.selected.lnglat && _this.sharedState.show.indexOf('parks') !== -1) {
              axios
                .get(`http://maps.co.mecklenburg.nc.us/api/nearest/v1/parks_all/${_this.sharedState.selected.lnglat.join(',')}/4326`,
                {
                    params: {
                        'columns': 'name, address, city, st_x(st_transform(geom, 4326)) as lng, st_y(st_transform(geom, 4326)) as lat',
                        'limit': '5'
                    }
                })
                .then(function(response) {
                    _this.privateState.results = response.data;
                });
            }
      },
      locationClick: function(index) {
          let poi = this.privateState.results[index];
          this.sharedState.poi = {
              'lnglat': [poi.lng, poi.lat],
              'address': poi.address,
              'label': poi.name
          };
      }
  }
}
</script>

<style lang="css">
</style>
