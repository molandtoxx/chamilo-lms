<template>
  <v-form>
    <v-container fluid>
      <v-row>
        <v-col cols="12" sm="6" md="6">
          <v-text-field
                  v-model="item.title"
                  :error-messages="titleErrors"
                  :label="$t('Title')"
                  required
                  @input="$v.item.title.$touch()"
                  @blur="$v.item.title.$touch()"
          />
          <editor
                  v-if="item.resourceNode && item.resourceNode.fileEditableText || item.newDocument"
                  :error-messages="contentFileErrors"
                  required
                  v-model="item.contentFile"
                  :init="{
                    skin_url: '/build/libs/tinymce/skins/ui/oxide',
                    content_css: '/build/libs/tinymce/skins/content/default/content.css',
                    branding:false,
                    height: 500,
                    toolbar_mode: 'sliding',
                    file_picker_callback : browser,
                  /*file_picker_callback: function(callback, value, meta) {
                    // Provide file and text for the link dialog
                    if (meta.filetype == 'file') {
                      callback('mypage.html', {text: 'My text'});
                    }

                    // Provide image and alt text for the image dialog
                    if (meta.filetype == 'image') {
                      callback('myimage.jpg', {alt: 'My alt text'});
                    }

                    // Provide alternative source and posted for the media dialog
                    if (meta.filetype == 'media') {
                      callback('movie.mp4', {source2: 'alt.ogg', poster: 'image.jpg'});
                    }
                  },*/
                  /*images_upload_handler: (blobInfo, success, failure) => {
                    const img = 'data:image/jpeg;base64,' + blobInfo.base64();
                    //console.log(img);
                    success(img);
                  },*/
                   //menubar: true,
                   autosave_ask_before_unload: true,
                   plugins: [
                     'fullpage advlist autolink lists link image charmap print preview anchor',
                     'searchreplace visualblocks code fullscreen',
                     'insertdatetime media table paste wordcount'
                   ],
                   toolbar: 'undo redo | bold italic underline strikethrough | fontselect fontsizeselect formatselect | alignleft aligncenter alignright alignjustify | outdent indent |  numlist bullist | forecolor backcolor removeformat | pagebreak | charmap emoticons | fullscreen  preview save print | insertfile image media template link anchor code codesample | ltr rtl',
                  }
              "
          />
        </v-col>
      </v-row>
    </v-container>
  </v-form>
</template>

<script>
import has from 'lodash/has';
import { validationMixin } from 'vuelidate';
import { required } from 'vuelidate/lib/validators';
//import UploadAdapter from './UploadAdapter';

import Editor from '../Editor'

export default {
  name: 'DocumentsForm',
  components: {
    'editor': Editor
  },
  mixins: [validationMixin],
  props: {
    values: {
      type: Object,
      required: true
    },
    errors: {
      type: Object,
      default: () => {}
    },
    initialValues: {
      type: Object,
      default: () => {}
    },
  },
  data() {
    return {
      title: null,
      contentFile: null,
      parentResourceNodeId: null,
      resourceNode: null,
    };
  },
  computed: {
    item() {
      return this.initialValues || this.values;
    },
    titleErrors() {
      const errors = [];
      if (!this.$v.item.title.$dirty) return errors;
      has(this.violations, 'title') && errors.push(this.violations.title);
      !this.$v.item.title.required && errors.push(this.$t('Field is required'));

      return errors;
    },
    contentFileErrors() {
      const errors = [];

      if (this.item.resourceNode && this.item.resourceNode.fileEditableText) {
        if (!this.$v.item.contentFile.$dirty) return errors;
        has(this.violations, 'contentFile') && errors.push(this.violations.contentFile);
        !this.$v.item.contentFile.required && errors.push(this.$t('Content is required'));
      }

      return errors;
    },
    violations() {
      return this.errors || {};
    }
  },
  methods: {
    browser (callback, value, meta) {
      tinymce.activeEditor.windowManager.openUrl({
        url: '/',// use an absolute path!
        title: 'file manager',
        /*width: 900,
        height: 450,
        resizable: 'yes'*/
      }, {
        oninsert: function (file, fm) {
          var url, reg, info;

          // URL normalization
          url = fm.convAbsUrl(file.url);

          // Make file info
          info = file.name + ' (' + fm.formatSize(file.size) + ')';

          // Provide file and text for the link dialog
          if (meta.filetype == 'file') {
            callback(url, {text: info, title: info});
          }

          // Provide image and alt text for the image dialog
          if (meta.filetype == 'image') {
            callback(url, {alt: info});
          }

          // Provide alternative source and posted for the media dialog
          if (meta.filetype == 'media') {
            callback(url);
          }
        }
      });
      return false;
    },
  },
  validations: {
    item: {
      title: {
        required,
      },
      contentFile: {
        //required,
      },
      parentResourceNodeId: {
      },
      resourceNode:{
      }
    }
  }
};
</script>
