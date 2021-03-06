<template>
  <div class="container">
    <div class="row">
      <div class="col-sm-10">
        <alert :message="message" v-if="showMessage"></alert>
        <div class="btn-group" role="group">
        <button type="button" class="btn btn-success btn-sm" v-b-modal.server-modal>Create</button>
        <button type="button" class="btn btn-warning btn-sm">Restart</button>
        </div>
        <br><br>
        <table class="table table-hover">
          <thead>
            <tr>
              <th scope="col">Server name</th>
              <th></th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(server, index) in servers" :key="index">
              <td>{{ server.server_name }}</td>
              <td>
                <div class="btn-group" role="group">
                  <button type="button" class="btn btn-warning btn-sm">Activate</button>
                  <button type="button" class="btn btn-danger btn-sm">Delete</button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <b-modal ref="createServerModal"
          id="server-modal"
          title="Create a new server"
          hide-footer>
    <b-form @submit="onSubmit" @reset="onReset" class="w-100">
    <b-form-group id="form-title-group"
                  label="Name (max 32 chars, only alphanumeric, hyphen, underscore):"
                  label-for="form-title-input">
        <b-form-input id="form-title-input"
                      type="text"
                      v-model="createServerForm.serverName"
                      required
                      placeholder="server name">
        </b-form-input>
      </b-form-group>
      <b-form-group id="form-activate-group">
        <b-form-checkbox-group v-model="createServerForm.activateServer" id="form-checks">
          <b-form-checkbox value="true">Activate?</b-form-checkbox>
        </b-form-checkbox-group>
      </b-form-group>
      <b-button type="submit" variant="primary">Create</b-button>
      <b-button type="reset" variant="danger">Reset</b-button>
    </b-form>
  </b-modal>
  </div>
</template>

<script>
import axios from 'axios';
import InfoBox from './InfoBox.vue';

export default {
  name: 'ListServersComponent',
  data() {
    return {
      servers: '',
      createServerForm: {
        serverName: '',
        activate: [],
      },
      message: '',
      showMessage: false,
    };
  },
  components: {
    alert: InfoBox,
  },
  methods: {
    getServers() {
      const path = 'http://localhost:8000/get_servers';
      axios.get(path)
        .then((res) => {
          this.servers = res.data.servers;
        })
        .catch((error) => {
          // eslint-disable-next-line
          console.error(error);
        });
    },
    createServer(payload) {
      const path = 'http://localhost:8000/create_server';
      axios.put(path, payload)
        .then(() => {
          this.getServers();
          this.message = 'Server created!';
        })
        .catch((error) => {
          // eslint-disable-next-line
          console.log(error);
          this.message = error;
          this.getServers();
        });
      this.showMessage = true;
    },
    initForm() {
      this.createServerForm.serverName = '';
      this.createServerForm.activate = [];
    },
    onSubmit(evt) {
      evt.preventDefault();
      this.$refs.createServerModal.hide();
      let activate = false;
      if (this.createServerForm.activate[0]) activate = true;
      const payload = {
        server_name: this.createServerForm.serverName,
        activate, // property shorthand
      };
      this.createServer(payload);
      this.initForm();
    },
    onReset(evt) {
      evt.preventDefault();
      this.$refs.createServerModal.hide();
      this.initForm();
      this.showMessage = false;
    },
  },
  created() {
    this.getServers();
  },
};
</script>
