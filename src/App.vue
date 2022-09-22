<template>
  <div id="app">
    <facebook-login
      class="button"
      appId="932010011091655"
      @login="onLogin"
      @logout="onLogout"
      @get-initial-status="getUserData"
      @sdk-loaded="sdkLoaded"
      :loginOptions="{
        scope:
          'public_profile,email, pages_show_list, pages_messaging, pages_manage_metadata, pages_read_engagement',
      }"
    >
    </facebook-login>
    <div style="display: flex;justify-content: space-around; width: 100%;">
      <div v-if="isConnected">
        <h1>My Facebook Information</h1>
        <div class="well">
          <div class="list-item">
            <img :src="picture" />
          </div>
          <div class="list-item">
            <i>{{ name }}</i>
          </div>
          <div class="list-item">
            <i>{{ email }}</i>
          </div>
          <div class="list-item">
            <i>{{ personalID }}</i>
          </div>
        </div>
      </div>
      <!-- <pre>{{ pages }}</pre> -->
      <div v-if="pages">
        <div style="display: flex;justify-content: center; width: 100%;">
          <h2>
            Connect Your Business Pages
          </h2>
        </div>
        <!-- {{ pages }} -->
        <table border="3" align="center">
          <tr>
            <th>Name</th>
            <th>Image</th>
            <th>Actions</th>
          </tr>
          <tr v-for="(page, index) in pages" :key="index">
            <td>{{ page.name }}</td>
            <td>
              <img
                :src="page.picture.data.url"
                alt=""
                height="100"
                width="100"
              />
            </td>
            <td v-if="page.is_webhooks_subscribed == true">
              Connected
              <button @click="disconnectPage(page.id, page.access_token)">Disconnect</button>
            </td>
            <td v-else-if="page.is_webhooks_subscribed == false">
              Disconnected
              <button @click="subscribePage(page.id, page.access_token)">
                Connect
              </button>
            </td>
          </tr>
        </table>
      </div>
    </div>
    <!-- <button @click="getPages">get pages</button> -->
  </div>
</template>

<script>
import facebookLogin from "facebook-login-vuejs";

export default {
  name: "app",
  data() {
    return {
      isConnected: false,
      name: "",
      email: "",
      personalID: "",
      picture: "",
      FB: undefined,
      pages: null,
    };
  },
  components: {
    facebookLogin,
  },
  methods: {
    getUserData() {
      this.FB.api("/me", "GET", { fields: "id,name,email,picture" }, (user) => {
        this.personalID = user.id;
        this.email = user.email;
        this.name = user.name;
        this.picture = user.picture.data.url;
      });
      this.getPages();
    },
    getPages() {
      this.FB.api(
        "me/accounts",
        "GET",
        { fields: "id,name,access_token,picture,is_webhooks_subscribed" },
        (response) => {
          this.pages = response.data;
        }
      );
    },
    disconnectPage(pageId, access_token) {
      let vm = this
      this.FB.api(
        `/${pageId}/subscribed_apps`,
        "delete",
        {
          subscribed_fields: "messages",
          access_token: access_token
        },
        (response) => {
          console.log("response", response);
          if (response && !response.error) {
            /* handle the result */
            vm.getPages();
          }
        }
      );
    },
    subscribePage(pageId, access_token) {
      let vm = this
      this.FB.api(
        `/${pageId}/subscribed_apps`,
        "POST",
        {
          subscribed_fields: "messages",
          access_token: access_token
        },
        (response) => {
          console.log("response", response);
          if (response && !response.error) {
            /* handle the result */
            vm.getPages();
          }
        },
      );
    },
    sdkLoaded(payload) {
      this.isConnected = payload.isConnected;
      this.FB = payload.FB;
      if (this.isConnected) this.getUserData();
    },
    onLogin() {
      this.isConnected = true;
      this.getUserData();
      console.log("dd");
    },
    onLogout() {
      this.isConnected = false;
      this.pages = null;
    },
  },
};
</script>

<style>
#app {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}
.information {
  margin-top: 100px;
  margin: auto;
  display: flex;
  flex-direction: column;
}
.well {
  background-color: rgb(191, 238, 229);
  margin: auto;
  padding: 50px 50px;
  border-radius: 20px;
  /* display:inline-block; */
}
.login {
  width: 200px;
  margin: auto;
}
.list-item:first-child {
  margin: 0;
}
.list-item {
  display: flex;
  align-items: center;
  margin-top: 20px;
}
.button {
  margin: auto;
}
</style>
