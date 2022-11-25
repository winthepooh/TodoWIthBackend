<template>
  <q-page class="flex flex-center">
    <q-card class="my-card q-px-md q-py-md" flat bordered>
      <div class="flex flex-center" v-if="isShowIcon">
        <q-icon name="account_circle" color="grey-6" size="4rem" />
      </div>
      <div class="flex flex-center" v-else>
        <q-img
          :src="imageUrl"
          :ratio="4 / 3"
          spinner-color="primary"
          spinner-size="82px"
        />
      </div>
      <q-card-section>
        <q-form
          @submit.prevent="onSubmit"
          @reset="onReset"
          class="q-gutter-md"
          ref="myRegisterForm"
        >
          <div>
            <q-input
              v-model="fullname"
              type="text"
              label="Name"
              lazy-rules
              :rules="[(val) => (val && val.length > 0) || 'Field is required']"
            />
          </div>
          <div>
            <q-input
              v-model="username"
              type="text"
              label="Username"
              lazy-rules
              :rules="[requiredValidate]"
            />
            <text-caption
              style="font-size: 0.9em"
              v-if="usernameCaption.show"
              :class="[
                usernameCaption.showClass ? 'text-positive' : 'text-negative',
              ]"
            >
              <q-icon :name="usernameCaption.icon" size="1.5em" />
              {{ usernameCaption.msg }}
            </text-caption>
          </div>
          <div>
            <q-input
              v-model="password"
              :type="isPwd ? 'password' : 'text'"
              label="Password"
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
            <!-- 1MB file size  -->
            <q-file
              standout
              v-model="upload_avatar"
              label="Avatar"
              max-file-size="1048576"
              accept=".jpg, .jpeg, .png"
              @rejected="onRejected"
              @update:model-value="uploadFile()"
            >
              <template #append>
                <q-icon name="attach_file" />
              </template>
            </q-file>
          </div>
          <div>
            <q-btn label="Submit" type="submit" color="black" />
            <q-btn
              label="Reset"
              type="reset"
              color="black float-right"
              flat
              class="q-ml-sm"
            />
          </div>
        </q-form>
      </q-card-section>
    </q-card>
    <DialogComponent v-model="showDialog" :propDialog="dialog" />
  </q-page>
</template>

<script>
import { emailValidate, requiredValidate } from "../utils/validations";
import { Notify } from "quasar";
import DialogComponent from "../components/DialogComponent.vue";
import { useLoginUserStore } from "../stores/loginUserStore.js";
export default {
  name: "RegisterPage",
  components: { DialogComponent },
  data() {
    return {
      imageUrl: "",
      fullname: "",
      email: "",
      username: "",
      password: "",
      isPwd: true,
      usernameCaption: {
        show: false,
        showClass: false,
        icon: null,
        msg: null,
      },
      isShowIcon: true,
      upload_avatar: null,
      dialog: {
        icon: "",
        msg: "",
        btnType: "",
        iconColor: "",
      },
      showDialog: false,
      storeLogUser: useLoginUserStore(),
    };
  },
  methods: {
    emailValidate,
    requiredValidate,
    submitData(filename) {
      const data = {
        name: this.fullname,
        username: this.username,
        password: this.password,
        img: filename,
      };
      this.$api
        .post("/user/register", data)
        .then((res) => {
          if (res.status == 200) {
            // console.log(res.data)
            this.showDialog = true;
            this.dialog.icon = "task_alt";
            this.dialog.iconColor = "primary";
            this.dialog.msg =
              "<div class='text-h6'>Welcome,<br>" +
              res.data.fullname +
              "</div> Your account has been created successfully.";
            this.dialog.btnType = "Signup";
            this.storeLogUser.userid = res.data.id;
            this.storeLogUser.fullname = res.data.fullname;
            this.storeLogUser.accessToken = res.data.accessToken;
            if (res.data.img != null) {
              this.storeLogUser.avatar =
                this.$RESTURL + "/file/" + res.data.img;
            } else {
              this.storeLogUser.avatar = "default-avatar.png";
            }
          }
          this.$refs.myRegisterForm.reset();
        })
        .catch((err) => {
          console.log(err);
          this.showErrDialog("on submit data error:" + err);
        });
    },
    onSubmit() {
      // this.showErrDialog("Test error case")
      // return;
      if (this.upload_avatar == "") this.upload_avatar = null;
      if (this.upload_avatar) {
        //with the upload file
        const formData = new FormData();
        formData.append("singlefile", this.upload_avatar);
        this.$api
          .post("/file/upload", formData)
          .then((response) => {
            if (response.status == 200) {
              //continue to submit form data
              this.submitData(response.data.uploadFileName);
            }
          })
          .catch((err) => {
            console.log(err);
            this.showErrDialog("on submit error: " + err);
          });
      } else {
        //without upload file
        this.submitData(null);
      }
    },
    showErrDialog(err) {
      this.showDialog = true;
      this.dialog.icon = "error";
      this.dialog.iconColor = "negative";
      this.dialog.msg = err;
      this.dialog.btnType = "Error";
    },
    usernameValidate() {
      if (this.username) {
        this.$api
          .get("/user/check/" + this.username)
          .then((response) => {
            // console.log(response.data)
            if (response.data.valid) {
              this.usernameCaption.show = true;
              this.usernameCaption.showClass = true;
              this.usernameCaption.icon = "check_circle_outline";
              this.usernameCaption.msg = "This username is Available.";
            } else {
              this.usernameCaption.show = true;
              this.usernameCaption.showClass = false;
              this.usernameCaption.icon = "highlight_off";
              this.usernameCaption.msg = "This username is NOT Available.";
            }
          })
          .catch((err) => {
            console.log(err);
          });
      } else {
        this.resetUserCaption();
      }
    },
    resetUserCaption() {
      this.usernameCaption.show = false;
      this.usernameCaption.showClass = false;
      this.usernameCaption.icon = null;
      this.usernameCaption.msg = null;
    },
    onRejected(rejectedEntries) {
      let msg;
      if (rejectedEntries[0].failedPropValidation == "accept")
        msg = "Only images (jpg, jpeg, png) are allowed.";
      else if (rejectedEntries[0].failedPropValidation == "max-file-size")
        msg = "File size cannot be larger than 1MB.";
      Notify.create({
        type: "negative",
        message: msg,
      });
    },
    uploadFile() {
      this.isShowIcon = false;
      this.imageUrl = URL.createObjectURL(this.upload_avatar);
    },
    onReset() {
      this.fullname = null;
      this.email = null;
      this.username = null;
      this.password = null;
      this.isPwd = true;
      this.resetUserCaption();
      this.upload_avatar = null;
      this.isShowIcon = true;
      this.imageUrl = "";
    },
  },
  watch: {
    username() {
      this.usernameValidate();
    },
  },
  created() {
    if (this.storeLogUser.getUserId) this.$router.push("/user");
  },
};
</script>

<style></style>
