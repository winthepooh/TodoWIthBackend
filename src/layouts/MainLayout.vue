<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar class="bg-black">
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="toggleLeftDrawer"
        />

        <q-toolbar-title> Account Manager </q-toolbar-title>

        <div v-if="storeLogUser.getUserId">
          Hello {{ storeLogUser.getFullname }}
          <q-space />
          <q-btn round>
            <q-avatar size="32px">
              <img :src="storeLogUser.getAvatar" alt="" />
            </q-avatar>
          </q-btn>
          <q-btn flat color="white" icon="logout" @click="logout" />
        </div>
        <div v-else>By. Phubhodee Junmanee</div>
      </q-toolbar>
    </q-header>

    <q-drawer v-model="leftDrawerOpen" show-if-above bordered>
      <q-list>
        <q-item-label header> Account Manager </q-item-label>

        <EssentialLink
          v-for="link in essentialLinks"
          :key="link.title"
          v-bind="link"
        />
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent, ref } from "vue";
import EssentialLink from "components/EssentialLink.vue";
import { useLoginUserStore } from "../stores/loginUserStore";
import { Notify } from "quasar";
const linksList = [
  {
    title: "Home",
    caption: "Normal Homepage",
    icon: "home",
    link: "/",
  },
  {
    title: "Register",
    caption: "Register Account",
    icon: "person_add",
    link: "/register",
  },
  {
    title: "List",
    caption: "Retrive users data",
    icon: "people_outline",
    link: "/user",
  },
];

export default defineComponent({
  name: "MainLayout",

  components: {
    EssentialLink,
  },

  setup() {
    const leftDrawerOpen = ref(false);
    const storeLogUser = useLoginUserStore();

    return {
      essentialLinks: linksList,
      leftDrawerOpen,
      toggleLeftDrawer() {
        leftDrawerOpen.value = !leftDrawerOpen.value;
      },
      storeLogUser,
    };
  },
  methods: {
    logout() {
      this.storeLogUser.clearStorage();
      Notify.create({
        type: "info",
        message: "Logout successfully.",
      });
      this.$router.push("/");
    },
  },
});
</script>
