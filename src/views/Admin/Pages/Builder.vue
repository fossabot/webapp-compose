<template>
    <div class="builder scrollable" v-if="page">
      <grid :blocks.sync="page.blocks" editable>
        <template slot-scope="{ block, index }">
          <div class="actions">
            <a class="action" @click="editBlock(block, index)">
              <i class="icon-edit"></i>
            </a>
            <a class="action"  @click="page.blocks.splice(index,1)">
              <i class="icon-x"></i>
            </a>
          </div>
          <block-preview :block="block"
                         :namespace="namespace"
                         :module="module"></block-preview>
        </template>
      </grid>

      <b-modal id="createBlockSelector" hide-footer :title="$t('page.build.selectBlockTitle')">
        <new-block-selector :record-page="!!module" @select="editBlock($event)"/>
      </b-modal>

      <b-modal
        :title="$t('block.general.title')"
        :ok-title="$t('page.build.addBlock')"
        ok-variant="dark"
        ok-only
        @ok="updateBlocks"
        @hide="editor=null"
        :visible="showCreator">
        <block-edit v-if="showCreator"
                    :namespace="namespace"
                    :module="module"
                    :page="page"
                    :block.sync="editor.block" />
      </b-modal>

      <b-modal
        :title="$t('block.general.changeBlock')"
        header-class="test"
        :ok-title="$t('general.label.saveAndClose')"
        ok-variant="dark"
        ok-only
        centered
        @ok="updateBlocks"
        @hide="editor=null"
        :visible="showEditor">
        <block-edit v-if="showEditor"
                    :namespace="namespace"
                    :module="module"
                    :page="page"
                    :block.sync="editor.block" />
      </b-modal>

      <editor-toolbar :back-link="{name: 'admin.pages'}"
                      :hide-delete="true"
                      @save="handleSave()"
                      @saveAndClose="handleSave({ closeOnSuccess: true })">
        <button v-b-modal.createBlockSelector class="btn">+ {{ $t('page.build.addBlock') }}</button>
        <button @click.prevent="handleSave({ previewOnSuccess: true })" class="btn">{{ $t('general.label.saveAndPreview') }}</button>
      </editor-toolbar>
    </div>
</template>

<script>
import { mapActions } from 'vuex'
import NewBlockSelector from '@/components/Admin/Page/Builder/Selector'
import Grid from '@/components/Common/Grid'
import Block from '@/lib/block'
import BlockPreview from '@/lib/block/BuilderPreview'
import BlockEdit from '@/lib/block/BuilderEdit'
import EditorToolbar from '@/components/Admin/EditorToolbar'
import Page from '@/lib/page'
import Namespace from '@/lib/namespace'

export default {
  components: {
    Grid,
    NewBlockSelector,
    BlockEdit,
    BlockPreview,
    EditorToolbar,
  },

  props: {
    namespace: {
      type: Namespace,
      required: true,
    },

    pageID: {
      type: String,
      required: true,
    },
  },

  data () {
    return {
      editor: null,
      page: null,
    }
  },

  computed: {
    showEditor () {
      return this.editor && this.editor.index !== undefined
    },

    showCreator () {
      return this.editor && this.editor.index === undefined
    },

    module () {
      if (this.page && this.page.moduleID !== '0') {
        return this.$store.getters['module/getByID'](this.page.moduleID)
      } else {
        return undefined
      }
    },
  },

  mounted () {
    const { namespaceID } = this.namespace
    this.findPageByID({ namespaceID, pageID: this.pageID, force: true }).then(page => {
      this.page = new Page(page)
    })
  },

  methods: {
    ...mapActions({
      findPageByID: 'page/findByID',
      updatePage: 'page/update',
    }),

    editBlock (block, index = undefined) {
      this.editor = { index, block: new Block({ ...block }) }
    },

    updateBlocks () {
      let block = new Block(this.editor.block) // make sure we get rid of the references
      // console.log(block, this.editor.index)
      if (this.editor.index !== undefined) {
        this.page.blocks.splice(this.editor.index, 1, block)
      } else {
        this.page.blocks.push(block)
      }
      // console.log(this.page)

      this.editor = null
    },

    handleSave ({ closeOnSuccess = false, previewOnSuccess = false } = {}) {
      this.findPageByID({ pageID: this.pageID, force: true }).then(page => {
        // Merge changes
        this.page = new Page({ ...page, blocks: this.page.blocks })

        this.updatePage(this.page).then((page) => {
          this.raiseSuccessAlert(this.$t('notification.page.saved'))
          if (closeOnSuccess) {
            this.$router.push({ name: 'admin.pages' })
          } else if (previewOnSuccess) {
            this.$router.push({ name: 'page' })
          }

          this.page = new Page(page)
        }).catch(this.defaultErrorHandler(this.$t('notification.page.saveFailed')))
      })
    },
  },
}
</script>

<style lang="scss" scoped>
@import "@/assets/sass/_0.declare.scss";
@import "@/assets/sass/btns.scss";

.builder {
  overflow: hidden;
  display: flex;
  flex-flow: column nowrap;
}

.actions {
  position: sticky;
  z-index: 2;
  top: 3px;
  margin-bottom: -25px;
  padding-right: 10px;

  a {
    i {
      font-weight: 900;
    }

    &:hover {
      i {
        color: $black;
        opacity: 0.8;
      }
    }

    &.action {
      padding: 0 5px;
    }
  }
}

</style>
