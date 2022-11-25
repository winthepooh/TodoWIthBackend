<template>
  <q-page padding>
    <!-- <div>
  <q-btn color="primary" icon="check"
  label="OK"
  @click="showErrDialog('This is an example error.')" />
</div> -->
    <div v-if="dataReady">
      <!-- display qtable -->
      <q-table
        title="List of Users"
        :rows="rows"
        :columns="columns"
        row-key="id"
        :pagination="paginations"
        flat
        bordered
      >
        <template #body="props">
          <q-tr :props="props">
            <q-td key="id" :props="props">
              {{ props.row.id }}
            </q-td>
            <q-td key="fullname" :props="props">
              {{ props.row.fullname }}
            </q-td>
            <q-td key="email" :props="props">
              {{ props.row.email }}
            </q-td>
            <q-td key="actions">
              <q-btn
                color="primary"
                icon="edit"
                flat
                round
                @click="editRecord(props.row)"
              />
              <q-btn
                color="primary"
                icon="delete"
                flat
                round
                @click="deleteRecord(props.row)"
              />
            </q-td>
          </q-tr>
        </template>
      </q-table>
    </div>
    <div v-else class="flex flex-center">
      <!-- display circular progress -->
      <q-circular-progress
        indeterminate
        size="90px"
        color="lime"
        center-color="grey-8"
        track-color="transparent"
        class="q-ma-md"
      />
    </div>
    <!-- Edit Dialog -->
    <q-dialog v-model="form_edit" persistent>
      <q-card>
        <q-card-section class="row items-center">
          <q-avatar icon="edit" color="primary" text-color="white" />
          <span class="q-ml-sm text-h6"> Edit User ID: {{ input.id }} </span>
        </q-card-section>
        <q-card-section>
          <q-input v-model="input.fullname" type="text" label="Fullname" />
          <q-input v-model="input.email" type="text" label="Email" />
          <q-img
            v-if="input.img"
            :src="input.img"
            :ratio="4 / 3"
            spinner-color="primary"
            spinner-size="82px"
          />
          <!-- 1048576 = 1MB -->
          <q-file
            v-model="uploadFile"
            label="Choose new Avartar"
            accept=".jpg, .jpeg, .png"
            max-file-size="1048576"
            @rejected="onRejected"
            @update:model-value="updateFile()"
          >
            <template #append>
              <q-icon name="attach_file" />
            </template>
          </q-file>
        </q-card-section>
        <q-card-actions align="right">
          <q-btn
            flat
            label="Cancel"
            color="primary"
            v-close-popup
            @click="onCancelEdit()"
          />
          <q-btn
            flat
            label="Edit"
            color="primary"
            v-close-popup
            @click="onEdit()"
          />
        </q-card-actions>
      </q-card>
    </q-dialog>
    <!-- Delete Dialog -->
    <q-dialog v-model="form_del" persistent>
      <q-card>
        <q-card-section class="row items-center">
          <q-avatar icon="delete" color="primary" text-color="white" />
          <span class="q-ml-sm text-h6"> Delete user ID: {{ input.id }} </span>
        </q-card-section>
        <q-card-section>
          <span class="q-ml-sm"> Fullname: {{ input.fullname }} </span>
        </q-card-section>
        <q-card-actions align="right">
          <q-btn flat label="NO" color="primary" v-close-popup />
          <q-btn
            flat
            label="YES"
            color="primary"
            v-close-popup
            @click="onDelete()"
          />
        </q-card-actions>
      </q-card>
    </q-dialog>
    <DialogComponent v-model="showDialog" :propDialog="dialog" />
  </q-page>
</template>

<script>
import { Notify } from "quasar";
import DialogComponent from "../components/DialogComponent.vue";
import { useLoginUserStore } from "../stores/loginUserStore.js";
const defaultAvatar = "default-avatar.png";
export default {
  name: "ListUserPage",
  components: { DialogComponent },
  data() {
    return {
      dataReady: false,
      storeLogUser: useLoginUserStore(),
      showDialog: false,
      dialog: {
        icon: "",
        msg: "",
        btnType: "",
        iconColor: "",
      },
      rows: [],
      columns: [
        {
          name: "id",
          label: "ID",
          align: "center",
          field: "id",
          sortable: true,
        },
        {
          name: "fullname",
          label: "Fullname",
          align: "left",
          field: "fullname",
          sortable: true,
        },
        {
          name: "email",
          label: "Email",
          align: "left",
          field: "email",
          sortable: true,
        },
        {
          name: "actions",
          label: "Actions",
          align: "left",
          field: "actions",
        },
      ],
      paginations: { rowsPerPage: 10 },
      input: [],
      form_edit: false,
      uploadFile: null,
      form_del: false,
    };
  },
  methods: {
    showErrDialog(err) {
      this.showDialog = true;
      this.dialog.icon = "error";
      this.dialog.iconColor = "negative";
      this.dialog.msg = err;
      this.dialog.btnType = "Error";
    },
    getAllUsers() {
      const headers = { "access-token": this.storeLogUser.accessToken };
      this.$api
        .get("/auth", { headers })
        .then((res) => {
          if (res.status == 200) {
            res.data.forEach((item, key) => {
              if (item.img != null) {
                res.data[key].img =
                  this.$RESTURL + "/file/" + res.data[key].img;
              }
            });
            this.rows = res.data;
          }
          // console.log(JSON.stringify(this.rows))
        })
        .catch((err) => {
          Notify.create({
            type: "negative",
            message: "Unauthorized",
          });
          this.storeLogUser.clearStorage();
          this.$router.push("/");
        });
    },
    editRecord(record) {
      // console.log(record);
      this.input = record;
      this.form_edit = true;
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
    updateFile() {
      this.input.img = URL.createObjectURL(this.uploadFile);
    },
    onCancelEdit() {
      this.getAllUsers();
    },
    onEdit() {
      if (this.uploadFile == "") this.uploadFile = null;
      if (this.uploadFile) {
        //user changed avartar
        const formData = new FormData();
        formData.append("singlefile", this.uploadFile);
        this.$api
          .post("/file/upload", formData)
          .then((res) => {
            if (res.status == 200) {
              //continue submit form data to API
              this.submitEditData(res.data.uploadFileName);
              this.uploadFile = null;
            }
          })
          .catch((err) => {
            console.log(err);
            this.showDialog(err);
          });
      } else {
        //user did not change avartar, submit form data to API
        this.submitEditData(null);
      }
      // console.log("call getalluser")
      this.getAllUsers();
    },
    submitEditData(filename) {
      let img = "";
      if (filename == null) {
        if (this.input.img == null) img = null;
        else img = this.getFileName(this.input.img);
      } else img = filename;
      const data = {
        fullname: this.input.fullname,
        email: this.input.email,
        img: img,
      };
      console.log("sumit data: " + data);
      const headers = {
        "access-token": this.storeLogUser.accessToken,
      };
      this.$api
        .put("/auth/" + this.input.id, data, { headers })
        .then((res) => {
          if (res.status == 200) {
            Notify.create({
              type: "positive",
              message: "Updated user ID: " + res.data.id,
            });
            if (this.storeLogUser.userid == res.data.id) {
              this.storeLogUser.fullname = res.data.fullname;
              if (res.data.img != null && this.uploadFile == null) {
                this.storeLogUser.avatar =
                  this.$RESTURL + "/file/" + res.data.img;
              } else {
                this.storeLogUser.avatar = defaultAvatar;
              }
            }
          }
        })
        .catch((err) => {
          console.log(err);
          this.showDialog(err);
        });
    },
    getFileName(filePath) {
      return filePath.substr(filePath.lastIndexOf("/") + 1);
    },
    deleteRecord(record) {
      this.input = record;
      this.form_del = true;
    },
    onDelete() {
      const headers = {
        "access-token": this.storeLogUser.accessToken,
      };
      this.$api
        .delete("/auth/" + this.input.id, { headers })
        .then((res) => {
          if (res.status == 200) {
            Notify.create({
              type: "positive",
              message: "Deleted user ID: " + res.data.id,
            });
            if (this.storeLogUser.userid == res.data.id) {
              this.storeLogUser.clearStorage();
              this.$router.push("/");
            } else {
              // console.log("call")
              this.getAllUsers();
            }
          }
        })
        .catch((err) => {
          console.log(err);
          this.showErrDialog(err);
        });
    },
  },
  async mounted() {
    await this.getAllUsers();
    this.dataReady = true;
  },
  created() {
    if (!this.storeLogUser.getUserId) this.$router.push("/");
  },
};
</script>

<style></style>
