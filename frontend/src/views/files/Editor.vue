<template>
  <div id="editor-container">
    <header-bar>
      <action icon="close" :label="$t('buttons.close')" @action="close()" />
      <title>{{ req.name }}</title>

      <action
        v-if="user.perm.modify"
        id="save-button"
        icon="save"
        :label="$t('buttons.save')"
        @action="save()"
      />
    </header-bar>

    <breadcrumbs base="/files" noLink />
    
    <editor-ui
    ref="toastuiEditor"
    :initialValue="startingValue"
    initialEditType="wysiwyg"
    height="500px"
    previewStyle="vertical"
    />
  
  </div>
</template>

<script>
import { mapState } from "vuex";
import { files as api } from "@/api";
import { theme } from "@/utils/constants";
import buttons from "@/utils/buttons";
import url from "@/utils/url";

import '@toast-ui/editor/dist/toastui-editor.css';
import { Editor as EditorUi } from '@toast-ui/vue-editor';

import HeaderBar from "@/components/header/HeaderBar";
import Action from "@/components/header/Action";
import Breadcrumbs from "@/components/Breadcrumbs";

export default {
  name: "file editor",
  components: {
    EditorUi,
    HeaderBar,
    Action,
    Breadcrumbs,
  },
  data: function () {
    return {
      fileContent: "",
      startingValue : "",
    };
  },
  computed: {
    ...mapState(["req", "user"]),
    breadcrumbs() {
      let parts = this.$route.path.split("/");

      if (parts[0] === "") {
        parts.shift();
      }

      if (parts[parts.length - 1] === "") {
        parts.pop();
      }

      let breadcrumbs = [];

      for (let i = 0; i < parts.length; i++) {
        breadcrumbs.push({ name: decodeURIComponent(parts[i]) });
      }

      breadcrumbs.shift();

      if (breadcrumbs.length > 3) {
        while (breadcrumbs.length !== 4) {
          breadcrumbs.shift();
        }

        breadcrumbs[0].name = "...";
      }

      return breadcrumbs;
    },
  },
  created() {
    window.addEventListener("keydown", this.keyEvent);
  },
  beforeDestroy() {
    window.removeEventListener("keydown", this.keyEvent);
  },
  mounted: function () {
    this.fileContent = this.req.content || "";

    this.$refs.toastuiEditor.invoke('setMarkdown', this.fileContent)
    
    },
  methods: {
    back() {
      let uri = url.removeLastDir(this.$route.path) + "/";
      this.$router.push({ path: uri });
    },
    keyEvent(event) {
      if (!event.ctrlKey && !event.metaKey) {
        return;
      }

      if (String.fromCharCode(event.which).toLowerCase() !== "s") {
        return;
      }

      event.preventDefault();
      this.save();
    },
    async save() {
      const button = "save";
      buttons.loading("save");

      this.fileContent = this.$refs.toastuiEditor.invoke('getMarkdown');
      try {
        await api.put(this.$route.path, this.fileContent);
        buttons.success(button);
      } catch (e) {
        buttons.done(button);
        this.$showError(e);
      }
    },
    close() {
      this.$store.commit("updateRequest", {});

      let uri = url.removeLastDir(this.$route.path) + "/";
      this.$router.push({ path: uri });
    },
  },
};
</script>
