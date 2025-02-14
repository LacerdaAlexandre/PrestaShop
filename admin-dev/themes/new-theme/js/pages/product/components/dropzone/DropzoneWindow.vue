<!--**
 * Copyright since 2007 PrestaShop SA and Contributors
 * PrestaShop is an International Registered Trademark & Property of PrestaShop SA
 *
 * NOTICE OF LICENSE
 *
 * This source file is subject to the Open Software License (OSL 3.0)
 * that is bundled with this package in the file LICENSE.md.
 * It is also available through the world-wide-web at this URL:
 * https://opensource.org/licenses/OSL-3.0
 * If you did not receive a copy of the license and are unable to
 * obtain it through the world-wide-web, please send an email
 * to license@prestashop.com so we can send you a copy immediately.
 *
 * DISCLAIMER
 *
 * Do not edit or add to this file if you wish to upgrade PrestaShop to newer
 * versions in the future. If you wish to customize PrestaShop for your
 * needs please refer to https://devdocs.prestashop.com/ for more information.
 *
 * @author    PrestaShop SA and Contributors <contact@prestashop.com>
 * @copyright Since 2007 PrestaShop SA and Contributors
 * @license   https://opensource.org/licenses/OSL-3.0 Open Software License (OSL 3.0)
 *-->
<template>
  <div class="dropzone-window">
    <div class="dropzone-window-header row">
      <div class="dropzone-window-header-left">
        <p
          class="dropzone-window-number"
          v-html="
            $t('window.selectedFiles', { '%filesNb%': selectedFiles.length })
          "
        />
      </div>
      <div class="dropzone-window-header-right">
        <i
          class="material-icons"
          data-toggle="pstooltip"
          :data-original-title="$t('window.zoom')"
        >search</i>
        <i
          class="material-icons"
          data-toggle="pstooltip"
          :data-original-title="$t('window.replaceSelection')"
          @click="openFileManager"
          v-if="selectedFile"
        >find_replace</i>
        <i
          class="material-icons"
          data-toggle="pstooltip"
          :data-original-title="$t('window.delete')"
          @click="$emit('removeSelection')"
        >delete</i>
        <i
          class="material-icons"
          data-toggle="pstooltip"
          :data-original-title="$t('window.close')"
          @click="$emit('unselectAll')"
        >close</i>
      </div>
    </div>

    <p
      class="dropzone-window-select"
      @click="$emit('selectAll')"
      v-if="files.length > 0 && selectedFiles.length !== files.length"
    >
      {{ $t("window.selectAll") }}
    </p>
    <p
      class="dropzone-window-unselect"
      v-if="selectedFiles.length === files.length"
      @click="$emit('unselectAll')"
      data-toggle="pstooltip"
      :data-original-title="$t('window.zoom')"
    >
      {{ $t("window.unselectAll") }}
    </p>

    <div
      class="dropzone-window-bottom"
      v-if="selectedFile"
    >
      <div
        class="md-checkbox dropzone-window-checkbox"
        v-if="selectedFile !== null"
        :data-toggle="showCoverTooltip"
        :data-original-title="$t('window.cantDisableCover')"
      >
        <label>
          <input
            type="checkbox"
            :disabled="isCover"
            :checked="isCover"
            @change="$emit('coverChanged', $event)"
          >
          <i class="md-checkbox-control" />
          {{ $t("window.useAsCover") }}
        </label>
      </div>

      <input
        type="file"
        class="dropzone-window-filemanager"
        @change="watchFiles"
      >

      <div class="dropzone-window-label">
        <label
          for="caption-textarea"
          class="control-label"
        >{{
          $t("window.caption")
        }}</label>
        <div class="dropdown">
          <button
            class="btn btn-outline-secondary btn-sm dropdown-toggle js-locale-btn"
            type="button"
            data-toggle="dropdown"
            aria-haspopup="true"
            aria-expanded="false"
            id="form_invoice_prefix"
          >
            {{ selectedLocale.iso_code }}
          </button>
          <div
            class="dropdown-menu locale-dropdown-menu"
            aria-labelledby="form_invoice_prefix"
          >
            <span
              v-for="locale in locales"
              :key="locale.name"
              class="dropdown-item js-locale-item"
              :data-locale="locale.iso_code"
            >
              {{ locale.name }}
            </span>
          </div>
        </div>
      </div>

      <textarea
        id="caption-textarea"
        name="caption-textarea"
        class="form-control"
        v-model="selectedCaption"
      />

      <div class="dropzone-window-button-container">
        <button
          type="button"
          class="btn btn-primary save-image-settings"
          @click="$emit('saveSelectedFile')"
        >
          <span v-if="!loading">
            {{ $t("window.saveImage") }}
          </span>
          <span
            class="spinner-border spinner-border-sm"
            v-if="loading"
            role="status"
            aria-hidden="true"
          />
        </button>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'DropzoneWindow',
    props: {
      selectedFiles: {
        type: Array,
        default: () => [],
      },
      files: {
        type: Array,
        default: () => [],
      },
      locales: {
        type: Array,
        required: true,
      },
      selectedLocale: {
        type: Object,
        default: () => {},
      },
      loading: {
        type: Boolean,
        default: false,
      },
    },
    computed: {
      selectedFile() {
        return this.selectedFiles.length === 1 ? this.selectedFiles[0] : null;
      },
      isCover() {
        return !!(this.selectedFile && this.selectedFile.is_cover);
      },
      showCoverTooltip() {
        if (this.isCover) {
          return 'pstooltip';
        }

        return false;
      },
      selectedCaption: {
        get() {
          if (
            this.selectedFile === null
            || !this.selectedFile.legends
            || !this.selectedFile.legends[this.selectedLocale.id_lang]
          ) {
            return '';
          }

          return this.selectedFile.legends[this.selectedLocale.id_lang];
        },
        set(value) {
          if (this.selectedFile === null) {
            return;
          }

          this.selectedFile.legends[this.selectedLocale.id_lang] = value;
        },
      },
    },
    mounted() {
      window.prestaShopUiKit.initToolTips();
    },
    updated() {
      window.prestaShopUiKit.initToolTips();
    },
    methods: {
      /**
       * Watch file change and send an event to the smart component
       */
      watchFiles(event) {
        this.$emit('replacedFile', event);
      },
      /**
       * Used to open the native file manager
       */
      openFileManager() {
        const fileInput = document.querySelector('.dropzone-window-filemanager');
        fileInput.click();
      },
    },
  };
</script>

<style lang="scss" type="text/scss">
@import "~@scss/config/_settings.scss";

.product-page {
  .dropzone-window {
    width: 45%;
    background-color: darken(#ffffff, 2%);
    align-self: stretch;
    padding: 1rem;
    min-width: 20rem;

    &-filemanager {
      display: none;
    }

    &-label {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 0.5rem;

      label {
        margin-bottom: 0;
      }

      .dropdown {
        > button {
          padding-right: 0.25rem;
        }

        &-item {
          cursor: pointer;
        }
      }
    }

    textarea {
      margin-bottom: 1rem;
    }

    &-button {
      &-container {
        display: flex;
        justify-content: flex-end;
      }
    }

    &-checkbox {
      margin-bottom: 1rem;

      label {
        font-size: 0.875rem;
      }
    }

    &-select,
    &-unselect {
      font-weight: 600;
      font-size: 0.925rem;
      color: $primary;
      cursor: pointer;
      margin-top: 0.5rem;
    }

    &-number {
      font-size: 1rem;

      span {
        color: $primary;
        font-weight: 600;
      }
    }

    &-header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 0 1rem;

      p {
        margin-bottom: 0;
      }

      .material-icons {
        cursor: pointer;
        color: $gray-500;
        transition: 0.25s ease-out;
        font-size: 1.5rem;
        margin: 0 0.25rem;

        &:last-child {
          margin-right: 0;
        }

        &:first-child {
          margin-left: 0;
        }

        &:hover {
          color: primary;
        }
      }
    }
  }
}
</style>
