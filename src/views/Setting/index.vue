<template>
  <div class="setting">
    <van-cell center title="Store Cache" :label="size.local | bytes">
      <template #right-icon>
        <van-button type="primary" size="small" @click="clearCache('local')">Clean Up</van-button>
      </template>
    </van-cell>
    <van-cell center title="Browser Cache" :label="size.session | bytes">
      <template #right-icon>
        <van-button type="info" size="small" @click="clearCache('session')">Clean Up</van-button>
      </template>
    </van-cell>
    <van-cell center title="R-18 Display" label="Contains nudity or sexual">
      <template #right-icon>
        <van-switch :value="currentSETTING.r18" @input="onR18Change($event, 1)" size="24" />
      </template>
    </van-cell>
    <van-cell center title="R-18G Display" label="Contains gore or disgusting content">
      <template #right-icon>
        <van-switch :value="currentSETTING.r18g" @input="onR18Change($event, 2)" size="24" />
      </template>
    </van-cell>
  </div>
</template>

<script>
import { Cell, Switch, Button, Dialog } from "vant";
import { mapState, mapActions } from "vuex";
import { LocalStorage, SessionStorage } from "@/utils/storage";
export default {
  name: "Setting",
  data() {
    return {
      currentSETTING: {
        r18: false,
        r18g: false
      },
      size: {
        local: 0,
        session: 0
      }
    };
  },
  computed: {
    ...mapState(["SETTING"])
  },
  watch: {
    $route() {
      this.calcCacheSize();
    }
  },
  methods: {
    onR18Change(checked, type) {
      let name;
      if (type === 1) name = "R-18";
      if (type === 2) name = "R-18G";

      if (checked) {
        Dialog.confirm({
          message: `Are you sure you want to turn on the display of works for ${name} Please make sure you are over 18 years old and have not violated local laws and regulations`,
          confirmButtonColor: "black",
          cancelButtonColor: "#1989fa",
          closeOnPopstate: true
        })
          .then(() => {
            if (type === 1) this.currentSETTING.r18 = checked;
            if (type === 2) {
              this.currentSETTING.r18g = checked;
              setTimeout(() => {
                Dialog.alert({
                  message: `Please note that turning on the ${name} switch may have irreversible effects on your physical and mental health, if you feel unwell, please close the app immediately and seek medical help`
                });
              }, 200);
            }
          })
          .catch(() => {
            console.log("操作取消");
          });
      } else {
        if (type === 1) this.currentSETTING.r18 = checked;
        if (type === 2) this.currentSETTING.r18g = checked;
      }
    },
    calcCacheSize() {
      this.size.local = LocalStorage.size;
      this.size.session = SessionStorage.size;
    },
    clearCache(type) {
      let showName;
      switch (type) {
        case "local":
          showName = "Store Cache";
          break;

        case "session":
          showName = "Browser Cache";
          break;

        default:
          break;
      }
      Dialog.confirm({
        message: `Are you sure you want to clean up ${showName} ?`,
        confirmButtonColor: "black",
        cancelButtonColor: "#1989fa",
        closeOnPopstate: true
      }).then(() => {
        if (type === "local") LocalStorage.clear();
        if (type === "session") SessionStorage.clear();

        this.calcCacheSize();
        this.$toast.success("Success");
      });
    },
    ...mapActions(["saveSETTING"])
  },
  filters: {
    bytes(bytes) {
      bytes = Number(bytes);
      if (bytes === 0) return "0 B";

      const k = 1024;
      const dm = 0;
      const sizes = ["B", "KB", "MB", "GB", "TB", "PB", "EB", "ZB", "YB"];

      const i = Math.floor(Math.log(bytes) / Math.log(k));

      return parseFloat((bytes / Math.pow(k, i)).toFixed(dm)) + " " + sizes[i];
    }
  },
  mounted() {
    this.currentSETTING = JSON.parse(JSON.stringify(this.SETTING));
    this.calcCacheSize();
  },
  updated() {
    this.saveSETTING(JSON.parse(JSON.stringify(this.currentSETTING)));
  },
  components: {
    [Cell.name]: Cell,
    [Button.name]: Button,
    [Switch.name]: Switch
  }
};
</script>

<style lang="stylus" scoped>
.setting {
}
</style>
