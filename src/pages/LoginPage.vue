<template>
  <q-page class="flex flex-center">
    <q-card class="my-card q-px-md q-py-md" flat bordered>
      <div class="flex flex-center">
        <q-icon name="account_circle" color="grey-6" size="4rem" />
      </div>
      <q-card-section>
        <q-form
          @submit.prevent="onSubmit"
          @reset="onReset"
          class="q-gutter-md"
          ref="myLoginForm"
        >
          <div>
            <q-input
              v-model="username"
              type="text"
              label="Your Username"
              lazy-rules
              :rules="[
                (val) =>
                  (val && val.length > 0) || 'Must be at least 1 character!',
              ]"
            />
          </div>
          <div>
            <q-input
              v-model="password"
              :type="isPwd ? 'password' : 'text'"
              label="Your Password"
              lazy-rules
              :rules="[
                (val) =>
                  (val && val.length >= 6) || 'Must be at least 6 characters',
              ]"
            >
              <template #append>
                <q-icon
                  :name="isPwd ? 'visibility_off' : 'visibility'"
                  class="cursor-pointer"
                  @click="isPwd = !isPwd"
                />
              </template>
            </q-input>
          </div>
          <div>
            <q-btn
              label="Login"
              type="submit"
              color="black"
              style="width: 100%"
            />
          </div>
          <div>
            <text-caption>
              Are you registered?
              <a href="/register">Create Account</a>
            </text-caption>
          </div>
        </q-form>
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script>
import { Notify } from "quasar";
import { useLoginUserStore } from "../stores/loginUserStore";
export default {
  name: "LoginPage",
  data() {
    return {
      username: null,
      password: null,
      isPwd: true,
      storeLogUser: useLoginUserStore(),
    };
  },
  methods: {
    onReset() {
      this.username = null;
      this.password = null;
      this.isPwd = true;
    },
    onSubmit() {
      const data = {
        username: this.username,
        password: this.password,
      };
      this.$api
        .post("/user/login", data)
        .then((res) => {
          if (res.status == 200) {
            // console.log(res.data)
            Notify.create({
              type: "positive",
              message: "Login successfully.",
            });
            this.storeLogUser.userid = res.data.id;
            this.storeLogUser.fullname = res.data.fullname;
            this.storeLogUser.accessToken = res.data.accessToken;
            if (res.data.img != null) {
              this.storeLogUser.avatar =
                this.$RESTURL + "/file/" + res.data.img;
            } else {
              this.storeLogUser.avatar = "default-avatar.png";
            }
            this.$router.push("/user");
          }
        })
        .catch((err) => {
          console.log(err);
          Notify.create({
            type: "negative",
            message: "Invalid Username or Password.",
          });
        });
      this.$refs.myLoginForm.reset();
    },
  },
  created() {
    if (this.storeLogUser.getUserId) this.$router.push("/user");
  },
};
</script>

<style></style>
