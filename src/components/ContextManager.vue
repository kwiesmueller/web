<template>
  <div class="container">
    <div class="col-md-8 col-md-offset-2">
      <h3 style="text-align:left; margin-bottom:30px">Manage a Context</h3>
      <div class="form-group">
        <label>Title</label>
        <input type="text" class="form-control" placeholder="Title" v-model="model.title">
      </div>
      <div class="form-group">
        <label>Description</label>
        <input type="text" class="form-control" placeholder="Description" v-model="model.desc">
      </div>
      <div class="form-group">
        <label>Url</label>
        <input type="text" class="form-control" placeholder="Description" v-model="model.url">
      </div>
      <div class="form-group">
        <label>Google KG ID</label>
        <input type="text" class="form-control" placeholder="Google KG ID" v-model="model.mid">
      </div>
      <div class="form-group">
        <label>Wikidata ID</label>
        <input type="text" class="form-control" placeholder="Wikidata ID" v-model="model.qid">
      </div>
      <v-btn @click="save(0)" color="primary" dark slot="activator" class="blue radius">
        Save
      </v-btn>
      <button @click="backPage()" class="mdl-button mdl-js-button mdl-button--raised mdl-button--colored">Cancel</button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import router from '../router';

/* eslint-disable no-undef */
const API_URL = API;

export default {
  data() {
    return {
      model: {},
    };
  },

  created() {
    const paramId = this.$route.params.id;
    if (paramId !== '0') {
      this.get(paramId);
    }
  },

  methods: {
    get(id) {
      axios.get(`${API_URL}/contexts/${id}`).then((response) => {
        this.model = response.data;
      });
    },

    save() {
      const paramId = this.$route.params.id;

      if (paramId === '0') {
        axios.post(`${API_URL}/contexts`, this.model).then(() => {
          this.backPage();
        }, (err) => {
          const obj = {
            title: 'Error',
            message: err.response.data.message,
            type: 'error',
          };
          this.$Simplert.open(obj);
        });
      } else {
        axios.put(`${API_URL}/contexts/${paramId}`, this.model).then(() => {
          this.backPage();
        }, (err) => {
          const obj = {
            title: 'Error',
            message: err.response.data.message,
            type: 'error',
          };
          this.$Simplert.open(obj);
        });
      }
    },

    backPage() {
      router.push('/context');
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
